---
title: virtualenv
date: 2019-01-04 13:21:39
tags:
- software
---

## 介绍
[Virtualenv](https://virtualenv.pypa.io/en/latest/)是一个独立的python环境的工具。
如果我们在开发中需要用到不同版本的Python，那么就可以使用它来创建不同的python虚拟环境

## 安装

首先，我们通过pip安装virtualenv
~~~bash
pip install virtualenv	# 针对pip
pip3 install virtualenv	# 针对pip3
~~~

### 1. 创建目录
例如，我想重新装一个python2.7的环境
~~~bash
mkdir myproject
cd myproject
~~~

### 2. 创建一个独立的Python运行环境，命名为env:
~~~bash
virtualenv --no-site-packages venv	# 不带任何第三方包
virtualenv venv --python=python2.7	# 使用python 2.7 创建env环境
virtualenv venv --python=python3.6	# 使用python 3.6 创建环境
~~~

新建的Python环境被放到当前目录下的venv目录。有了venv这个Python环境，可以用source进入该环境：
~~~bash
source venv/bin/activate
~~~
这时候命令提示符会变化，有个(venv)前缀，表示当前环境是一个名为venv的Python环境。

下面正常安装各种第三方包，并运行python命令：
~~~bash
pip install package_name
~~~
在venv环境下，用pip安装的包都被安装到venv这个环境下，系统Python环境不受任何影响。也就是说，venv环境是专门针对myproject这个应用创建的。

退出当前的venv环境，使用deactivate命令：
~~~bash
deactivate 
~~~
此时就回到了正常的环境，现在pip或python均是在系统Python环境下执行。

完全可以针对每个应用创建独立的Python运行环境，这样就可以对每个应用的Python环境进行隔离。

virtualenv是如何创建“独立”的Python运行环境的呢？原理很简单，就是把系统Python复制一份到virtualenv的环境，用命令source venv/bin/activate进入一个virtualenv环境时，virtualenv会修改相关环境变量，让命令python和pip均指向当前的virtualenv环境。

## 删除环境
删除虚拟环境只需通过停用虚拟环境并删除环境文件夹及其所有内容即可完成：
~~~bash
(ENV)$ deactivate
$ rm -r /path/to/ENV
~~~

## 参考
廖雪峰的官方网站.[廖雪峰的官方网站](https://www.liaoxuefeng.com/wiki/0014316089557264a6b348958f449949df42a6d3a2e542c000/001432712108300322c61f256c74803b43bfd65c6f8d0d0000)

virtualenv[virtualenv](https://virtualenv.pypa.io/en/latest/)
