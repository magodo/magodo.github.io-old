---
layout: "post"
title: "PulseAudio"
categories:
- "audio"
---

<!--more-->

***
Table of Content

* TOC
{:toc}
***

# 1. 简介

PulseAudio是一款面向POSIX操作系统的音频系统。它的主要功能是允许用户在不同程序，声卡，甚至不同主机上的声音数据进行传递和混合。同时，它也可以改变音频数据的格式，采样率，通道数。

# 2. 软件架构

PulseAudio会启动一个后台进程，用于接收来自一个或多个Source的音频数据（进程或者声卡），可选择地做一些音频数据的转换（格式，采样率，通道数），接着将数据传到一个或多个Sink（其他进程，声卡，其他网络上的PulseAudio进程）。

PulseAudio的一个目标是使所有的音频流都通过自己的后台进程，包括那些直接操作声卡设备的进程。之所以PulseAudio可以做到这点，是因为它提供给那些使用其他音频系统（例如:ALSA）的应用程序各种适配器（Adapter）。

对于Linux，默认的音频系统用的是ALSA，与声卡设备交互的数据都是通过ALSA driver来实现写入或者接收的。PulseAudio在安装到Linux系统时会通过配置ALSA，将它的默认设备设置为PulseAudio提供的一个虚拟声卡设备。于是，使用ALSA的应用程序会将音频数据输出值PulseAudio(虽然这些应用程序以为是在操作ALSA设备)，然后PulseAudio再使用ALSA接口去操作真正的声卡设备。

同时，PulseAudio也提供native的接口给应用程序，使他们可以直接和PulseAudio交互。

对于不同的应用程序，在使用PulseAudio的操作系统中，音频数据的流通路线大致有以下几种：

1. 使用ALSA接口与pulse设备交互的应用程序

    Sound source -> libALSA -> PulseAudio -> ALSA driver -> hardware

2. 使用PulseAudio接口的本地应用程序

    Sound source -> PulseAudio -> ALSA driver -> hardware

3. 使用PulseAudio接口的网络应用程序

    Sound source -> PulseAudio -> network -> PulseAudio -> ALSA driver -> hardware

4. 使用ALSA接口与其他ALSA设备交互的应用程序

    Sound source -> libALSA -> ALSA driver -> hardware

5. 直接与声卡设备交互

    Sound source -> ALSA driver -> hardware

Manuel Amador Briz 提供了一幅很详细的图示：

![pulseAudio architecture](/images/pulseaudio/Pulseaudio-diagram.png)


# 3. 进入PulseAudio命令行

首先，请检查下在当前系统中PulseAudio是否已经启动。如果没有，那么可以执行：

    # pulseaudio -nC

来进入命令行。

否则，有两种情况：

1. PulseAudio在deamoniz的时候传入`--disallow-module-loading`参数：

    # # 先将该参数从启动脚本/service文件中去掉并重启该进程
    # pacmd

2. PulseAudioz在daemonize的时候没有该参数：

    # pacmd

