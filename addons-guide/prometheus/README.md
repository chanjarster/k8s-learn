# Prometheus安装指南

本文根据[kube-prometheus官方文档][kube-prometheus]来安装Prometheus，但是做了一些修改。

1. 到[prometheus github][prometheus-github]下载代码
1. 把`master/contrib/kube-prometheus/manifests`copy出来，取个名字叫做`prometheus-install`
1. 根据[镜像/加速仓库清单][mirrors.md]的方法替换仓库地址
1. 根据[grafana behind proxy][grafana-behind-proxy]的说明，修改`grafana-deployment.yaml`，添加`GF_SERVER_ROOT_URL`的环境变量，这个关系到后面Ingress是否能够正常反向代理：
   ```yaml
   ...
   containers:
   - image: grafana/grafana:5.1.0
     name: grafana
     ports:
     - containerPort: 3000
       name: http
     resources:
       limits:
         cpu: 200m
         memory: 200Mi
       requests:
         cpu: 100m
         memory: 100Mi
     env:
       - name: GF_SERVER_ROOT_URL
         value: http://127.0.0.1:3000/grafana
   ...
   ```
1. 安装：`kubectl apply -f prometheus-install`
1. 用Dashboard登录，切换到monitoring namespace，到Stateful Sets，找到prometheus-k8s stateful set，编辑，在containers.args下添加`--web.external-url`参数，比如下面这样：
   ```json
   ...
   "containers": [
       {
         "name": "prometheus",
         "image": "quay.mirrors.ustc.edu.cn/prometheus/prometheus:v2.2.1",
         "args": [
           "--config.file=/etc/prometheus/config_out/prometheus.env.yaml",
           "--storage.tsdb.path=/prometheus",
           "--storage.tsdb.retention=24h",
           "--web.enable-lifecycle",
           "--storage.tsdb.no-lockfile",
           "--web.route-prefix=/",
           "--web.external-url=http://<any-node-ip>:<ingress-http-port>/prometheus-k8s"
         ],
         ...
       }
    ...
   ```
   注意设置正确的`<any-node-ip>`和`<ingress-http-port>`。
   添加这个参数的目的是为了后面Ingress能走通

**配置Ingress**

```
kubectl apply -f alertmanager-main-ingress.yamlkubectl apply -f grafana-ingress.yamlkubectl apply -f prometheus-k8s-ingress.yaml
```

然后就可以通过以下地址访问相关组件了：

* `http://<any-node-ip>:<ingress-http-port>/grafana`，默认用户名密码：admin/admin
* `http://<any-node-ip>:<ingress-http-port>/alertmanager-main`
* `http://<any-node-ip>:<ingress-http-port>/prometheus-k8s`

不过以上方式是将这些组件公开的暴露的，如果要添加认证控制，请参考[Exposing Prometheus, Alertmanager and Grafana UIs via Ingress][exposing-prometheus]，这篇文档的大致意思是通过http basic auth来做认证。

[prometheus-github]: https://github.com/coreos/prometheus-operator
[kube-prometheus]: https://github.com/coreos/prometheus-operator/tree/master/contrib/kube-prometheus
[mirrors.md]: ../../installation-guide/mirrors.md
[exposing-prometheus]: https://github.com/coreos/prometheus-operator/blob/master/contrib/kube-prometheus/docs/exposing-prometheus-alertmanager-grafana-ingress.md
[grafana-behind-proxy]: http://docs.grafana.org/installation/behind_proxy/