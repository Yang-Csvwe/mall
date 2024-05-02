# nfs-subdir-external-provisioner

## 介绍
- 前提
  - 搭建好kubernetes集群，集群中各节点并安装了nfs-utils
  - 搭建好NFS服务器，并共享一个目录给kubernetes集群使用
- nfs-subdir-external-provisioner是一个用于kubernetes的NFS外部provisioner，它允许用户将NFS服务器挂载到kubernetes集群中，并使用它来创建PersistentVolume（PV）和PersistentVolumeClaim（PVC）
- nfs-subdir-external-provisioner通过在NFS服务器上创建子目录来支持多个PVC同时使用同一个NFS路径
- nfs-subdir-external-provisioner还支持动态卷创建和删除
- 当PVC被删除时，nfs-subdir-external-provisioner会自动删除NFS服务器上的子目录
- 当PVC被创建时，nfs-subdir-external-provisioner会自动创建NFS服务器上的子目录

## 安装nfs-subdir-external-provisioner
- 添加nfs-subdir-external-provisioner镜像仓库
```shell
helm repo add nfs-subdir-external-provisioner https://kubernetes-sigs.github.io/nfs-subdir-external-provisioner/
```
- 安装nfs-subdir-external-provisioner release
```shell
# --set nfs.server：NFS服务器地址
# --set nfs.path：NFS服务器共享目录
helm install nfs-subdir-external-provisioner nfs-subdir-external-provisioner/nfs-subdir-external-provisioner \
  --version 4.0.18 \
  --set nfs.server=k8s-master \
  --set nfs.path=/nfs \
  --namespace nfs-subdir-external-provisioner \
  --create-namespace
```
- 卸载nfs-subdir-external-provisioner release
```shell
helm unintall nfs-subdir-external-provisioner --namespace nfs-subdir-external-provisioner
```
