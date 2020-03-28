---
title: aria2简明教程
date: 2018-12-30 10:44:49
tags:
- 工具
---

# [aria2](https://aria2.github.io)

> aria2是一个轻量级的多协议和多源命令行下载实用程序。它支持HTTP / HTTPS，FTP，SFTP，BitTorrent和Metalink。 aria2可以通过内置的JSON-RPC和XML-RPC接口进行操作。
> Aria2作为一款Linux下的下载神器，很多极客都在使用，可以下载http资源、种-子文件、磁力链接等，功能强大，而且整合Chrome插件可以摆脱百度云盘的速度限制。Aria2具有特点：
- 高速，自动多线程下载；断点续传；
- 轻量占用内存非常少，通常情况平均4~9MB内存占用(官方介绍)；
- 多平台。支持 Win/Linux/OSX/Android 等操作系统下的部署；
- 模块化。分段下载引擎，文件整合速度快；
- 支持RPC界面远程；
- 全面支持BitTorrent协议；

## 下载

[github 下载链接](https://github.com/aria2/aria2/releases/tag/release-1.34.0)

## 使用方法

### Download from WEB:
~~~bash
$ aria2c http://example.org/mylinux.iso
~~~

### Download from 2 sources:
~~~bash
$ aria2c http://a/f.iso ftp://b/f.iso
~~~

### Download using 2 connections per host:
~~~bash
$ aria2c -x2 http://a/f.iso
~~~

### BitTorrent:
~~~bsah
$ aria2c http://example.org/mylinux.torrent
~~~

### BitTorrent Magnet URI:
~~~bash
$ aria2c 'magnet:?xt=urn:btih:248D0A1CD08284299DE78D5C1ED359BB46717D8C'
~~~

### Metalink:
~~~bash
$ aria2c http://example.org/mylinux.metalink
~~~

### Download URIs found in text file:
~~~bash
$ aria2c -i uris.txt
~~~
## 参数使用

具体参数可以使用aria2c --help来查看

常用参数:


~~~bash
aria2c -d /home/Download -c -x 16
~~~

## Aria2 配置说明
[Aria2 & YAAW 使用说明](http://aria2c.com/usage.html)
[aria2c(1)](https://aria2.github.io/manual/en/html/aria2c.html)

## 贴一下我的aria2.conf配置文件
~~~bash

## `#`开头为注释内容, 选项都有相应的注释说明, 根据需要修改 ##
## 被注释的选项填写的是默认值, 建议在需要修改时再取消注释  ##
## 如果出现`Initializing EpollEventPoll failed.`之类的
## 错误提示, 可以取消event-poll选项的注释                  ##

## 文件保存相关 ##

# 文件的保存路径(可使用绝对路径或相对路径), 默认: 当前启动位置
dir=Aria2Data
# 启用磁盘缓存, 0为禁用缓存, 需1.16以上版本, 默认:16M
disk-cache=32M
# 文件预分配方式, 能有效降低磁盘碎片, 默认:prealloc
# 预分配所需时间: none < falloc ? trunc < prealloc
# falloc和trunc则需要文件系统和内核支持, NTFS建议使用falloc, EXT3/4建议trunc
file-allocation=falloc
# 断点续传
continue=true

## 下载连接相关 ##

# 最大同时下载任务数, 运行时可修改, 默认:5
max-concurrent-downloads=5
# 同一服务器连接数, 添加时可指定, 默认:1
max-connection-per-server=5
# 最小文件分片大小, 添加时可指定, 取值范围1M -1024M, 默认:20M
# 假定size=10M, 文件为20MiB 则使用两个来源下载; 文件为15MiB 则>使用一个来源下载
min-split-size=10M
# 单个任务最大线程数, 添加时可指定, 默认:5
split=5
# 整体下载速度限制, 运行时可修改, 默认:0
#max-overall-download-limit=0
# 单个任务下载速度限制, 默认:0
#max-download-limit=0
# 整体上传速度限制, 运行时可修改, 默认:0
#max-overall-upload-limit=0
# 单个任务上传速度限制, 默认:0
#max-upload-limit=0
# 禁用IPv6, 默认:false
disable-ipv6=true

## 进度保存相关 ##

# 从会话文件中读取下载任务
input-file=aria2.session
# 在Aria2退出时保存`错误/未完成`的下载任务到会话文件
save-session=aria2.session
# 定时保存会话, 0为退出时才保存, 需1.16.1以上版本, 默认:0
save-session-interval=60

## RPC相关设置 ##

# 启用RPC, 默认:false
enable-rpc=true
# 允许所有来源, 默认:false
rpc-allow-origin-all=true
# 允许非外部访问, 默认:false
rpc-listen-all=true
# 事件轮询方式, 取值:[epoll, kqueue, port, poll, select], 不同>系统默认值不同
#event-poll=select
# RPC监听端口, 端口被占用时可以修改, 默认:6800
#rpc-listen-port=6800

## BT/PT下载相关 ##
# 当下载的是一个种子(以.torrent结尾)时, 自动开始BT任务, 默认:true
#follow-torrent=true
# BT监听端口, 当端口被屏蔽时使用, 默认:6881-6999
listen-port=51413
# 单个种子最大连接数, 默认:55
#bt-max-peers=55
# 打开DHT功能, PT需要禁用, 默认:true
enable-dht=false
# 打开IPv6 DHT功能, PT需要禁用
#enable-dht6=false
# DHT网络监听端口, 默认:6881-6999
#dht-listen-port=6881-6999
# 本地节点查找, PT需要禁用, 默认:false
#bt-enable-lpd=false
# 种子交换, PT需要禁用, 默认:true
enable-peer-exchange=false
# 每个种子限速, 对少种的PT很有用, 默认:50K
#bt-request-peer-speed-limit=50K
# 客户端伪装, PT需要
peer-id-prefix=-TR2770-
user-agent=Transmission/2.77
# 当种子的分享率达到这个数时, 自动停止做种, 0为一直做种, 默认:1.0
seed-ratio=0
# 强制保存会话, 话即使任务已经完成, 默认:false
# 较新的版本开启后会在任务完成后依然保留.aria2文件
#force-save=false
# BT校验相关, 默认:true
#bt-hash-check-seed=true
# 继续之前的BT任务时, 无需再次校验, 默认:false
bt-seed-unverified=true
# 保存磁力链接元数据为种子文件(.torrent文件), 默认:false
bt-save-metadata=true
~~~
