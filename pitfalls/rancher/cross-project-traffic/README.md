# 无法访问跨Project Service的问题

2018-08-14: 从v2.0.7开始，默认关掉了Project的NetworkPolicy，此问题得到解决，详见[Rancher Release - v2.0.7][rancher-2.0.7]。

Rancher版本: 2.0.6

先说结论：Rancher会自动给Project里的Namespace添加NetworkPolicy，只允许相同Project下的Namespace里的网络流量互通，跨Project则不行。实际上这个问题是很坑的，特别当你要用到service mesh比如linkerd、istio的时候，因为网络不通导致无法运作。 

下面开始实验来证明：

## 不挂在Project下的Namespace

创建两个namespace：nginx-1和nginx-2，并且里面都跑了一个nginx和busybox。

```bash
kubectl apply -f https://raw.githubusercontent.com/chanjarster/k8s-learn/master/pitfalls/rancher/cross-project-traffic/nginx-1.yaml
kubectl apply -f https://raw.githubusercontent.com/chanjarster/k8s-learn/master/pitfalls/rancher/cross-project-traffic/nginx-2.yaml
```

观察这两个namespace下是否有networkpolicy，结果是看不到：

```bash
kubectl get networkpolicy --all-namespaces
```

到namespace nginx-1里观察网络连通情况：

```bash
kubectl -n nginx-1 exec -it busybox-<podid> /bin/sh

# 访问同Namespace下的简称
wget -O - nginx-svc
# 访问同Namespace下的全称
wget -O - nginx-svc.nginx-1.svc.cluster.local
# 跨Namespace访问，结果是通的
wget -O - nginx-svc.nginx-2.svc.cluster.local
```

到namespace nginx-2里观察网络连通情况：

```bash
kubectl -n nginx-2 exec -it busybox-<podid> /bin/sh

# 访问同Namespace下的简称
wget -O - nginx-svc
# 访问同Namespace下的全称
wget -O - nginx-svc.nginx-2.svc.cluster.local
# 跨Namespace访问，结果是通的
wget -O - nginx-svc.nginx-1.svc.cluster.local
```

## 把Namespace挂到Project下

在Rancher创建nginx-1, nginx-2两个Project，把这两个namespace nginx-1和nginx-2分别移动到这两个Project里。

观察这两个namespace下是否有networkpolicy，发现有了：

```bash
kubectl get networkpolicy --all-namespaces

nginx-1         hn-nodes     <none>   28m
nginx-1         np-default   <none>   28m
nginx-2         hn-nodes     <none>   28m
nginx-2         np-default   <none>   28m
```

到namespace nginx-1里观察网络连通情况：

```bash
kubectl -n nginx-1 exec -it busybox-<podid> /bin/sh

# 跨Namespace访问，stuck住了
wget -O - nginx-svc.nginx-2.svc.cluster.local
```

到namespace nginx-2里观察网络连通情况：

```bash
kubectl -n nginx-2 exec -it busybox-<podid> /bin/sh

# 跨Namespace访问，stuck住了
wget -O - nginx-svc.nginx-1.svc.cluster.local
```

观察networkpolicy的内容：

```bash
$ kubectl -n nginx-1 get networkpolicy hn-nodes -o yaml
apiVersion: extensions/v1beta1
kind: NetworkPolicy
metadata:
  name: hn-nodes
  namespace: nginx-1
spec:
  ingress:
  - from:
    - ipBlock:
        cidr: 10.42.0.0/32
    - ipBlock:
        cidr: 10.42.1.0/32
    - ipBlock:
        cidr: 10.42.2.0/32
  podSelector: {}
  policyTypes:
  - Ingress

$ kubectl -n nginx-1 get networkpolicy np-default -o yaml

apiVersion: extensions/v1beta1
kind: NetworkPolicy
metadata:
  labels:
    field.cattle.io/projectId: p-ts6bj
  name: np-default
  namespace: nginx-1
spec:
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          field.cattle.io/projectId: p-ts6bj
  podSelector: {}
  policyTypes:
  - Ingress
```

## 解决办法

为此我在Rancher Forum发了一篇帖子，得到的答案是安装Cluster的时候选用Flannel插件（见[这里][forum]），怎么选用呢？见[RKE Network Plugi-ns][rke-network-plug-in]。

[forum]: https://forums.rancher.com/t/pods-cannot-communicate-across-projects/11415/4?u=chanjarster
[rke-network-plug-in]: https://rancher.com/docs/rke/v0.1.x/en/config-options/add-ons/network-plugins/

[rancher-2.0.7]: https://forums.rancher.com/t/rancher-release-v2-0-7/11435
