---
title: 使用HAProxy加速Shadowsocks
date: 2019-03-01 09:52:51
tags:
- software
---

## 一、介绍

HAProxy
The Reliable, High Performance TCP/HTTP Load Balancer
HAProxy 是一个高效的负载均衡软件，可以实现 TCP/HTTP 的代理。这里使用它将我们发给它的请求转发给 ss 服务器。

ss 支持中继，那么只要有一个服务器，连接自己电脑和 ss 服务器都很快的话就能实现加速
~~~bash
客户端 < - > 中继服务器 < - > Shadowsocks 服务器
~~~

## 二、 安装
~~~bash
// 以 CentOS 7 为例
yum install haproxy
~~~

## 三、配置
编辑 /etc/haproxy/haproxy.cfg，保存以下内容
~~~bash
global
        ulimit-n  51200

defaults
        log global
        mode    tcp
        option  dontlognull
        contimeout 1000
        clitimeout 150000
        srvtimeout 150000

frontend ss-in
        bind *:8388
        default_backend ss-out

backend ss-out
        server server1 222.222.222.222:2222 maxconn 20480
~~~
其中，*:8388 中的 8388 是中继服务器接受请求的端口，222.222.222.222:2222 是 ss 服务器的 IP 地址加端口号。
然后执行
~~~bash
service haproxy restart
~~~
HAProxy 就会在后台进行启动。可以使用 ps -ef 查看进程，lsof -i 查看端口占用情况来验证 HAProxy 是否已经运行。若无法连接中继服务器，使用 iptables -L 查看防火墙规则是否有问题。

客户端的配置，只要将原来配置的 ip 地址和端口更换成中继服务器的 ip 地址和端口号就可以了。

### 参考
[Setup a Shadowsocks relay](https://github.com/shadowsocks/shadowsocks/wiki/Setup-a-Shadowsocks-relay)

[使用 HAProxy 加速 Shadowsocks](https://kaywu.xyz/2016/06/19/Shadowsocks-HAProxy/)

[HAProxy](http://www.haproxy.org/)
