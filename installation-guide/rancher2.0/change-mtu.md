## 修改MTU

好了，我们又遇到了坑爹的MTU问题了，见这个[issue 13984][issue-13984]，Rancher安装了Calico和Flannel两个CNI，这两个CNI的默认MTU是这样的：

* Calico: 1500
* Fannel: 自动计算，且不能修改，见[Flannel文档][flannel-mtu]

我们要先确定Calico的MTU设置成多少才合适，根据[Calico Configuring MTU][calico-mtu]，MTU的公式如下：

* 默认模式，MTU = 网卡MTU
* IP-in-IP模式，MTU = 网卡MTU - 20
* VXLAN模式，MTU = 网卡MTU - 50

那我们需要知道Calico到底采用了哪种模式，根据[Calico Configuring IP-in-IP][calico-ip-in-ip]，我们运行以下命令确认，发现Calico没有启用IP-in-IP模式：

```
$ kubectl get ipPool -o json
{
    "apiVersion": "v1",
    "items": [
        {
            "apiVersion": "crd.projectcalico.org/v1",
            "kind": "IPPool",
            ...
            "spec": {
                "cidr": "192.168.0.0/16",
                "ipipMode": "Never",
                "natOutgoing": true
            }
        }
    ],
    ...
}
```

我们根据[Configuring the Calico CNI plugins][calico-cni]，发现使用了VXLAN模式：

```
$ kubectl -n kube-system get configmap canal-config -o yaml
apiVersion: v1
data:
  canal_iface: ens3
  cni_network_config: |-
    {
      "name": "k8s-pod-network",
      "cniVersion": "0.3.0",
      "plugins": [
        {
          "type": "calico",
          "log_level": "info",
          "datastore_type": "kubernetes",
          "nodename": "__KUBERNETES_NODE_NAME__",
          "ipam": {
            "type": "host-local",
            "subnet": "usePodCidr"
          },
          ...
        },
        ...
      ]
    }
  masquerade: "true"
  net-conf.json: |-
    {
      "Network": "10.42.0.0/16",
      "Backend": {
        "Type": "vxlan"
      }
    }
kind: ConfigMap
metadata:
  ...
  name: canal-config
  namespace: kube-system
  ...
```

所以我们可以知道，Calico的MTU应该等于网卡MTU - 50。

接下来看怎么改这个MTU，K8S Network Plugin文档里说了可以传递`--network-plugin-mtu`来[设置MTU][k8s-mtu]，但这个方法只适用于`kubernet`，对于其他CNI是不起作用的。于是根据[K8S CNI文档][k8s-cni]，在每个Node的`/etc/cni/net.d`目录下，找到了`10-calico.conflist`这个文件。但我们不能直接改这个文件，因为有两个坑：

1. 这个文件是Rancher生成的，修改了可能会被改回去
1. 光改文件没有用，需要重启Calico服务才可以

为了解决第一个问题，我们要修改ConfigMap `canal-config`，在Rancher登录，跑到namespace `kube-system`，Resources -> Config Maps，找到`canal-config`，根据[Calico Configuring MTU - MTU configuration with CNI][calico-mtu-cni]文档里的方法修改`cni_network_config`，它的值就是生成`/etc/cni/net.d/10-calico.conflist`的模板，在`"type": "calico"`属性下面加上mtu参数，比如这样：

```yaml
{
    "name": "any_name",
    "cniVersion": "0.1.0",
    "type": "calico",
    "mtu": 1480,
    "ipam": {
        "type": "calico-ipam"
    }
}
```

不过这里又有一个坑，Rancher在安装的时候生成的ConfigMap `canal-config`有一个key叫做`canal_iface`是空值（见[RKE源码][rke-canal-template]），然而在Rancher里修改ConfigMap都会把空值key给干掉，导致后面一步的DaemonSet `canal`启动不了，报`Couldn't find key canal_iface in ConfigMap kube-system/canal-config`的错，所以我们要手动添加`canal_iface`这个key到ConfigMap `canal-config`里，值填物理网卡名称就好了（关于这个问题我已经提交[issue #15010][issue-15010]到Rancher，2018-08-14更新：此问题在2.0.7里已修复），如果你的Node的物理网卡名称不一样那就歇菜了。

解决第二个问题，我们得重启DaemonSet `canal`，重启方法很简单，只要删除它的Pod就可以了，K8S会自动运行新的Pod。

之后你就可以到Node上执行ifconfig观察`cali*`网卡的MTU，不过你会失望，因为这样的修改只对之后创建的Pod有效，对于已有Pod不发生作用，解决办法有两个：

1. 干掉这些Pod，让K8S的自动重启机制创建新Pod，不过这个办法不好，可能对Stateful Pod有删除数据的风险（这个我也没试过）。
1. `ifconfig <interface> mtu <size> up`到每个Node上把已经存在的`cali*`网卡的mtu都设置一遍，运行这个命令：
   ```
   ifconfig | grep cali | cut -d ' ' -f 1 | xargs -n1 -I{} sudo ifconfig {} mtu <size> up
   ```


[flannel-mtu]: https://github.com/coreos/flannel/blob/master/Documentation/configuration.md#key-command-line-options
[k8s-cni]: https://kubernetes.io/docs/concepts/extend-kubernetes/compute-storage-net/network-plugins/#cni
[k8s-mtu]: https://kubernetes.io/docs/concepts/extend-kubernetes/compute-storage-net/network-plugins/#customizing-the-mtu-with-kubenet
[calico-mtu]: https://docs.projectcalico.org/v2.2/usage/configuration/mtu
[calico-ip-in-ip]: https://docs.projectcalico.org/v2.2/usage/configuration/ip-in-ip
[calico-cni]: https://docs.projectcalico.org/v2.0/reference/cni-plugin/configuration
[calico-mtu-cni]: https://docs.projectcalico.org/v2.2/usage/configuration/mtu#mtu-configuration-with-cni
[rke-canal-template]: https://github.com/rancher/rke/blob/master/templates/canal.go
[issue-15010]: https://github.com/rancher/rancher/issues/15010
[issue-13984]: https://github.com/rancher/rancher/issues/13984
