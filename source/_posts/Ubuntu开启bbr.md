---
title: Ubuntu开启bbr
date: 2020-05-13 08:21:42
tags:
- os
---
当前场景为Ubuntu 2020.04LTS ,内核5.4，只需要开启配置即可

## 1. 修改系统变量
~~~bash
# echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf
# echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf
~~~

## 保存生效，配置内核
~~~bash
# sysctl   -p
~~~

显示如下
~~~bash
net.core.default_qdisc = fq
net.ipv4.tcp_congestion_control = bbr
~~~

## 3、  查看内核是否已开启BBR
~~~bash
# sysctl net.ipv4.tcp_available_congestion_control
~~~
结果如下
~~~bash
net.ipv4.tcp_available_congestion_control = reno cubic bbr
~~~

~~~bash
# sysctl net.ipv4.tcp_congestion_control
~~~

~~~bash
net.ipv4.tcp_congestion_control = bbr
~~~

## 4、 验证BBR是否已经启动

~~~
# lsmod | grep bbr
~~~

结果如下
~~~bash
tcp_bbr                20480  1 
~~~
