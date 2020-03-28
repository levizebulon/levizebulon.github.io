---
title: ubuntu下安装TexLive
date: 2019-01-03 16:56:38
tags:
---
#### 下载安装包
如果身在国内，推荐改用国内的镜像，比如清华大学的[tuna](https://mirrors.tuna.tsinghua.edu.cn/)。以下都以这个镜像为例

在 [https://mirrors.tuna.tsinghua.edu.cn/CTAN/systems/texlive/tlnet/]( https://mirrors.tuna.tsinghua.edu.cn/CTAN/systems/texlive/tlnet/) 下载 install-tl-unx.tar.gz，解压并进入文件夹 install-tl-2018*。这里的 * 是一个日期。

首先安装 perl-tl 和 perl-doc
~~~bash
sudo apt-get install perl-tk perl-doc
~~~
#### 安装

##### 通过下载安装包安装

终端中的操作如下：
~~~bash
wget https://mirrors.tuna.tsinghua.edu.cn/CTAN/systems/texlive/tlnet/install-tl-unx.tar.gz
tar -xzf install-tl-unx.tar.gz
cd install-tl-2018*
sudo ./install-tl 	# 终端安装
sudo ./install-tl -gui # 通过图形界面安装
~~~

如果安装过程中发生错误
~~~bash
sudo ./install-tl --profile installation.profile
~~~
可以按照之前的配置重装

##### 通过镜像按装

这样可以得到较新的发行版版本，同时有丰富的包支持及更新。
ISO安装文件可以从下面几个开源镜像站点的CTAN同步镜像下载：
- 中科大开源镜像站 [(http://mirrors.ustc.edu.cn/CTAN/systems/texlive/Images/)](http://mirrors.ustc.edu.cn/CTAN/systems/texlive/Images/)
- 清华大学开源镜像站 [(http://mirrors.tuna.tsinghua.edu.cn/CTAN/systems/texlive/Images/)](http://mirrors.tuna.tsinghua.edu.cn/CTAN/systems/texlive/Images/)
- 厦门大学开源镜像站 [(http://mirrors.xmu.edu.cn/CTAN/systems/texlive/Images/)](http://mirrors.xmu.edu.cn/CTAN/systems/texlive/Images/)
###### 挂载镜像并安装
~~~bash
sudo mount /path_to_iso/texlive20*.iso /mnt
cd /mnt
sudo ./install-tl
~~~

#### 环境变量设置

此时 TeX Live 虽已安装，但其路径对于 Linux 来说仍是不可识别的。所以需要更改环境变量。
打开 ~/.bashrc，在最后添加
~~~bash
export PATH=/usr/local/texlive/2018/bin/x86_64-linux:$PATH
export MANPATH=/usr/local/texlive/2018/texmf-dist/doc/man:$MANPATH
export INFOPATH=/usr/local/texlive/2018/texmf-dist/doc/info:$INFOPATH
~~~
还需保证开启 sudo 模式后路径仍然可用。命令行中执行
~~~bash
sudo visudo
~~~
找到如下一段代码
~~~bash
Defaults        env_reset
Defaults        mail_badpass
Defaults        secure_path="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin"
~~~
将第三行改为
~~~bash
Defaults        secure_path="/usr/local/texlive/2018/bin/x86_64-linux:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin"
~~~
也就是加入 TeX Live 的执行路径。如果在安装时作了修改，这里的路径也都要与安装时的保持一致。

#### 字体设置
要在整个系统中使用 TeX 字体，还需要将 TeX 自带的配置文件复制到系统目录下。命令行中执行
~~~bash
sudo cp /usr/local/texlive/2018/texmf-var/fonts/conf/texlive-fontconfig.conf /etc/fonts/conf.d/09-texlive.conf
~~~
之后再执行
~~~bash
sudo fc-cache -fv
~~~
刷新字体数据库。

#### 检查
到此整个 TeX Live 2018 就已经安装完毕。可以做下面的一些检查：

##### 基本命令：
~~~bash
tlmgr --version
pdftex --version
xetex --version
luatex --version
~~~

#### 包管理器
~~~bash
sudo tlmgr update --list
~~~
这一步是检查更新，如果有就顺手更了吧：
~~~bash
sudo tlmgr update --self --all
~~~
--self 用来更新 tlmgr 自身，如果上一步没有提示可以不加这个选项。

##### Hello world
可以编译一个简短的测试文件：
~~~bash
% hello.tex
\documentclass[UTF8]{ctexart}
\begin{document}
欢迎来到 \TeX{} 世界！
\end{document}
~~~
用 xelatex 或 lualatex 编译：
~~~bash
xelatex hello
lualatex hello
~~~
编译得到的 PDF 文件如果显示正常，则说明 TeX Live 基本工作正常。

#### 使用Sublime Text 3编写LaTeX
[LaTeXTools: A LaTeX Plugin for Sublime Text 2 and 3](https://latextools.readthedocs.io/en/latest/)

[基于TeXlive,使用Sublime Text 3编写LaTeX](https://blog.csdn.net/qazxswed807/article/details/51234834)
##### 打开Perference->Package Settings->LaTexTools->Settings-User
在打开文件的227行”linux”中配置你的TexLive路径

- “texpath” : “$PATH:/usr/local/texlive/2018/bin/x86_64-linux”

- “python”: “/home/youname/anaconda3/bin”（Python的安装路径，如果你没有安装anaconda或者设置路径过你的路径可能不是这个你需要sudo find / -name python*）找到你的Python路径，设置487中 “viewer”: “evince”（不设置空着应该也可以，系统默认用自己的evince）。

##### 安装dbus（为了编译后自动打开pdf）

用conda[下载](https://anaconda.org/scottwales/dbus-python)
~~~bash
conda install -c scottwales dbus-python 
~~~

#### Latex文档
[一份(不太)简短的LATEX 2介绍](http://mirrors.ustc.edu.cn/CTAN/info/lshort/chinese/)

#### 参考
stone-zeng.[在 Ubuntu 中安装 TeX Live 2018](https://stone-zeng.github.io/fduthesis/2018-05-13-install-texlive-ubuntu/#1)

瞑夜-q.[基于TeXlive,使用Sublime Text 3编写LaTeX](https://blog.csdn.net/qazxswed807/article/details/51234834)

bleedingfight.[Ubuntu下用sublime text3和latextool插件写latex文档](https://blog.csdn.net/bleedingfight/article/details/72810606)

ptbsare.[Ubuntu下部署Latex编译环境](http://ptbsare.org/2014/05/12/ubuntu%E4%B8%8B%E9%83%A8%E7%BD%B2latex%E7%BC%96%E8%AF%91%E7%8E%AF%E5%A2%83/)
