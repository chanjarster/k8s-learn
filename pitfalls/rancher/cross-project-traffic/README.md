# Rancher - 无法访问跨Project Service的问题

先说结论：Rancher会自动给Project里的Namespace添加NetworkPolicy，只允许相同Project下的Namespace里的网络流量互通，跨Project则不行。下面开始实验来证明：

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

## 解决办法

怎么能够跨Project访问呢？抱歉目前我还不知道。
