---
title: K2pdfopt简明教程 
date: 2018-12-29 20:20:14
tags:
- 工具
- 阅读
---

# What's K2pdfopt

> K2pdfopt 是一个开源软件。可以优化 PDF/DJVU 文件以适配移动阅读设备（比如 Kindle）以及智能手机。它能很好的处理有多栏内容的 PDF/DJVU 文件以及重排甚至是扫描版 PDF 文件的文本。它也能被当作一个标准的 PDF 操作工具，如复制、裁切、调整尺寸、ORC识别。它能生成原生或位图形式的 PDF，带有可选的 OCR 层。支持 Windows、Mac OS X、Linux 系统，其中 Windows 系统集成了一个带界面的版本。

# Why's K2pdfopt

> 用kindle看科学类书籍或文献


## 这里介绍Linux下的安装与使用

> 测试环境: Ubuntu 16.04

### 一、 下载

下载页面: [K2pdfopt Download Page]("http://www.willus.com/k2pdfopt/download/")

在 Linux 系统中，需要将下载的 k2pdfopt 文件移动到你自己的路径，并将其修改为可执行，然后再通过终端运行它。具体步骤请打开一个终端然后参照下面的命令依次输入：
~~~bash
$ cd ~/Downloads/ #这里的“/Downloads/”是指下载 K2pdfopt 所在的路径
$ sudo mv k2pdfopt /usr/bin #这里的路径可以按照你的喜好设置
$ chmod +x /usr/bin/k2pdfopt #将 k2pdfopt 变成可执行文件
$ cd /my/pdf/folder #定位到 PDF 文档所在目录
$ k2pdfopt myfile.pdf #开始转换 PDF 文档
~~~
### 二、 转换
{% asset_img k2pdfopt_help.png k2pdfopt_help %}

### 三、 传输

> 注意：当选择邮箱推送时,邮件主题不要写成"Convert"否则邮件服务器会再进行一次格式转换，此时的格式很可能严重影响阅读。 
