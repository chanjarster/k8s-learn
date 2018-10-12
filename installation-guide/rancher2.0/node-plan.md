# 节点规划

在生产环境下的规划。

## rancher集群

* 管理节点：>=3个（单数），负责etcd、controlplane，**绝对不能**担任worker职能。
* 工作节点：根据情况配备，只担任worker职能。

每个节点的硬件配置见[这里](https://rancher.com/docs/rancher/v2.x/en/installation/requirements/)。

## 工作集群

* 管理节点：>=3个（单数），这3个节点只负责etcd、controlplane，**绝对不能**担任worker职能
* 工作节点：根据情况配备，只担任worker职能。

管理节点的硬件配置不低于[Rancher的节点配置要求](https://rancher.com/docs/rancher/v2.x/en/installation/requirements/)。

工作节点的硬件配置根据情况来。

## 注意事项

* 一定要将rancher集群和工作集群分开，如果不分开，很容易因为工作负载过重导致rancher歇菜。
* 一定要将集群内部的管理节点和工作节点分开，如果不分开，那么很容易因为工作负载过重导致整个k8s集群歇菜，而且rancher也管不了、kubectl也管不了。
