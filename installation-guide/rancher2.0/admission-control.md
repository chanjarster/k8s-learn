# 配置admission-control

Rancher版本：v2.0.7

我们在安装某些组件的时候会需要开放一些[Admission Control][k8s-admission-control]。下面介绍步骤：

1. 登录到一个Control Plane Node，运行`ps -ef | grep admission-control`查看当前开放的admission control有哪些，把当前值记录下来：
   ```
   root     31853 31835  8 08:05 ?        00:00:54 kube-apiserver
   ...
   --admission-control=ServiceAccount,NamespaceLifecycle,LimitRanger,PersistentVolumeLabel,DefaultStorageClass,ResourceQuota,DefaultTolerationSeconds
   ...
   ```
1. Cordon所有Node
1. 登录Rancher，修改Cluster，Edit as YAML，在下面位置添加这么一段：
   ```yaml
   nodes:
      ...
   
   services:
     kube-api:
       extra_args:
         admission-control:<之前记录下的值>,<你要添加的东西>
   
   addons: ...
   ```
1. 到每个Control Plane Node上，运行`ps -ef | grep admission-control`检查配置是否真的生效了。
1. Uncordon所有Node

## 参考资料

* [RKE - Kubernetes Default Services][rke-services]
* [RKE - Extra Args, Extra Binds, and Extra Environment Variables][rke-services-extras]

[rancher-install]: install.md
[k8s-admission-control]: https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/
[rke-services]: https://rancher.com/docs/rke/v0.1.x/en/config-options/services/
[rke-services-extras]: https://rancher.com/docs/rke/v0.1.x/en/config-options/services/services-extras/
