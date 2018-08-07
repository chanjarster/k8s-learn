# linkerd2(conduit)安装指南

本文写的时候版本`v18.7.2`

## 导入image

先找一个能够翻墙的、安装了Docker的机器，然后运行以下命令把相关image拉下来：

```
docker pull gcr.io/linkerd-io/proxy:v18.7.2
docker pull gcr.io/linkerd-io/controller:v18.7.2
docker pull gcr.io/linkerd-io/grafana:v18.7.2
docker pull gcr.io/linkerd-io/proxy-init:v18.7.2
docker pull gcr.io/linkerd-io/web:v18.7.2
docker pull prom/prometheus:v2.3.1
```

然后运行以下命令把这些image打成tar包：

```
docker save \
  gcr.io/linkerd-io/proxy:v18.7.2 \
  gcr.io/linkerd-io/controller:v18.7.2 \
  gcr.io/linkerd-io/grafana:v18.7.2 \
  gcr.io/linkerd-io/proxy-init:v18.7.2 \
  prom/prometheus:v2.3.1 \
  gcr.io/linkerd-io/web:v18.7.2 \
  | gzip -c > linkerd2-images-v18.7.2.tar.gz
```

把这个tar上传到每个node上，然后执行：

```
docker load --input linkerd2-images-v18.7.2.tar.gz
```

然后根据[官方文档](https://conduit.io/)的方法安装即可。

## 备注

因为linkerd的版本是在变化中的，其所需要image可能会发生变化，那么怎么知道要pull哪些image呢？

办法就是先按照[官方文档](https://conduit.io/)的方法安装。然后运行这个命令获得container的image清单:

```
kubectl -n linkerd get pods -o jsonpath='{.items[*].spec.containers[*].image}'
```

然后根据获得的结果导入image。