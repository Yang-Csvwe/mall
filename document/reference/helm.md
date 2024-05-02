# Helm常用指令

## Helm 仓库常用命令
### 添加仓库
- helm repo add bitnami https://charts.bitnami.com/bitnami
- helm repo add harbor https://helm.goharbor.io
### 查看仓库
- helm repo list
### 删除仓库
- helm repo remove bitnami
### 更新仓库
- helm repo update

## Helm chart常用命令
### 查找chart
- helm search hub harbor
- helm search repo harbor/harbor
- helm search repo harbor/harbor --versions
### 拉取chart
- helm pull harbor/harbor
- helm pull harbor/harbor --version 1.14.1
- helm pull harbor/harbor --version 1.14.1 --untar
### 安装chart
- helm install harbor/harbor --version 1.14.1 --namespace harbor --create-namespace \
  --set registry.internal.hostname=registry.harbor.svc.cluster.local
### 查看部署的清单
- helm template ingress-nginx ingress-nginx/ingress-nginx --namespace ingress-nginx --create-namespace --version 4.3.0
### 卸载chart
- helm uninstall harbor --namespace harbor
### 查看chart
- helm show all harbor/harbor --version 1.14.1

## Helm 其他
### 下载chart的values.yaml
- helm show values harbor/harbor --version 1.14.1 > values-harbor-harbor-1.14.1.yaml
