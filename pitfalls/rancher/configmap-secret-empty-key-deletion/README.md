# ConfigMap/Secret自动删除空值key

Rancher版本: 2.0.6

通过Rancher web ui配置ConfigMap/Secret/Environment Variable，如果你有一个key是空值的话，Rancher是会把你的这个key删除的，这个就很坑了。

这个问题已经提交：https://github.com/rancher/rancher/issues/15010