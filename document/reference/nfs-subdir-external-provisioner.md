# nfs-subdir-external-provisioner

## 安装nfs服务器
- 安装nfs-utils、rpcbind
```shell
[root@k8s-master ~]# yum install rpcbind nfs-utils -y
```
```shell
# 在nfs服务端创建共享目录，设置文件夹权限
[root@k8s-master ~]# mkdir -p /data/nfs && chmod 777 -R /data/nfs
# 编辑NFS配置并加入以下内容，192.168.11.0为允许访问的网段
[root@k8s-master ~]# cat > /etc/exports <<EOF
/data/nfs 192.168.11.0/24(rw,sync,no_all_squash,no_subtree_check)
EOF
# 载入配置
[root@k8s-master ~]# exportfs -rv
# 启动并设置开机自启
[root@k8s-master ~]# systemctl enable rpcbind --now && systemctl enable nfs --now
```
## k8s各节点安装nfs-utils
```shell
[root@k8s-master /]# yum install nfs-utils -y
```
## 安装nfs-subdir-external-provisioner
- 添加nfs-subdir-external-provisioner镜像仓库
```shell
[root@k8s-master ~]# helm repo add nfs-subdir-external-provisioner https://kubernetes-sigs.github.io/nfs-subdir-external-provisioner/
```
- 安装nfs-subdir-external-provisioner release
```shell
# --set nfs.server：NFS服务器地址
# --set nfs.path：NFS服务器共享目录
[root@k8s-master ~]# helm install nfs-subdir-external-provisioner nfs-subdir-external-provisioner/nfs-subdir-external-provisioner \
  --version 4.0.18 \
  --set nfs.server=k8s-master \
  --set nfs.path=/data/nfs \
  --namespace nfs-subdir-external-provisioner \
  --create-namespace
```
- 卸载nfs-subdir-external-provisioner release
```shell
[root@k8s-master ~]# helm unintall nfs-subdir-external-provisioner --namespace nfs-subdir-external-provisioner
```
```shell
# 验证nfc-client安装是否成功，storageclass可以简写sc，kubectl get sc
[root@k8s-master ~]# kubectl get storageclass
```
