---
layout: "post"
title: "使用supervisor+uWSGI+nginx部署flaskbb"
categories:
- "python"

<!--more-->

***
Table of Conetent

* TOC
{:toc}
***

最近在研究部署一个论坛软件，供部门内部交流学习用。选择了自己有一定基础的 (`flaskbb`)[http://flaskbb.readthedocs.io/en/latest/index.html]。不过，这个东东好像不怎么维护的样子。虽然有一些文档，但是市面上貌似没有哪个论坛是基于它的。不过，为了减少学习成本我打算还是试用一下再说。

官网的部署步骤使用的是 `supervisor + uWSGI + nginx` 的方法，我也就先照做。

# 1 supervisor

Supervisor是一个用于在类Unix系统中控制和监视进程的一个服务器/客户端形式的系统，类似于 `systemd`, `launchd` 之类的 `init system` 但并不是它们的替代品。

Supervisor和其他进程一样，在系统启动时被运行。同时，Supervisor会根据用户的配置，将指定的程序相应地运行起来。

## 1.1 组件

### supervisord

Supervisor的服务端进程。它负责启动，响应客户端程序，在出错的时候重启客户端程序，以及将客户端程序的标准输出和错误记录到日志中。

### supervisorctl

命令行工具，用于控制客户端程序。它是通过UDS(Unix domain socket)或者TCP socket的方式supervisord进行通信。

### Web Server

Supervisor提供了一个功能类似 `supervisorctl` 的网页界面。通过访问服务端的URL(e.g. `http://localhost:9001/` )，用户可以查看和控制客户端程序的状态。

这个组件工作的前提条件是配置文件中定义了 `[inet_http_server]` 段。

### XML-RPC Interface

启动上面那个Web UI的HTTP服务器同时启动了一个 XML-RPC 接口服务，它可以用于查询和控制supervisor和它启动的程序。

## 1.2 安装Supervisor

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

## 1.3 配置文件

安装完毕后，可以在命令行中执行 `echo_supervisord_conf`. 它会打印出一个配置文件的样本。我们可以利用这个样本来创建。不过arch版本的包中已经自带了配置文件，因此使用默认的配置文件。

具体的配置项还请参考[官方文档](http://supervisord.org/configuration.html).

## 1.4 运行程序

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

# 2 uWSGI

uWSGI是一个web server，它实现了包括 uwsgi在内的多个通信协议。

## 2.1 安装

## 2.2 配置

创建如下目录和文件：

    [magodo@t460p uwsgi]$ pwd
    /etc/uwsgi
    [magodo@t460p uwsgi]$ tree
    .
    ├── apps-available
    │   └── flaskbb
    │       └── flaskbb.ini
    └── apps-enable -> apps-available/flaskbb/

其中，flaskbb.ini 中的内容为：

    [uwsgi]
    base = /home/magodo/github/code/flaskbb
    virtualenv = /home/magodo/.virtualenvs/flaskbb
    pythonpath = %(base)
    http = 127.0.0.1:30002
    module = wsgi
    callable = flaskbb
    #logto = /tmp/apps/flaskbb/logs/uwsgi.log
    #uid = flaskbb
    #gid = flaskbb

这里我把 `uid` 和 `gid` 注释掉的原因是 `flaskbb` 是我创建的一个system user (nologin/nohome), 而我的virtualenv创建在 `magodo` 这个用户的home目录下，如果uwsgi启动的flaskbb以 `flaskbb` 用户来跑的话，它无法正确使用virtualenv，最后会在uwsgi的log中找到类似：

    ImportError: no module named site

的错误。

# 3 nginx

# 4 flaskbb

对于 `flaskbb` 本身，我也暂时做了一些改动。在文件 `wsgi.py`中，改为如下内容：

    from flaskbb import create_app
    #from flaskbb.configs.production import ProductionConfig
    from flaskbb.configs.development import DevelopmentConfig

    #flaskbb = create_app(config=ProductionConfig())
    flaskbb = create_app(config=DevelopmentConfig())

也就是说，在uWSGI启动的时候，使用开发模式的配置。

