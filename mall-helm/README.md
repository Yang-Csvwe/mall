## Idea配置kubernetes插件
### kubectl.exe下载
+ 网址：https://kubernetes.io/docs/tasks/tools/install-kubectl-windows/#install-kubectl-binary-with-curl-on-windows

### helm.exe下载
+ 网址：https://github.com/helm/helm/tags

## helm
+ helm添加chart仓库
```shell
helm repo add aliyun https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts
helm repo add aliyun bitnami	https://charts.bitnami.com/bitnami                    
helm repo add aliyun stable 	http://mirror.azure.cn/kubernetes/charts
```

## k8s集群版本升级
+ master节点
```shell
yum install kubeadm-1.22.17-0 kubelet-1.22.17-0 kubectl-1.22.17-0 --disableexcludes=kubernetes

kubectl drain k8s-master --ignore-daemonsets
kubeadm upgrade plan

kubeadm upgrade apply 1.22.17

systemctl daemon-reload
systemctl restart kubelet
kubectl uncordon k8s-master
```
+ node节点
```shell
yum install kubeadm-1.22.17-0 kubelet-1.22.17-0 kubectl-1.22.17-0 --disableexcludes=kubernetes
kubeadm upgrade node
systemctl daemon-reload
systemctl restart kubelet
```

+ 验证
```shell
yum list --showduplicates kubeadm --disableexcludes=kubernetes
kubectl get nodes
kubectl logs -f kube-controller-manager-k8s-master -n kube-system
```

## 安装nfs
### nfs服务端
+ 安装nfs服务端
```shell
yum install rpcbind nfs-utils -y
```
+ 前提手动创建的共享目录：/shared_folder，配置共享目录以及客户端权限
```shell
vim /etc/exports
/storage_nfs *(rw,sync,insecure,no_subtree_check,no_root_squash)
```
+ 加载配置
```shell
exportfs -a
```
+ 重启服务
```shell
systemctl restart rpcbind && systemctl enable rpcbind 
systemctl restart nfs-server && systemctl enable nfs-server
```
### nfs客户端
+ 安装nfs客户端
```shell
yum install nfs-utils
```
+ 前提手动创建的共享目录：/shared_folder，配置挂载目录
```shell
vim /etc/fstab
## 添加如下内容
k8s-master:/storage_nfs /storage_nfs nfs defaults,_netdev 0 0
```
+ 配置挂载目录
```shell
mount k8s-master:/storage_nfs /storage_nfs
```




## k8s集群配置
### StorageClass
```shell
helm repo add nfs-subdir-external-provisioner https://kubernetes-sigs.github.io/nfs-subdir-external-provisioner/
helm install nfs-subdir-external-provisioner nfs-subdir-external-provisioner/nfs-subdir-external-provisioner --set nfs.server=k8s-master --set nfs.path=/nfs
```
+ 验证，生成storageclass：nfs-client，pvc的storageclass为nfs-client
```shell
[root@k8s-master /]# kubectl get storageclass
NAME         PROVISIONER                                     RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
nfs-client   cluster.local/nfs-subdir-external-provisioner   Delete          Immediate           true                   2m43s
```
