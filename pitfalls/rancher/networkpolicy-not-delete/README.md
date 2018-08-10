# NetworkPolicy不自动删除

Rancher版本: 2.0.6

在[无法访问跨Project Service的问题](../cross-project-traffic)里我们提到了如果把一个Namespace挂到Project下，Rancher会自动给Namespace添加NetworkPolicy。

不过Rancher还有一个坑，就是如果我们把这个Namespace移出Project（到None），那么这些NetworkPolicy不会被删除。下面是重现步骤：


1. 创建namespace: `kubectl create namespace np-test`
1. 打开Rancher web ui把namespace `np-test` 挂到任意Project
1. 发现`np-test`下创建了NetworkPolicy: `kubectl -n np-test get networkpolicy`
   ```
   NAME         POD-SELECTOR   AGE
   hn-nodes     <none>         45m
   np-default   <none>         45m
   ```
1. 打开Rancher web ui移动namespace `np-test` 到 **None**
1. NetworkPolicy依然存在: `kubectl -n np-test get networkpolicy`
```
NAME         POD-SELECTOR   AGE
hn-nodes     <none>         45m
np-default   <none>         45m
```

我已经把这个问题提交到Rancher：https://github.com/rancher/rancher/issues/15029