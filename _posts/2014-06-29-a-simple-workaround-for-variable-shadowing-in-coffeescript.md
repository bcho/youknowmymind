---
title: "Coffeescript 中简单的显式 variable shadowing"
data: 2014-06-29 12:00:00
layout: post
categories: javascript programming
---

Coffeescript 里面一个比较特别的地方就是没有显式的 variable shadowing：

[E.g. 1](http://dou.bz/3ROhYm):

```coffeescript
outer = 1
innerGlobal = 42

changeNumbers = ->
  inner = -1
  innerGlobal = -1
  outer = 10

inner = changeNumbers()

alert "#{inner} #{innerGlobal}"
```

这个做法最大的问题就是开发者很容易会在某一个层级的作用域把高层级作用域中的同名变量（name conflict）给覆盖掉，从而污染了上级作用域：

[E.g. 2](http://dou.bz/3kjeWs):

```coffeescript
{sin, cos, PI} = math

coolCos = (x) ->
  cos = sin(x - PI / 2)

alert cos  # should be a function
alert cos PI / 2  # should be 0
alert coolCos(PI / 2)  # should be 0
alert cos  # cos is 0 now!
```

现实生活中的[例子](http://stackoverflow.com/questions/15223430/why-is-coffeescript-of-the-opinion-that-shadowing-is-a-bad-idea/15228602#15228602)，请务必看 EDIT 部分。


虽然我很理解和认同 Jeremy 老师的[说法](https://github.com/jashkenas/coffeescript/issues/712#issuecomment-430673)：

> The real solution to this is to keep your top-level scopes clean, and be aware of what's in your lexical scope. If you're creating a variable that's actually a different thing, you should give it a different name.


但如果真的要实实在在多人合作的话，还是提供一个官方的 work around 会比较好吧？（参考 lua 的 local）


------------------


在 [#712](https://github.com/jashkenas/coffeescript/issues/712) 这个 issue 中，也有人提出了一个看起来还不错的[解决方法](https://github.com/jashkenas/coffeescript/issues/712#issuecomment-8595484)。


具体原理就是利用 JavaScript 的[函数参数优先级比外层作用域高](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions_and_function_scope?redirectlocale=en-US&redirectslug=JavaScript%2FReference%2FFunctions_and_function_scope#Name_conflicts)的特点，明确限定我们的局部变量的作用域（当然用 literal 的 ``var`` 也行，哪个好看就见仁见智了。）


于是 E.g. 2 可以改成如下：

[E.g. 3](http://dou.bz/0gUswl):

```coffeescript
outer = 1
innerGlobal = 42

changeNumbers = -> do (innerGlobal) ->
  inner = -1
  innerGlobal = -1
  outer = 10
  alert innerGlobal  # should be -1
  return outer

inner = changeNumbers()

alert "#{inner} #{innerGlobal}"  # innerGlobal should be 42
```


通过上述的方法（``do`` 不是必须的），妈妈就再也不用担心<del>同事</del>把我的 coffeescript 代码弄坏啦！（虽然就是丑了点。）
