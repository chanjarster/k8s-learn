# 如何重启Deployment

有时候我们会需要重启Deployment，原因可能是：

1. docker image使用的是latest tag，这个latest在docker image registry已经更新了，我们需要重启deployment来使用新的latest
2. Pod运行缓慢但是还活着，我们就是想重启一下

上面两种情况的共同之处在于，Deployment spec没有发生任何变化，因此即使你`kubectl appply -f deployment-spec.yaml`也是没用的，因为K8S会认为你这个没有变化就什么都不做了。

但是我们又不想使用手工删除Pod-让K8S新建Pod的方式来重启Deployment，最好的办法应该是像[Updating a deployment][updating-a-deployment]一样，让K8S自己滚动的删除-新建Pod。

有人对此给了一个[workaround][workaround]：

```
kubectl patch deployment <deployment-name> \
  -p '{"spec":{"template":{"spec":{"containers":[{"name":"<container-name>","env":[{"name":"RESTART_","value":"'$(date +%s)'"}]}]}}}}'
```

基本思路就是给Container添加一个无关紧要的环境变量，这个环境变量的值就是时间戳，而这个时间戳则是每次执行上述命令的系统当前时间。这样一来对于K8S来讲这个Deployment spec就变化了，就可以像[Updating a deployment][updating-a-deployment]一样，重启Pod了。

[workaround]: https://github.com/kubernetes/kubernetes/issues/13488#issuecomment-356892053
[updating-a-deployment]: https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#updating-a-deployment