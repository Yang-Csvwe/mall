# k8s集群搭建
源博客地址：[CentOS 7.9基于kubeadm部署Kubernetes v1.28.2（Cri-Dockerd）](https://www.znetcore.com/2024/02/26/centos-7-9%E5%9F%BA%E4%BA%8Ekubeadm%E9%83%A8%E7%BD%B2kubernetes-v1-28-2%EF%BC%88cri-dockerd%EF%BC%89/)

## 配置主机名
**_根据规划定义主机名_**
```shell
# k8s-master
[root@k8s-master ~]# hostnamectl set-hostname k8s-master

# k8s-node1
[root@k8s-node1 ~]# hostnamectl set-hostname k8s-node1

# k8s-node2
[root@k8s-node2 ~]# hostnamectl set-hostname k8s-node2
```

## 配置hosts
**_以下操作所有节点需要执行_**
```shell
[root@k8s-master ~]# cat /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6

192.168.11.130 k8s-master
192.168.11.131 k8s-node1
192.168.11.132 k8s-node2
```

## 关闭防火墙
**_以下操作所有节点需要执行_**
```shell
[root@k8s-master ~]# systemctl stop firewalld && systemctl disable firewalld
```

## 关闭selinux
**_以下操作所有节点需要执行_**
```shell
[root@k8s-master ~]# sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config
[root@k8s-master ~]# setenforce 0
```

## 关闭swap分区
**_以下操作所有节点需要执行_**
```shell
[root@k8s-master ~]# swapoff -a
[root@k8s-master ~]# sed -i '/swap/s/^/#/g' /etc/fstab
```

## 修改内核参数
**_以下操作所有节点需要执行_**
```shell
[root@k8s-master ~]# cat >/etc/sysctl.d/k8s.conf <<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF
```
```shell
[root@k8s-master ~]# sysctl --system
```

## 安装ipset、ipvsadm
**_以下操作所有节点需要执行_**
```shell
[root@k8s-master ~]# yum install ipset ipvsadmin -y
```
```shell
[root@k8s-master ~]# cat >/etc/modules-load.d/ipvs.conf <<EOF
# Load IPVS at boot
ip_vs
ip_vs_rr
ip_vs_wrr
ip_vs_sh
nf_conntrack
nf_conntrack_ipv4
EOF
```
```shell
[root@k8s-master ~]# systemctl enable systemd-modules-load.service --now 
```
**确认内核模块加载成功**
```shell
[root@k8s-master ~]# lsmod |egrep "ip_vs|nf_conntrack_ipv4"
nf_conntrack_ipv4      15053  0 
nf_defrag_ipv4         12729  1 nf_conntrack_ipv4
ip_vs_sh               12688  0 
ip_vs_wrr              12697  0 
ip_vs_rr               12600  0 
ip_vs                 141432  6 ip_vs_rr,ip_vs_sh,ip_vs_wrr
nf_conntrack          133053  2 ip_vs,nf_conntrack_ipv4
libcrc32c              12644  3 xfs,ip_vs,nf_conntrack
```

## 安装docker
**_以下操作所有节点需要执行_**
```shell
[root@k8s-master ~]# yum install yum-utils device-mapper-persisent-data lvm2 -y
```
```shell
[root@k8s-master ~]# yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```
```shell
[root@k8s-master ~]# yum install docker-ce-20.10.23 docker-ce-cli-20.10.23 -y 
```
```shell
[root@k8s-master ~]# mkdir -p /etc/docker
[root@k8s-master ~]# cat >/etc/docker/daemon.json <<EOF
{
    "registry-mirrors": [
    "https://hub-mirror.c.163.com",
    "https://cr.console.aliyun.com",
    "https://reg-mirror.qiniu.com",
    "https://zwi8oxzp.mirror.aliyuncs.com"
    ],
    "exec-opts": ["native.cgroupdriver=systemd"]
}
EOF
[root@k8s-master ~]# systemctl enable docker --now
```

## 安装cri-dockerd
**_以下操作所有节点需要执行_**
```shell
[root@k8s-master ~]# yum -y install wget
[root@k8s-master ~]# wget -c https://github.com/Mirantis/cri-dockerd/releases/download/v0.3.1/cri-dockerd-0.3.1-3.el7.x86_64.rpm
[root@k8s-master ~]# yum -y install cri-dockerd-0.3.1-3.el7.x86_64.rpm
[root@k8s-master ~]# sed -i '/ExecStart/s#dockerd#& --network-plugin=cni --pod-infra-container-image=registry.aliyuncs.com/google_containers/pause:3.9#' /usr/lib/systemd/system/cri-docker.service
[root@k8s-master ~]# systemctl enable cri-docker --now
```

## 安装kubectl、kubeadm、kubelet
**_以下操作所有节点需要执行_**
```shell
[root@k8s-master ~]# cat >/etc/yum.repos.d/kubernetes.repo <<EOF
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
```
```shell
[root@k8s-master ~]# yum --showduplicates list kubelet |grep 1.28
kubelet.x86_64     1.28.0-0     kubernetes
kubelet.x86_64     1.28.1-0     kubernetes
kubelet.x86_64     1.28.2-0     kubernetes
```
```shell
[root@k8s-master ~]# yum -y install kubectl-1.28.2 kubelet-1.28.2 kubeadm-1.28.2
```
```shell
[root@k8s-master ~]# systemctl enable kubelet --now
```

## 初始化master节点
**_以下操作master节点执行_**
```shell
[root@k8s-master ~]# kubeadm init --kubernetes-version=1.28.2 \
    --apiserver-advertise-address=192.168.11.130 \
    --image-repository registry.aliyuncs.com/google_containers \
    --service-cidr=172.15.0.0/16 --pod-network-cidr=172.16.0.0/16 \
    --cri-socket unix:///var/run/cri-dockerd.sock
```
**此处根据页面提示**
```shell
[root@k8s-master ~]# mkdir -p $HOME/.kube
[root@k8s-master ~]# cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
[root@k8s-master ~]# chown $(id -u):$(id -g) $HOME/.kube/config
```

## 安装calico
**_以下操作master节点执行_**
```shell
[root@k8s-master ~]# kubectl apply -f https://docs.tigera.io/archive/v3.24/manifests/calico.yaml
```

## 添加worker节点
**_以下操作worker节点执行，此处根据页面提示_**
```shell
[root@k8s-master ~]# kubeadm join 192.168.11.130:6443 --token kyq8uq.51j5jmotl5qewl83 \
    --discovery-token-ca-cert-hash sha256:ca6b67365e3a4f57ff784687a06aaa9541dd2adbdaa5888869ed5b739439d39e \
    --cri-socket unix:///var/run/cri-dockerd.sock
```

## kubectl命令补全功能
**_以下操作master节点执行_**
```shell
[root@k8s-master ~]# yum -y install bash-completion
[root@k8s-master ~]# echo "source <(kubectl completion bash)" >> /etc/profile
[root@k8s-master ~]# source /etc/profile
```
