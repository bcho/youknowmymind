---
title: "DevOps 101: 一个简单的部署模型"
date: 2014-06-08 23:00:00
layout: post
categories: devops, note
---

DevOps is [hard](http://jeffknupp.com/blog/2014/04/15/how-devops-is-killing-the-developer/)! 但在全<del>站</del>栈式大行其道的今天，不一个人揽起所有的工作的话就拿不到工资啦！


（对我来说，）SA 的工作比 developer 难得多，原因就是 SA 需要考虑更加多的环境和对应环境下可能出现的问题。
而这些没有章法的问题却经常是 ungoogleable 的。


一个项目对于 DevOps 来说，关键的地方是：怎么部署和怎么**持续地**进行部署 (continuous deployment)。


就目前遇到的一些项目来说，一般都是单台主机跑同一个应用的一到两个实例（初级阶段只能忍忍了），
某些极端（非盈利）的项目则可能会一台主机上跑多个应用。


根据这种情况，我一般会采用如下的部署模型：


## 文件组织结构

根据的 [FHS](http://en.wikipedia.org/wiki/Filesystem_Hierarchy_Standard)，
Web 应用的代码应该放到 `/srv` 或者 `/var/www` 下。但大部分服务器实例的根分区大小都会比较小，
而且没有另外分区（初级阶段很少有机会去定制磁盘），所以我更倾向于将实例放到外挂硬盘目录 `/mnt` 下。
当然将外挂硬盘挂到 `/var` 或者 `/srv` 下也是可以的。


总之原则就是：服务代码和服务数据尽可能和系统的分区隔离，将服务数据放到空间比较大的磁盘上。


在实例文件夹下，又可以划分为以下几个部分：

```
/mnt
  |- apps
  |- logs
  |- public
```

其中 `apps` 用来放置服务源代码， `logs` 则如其名，存放日志记录（所有的，包括分卷之后的，初级阶段没有中央日志系统），
`public` 用来放置一般的静态文件。


同时为了满足同时部署多个应用的多个实例（分支）的要求，这三个父级文件夹下还应该按照如下命名规则进行划分：

```
<%= app_name %>_<%= instance_name %>
```

通过以上的方法可以做到比较基本的隔离。各路径只需要在具体的配置文件里指明即可。


### Bouns: 一个简单的 Nginx 配置

```nginx
server {
    listen 80;
    server_name _;

    root /mnt/apps/sample_app_production;

    access_log /mnt/logs/sample_app_production/nginx.access.log;
    error_log /mnt/logs/sample_app_production/nginx.error.log;

    location ~* /static/(.*) {
        root /mnt/public/sample_app_production;

        access_log off;
    }

    location / {
        proxy_pass http://sample_app_production_upperstream;
    }
}
```

## 配置

应用配置处理方法和 [The Twelve-Factor App](http://12factor.net/config) 一样，采用环境变量注入。


## 部署

部署方面暂时使用了 [fabric](http://fabric.readthedocs.org/en/latest) 来写一系列的脚本。


之所以说是**一系列**是因为如果把任务都写在一个 `fabfile.py` 里面的话，
一段时间后（我）往往会被庞大的 `fabfile` 吓得忘记怎么执行某个操作了。


更好的做法是将 `fabfile` 分拆成按照**组件**划分的模块（OO 化）：

```bash
$ ls
alembic.py  codebase.py  constants.py  database.py  environment.py  fabfile.py  nginx.py  paths.py  supervisor.py  utils.py  virtualenv.py
```


然后在各个组件中定义基本的操作，如 `provision`、`deploy`、`reload` 等等。
顶层的 `fabfile` 就可以通过组合各个模块的基本操作进行任务拼装。


当然采取这种方法来写的话需要写大量的代码来实现某个系统服务的封装，所以下次可能会尝试一下 puppet 或者 ansible 这种内置语法糖做配置的工具。


**bouns**: fabric 的话还可以配合 [fabtools](http://fabtools.readthedocs.org) 来使用，能少写很多代码。


### 初始部署 (provision)

初始部署最坑爹的地方在于环境配置。 PHP 项目还算简单，只要连同 `vendor` 一起打包上传解压即可。
Python 的话就麻烦许多了。一般来说就是一个应用的一个实例配置一个 `virtualenv` 环境，
分别从 `requirements.txt` 里安装依赖。


我习惯把 `virtualenv` 的环境都统一放置在 `$HOME/.vritualenvs` 下。


监控管理应用运行一般使用 [supervisor](http://supervisord.org)，在配置里指定对应环境变量即可。


### 持续部署 (continuous deployment)

同样的，PHP 有先天优势（当然速度上来看也是缺陷），更新文件即可以完成应用部署更新了；所以持续部署问题不大。


但 Python 应用一般需要进行以下几个步骤：

- 同步代码
- 重启应用进程


同样地，配合 `supervisor` 来用即可（reload）。


**bouns a**: `supervisor` 中如果一个服务杀不死（不干净，经常出现在多进程的应用中），
可以指明 `killasgroup` / `stopasgroup` 来进行 `kill`。


**bouns b**: `supervisor` 的 `command` 不要用 shell 脚本来启动实例，
否则 reload 的时候不能正确 kill 实例的 pid，而是 kill shell 脚本的 pid；从而导致端口依旧被占用。
