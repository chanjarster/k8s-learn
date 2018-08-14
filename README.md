# k8s-learn

**注意：在中国安装使用Docker之前，请务必使用[Docker镜像清单](installation-guide/docker-mirrors.md)**

## 集群安装指南

* [使用Kubespray安装](installation-guide/kubespray/README.md)
* [使用Rancher 2.0安装](installation-guide/rancher2.0/README.md)(无DNS)

## 组件安装指南

* [Ingress Nginx](addons-guide/ingress)
* [Dashboard配置Ingress](addons-guide/dashboard)
* [Elasticsearch, Fluentd, Kibana](addons-guide/efk)
* [Prometheus](addons-guide/prometheus)
* [linkred2(conduit)](addons-guide/linkerd2)
* [istio](addons-guide/istio)
* [helm](addons-guide/helm)

## Sample App

* [Echo Server](sample-apps/echo-server)
* [Emojivoto](sample-apps/emojivoto)(linkerd2 demo app)

## 坑

* [Http Redirect端口丢失问题](pitfalls/http-302)
* [K8S - 删除Namespace一直Terminating的问题](pitfalls/k8s/namespace-deletion-stuck)
* [Rancher - 无法访问跨Project Service的问题](pitfalls/rancher/cross-project-traffic)(v2.0.7开始没有这个问题了)
* [Rancher - NetworkPolicy不自动删除](pitfalls/rancher/networkpolicy-not-delete)
* [Rancher - ConfigMap/Secret自动删除空值key](pitfalls/rancher/configmap-secret-empty-key-deletion)

## 镜像清单

* [Docker镜像清单](installation-guide/docker-mirrors.md)
* [K8S Image仓库镜像](installation-guide/k8s-image-repo-mirrors.md)

