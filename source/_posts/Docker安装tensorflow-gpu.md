---
title: Docker安装tensorflow-gpu
date: 2019-03-20 16:36:12
tags:
- Machine Learning
---
### 安装Docker
[Get docker-ce](https://docs.docker.com/install/)

[Tuna Docker-ce](https://mirror.tuna.tsinghua.edu.cn/help/docker-ce/)

如果你过去安装过 docker，先删掉:
~~~bash
sudo apt-get remove docker docker-engine docker.io
~~~
首先安装依赖:
~~~bash
sudo apt-get install apt-transport-https ca-certificates curl gnupg2 software-properties-common
~~~
根据你的发行版，下面的内容有所不同。这里用的是Ubuntu
信任 Docker 的 GPG 公钥:
~~~bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
~~~
对于 amd64 架构的计算机，添加软件仓库:
~~~bash
sudo add-apt-repository \
   "deb [arch=amd64] https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
~~~
最后安装
~~~bash
sudo apt-get update
sudo apt-get install docker-ce
~~~

### 安装nvidia-docker

#### GPU 支持
Docker 是在 GPU 上运行 TensorFlow 的最简单方法，因为主机只需安装[NVIDIA®](https://github.com/NVIDIA/nvidia-docker/wiki/Frequently-Asked-Questions#how-do-i-install-the-nvidia-driver) 驱动程序（无需安装 NVIDIA® CUDA® 工具包）。

安装 [nvidia-docker](https://github.com/NVIDIA/nvidia-docker) 可启动支持 NVIDIA® GPU 的 Docker 容器。nvidia-docker 仅适用于 Linux，详情请参阅对应的[平台支持常见问题解答](https://github.com/NVIDIA/nvidia-docker/wiki/Frequently-Asked-Questions#platform-support)了解详情。

检查 GPU 是否可用：
~~~bash
lspci | grep -i nvidia
~~~
验证 nvidia-docker 安装：
~~~bash
docker run --runtime=nvidia --rm nvidia/cuda nvidia-smi
~~~
注意：nvidia-docker v1 使用 ```nvidia-docker``` 别名，而 v2 使用 ```docker --runtime=nvidia```。

~~~bash
# If you have nvidia-docker 1.0 installed: we need to remove it and all existing GPU containers
docker volume ls -q -f driver=nvidia-docker | xargs -r -I{} -n1 docker ps -q -a -f volume={} | xargs -r docker rm -f
sudo apt-get purge -y nvidia-docker

# Add the package repositories
curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | \
  sudo apt-key add -
distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | \
  sudo tee /etc/apt/sources.list.d/nvidia-docker.list
sudo apt-get update

# Install nvidia-docker2 and reload the Docker daemon configuration
sudo apt-get install -y nvidia-docker2
sudo pkill -SIGHUP dockerd

# Test nvidia-smi with the latest official CUDA image
docker run --runtime=nvidia --rm nvidia/cuda:9.0-base nvidia-smi
~~~

### 安装测试Tensorflow

#### 使用支持 GPU 的映像的示例
下载并运行支持 GPU 的 TensorFlow 映像（可能需要几分钟的时间）：
~~~bash
docker run --runtime=nvidia -it --rm tensorflow/tensorflow:latest-gpu \
   python -c "import tensorflow as tf; tf.enable_eager_execution(); print(tf.reduce_sum(tf.random_normal([1000, 1000])))"
~~~
设置支持 GPU 的映像可能需要一段时间。如果重复运行基于 GPU 的脚本，您可以使用 ```docker exec```使用最新的 TensorFlow GPU 映像在容器中启动 ```bash``` shell 会话： 重用容器。
~~~bash
docker run --runtime=nvidia -it tensorflow/tensorflow:latest-gpu bash
~~~
#### [Get Started with TensorFlow](https://www.tensorflow.org/tutorials)
TensorFlow is an open-source machine learning library for research and production. TensorFlow offers APIs for beginners and experts to develop for desktop, mobile, web, and cloud. See the sections below to get started.

### 参考
