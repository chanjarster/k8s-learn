# istio安装指南

本文写的时候版本`v1.0.0`

## 导入image

先找一个能够翻墙的、安装了Docker的机器，然后运行以下命令把相关image拉下来：

```
docker pull docker.io/jaegertracing/all-in-one:1.5
docker pull docker.io/prom/prometheus:v2.3.1
docker pull docker.io/prom/statsd-exporter:v0.6.0
docker pull gcr.io/istio-release/citadel:1.0.0
docker pull gcr.io/istio-release/galley:1.0.0
docker pull gcr.io/istio-release/grafana:1.0.0
docker pull gcr.io/istio-release/mixer:1.0.0
docker pull gcr.io/istio-release/pilot:1.0.0
docker pull gcr.io/istio-release/proxyv2:1.0.0
docker pull gcr.io/istio-release/servicegraph:1.0.0
docker pull gcr.io/istio-release/sidecar_injector:1.0.0
docker pull quay.io/coreos/hyperkube:v1.7.6_coreos.0
```

然后运行以下命令把这些image打成tar包：

```
docker save \
  docker.io/jaegertracing/all-in-one:1.5 \
  docker.io/prom/prometheus:v2.3.1 \
  docker.io/prom/statsd-exporter:v0.6.0 \
  gcr.io/istio-release/citadel:1.0.0 \
  gcr.io/istio-release/galley:1.0.0 \
  gcr.io/istio-release/grafana:1.0.0 \
  gcr.io/istio-release/mixer:1.0.0 \
  gcr.io/istio-release/pilot:1.0.0 \
  gcr.io/istio-release/proxyv2:1.0.0 \
  gcr.io/istio-release/servicegraph:1.0.0 \
  gcr.io/istio-release/sidecar_injector:1.0.0 \
  quay.io/coreos/hyperkube:v1.7.6_coreos.0 \
  | gzip -c > istio-images-v1.0.0.tar.gz
```

把这个tar上传到每个node上，然后执行：

```
docker load --input istio-images-v1.0.0.tar.gz
```

## 安装

1. 根据[Installing the sidecar][istio-sidecar-injection]提到，需要给kube-apiserver的`--admission-control`添加`MutatingAdmissionWebhook`和`ValidatingAdmissionWebhook`
   1. 如果你用的是Rancher，那么可以参考[这个方法](../../../installation-guide/rancher2.0/admission-control.md)
1. [下载istio][download-istio]
1. 然后根据[官方文档Installation with Helm][istio-helm-install]安装
   * 先[安装Helm](../../helm/README.md)
   * 然后根据文档说明安装Tiller
   * 注意文档开头要安装CRDs，里面提到一句话：
     > If you are enabling `certmanager`, you also need to install its CRDs...
     
     certmanager是否启用看安装参数，见`certmanager.enabled`[安装参数][istio-install-options]。这个参数默认是false的。
1. 选择`Option 2`的方式，安装命令，更多自定义参数见[安装参数][istio-install-options]：
   ```
   helm install install/kubernetes/helm/istio --name istio --namespace istio-system \
     --set gateways.istio-ingressgateway.type=NodePort \
     --set gateways.istio-egressgateway.type=NodePort \
     --set tracing.enabled=true \
     --set servicegraph.enabled=true \
     --set grafana.enabled=true
   ```
   上面的安装命令我们启用了链路调用tracing、servicegraph和grafana。并且把istio的ingressgateway/egressgateway设置成NodePort模式（默认是LoadBalancer）。
1. 为tracing、prometheus、servicegraph、grafana配置Ingress：
   1. 修改目录下的`istio-ingresses.yaml`，把`<your host name>`替换成你自己的值
   1. 执行`kubectl apply -f istio-ingresses.yaml`

删除istio的方法：

* `helm delete --purge istio`
* `kubectl delete -f install/kubernetes/helm/istio/templates/crds.yaml -n istio-system`

## 备注

因为istio的版本是在变化中的，其所需要image可能会发生变化，那么怎么知道要pull哪些image呢？

办法就是先按照前面讲的办法安装，然后运行这个命令获得container的image清单:

```
printf '%s\n' \
$(kubectl -n istio-system get pods -o jsonpath='{range .items[*]}{@.spec.containers[*].image}{" "}{@.spec.initContainers[*].image}{" "}') \
| sort | uniq
```

然后根据获得的结果导入image。

## 参考文档

* [Istio - Installation with Helm][istio-helm-install]
* [Istio - Installation Options][istio-install-options]
* [Istio - Verify installed Istio plugins][istio-plugins]

[istio-helm-install]: https://istio.io/docs/setup/kubernetes/helm-install/
[download-istio]: https://istio.io/docs/setup/kubernetes/download-release/
[istio-sidecar-injection]: https://istio.io/docs/setup/kubernetes/sidecar-injection/
[istio-install-helm-tiller]: https://istio.io/docs/setup/kubernetes/helm-install/#option-2-install-with-helm-and-tiller-via-helm-install
[istio-install-options]: https://istio.io/docs/reference/config/installation-options/
[istio-plugins]: https://istio.io/docs/setup/kubernetes/quick-start-gke-dm/#verify-installed-istio-plugins
