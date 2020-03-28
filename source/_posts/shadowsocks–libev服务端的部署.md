---
title: Ubuntu shadowsocks–libev服务端的部署
date: 2019-03-01 09:23:47
tags:
- software
---

## 一、介绍
Shadowsocks-libev是一款轻量级安全SOCKS5代理，适用于嵌入式设备和低端机箱。
[shadowsocks-libev README.md](https://github.com/shadowsocks/shadowsocks-libev/blob/master/README.md)

## 二、特征
Shadowsocks-libev是用纯C编写的，取决于libev。它旨在成为shadowsocks协议的轻量级实现，以便尽可能降低资源使用率。

## 三、安装
For Ubuntu 14.04 and 16.04 users, please install from PPA:
~~~bash
sudo apt-get install software-properties-common -y
sudo add-apt-repository ppa:max-c-lv/shadowsocks-libev -y
sudo apt-get update
sudo apt install shadowsocks-libev
~~~

Configure and start the service

~~~bash
# Edit the configuration file
sudo vim /etc/shadowsocks-libev/config.json

# Edit the default configuration for debian
sudo vim /etc/default/shadowsocks-libev

# Start the service
sudo /etc/init.d/shadowsocks-libev start    # for sysvinit, or
sudo systemctl start shadowsocks-libev      # for systemd
~~~

## 四、配置与启动
1、配置文件为：/etc/shadowsocks-libev/config.json，格式说明：
~~~bash
{
	"server":"example.com or X.X.X.X",
	"mode":"tcp_and_udp",
	"server_port":443,
	"password":"password",
	"method":"aes-128-gcm",
	"fast_open":true,
	"timeout":60
}
~~~
其中：
~~~bash
server：主机域名或者IP地址，尽量填IP

server_port：服务器监听端口

password：密码

method：加密方式 所有支持的加密方式请参照官方文档。这里本人推荐只使用支持AEAD的加密方式，包括以下五种： aes-128-gcm/aes-192-gcm/aes-256-gcm/chacha20-ietf-poly1305/xchacha20-ietf-poly1305

timeout：连接超时时间，单位秒。要适中。
~~~
注意引号。

## 五、服务器优化
1.  开启TCP Fast Open：
需求： 系统内核版本≥3.7，shadowsocks-libev≥3.0.4。
修改 /etc/sysctl.conf ，加入如下一行：
~~~bash
net.ipv4.tcp_fastopen = 3
~~~
执行如下命令使之生效：
~~~bash
sysctl -p
~~~
配置文件 /etc/shadowsocks.json 增加一行：

~~~bash
{
      "server": "X.X.X.X",
      "server_port": "443",
      "password": "password",
      "method": "aes-128-gcm",
      "fast_open": true
}
~~~
2. 其他优化
[魔改BBR](https://github.com/FunctionClub/YankeeBBR)

## 参考
shadowsocks-libev [shadowsocks-libev](https://github.com/shadowsocks/shadowsocks-libev/blob/master/README.md)
shadowsocks – libev 服务端的部署 [shadowsocks – libev 服务端的部署
](https://cokebar.info/archives/767)
YankeeBBR [YankeeBBR](https://github.com/FunctionClub/YankeeBBR)

