---
layout: "post"
title: "Supervisor"
categories:
- "python"
---

<!--more-->

***
Table of Conetent

* TOC
{:toc}
***

Supervisor是一个用于在类Unix系统中控制和监视进程的一个服务器/客户端形式的系统，类似于 `systemd`, `launchd` 之类的 `init system` 但并不是它们的替代品。

Supervisor和其他进程一样，在系统启动时被运行。同时，Supervisor会根据用户的配置，将指定的程序相应地运行起来。

## 1 组件

### supervisord

Supervisor的服务端进程。它负责启动，响应客户端程序，在出错的时候重启客户端程序，以及将客户端程序的标准输出和错误记录到日志中。

### supervisorctl

命令行工具，用于控制客户端程序。它是通过UDS(Unix domain socket)或者TCP socket的方式supervisord进行通信。

### Web Server

Supervisor提供了一个功能类似 `supervisorctl` 的网页界面。通过访问服务端的URL(e.g. `http://localhost:9001/` )，用户可以查看和控制客户端程序的状态。

这个组件工作的前提条件是配置文件中定义了 `[inet_http_server]` 段。

### XML-RPC Interface

启动上面那个Web UI的HTTP服务器同时启动了一个 XML-RPC 接口服务，它可以用于查询和控制supervisor和它启动的程序。

## 2 安装Supervisor

由于我使用Archlinux发型版，社区已经提供了Supervisor的包，所以只要简单地执行：

    [magodo@t460p _posts]$ sudo pacman -S supervisor

即可。值得注意的是，supervisord是通过 `systemd` 启动的:

    [magodo@t460p _posts]$ cat /usr/lib/systemd/system/supervisord.service
    [Unit]
    Description=Process Monitoring and Control Daemon
    Documentation=http://supervisord.org
    After=network.target

    [Service]
    Type=forking
    ExecStart=/usr/bin/supervisord -c /etc/supervisord.conf
    ExecReload=/usr/bin/supervisorctl reload
    ExecStop=/usr/bin/supervisorctl shutdown

    [Install]
    WantedBy=multi-user.target

因此，需要enable/start。

## 3 配置文件

安装完毕后，可以在命令行中执行 `echo_supervisord_conf`. 它会打印出一个配置文件的样本。我们可以利用这个样本来创建。不过arch版本的包中已经自带了配置文件，因此使用默认的配置文件。

具体的配置项还请参考[官方文档](http://supervisord.org/configuration.html).

## 4 运行程序

supervisord会在启动的时候解析配置文件，其中关于要启动哪些客户端程序的信息记录在配置文件中的 `[program:xxx]` 段（其中的 `xxx` 准确地应该称为 *同类别程序组*  (详见)[http://supervisord.org/configuration.html#program-x-section-settings]）

在添加某个程序的时候，用户可以直接修改配置文件，不过这是不推荐的。推荐的做法是在配置文件中加入一个 `[include]` 段，在其中指定需要append的文件的路径（模式）。例如，arch的supervisor包中的配置文件有如下配置：

    [include]
    files = /etc/supervisor.d/*.ini

假设我需要运行 `uwsgi`，那么我可以在 `/etc/supervisor.d/` 中创建文件 `uwsgi.ini` ：

    [program:uwsgi]
    command=/usr/bin/uwsgi --emperor /etc/uwsgi/apps-enabled
    stopsignal=QUIT
    autostart=true
    autorestart=true
    redirect_stderr=true

这里如果不指定 `user` ，uwsgi进程将会以supervisord的uid 启动。
