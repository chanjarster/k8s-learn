# Etcd Operator安装指南

本文用的版本：v0.9.2

下载代码：https://github.com/coreos/etcd-operator

切换到最新的tag。

```bash
kubectl create ns etcd
NAMESPACE=etcd example/rbac/create_role.sh
kubectl -n etcd create -f example/deployment.yaml
```

参考文档：

[Etdc Operator - Installation guide](https://github.com/coreos/etcd-operator/blob/v0.9.2/doc/user/install_guide.md)