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

然后根据[官方文档][official-install]的方法下载istio，然后跑到`install/kubernetes`目录下，执行`kubectl apply -f .`。这个命令有可能第一次会失败，不要紧再执行一遍就行了。


## 备注

因为istio的版本是在变化中的，其所需要image可能会发生变化，那么怎么知道要pull哪些image呢？

办法就是先按照前面讲的办法安装，然后运行这个命令获得container的image清单:

```
printf '%s\n' \
$(kubectl -n istio-system get pods -o jsonpath='{range .items[*]}{@.spec.containers[*].image}{" "}{@.spec.initContainers[*].image}{" "}') \
| sort | uniq
```

然后根据获得的结果导入image。

[official-install]: https://istio.io/docs/setup/kubernetes/download-release/