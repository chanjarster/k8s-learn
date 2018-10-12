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
