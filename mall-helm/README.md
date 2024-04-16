

## Idea配置kubernetes插件

+ 根据k8s集群版本，下载指定版本的插件

| Idea插件    | 网址                                                         | 版本     |
| ----------- | ------------------------------------------------------------ | -------- |
| kubectl.exe | https://kubernetes.io/docs/tasks/tools/install-kubectl-windows/#install-kubectl-binary-with-curl-on-windows | v1.22.17 |
| helm.exe    | https://github.com/helm/helm/tags                            | v3.9.4   |

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



## 应用安装

+ Helm仓库

| Helme仓库别名（自己定义）       | Helm仓库地址                                                 |
| ------------------------------- | ------------------------------------------------------------ |
| aliyun                          | https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts       |
| bitnami                         | https://charts.bitnami.com/bitnami                           |
| stable                          | http://mirror.azure.cn/kubernetes/charts                     |
| nfs-subdir-external-provisioner | https://kubernetes-sigs.github.io/nfs-subdir-external-provisioner/ |
| ingress-nginx                   | https://kubernetes.github.io/ingress-nginx                   |

+ 应用安装（选用版本）

| 应用                            | Helm仓库别名                    | CHART  | 指令                                                         |
| :------------------------------ | ------------------------------- | ------ | ------------------------------------------------------------ |
| ingress-nginx                   | ingress-nginx                   | 4.4.2  | 创建控制器：ingress-nginx                                    |
| nfs-subdir-external-provisioner | nfs-subdir-external-provisioner | 4.0.18 | --set nfs.server=k8s-master 指定nfs服务器地址<br/>--set nfs.path=/nfs 指定nfs服务器共享目录<br/><br/>pv实现动态创建，StorageClass：nfs-client |
| mysql                           | stable                          | 1.6.9  | /                                                            |
| redis                           | stable                          | 10.5.7 | /                                                            |
|                                 |                                 |        |                                                              |
|                                 |                                 |        |                                                              |
|                                 |                                 |        |                                                              |
|                                 |                                 |        |                                                              |
|                                 |                                 |        |                                                              |





