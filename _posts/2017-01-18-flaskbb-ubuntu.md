---
layout: "post"
title: "flaskbb-ubuntu"
categories:
- "python"
---

<!--more-->

***
Table of Conetent

* TOC
{:toc}
***

# 1 激活 `www-data` 用户

我们希望nginx和uwsgi都以 `www-data` 这个系统默认的给web应用使用的用户来执行。这要求我们将项目文件和python的虚拟环境的owner设置为 `www-data`。所以，在一开始的时候就以这个用户来配置环境会比较方便。

所谓的激活，就是使用 `su` 指令切换当前用户为 `www-data` 而已。先看下这个用户的默认配置（也可以直接查看 `/etc/passwd` ）:

    www-data@hiv8039u:/srv/www/flaskbb$ finger www-data
    Login: www-data                         Name: www-data
    Directory: /var/www                     Shell: /bin/sh
    Never logged in.
    No mail.
    No Plan.

可见这个用户并不是 *no login* 用户，也就是说它是有shell的。

似乎这些系统用户（例如: root也是）是没有默认密码的，所以需要首先给它们设个密码才可以使用 `su` 切换：

    $ sudo passwd www-data
    # 输入密码
    # 验证密码

现在，可以正常切换到 `www-data` 了：

    $ su -s /bin/bash www-data

下面的操作如果没有特殊说明，我们都以 `www-data` 的用户来执行（以 `>` 来标识）。

# 2 下载安装flaskbb

这里按照(官网)[http://flaskbb.readthedocs.io/en/latest/installation.html#installation]上面的 *简陋* 的指令去安装flaskbb.

同时，根据(FHS)[http://www.pathname.com/fhs/] 的建议，我们将工程目录放置在 `/srv/www` 下面。

## 2.1 virtualenvwrapper

首先，安装这个virtualenv的封装工具：

    > sudo pip install virtualenvwrapper

从github上面下载flaskbb:

    > git clone https://github.com/sh4nks/flaskbb.git

为了便于管理，我选择将 `virtualenv` 放置在flaskbb的项目工程下：
    
    > # 设置环境变量，source脚本
    > export WORKON_HOME=/srv/www/flaskbb/.virtualenvs
    > export VIRTUALENVWRAPPER_SCRIPT=/usr/local/bin/virtualenvwrapper.sh
    > source /usr/local/bin/virtualenvwrapper_lazy.sh
    > # 创建virtualenv
    > cd /srv/www/flaskbb
    > mkvirtualenv -a . -p $(which python2) flaskbb
    > # 自动进入virtualenv flaskbb
    > # 安装dependency
    > pip install -r requirements.txt
    > # 接下来也可以安装一些可选的依赖包（例如redis），这里就不介绍了
    > # 退出 virtualenv
    > deactivate

# 2.2 flaskbb的配置文件

flaskbb提供了2个配置文件可供选择，它们都在 `flaskbb/flaskbb/configs/` 目录下面。不过，我发现现有的配置文件貌似忘记了在文件的第一行指定encoding，如果不加的话在后面用uWSGI启动貌似会出错。所以，在这里给 `development.py.example` 和 `production.py.example` 都加上这行：

    # -*- coding: utf-8 -*-

然后，按照官网上说的，把两个 `.example` 文件都copy成不带这个后缀的文件：

    > cp flaskbb/configs/development.py.example flaskbb/configs/development.py
    > cp flaskbb/configs/production.py.example flaskbb/configs/production.py

其中，`production.py` 中我们有一些配置一定要修改才能用，否则运行时会出错。包括：

* SERVER_NAME

这个选项我把它注释掉了，因为这个选项只在sub domain name需要用到的时候才需要设置，我这台机器当前应该只跑一个site，所以先不设了。

另外，如果有SMTP邮件服务器的，也可以配置E-mail部分。不过，我现在没有配置。一来，我不知道怎么配置；二来flaskbb允许不需要邮件验证的注册，后面会提到。

# 2.3 安装flaskbb

先进入virtualenv:

    > workon flaskbb

然后，按照官网上说的，执行：

    > make install

在安装过程中，会需要你输入一些账户信息，这个账户是flaskbb会为你创建的一个账户，一般把group设置为管理员。

# 3 uWSGI

我们使用uWSGI来作为web服务器，它会被配置为通过某个UDS与Nginx通讯，将用户的请求从Nginx读取，使用uwsgi接口传给flaskbb。再将flaskbb的响应利用uwsgi接口传给Nginx。

# 3.1 安装

根据(uWSGI官网)[https://uwsgi-docs.readthedocs.io/en/latest/WSGIquickstart.html]的指令：

    # 注意，这里我将uwsgi安装到了系统当中，我没有安装到virtualenv中，因为我觉得没什么必要
    $ sudo pip install uwsgi

# 3.2 配置文件

建议的做法是，在 `/etc/uwsgi/` 下创建如下目录结构：

    ├── apps-available
    │   └── flaskbb
    │       └── flaskbb.ini
    └── apps-enable -> apps-available/flaskbb/

然后，通过 `sudo uwsgi --emperor /etc/uwsgi/apps-enabled` 来执行。

`flaskbb.ini` 的内容如下：

值得注意的是，由于配置文件中指定了`uid` 和 `gid` ，所以 `uwsgi` 需要以root权限执行。

# 4 Nginx

## 4.1 安装

使用发行版的包安装：

    $ sudo apt-get install nginx

## 4.2 启动

因为现在公司里的运行环境是Ubuntu12.04LTS, 使用的是sysv init初始化系统，所以需要执行：

    # start
    $ sudo service nginx start
    # enable
    $ sudo update-rc.d nginx defaults

验证Nginx是否正确启动，打开浏览器，输入你的eth0的ip地址，页面上会出现类似 **welcome nginx** 之类的字眼就代表安装成功。

## 4.3 配置flaskbb

根据flaskbb官网的建议配置，做一点微小的改动即可，大概包括：

* server_name 使用的localhost
* 执行工程的路径使用的绝对路径
* location @flaskbb中使用UDS，而不是TCP socket

具体配置文件如下：

文件路径为 `/etc/nginx/sites-available/flaskbb`。同样，需要创建一个link指向该文件，link为 `/etc/nginx/sites-enabled/flaskbb`

配置完毕后，别忘了重启nginx service:

    $ sudo service nginx restart 

# 5 flaskbb bug fix

flaskbb本身好像没人维护了的样子，而它官网又说是under development，不建议使用production的配置。所以可以预料的到，有很多坑在里面。这里我列举了一些 *可能* 的问题。

# 6 TODO

这里列了一些按如上配置之后运行中的错误/警告，暂时还不知道原因，需要进一步调查。

* 执行uWSGI之后出现如下错误讯息
    
    ... 
    added /srv/www/flaskbb/ to pythonpath.
    Error opening file for reading: Permission denied
    ...



