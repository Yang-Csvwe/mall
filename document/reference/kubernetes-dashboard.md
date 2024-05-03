# kubernetes-dashboard

## 安装kubernetes-dashboard
- 添加kubernetes-dashboard chart镜像仓库
```shell
[root@k8s-master ~]# helm repo add kubernetes-dashboard https://kubernetes.github.io/dashboard/
```
- 安装kubernetes-dashboard
```shell
[root@k8s-master ~]# helm upgrade --install kubernetes-dashboard kubernetes-dashboard/kubernetes-dashboard \
  --set service.type=NodePort \
  --set service.nodePort=30130 \
  --set metricsScraper.enabled=true \
  --namespace kubernetes-dashboard \
  --create-namespace \
  --version 6.0.8
```
- 添加权限，Kubernetes Dashboard 需要较大的权限才能够访问所有资源，所以我们可以创建一个 admin-user 的 ServiceAccount 来获取对应的 token
```yaml
# admin-user.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kubernetes-dashboard
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
  - kind: ServiceAccount
    name: admin-user
    namespace: kubernetes-dashboard
```
- 获取token
```shell
[root@k8s-master ~]# kubectl -n kubernetes-dashboard create token admin-user
```
- 卸载kubernetes-dashboard
```shell
helm uninstall kubernetes-dashboard --namespace kubernetes-dashboard
```

