# 删除Namespace一直Terminating的问题

K8S版本: 1.10.5

之前在安装了istio之后想删除，最后删除namespace`istio-system`的时候发现这个namespace一直处于`Terminating`的状态。后来发现是因为没有删除istio安装的CRDs造成的，下面是两个相关的issue：

* [kubectl delete should wait for resource to be deleted before returning][issue-42594] TL;DR 
* [deleting namespace stuck at "Terminating" state][issue-60807] TL;DR 


[issue-42594]: https://github.com/kubernetes/kubernetes/issues/42594
[issue-60807]: https://github.com/kubernetes/kubernetes/issues/60807