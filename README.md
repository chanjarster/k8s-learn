# k8s-learn

**注意：在中国安装使用Docker之前，请务必使用[Docker镜像清单](installation-guide/docker-mirrors.md)**

## 集群安装指南

* [使用Kubespray安装](installation-guide/kubespray/README.md)
* [使用Rancher 2.0安装](installation-guide/rancher2.0/README.md)(无DNS)

## 使用技巧

**K8S**

* [如何重启Deployment](skills/k8s/how-to-restart-deployment.md)
* [如何关掉Node](skills/k8s/how-to-shutdown-node.md)
* [如何创建指向外部资源的Service](https://kubernetes.io/docs/concepts/services-networking/service/#services-without-selectors)
  * [Kubernetes best practices: mapping external services](https://chinagdg.org/2018/05/kubernetes-best-practices-mapping-external-services/)
* [管理TLS证书](https://chanjarster.github.io/post/k8s-how-to-manage-tls-certs/)
* [PV中目录权限不符合要求的解决办法](https://cloud.ibm.com/docs/containers?topic=containers-cs_troubleshoot_storage#nonroot)

## 组件

**安装**

* [Ingress Nginx](addons-guide/ingress)
* [Dashboard配置Ingress](addons-guide/dashboard)
* [Elasticsearch, Fluentd, Kibana](addons-guide/efk)
* [Prometheus Operator](addons-guide/prometheus-operator)
* [linkred2(conduit)](addons-guide/linkerd2)
* [istio](addons-guide/istio/install)
* [helm](addons-guide/helm)
* Elastic Stack
  * [beats](https://github.com/elastic/beats/tree/master/deploy/kubernetes)
* [Etcd Operator](addons-guide/etcd-operator)

**Istio**：

* [完整配置文件](addons-guide/istio/learn/effective-traffic-routing.md)

## Sample App

* [Echo Server](sample-apps/echo-server)(Nginx Ingress版)
* [Echo Server](sample-apps/echo-server-istio)(Istio版)
* [Emojivoto](sample-apps/emojivoto)(linkerd2 demo app)

## 坑

**K8S使用**

* [Http Redirect端口丢失问题](pitfalls/http-302)
* [K8S - 删除Namespace一直Terminating的问题](pitfalls/k8s/namespace-deletion-stuck)
* NFS卷不控制容量上限，即你可以使用任意容量，所以也不支持扩容，见这个[issue](https://github.com/kubernetes-incubator/external-storage/issues/1235#issuecomment-542043620)

**Rancher**

* v2.0.6 ~~[Rancher - 无法访问跨Project Service的问题](pitfalls/rancher/cross-project-traffic)~~(v2.0.7已修复)
* v2.0.6 ~~[Rancher - ConfigMap/Secret自动删除空值key](pitfalls/rancher/configmap-secret-empty-key-deletion)~~(v2.0.7已修复)
* v2.0.6 [Rancher - NetworkPolicy不自动删除](pitfalls/rancher/networkpolicy-not-delete)

**Istio**

* v1.0 bug [从Ingress Nginx过来的流量不会被trace][istio-issue-7963]
* v1.0 [`x-forwarded-port`错误，导致SSL Termination情况下Redirect会被浏览器拒绝][istio-issue-7964]
* v1.0 [Redis流量无法被trace][istio-issue-5725]
* v1.0 ~~[追加http request header时逻辑错误][istio-issue-8019]~~

  这其实是bug中提到的echoserver的问题。Istio追加的header的行为是添加一个header，而不是追加值到原来的header里。
* v1.0 ~~[Jaeger无法记录多于2层的span][istio-issue-8155]~~

  还是程序的bug，应用代码要自己将trace相关的头再转发出去，详情见[文档](https://istio.io/docs/tasks/telemetry/distributed-tracing/#understanding-what-happened)
  

## 镜像清单

* [Docker镜像清单](installation-guide/docker-mirrors.md)
* [K8S Image仓库镜像](installation-guide/k8s-image-repo-mirrors.md)
* [Helm Chart仓库镜像](installation-guide/helm-mirrors.md)

[istio-issue-7963]: https://github.com/istio/istio/issues/7963
[istio-issue-7964]: https://github.com/istio/istio/issues/7964
[istio-issue-5725]: https://github.com/istio/istio/issues/5725
[istio-issue-8019]: https://github.com/istio/istio/issues/8019
[istio-issue-8155]: https://github.com/istio/istio/issues/8155
