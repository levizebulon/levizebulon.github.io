---
title: CentOS搭建K8s集群
date: 2020-07-30 11:15:54
tags:
- distribute system
---

# 配置说明

- 3台虚拟机
~~~
Architecture:          x86_64
CPU op-mode(s):        32-bit, 64-bit
Byte Order:            Little Endian
CPU(s):                4
On-line CPU(s) list:   0-3
Thread(s) per core:    1
Core(s) per socket:    1
Socket(s):             4
NUMA node(s):          1
Vendor ID:             GenuineIntel
CPU family:            6
Model:                 85
Model name:            Intel Xeon Processor (Skylake, IBRS)
Stepping:              4
CPU MHz:               2194.842
BogoMIPS:              4389.68
Hypervisor vendor:     KVM
Virtualization type:   full
L1d cache:             32K
L1i cache:             32K
L2 cache:              4096K
L3 cache:              16384K
NUMA node0 CPU(s):     0-3
Flags:                 fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush mmx fxsr sse sse
2 ss syscall nx pdpe1gb rdtscp lm constant_tsc rep_good nopl xtopology eagerfpu pni pclmulqdq ssse3 fma cx16 pcid sse4_
1 sse4_2 x2apic movbe popcnt tsc_deadline_timer aes xsave avx f16c rdrand hypervisor lahf_lm abm 3dnowprefetch fsgsbase
 tsc_adjust bmi1 hle avx2 smep bmi2 erms invpcid rtm mpx avx512f avx512dq rdseed adx smap clflushopt clwb avx512cd avx5
12bw avx512vl xsaveopt xsavec xgetbv1 ibpb ibrs stibp arat pku ospke spec_ctrl intel_stibp
~~~

- CentOS 7.5

安装后的软件版本为

- Kubernetes v1.16.0 
  - calico 3.8.2
- Docker 18.09.7

网络结构为
- Master
  - master-192-168-21-188
- Worker
  - worker-192-168-21-224
  - worker-192-168-21-127

采用 kubernetes.io 官方推荐的 kubeadm 工具安装 kubernetes 集群

# 2. 检查CentOS hostname

~~~bash
# 在 master 节点和 worker 节点都要执行
cat /etc/redhat-release

# 此处 hostname 的输出将会是该机器在 Kubernetes 集群中的节点名字
# 不能使用 localhost 作为节点的名字
hostname

# 请使用 lscpu 命令，核对 CPU 信息
# Architecture: x86_64    本安装文档不支持 arm 架构
# CPU(s):       2         CPU 内核数量不能低于 2
lscpu
~~~

修改hostname，可执行如下命令：

~~~bash
# 修改 hostname
hostnamectl set-hostname your-new-host-name
# 查看修改结果
hostnamectl status
# 设置 hostname 解析
echo "127.0.0.1   $(hostname)" >> /etc/hosts
~~~

请确保：

- 我的任意节点 centos 版本在兼容列表中
- 我的任意节点 hostname 不是 localhost
- 我的任意节点 CPU 内核数量大于等于 2

# 3. 安装 docker / kubelet

使用 root 身份在所有节点执行如下代码，以安装软件：

- docker
- nfs-utils
- kubectl / kubeadm / kubelet

~~~
#!/bin/bash

# 在 master 节点和 worker 节点都要执行

# 安装 docker
# 参考文档如下
# https://docs.docker.com/install/linux/docker-ce/centos/ 
# https://docs.docker.com/install/linux/linux-postinstall/
# https://mirrors.tuna.tsinghua.edu.cn/help/docker-ce/

# 卸载旧版本
yum remove -y docker \
docker-client \
docker-client-latest \
docker-common \
docker-latest \
docker-latest-logrotate \
docker-logrotate \
docker-selinux \
docker-engine-selinux \
docker-engine

# 安装一些依赖
yum install -y yum-utils device-mapper-persistent-data lvm2

# 设置 yum repository
yum install -y yum-utils \
device-mapper-persistent-data \
lvm2
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

# 安装并启动 docker
yum makecache fast
yum install -y docker-ce-18.09.7 docker-ce-cli-18.09.7 containerd.io
systemctl enable docker
systemctl start docker

# 安装 nfs-utils
# 必须先安装 nfs-utils 才能挂载 nfs 网络存储
yum install -y nfs-utils

# 关闭 防火墙
systemctl stop firewalld
systemctl disable firewalld

# 关闭 SeLinux
setenforce 0
sed -i "s/SELINUX=enforcing/SELINUX=disabled/g" /etc/selinux/config

# 关闭 swap
swapoff -a
yes | cp /etc/fstab /etc/fstab_bak
cat /etc/fstab_bak |grep -v swap > /etc/fstab

# 修改 /etc/sysctl.conf
# 如果有配置，则修改
sed -i "s#^net.ipv4.ip_forward.*#net.ipv4.ip_forward=1#g"  /etc/sysctl.conf
sed -i "s#^net.bridge.bridge-nf-call-ip6tables.*#net.bridge.bridge-nf-call-ip6tables=1#g"  /etc/sysctl.conf
sed -i "s#^net.bridge.bridge-nf-call-iptables.*#net.bridge.bridge-nf-call-iptables=1#g"  /etc/sysctl.conf
# 可能没有，追加
echo "net.ipv4.ip_forward = 1" >> /etc/sysctl.conf

echo "net.bridge.bridge-nf-call-ip6tables = 1" >> /etc/sysctl.conf
echo "net.bridge.bridge-nf-call-iptables = 1" >> /etc/sysctl.conf
# 执行命令以应用
sysctl -p

# 配置K8S的yum源
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=http://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=http://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg
       http://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF

# 卸载旧版本
yum remove -y kubelet kubeadm kubectl

# 安装kubelet、kubeadm、kubectl
yum install -y kubelet-1.16.0 kubeadm-1.16.0 kubectl-1.16.0

# 修改docker Cgroup Driver为systemd
# # 将/usr/lib/systemd/system/docker.service文件中的这一行 ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
# # 修改为 ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock --exec-opt native.cgroupdriver=systemd
# 如果不修改，在添加 worker 节点时可能会碰到如下错误
# [WARNING IsDockerSystemdCheck]: detected "cgroupfs" as the Docker cgroup driver. The recommended driver is "systemd".

# Please follow the guide at https://kubernetes.io/docs/setup/cri/
sed -i "s#^ExecStart=/usr/bin/dockerd.*#ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock --exec-opt native.cgroupdriver=systemd#g" /usr/lib/systemd/system/docker.service

# 设置 docker 镜像，提高 docker 镜像下载速度和稳定性
# 如果您访问 https://hub.docker.io 速度非常稳定，亦可以跳过这个步骤
curl -sSL https://get.daocloud.io/daotools/set_mirror.sh | sh -s http://f1361db2.m.daocloud.io

# 重启 docker，并启动 kubelet
systemctl daemon-reload
systemctl restart docker
systemctl enable kubelet && systemctl start kubelet

docker version

echo -e "\033[31;1mDocker and kubelet has installed.\033[0m"
~~~

# 4. 初始化 master 节点

- 以 root 身份在 master-192-168-21-188 机器上执行
- 初始化 master 节点时，如果因为中间某些步骤的配置出错，想要重新初始化 master 节点，请先执行 kubeadm reset 操作
- POD_SUBNET 所使用的网段不能与 master节点/worker节点 所在的网段重叠。该字段的取值为一个 CIDR 值，如果您对 CIDR 这个概念还不熟悉，请不要修改这个字段的取值 10.100.0.1/20

~~~
# 只在 master 节点执行
# 替换 x.x.x.x 为 master 节点实际 IP（请使用内网 IP）
# export 命令只在当前 shell 会话中有效，开启新的 shell 窗口后，如果要继续安装过程，请重新执行此处的 export 命令
export MASTER_IP=192.168.21.188
# 替换 apiserver.demo 为 您想要的 dnsName (不建议使用 master 的 hostname 作为 APISERVER_NAME)
export APISERVER_NAME=apiserver.demo
# Kubernetes 容器组所在的网段，该网段安装完成后，由 kubernetes 创建，事先并不存在于您的物理网络中
export POD_SUBNET=10.100.0.1/20
echo "${MASTER_IP}    ${APISERVER_NAME}" >> /etc/hosts

# 查看完整配置选项 https://godoc.org/k8s.io/kubernetes/cmd/kubeadm/app/apis/kubeadm/v1beta2
rm -f ./kubeadm-config.yaml
cat <<EOF > ./kubeadm-config.yaml
apiVersion: kubeadm.k8s.io/v1beta2
kind: ClusterConfiguration
kubernetesVersion: v1.16.0
imageRepository: registry.cn-hangzhou.aliyuncs.com/google_containers
controlPlaneEndpoint: "${APISERVER_NAME}:6443"
networking:
  serviceSubnet: "10.96.0.0/16"
  podSubnet: "${POD_SUBNET}"
  dnsDomain: "cluster.local"
EOF

# kubeadm init
# 根据您服务器网速的情况，您需要等候 3 - 10 分钟
kubeadm init --config=kubeadm-config.yaml --upload-certs

# 配置 kubectl
rm -rf /root/.kube/
mkdir /root/.kube/
cp -i /etc/kubernetes/admin.conf /root/.kube/config

# 安装 calico 网络插件
# 参考文档 https://docs.projectcalico.org/v3.8/getting-started/kubernetes/
rm -f calico.yaml
wget https://docs.projectcalico.org/v3.8/manifests/calico.yaml
sed -i "s#192\.168\.0\.0/16#${POD_SUBNET}#" calico.yaml
kubectl apply -f calico.yaml

echo -e "\033[31;1mmaster has initialized\033[0m"
~~~

检查 master 初始化结果

~~~
# 只在 master 节点执行

# 执行如下命令，等待 3-10 分钟，直到所有的容器组处于 Running 状态
watch kubectl get pod -n kube-system -o wide

# 查看 master 节点初始化结果
kubectl get nodes -o wide
~~~

# 5. 初始化 worker节点

## 获得Join命令参数

在master节点上执行

~~~
# 只在 master 节点执行
kubeadm token create --print-join-command
~~~

可获取kubeadm join 命令及参数，如下所示

~~~
# kubeadm token create 命令的输出
kubeadm join apiserver.demo:6443 --token ij9run.pkdknewa5w6uqeck     --discovery-token-ca-cert-hash sha256:1483024cd4c42424497214c3a16ba94e3f6f07e458503bb65673dead208194d9
~~~

将上面的结果保存到文件`worker-join-command.sh`，然后通过`scp`命令将`worker-join-command.sh`发送到各个worker。

~~~
scp worker-join-command.sh root@192.168.21.xx:~/ 
~~~

## 初始化worker

针对所有的 worker 节点执行

~~~
# 只在 worker 节点执行
MASTER_IP=192.168.21.188
APISERVER_NAME=apiserver.demo
echo "${MASTER_IP}    ${APISERVER_NAME}" >> /etc/hosts

# get from Master. kubeadm token create --print-join-command
bash worker-join-command.sh
~~~

## 检查初始化结果

在 master 节点上执行

~~~
# 只在 master 节点执行
kubectl get nodes -o wide
~~~

输出结果如下所示：

~~~
NAME                    STATUS   ROLES    AGE    VERSION
master-192-168-21-188   Ready    master   170m   v1.16.0
worker-192-168-21-127   Ready    <none>   153m   v1.16.0
worker-192-168-21-224   Ready    <none>   153m   v1.16.0
~~~

## 移除 worker 节点

正常情况下，您无需移除 worker 节点，如果添加到集群出错，您可以移除 worker 节点，再重新尝试添加

### 在准备移除的 worker 节点上执行

~~~
# 只在 worker 节点执行
kubeadm reset
~~~

### 在 master 节点上执行

~~~
# 只在 master 节点执行
kubectl delete node worker-x-x
~~~

- 将 worker-x-x 替换为要移除的 worker 节点的名字
- worker 节点的名字可以通过在master节点上执行 kubectl get nodes 命令获得

# 6. 安装Kuboard

[安装 Kuboard稳定版](https://kuboard.cn/install/install-dashboard.html#%E5%AE%89%E8%A3%85kuboard)

~~~
kubectl apply -f https://kuboard.cn/install-script/kuboard.yaml
kubectl apply -f https://addons.kuboard.cn/metrics-server/0.3.7/metrics-server.yaml
~~~

查看 Kuboard 运行状态：

~~~
kubectl get pods -l k8s.kuboard.cn/name=kuboard -n kube-system
~~~

输出结果如下所示：

~~~
NAME                       READY   STATUS        RESTARTS   AGE
kuboard-54c9c4f6cb-6lf88   1/1     Running       0          45s
~~~
