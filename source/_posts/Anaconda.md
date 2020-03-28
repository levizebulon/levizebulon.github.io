---
title: Anaconda
date: 2019-01-09 09:55:46
tags:
---
# Anaconda
[Anaconda下载](https://www.anaconda.com/download/#linux)
### conda 安装

### conda源
[清华源](https://mirror.tuna.tsinghua.edu.cn/help/anaconda/)

### 如果conda安装错误
~~~bash
conda install /path/package_name.tar.bz2
~~~
例如安装pytorch时出现错误：
~~~bash
Downloading and Extracting Packages
pytorch-1.0.0        | 437.5 MB  | ###################1                  |  52% 

CondaHTTPError: HTTP 000 CONNECTION FAILED for url <https://conda.anaconda.org/pytorch/linux-64/pytorch-1.0.0-py3.6_cuda8.0.61_cudnn7.1.2_1.tar.bz2>
Elapsed: -

An HTTP error occurred when trying to retrieve this URL.
HTTP errors are often intermittent, and a simple retry will get you on your way.
~~~
可以尝试用ariac2下载安装,下载地址已经在错误信息中
~~~bash
aria2c -c -x 16 -d ~/Downloads https://conda.anaconda.org/pytorch/linux-64/pytorch-1.0.0-py3.6_cuda8.0.61_cudnn7.1.2_1.tar.bz2
~~~
下载完成后
~~~bash
conda install ~/anaconda3/pkgs/pytorch-1.0.0-py3.6_cuda8.0.61_cudnn7.1.2_1.tar.bz2
~~~
