---
title: "Quick note: ELF 格式"
date: 2015-05-28 11:00:00
layout: post
categories: notes, programming
---

在阅读 [greenify][_greenify] 的源代码的时候，发现了一个比较好玩的实现技巧：[elf redirect][_elf-redirect]。这当中的实现原理就要参考 ELF 的格式定义了。

[_greenify]: https://github.com/douban/greenify/tree/master/src
[_elf-redirect]: http://www.codeproject.com/Articles/70302/Redirecting-functions-in-shared-ELF-libraries


#### 参考资料：

- [http://flint.cs.yale.edu/cs422/doc/ELF_Format.pdf](http://flint.cs.yale.edu/cs422/doc/ELF_Format.pdf)
- [http://imgur.com/a/JEObT](http://imgur.com/a/JEObT) (很赞的一个 infographic)
- [http://linux.die.net/man/1/readelf](http://linux.die.net/man/1/readelf)

## 文件布局

每个 ELF 文件基本上由以下几个部分组成（序号为出现顺序）：

1. ELF Header
2. Program Header Table (optional)
3. Sections
4. Section Header Table (optional)

### ELF Header

其实就是一个 meta 头，包含了关于这个 ELF 文件的组织信息(road map)。值得一提的就是为了跨平台，header 里面有 16 个字节(magic) 是平台、字节序无关的：

```
ELF Header:
  Magic:   7f 45 4c 46 02 01 01 00 00 00 00 00 00 00 00 00
```

### Program Header Table (optional)

顾名思义，这个 table 是用来描述 program 如何可以被运行的，其中：

- `p_offset` 字段指明了第一个 segment 的开始偏移（也就是头部信息的大小）
- `p_vaddr` 来指定在虚拟地址中的地址

但不是所有 ELF 文件都有这个 table，因为不是所有 ELF 文件都是用来执行的。

### Sections

包含了一系列 **不定长** 的 object file 数据（如 data / symbol tale）。
注意这个 section 的概念只在 linking 的时候有意义（也就是可以改写指定块的数据）；而运行时是只会有定长的 segment 定义。也就是说，一个不定长的 section 会落到一个或多个 segment 中：

```
Section to Segment mapping:
  Segment Sections…
   00
   01     .interp
   02     .interp .note.ABI-tag .note.gnu.build-id .gnu.hash .dynsym .dynstr .gnu.version .gnu.version_r .rela.dyn .rela.plt .init .plt .text .fini .rodata .typelink .gopclntab .eh_frame_hdr .eh_frame

… remains omitted
```

另外要注意的是因为 index 都是非负数(``uint32``)，所以特意用 0 来代替一个空的 section 。


### Section Header Table (optinal)

这个 table 包含了 sections 的 meta 信息，如：

- section 名称
- section 大小

只在 linking 的时候有意义。

------

## ELF Redirect 原理

程序调用外部函数的时候是需要通过 call 某个地址来完成的。从编程语言的角度来看，一个函数的外部调用：

```
call_rv = call_extern_func(args);
```

其实就是对某个地址的调用，而在这里 `call_extern_func` 本质上只是一个地址 `A` 的 alias。所以完全可以在运行时对这个 `call_extern_func` 和 `A` 的映射关系进行一个 hijack，把 `call_extern_func` 重新指向另外一个地址。

为了实现上面的工作，要借助以下几个 section：

1. Dynamic Symbol Table (`.dynsym`)
2. Relocations (`.rel.plt` 和 `.rel.dyn`)

其中 dynamic symbol table 包含了 `call_extern_func` 在对应 section 中的位置信息：

```
Symbol table ‘.dynsym’ contains 49 entries:
   Num:    Value          Size Type    Bind   Vis      Ndx Name
     0: 0000000000000000     0 NOTYPE  LOCAL  DEFAULT  UND
     1: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND free@GLIBC_2.2.5 (2)
     2: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND pthread_create@GLIBC_2.2.5 (3)
     3: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND pthread_sigmask@GLIBC_2.2.5 (3)
     4: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND abort@GLIBC_2.2.5 (2)
     5: 0000000000000000     0 NOTYPE  WEAK   DEFAULT  UND _ITM_deregisterTMCloneTab
     6: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND puts@GLIBC_2.2.5 (2)

… remains omitted
```

注意 num 字段的值是我们需要的(以 `puts` 为例，这里是 6)。

而 relocations tables 中包含了 `call_extern_func` 和 `A` 的关系：

```
Relocation section ‘.rela.plt’ at offset 0xb68 contains 18 entries:
  Offset          Info           Type           Sym. Value    Sym. Name + Addend
0000007a9018  000100000007 R_X86_64_JUMP_SLO 0000000000000000 free + 0
0000007a9020  000200000007 R_X86_64_JUMP_SLO 0000000000000000 pthread_create + 0
0000007a9028  000300000007 R_X86_64_JUMP_SLO 0000000000000000 pthread_sigmask + 0
0000007a9030  000400000007 R_X86_64_JUMP_SLO 0000000000000000 abort + 0
0000007a9038  000600000007 R_X86_64_JUMP_SLO 0000000000000000 puts + 0

… remains omitted
```

从上面的 listing 可以发现， `puts` 的 Info 值为 `000600000007` 中的高 32 位和 symbol table 中的 num 值一致，所以我们可以通过这个对应关系来从 (dynamic) symbol table 中找到在 relocation tables 中的位置。

Relocation tables 中则是包含了具体调用的地址的地址，我们只需要将对应 entry 的 offset 的引用值(`A`)替换为 `B` 即可完成对调用的转移。不过需要注意的是 `.rela.plt` 和 `.rela.dyn` 的 offset 含义区别。`
