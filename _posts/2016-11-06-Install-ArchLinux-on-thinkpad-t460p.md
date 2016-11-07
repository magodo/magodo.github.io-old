---
layout: "post"
title: "Install ArchLinux on thinkpad t460p"
categories:
- "OS"
---

<!--more-->

***
Table of Content

* TOC
{:toc}
***

BIOS设置
-------

1. Security -> Secure Boot -> Secure Boot: Disabled
2. Startup -> EFI/Legacy Boot: UEFI Only
3. Config -> Display -> Total Graphics Memory: 512MB

使用脚本安装
------------

详情请参见[这里](https://github.com/magodo/t460p-arch)


安装完之后的第一次登录
----------------------

第一次登录之后，会发现终端输出类似如下的错误：

    nouveau:02:00.0: DRM: Pointer to TMDS table invalid
    nouveau 0000:02:00.0: DRM: Pointer to flat panel table invalid

以及:

    mei_wdt mei::xxxxxx: Could not register event ret=-22

桌面环境
--------

### 安装xorg

1. `# pacman -S xorg-server`
2. `# pacman -S xorg-xinit`

  * 创建`~/.xserverrc`:
  
      #! /bin/sh
      exec /usr/bin/Xorg -nolisten tcp "$@" vt$XDG_VTNR

  * `# cp /etc/X11/xinit/xinitrc ~/.xinitrc`

  测试：

  * `# pacman -S xorg-xclock xterm`
  * `# startx`

### 安装gnome

1. `# pacman -S gnome3`
2. 按照[wiki](https://wiki.archlinux.org/index.php/GNOME), 配置成手动启动: 在`~/.xinitrc`中，把下面的部分从`twm &`开始都注释，然后加上`exec gnome-session`



显卡驱动
--------

由于thinkpad t460p有两个显卡设备：

    # lspci -k | grep -A 3 -E "(VGA|3D)" 

    00:02.0 VGA compatible controller: Intel Corporation HD Grahics 530 (rev 06)
        Subsystem: Lenovo Device 5050
        Kernel driver in use: i915
        Kernel modules: i915

    02:00.0 3D controller: NVIDIA Corporation GM108M [GeForce 940MX] (rev a2)
        Subsystem: Lenovo Device 5050
        Kernel driver in use: nouveau
        Kernel modules: nouveau

NVIDIA的Optimus技术允许集成GPU与独立的NVIDIA GPU被笔记本电脑一起使用。在t460p上，想要让两张显卡正常工作（不一定同时工作）的话，有以下几种配置方式：

1. 在BIOS里将其中一张显卡关闭。不过并不是所有的BIOS都支持这种设置，例如t460p的貌似就不支持 (请参考[这里](https://www.reddit.com/r/thinkpad/comments/4q72qt/t460p_fedora_24_nvidia_940mx_driver_cant_disable/))

2. 使用NVIDIA官方驱动所提供的Optimus支持，这个能提供最好的NVIDIA性能，但是不允许GPU切换，也可能比开源的驱动有更多bug

3. 使用第三方工具*bumblebee*来实现类似Optimus的功能，它提供了GPU切换(参见[这里](https://wiki.archlinux.org/index.php/Bumblebee))

### bumblebee 方案 ###

安装前，先保证删除`/etc/X11/xorg.conf`和`/etc/X11/xorg.conf.d`

    # pacman -S bumblebee

接着，它会让你选择一个提供libgl的包，根据[arch wiki](https://wiki.archlinux.org/index.php/Bumblebee):

>>> Note: bumblebee depends on mesa-libgl and provides all nvidia-libgl, nvidia-340xx-libgl and nvidia-304xx-libgl to avoid dependency conflict between the respective libgl versions.

我选择了*mesa-libgl*.

然后，安装*mesa*, *xf86-video-intel*, 以及nvidia的包：*nvidia* or *nvidia-340xx* or *nvidia-304xx*. 通过，这里个[wiki](https://wiki.archlinux.org/index.php/NVIDIA)知道，t460p的GeForce 940MX是比400系列更新的版本，应该安装*nvidia*或者*nvidia-lts*. 我选择的是*nvidia*. 安装过程中，会让你选择安装*xf86-input-evdev* 或者 *xf86-input-libinput*。根据[这里](https://www.reddit.com/r/archlinux/comments/48tqj9/difference_between_libinput_and_evdev/)，我选择安装后者。

安装完毕后，将系统中的用户（例如*root*）加到`bumblebee`的group里：

    # gpasswd -a magodo bumblebee

然后，enable `bumblebeed.service`:

    # systemctl enable bumblebeed.service

重启。。。

重启之后，发现之前第一次重启的两个错误log没了：

    nouveau:02:00.0: DRM: Pointer to TMDS table invalid
    nouveau 0000:02:00.0: DRM: Pointer to flat panel table invalid

接下来，我们要检查一下bumblebee是否工作。安装`mesa-demos`，并且执行`glxgears`:

    # optirun glxspheres64

然后，我就遇到了如下的问题：

[[ERROR]Cannot access secondary GPU: No devices detected](https://wiki.archlinux.org/index.php/bumblebee#.5BERROR.5DCannot_access_secondary_GPU:_No_devices_detected)

这里的话，按照wiki上面是说的，在`/etc/bumblebee/xorg.conf.nvidia`中，把`BusID`那行去注释，然后设上N卡的BusID（`# lspci | grep 3D`），在我这里得到的是`02:00.0`。这里有两点要注意：

1. 写到文件里的时候把最后的`.`改成`:`
2. `lspci`的输出是16进制的，而写到文件里的应该是10进制的

这下，`# optirun glxspheres64` 可以跑了


现有问题
=======

1. ctrl-alt-NUM 切换到虚拟终端显示错误
2. wifi不能自动连接
3. 在gnome3上没法打开terminal
