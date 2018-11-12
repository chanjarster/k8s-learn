# Prometheus Operator安装指南

本文用的是版本：v0.25.0

## 前期准备

先有一个K8S集群，根据[Prerequisites][kube-prometheus-prerequisites]要求给`kubelet`设置两个启动参数：`--authentication-token-webhook=true`和`--authorization-mode=Webhook`。

**Rancher注意**

Rancher v2.1.1里默认开启了`--authentication-token-webhook=true`，但是无法开启`--authorization-mode=Webhook`（开启了cluster就起不来了），不过经过测试[Promtheus Operator][prometheus-operator]依然能够正常运行。

## 安装

本文安装的是[Promtheus Operator][prometheus-operator]的[kube-prometheus][kube-prometheus]来安装Prometheus，但是做了一些修改。

1. 到[Promtheus Operator][prometheus-operator]下载代码，切换到最新的tag
1. 到`contrib/kube-prometheus/`目录下，根据[Quickstart][kube-prometheus-quickstart]安装


## 通过Ingress暴露Prometheus、Alertmanager、Grafana


### 如果你没有办法创建A Record


1. 根据[grafana behind proxy][grafana-behind-proxy]的说明，到`monitoring` namespace下修改Deployment `grafana`，添加`GF_SERVER_ROOT_URL`的环境变量，这个关系到后面Ingress是否能够正常反向代理：
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
1. 到`monitoring` namespace下修改StatefulSets `prometheus-k8s`，在containers.args下添加`--web.external-url`参数，比如下面这样：
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
1. 使用本目录下的`no-dns-ingress.yaml`，创建Ingress

然后就可以通过以下地址访问相关组件了：

* `http://<any-node-ip>:<ingress-http-port>/grafana`，默认用户名密码：admin/admin
* `http://<any-node-ip>:<ingress-http-port>/alertmanager-main`
* `http://<any-node-ip>:<ingress-http-port>/prometheus-k8s`

### 如果你有办法创建A Record

1. 创建这样几个A Record：
   * `mon-alertmanager.<dns-name>`
   * `mon-grafana.<dns-name>`
   * `mon-prometheus.<dns-name>`
1. 使用本目录下的`dns-ingress.yaml`，修改里面的`<dns-name>`。

然后就可以通过以下地址访问相关组件了：

* `http://mon-alertmanager.<dns-name>`
* `http://mon-grafana.<dns-name>`，默认用户名密码：admin/admin
* `http://mon-prometheus.<dns-name>`
   
### 开启Http basic auth

Prometheus和Alertmanager本身都是无密码访问的，为了安全起见可以利用Ingress开启Basic Auth来做一层防护，详见这篇文档：[Ingress Basic Authentication][ingress-basic-auth]。

## 如何使用呢？

你可以参照[Getting Started][p8s-operator-getting-started]。阅读的时候可以忽略Prometheus Operatator安装的这一部分。

这篇文档大致思路是创建一个Prometheus对象和ServiceMonitor对象，这两个都是Prometheus Operator的CRDs。

如果你想使用Prometheus Operator自己的prometheus实例（在namespace monitoring里有pod叫做prometheus-k8s），那么跳过文档里的创建Prometheus对象的步骤，创建RBAC：

```yaml
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRole
metadata:
  name: prometheus-all-ns
rules:
- apiGroups: [""]
  resources:
  - nodes
  - services
  - endpoints
  - pods
  verbs: ["get", "list", "watch"]
- apiGroups: [""]
  resources:
  - configmaps
  verbs: ["get"]
- nonResourceURLs: ["/metrics"]
  verbs: ["get"]
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: prometheus-all-ns
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: prometheus-all-ns
subjects:
- kind: ServiceAccount
  name: prometheus-k8s
  namespace: monitoring
```

然后再创建ServiceMonitor对象就行了。

[prometheus-operator]: https://github.com/coreos/prometheus-operator
[kube-prometheus]: https://github.com/coreos/prometheus-operator/tree/master/contrib/kube-prometheus
[kube-prometheus-quickstart]: https://github.com/coreos/prometheus-operator/tree/master/contrib/kube-prometheus#quickstart
[kube-prometheus-prerequisites]: https://github.com/coreos/prometheus-operator/tree/master/contrib/kube-prometheus#prerequisites
[mirrors.md]: ../../installation-guide/mirrors.md
[exposing-prometheus]: https://github.com/coreos/prometheus-operator/blob/master/contrib/kube-prometheus/docs/exposing-prometheus-alertmanager-grafana-ingress.md
[grafana-behind-proxy]: http://docs.grafana.org/installation/behind_proxy/
[ingress-basic-auth]: https://kubernetes.github.io/ingress-nginx/examples/auth/basic/
[p8s-operator-getting-started]: https://github.com/coreos/prometheus-operator/blob/v0.25.0/Documentation/user-guides/getting-started.md
