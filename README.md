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

* [Echo Server](sample-apps/echo-server)(Nginx Ingress版)
* [Echo Server](sample-apps/echo-server-istio)(Istio版)
* [Emojivoto](sample-apps/emojivoto)(linkerd2 demo app)

## 坑

**K8S使用**

* [Http Redirect端口丢失问题](pitfalls/http-302)
* [K8S - 删除Namespace一直Terminating的问题](pitfalls/k8s/namespace-deletion-stuck)

**Rancher**

* v2.0.6 [Rancher - 无法访问跨Project Service的问题](pitfalls/rancher/cross-project-traffic)(v2.0.7已修复)
* v2.0.6 [Rancher - ConfigMap/Secret自动删除空值key](pitfalls/rancher/configmap-secret-empty-key-deletion)(v2.0.7已修复)
* v2.0.6 [Rancher - NetworkPolicy不自动删除](pitfalls/rancher/networkpolicy-not-delete)

**Istio**

* v1.0 [从Ingress Nginx过来的流量不会被trace][istio-issue-7963]
* v1.0 [`x-forwarded-port`错误，导致SSL Termination情况下Redirect会被浏览器拒绝][istio-issue-7964]
* v1.0 [Redis流量无法被trace][istio-issue-5725]
* v1.0 [追加http request header时逻辑错误][istio-issue-8019]

## 镜像清单

* [Docker镜像清单](installation-guide/docker-mirrors.md)
* [K8S Image仓库镜像](installation-guide/k8s-image-repo-mirrors.md)


[istio-issue-7963]: https://github.com/istio/istio/issues/7963
[istio-issue-7964]: https://github.com/istio/istio/issues/7964
[istio-issue-5725]: https://github.com/istio/istio/issues/5725
[istio-issue-8019]: https://github.com/istio/istio/issues/8019