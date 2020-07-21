---
title: centos非root用户yum安装软件方法
date: 2020-07-21 20:21:07
tags:
- Linux
---
# 1. 查看yum中是否有你需要的包

比如想安装`tmux`，可以这样查看

~~~bash
yum list tmux
~~~

# 2. 下载rpm包

然后我们从仓库中下载rpm包，比如我们要下载`tmux.x86_64`，我们可以这样下载：

~~~bash
yumdownloader tmux.x86_64
~~~

# 3. 解压RPM包

~~~bash
rpm2cpio tmux-1.8-4.el7.x86_64.rpm | cpio -idvm
~~~

# 添加环境变量

如果解压的路径是在home目录的话，那么需要这样添加即可

~~~bash
vim  ~/.bashrc
export PATH=$PATH:$HOME/usr/bin/
~~~

# 引用

[centos 非root用户(普通用户)替换yum安装软件方法](https://blog.csdn.net/sty945/article/details/80888872)

