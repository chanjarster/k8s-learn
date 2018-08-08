# k8s-learn

**注意：在中国安装使用Docker之前，请务必使用[Docker镜像清单](installation-guide/docker-mirrors.md)**

## 集群安装指南

* [使用Kubespray安装](installation-guide/kubespray/README.md)
* [使用Rancher 2.0安装](installation-guide/rancher2.0/README.md)(无DNS)

## 组件安装指南

* [Ingress](addons-guide/ingress)
* [Dashboard配置Ingress](addons-guide/dashboard)
* [Elasticsearch, Fluentd, Kibana](addons-guide/efk)
* [Prometheus](addons-guide/prometheus)
* [linkred2(conduit)](addons-guide/linkerd2)

## Sample App

* [Echo Server](sample-apps/echo-server)
* [Emojivoto](sample-apps/emojivoto)(linkerd2 demo app)

## 坑

* [Http Redirect端口丢失问题](pitfalls/http-302)

## 镜像清单

* [Docker镜像清单](installation-guide/docker-mirrors.md)
* [K8S Image仓库镜像](installation-guide/k8s-image-repo-mirrors.md)

