# EFK安装指南

K8S是将Pod中的stdout和stderr都输出到node机器上的某个目录下的，可以利用Elasticsearch、FLuentd、Kibana来收集和查询日志。

但是很遗憾[官方文档 - Logging Using Elasticsearch and Kibana][k8s-efk]所讲的部署方法似乎不试用与kubeadm之外的安装方法，所以本文在此做一个通用的安装方法。

1. 到[k8s github][k8s-github]下载最新代码
1. 到`cluster/addons/fluentd-elasticsearch`目录把`*.yaml`copy出来到一个目录，比如`efk-install`
1. 根据[镜像/加速仓库清单][mirrors.md]的方法替换仓库地址
1. 注意`fluentd-es-ds.yaml`里的`nodeSelector`，给每个node添加label：
   ```
   kubectl label node <node-name> beta.kubernetes.io/fluentd-ds-ready=true
   ```
1. 修改`kibana-deployment.yaml`，添加`SERVER_BASEPATH`环境变量：
   ```yaml
   env:
     -name: SERVER_BASEPATH
      value: /k8s-kibana
   ```
   这一步很重要，因为之后我们要用Ingress做反向代理的。
1. 安装EFK：`kubectl apply -f efk-install`

**配置Ingress**

使用本项目提供的配置文件：

```
kubectl apply -f es-ingress.yaml
kubectl apply -f kibana-ingress.yaml
```

然后你就可以通过一下地址访问elasticsearch和kibana了：

* `http://<any-node-ip>:<ingress-http-port>/k8s-kibana`
* `http://<any-node-ip>:<ingress-https-port>/k8s-es`
 
参考文档：[k8s addon/efk仓库][k8s-github-efk]。

[mirrors.md]: ../../installation-guide/mirrors.md
[k8s-efk]: https://kubernetes.io/docs/tasks/debug-application-cluster/logging-elasticsearch-kibana/
[k8s-github]: https://github.com/kubernetes/kubernetes
[k8s-github-efk]: https://github.com/kubernetes/kubernetes/tree/master/cluster/addons/fluentd-elasticsearch
