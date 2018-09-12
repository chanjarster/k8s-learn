# 如何关掉Node

虽然我们可以直接将Node关机，让K8S自己将这个Node上的Pod在别的Node上重新创建，但是这样可能会对K8S集群造成冲击。

我们设想中的顺序应该是这样的：

1. 禁止有新的Pod安排到Node上
1. 把Node上的Pod迁移到别处
1. 把Node关掉

我们可以利用[`kubectl drain`][kubectl-drain]来做到前面的两点，大致方式如下：

1. `kubectl get nodes`找到Node
1. `kubectl drain <node-name>`
1. 然后关机就行了

当这个Node后来又启动了，那么只需要`kubectl uncordon <node name>`。


[kubectl-drain]: https://kubernetes.io/docs/tasks/administer-cluster/safely-drain-node/#use-kubectl-drain-to-remove-a-node-from-service