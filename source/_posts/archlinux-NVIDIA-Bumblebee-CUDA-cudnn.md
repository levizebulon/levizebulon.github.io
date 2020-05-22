---
title: archlinux+NVIDIA+Bumblebee+CUDA+cudnn
date: 2020-05-22 22:04:13
tags:
- software
---

## [NVIDIA](https://wiki.archlinux.org/index.php/NVIDIA_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)#%E6%9C%80%E5%B0%8F%E9%85%8D%E7%BD%AE%E6%A8%A1%E5%BC%8F)

### 1. 安装
`pacman -S nvidia`
或者
`pacman -S nvidia-lts`

### 2. 配置
安装完驱动之后，您需要创建Xorg的配置文件。您可以运行一次测试来检验没有配置
文件Xorg能否正确运行。但是，您可能需要创建配置文件`/etc/X11/xorg.conf`来调
整Xorg运行时的一些变量。您可以用nvidia-xconfig配置工具来生成这些文件，也可
以手动创建。假如您是手动创建的话，它可以是一个最小的配置文件(也就是意味着
它仅仅把一些基础的选项传给Xorg服务器)，或者包含一些自定义的配置来绕开Xorg
的自动发现或者是预配置的选项。

#### 自动配置

英伟达的软件包已经包含一个自动配置的工具来帮助您创建Xorg的配置文件(xorg.conf)您可以通过运行下面的命令来实现自动配置：

~~~bash
nvidia-xconfig
~~~

当Xorg的配置文件`xorg.conf`不存在时，这条命令会自动检测您的硬件，并创建文件`/etc/X11/xorg.conf`。假如配置文件已经存
在的话，它会进行一些编辑，以方便在Xorg运行时能成功载入英伟达的专有驱动。

## [Bumblebee](https://wiki.archlinux.org/index.php/Bumblebee_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87))

安装 Bumblebee 之前，检查你的 BIOS 并尽可能激活 Optimus (老式电脑称之为"可切换显卡"，
BIOS有可能没有提供此项设置)。如果 "Optimus" 和 "switchable" 都没有在BIOS里，就保证
两种GPU都已启用并且集成显卡是主要显示设备。显示应该连接在主板上的集成显卡，而不是
独立显卡。如果集成显卡之前被禁用而安装了独立显卡的驱动，那就删除 `/etc/X11/xorg.conf` 
或者有关独立显卡的 `/etc/X11/xorg.conf.d` 中的文件。

### 1. 安装
- [bumblebee](https://www.archlinux.org/packages/?name=bumblebee) - 提供守护进程以及程序的主要安装包。
- [mesa ](https://www.archlinux.org/packages/?name=mesa)- 开源的 OpenGL 标准实现。
- 对于合适的NVIDIA驱动，参看[NVIDIA](https://wiki.archlinux.org/index.php/NVIDIA_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)#.E5.AE.89.E8.A3.85)安装 。
- [xf86-video-intel](https://www.archlinux.org/packages/?name=xf86-video-intel)- Intel 驱动（可选,用来做显示，nvidia只用来做计算）。

## [CUDA](https://wiki.archlinux.org/index.php/GPGPU#CUDA)

## [cudnn](https://security.archlinux.org/package/cudnn)
