# Helm安装

本文写的时候版本`v2.9.1`

## 导入docker image

1. 找一个能够翻墙的、安装了Docker的机器，然后运行以下命令把相关image拉下来：
   ```
   docker pull gcr.io/kubernetes-helm/tiller:v2.9.1
   ```
1. 运行一下命令把tiller的image打成tar包：
   ```
   docker save gcr.io/kubernetes-helm/tiller:v2.9.1 \
     | gzip -c > tiller-v2.9.1.tar.gz
   ```
1. 把这个tar上传到每个node上，然后执行`docker load --input tiller-v2.9.1.tar.gz`

## 不导入docker image的方法

```
helm init --service-account tiller \
  --tiller-image registry.cn-hangzhou.aliyuncs.com/google_containers/tiller:<tag>
```

`<tag>`是helm的版本

## 安装

1. [下载helm][helm-binary]，解压缩，把路径添加到`PATH`环境变量
1. [安装Tiller][install-tiller]，运行`helm init`安装tiller


## 备注

因为helm的版本是在变化中的，其所需要image可能会发生变化，那么怎么知道要pull哪些image呢？

办法就是先按照前面讲的办法安装，然后运行这个命令获得container的image清单:

```
kubectl -n kube-system get deployments tiller-deploy -o jsonpath='{.spec.template.spec.containers[0].image}{"\n"}'
```

然后根据获得的结果导入image。

[helm-binary]: https://docs.helm.sh/using_helm/#from-the-binary-releases
[install-tiller]: https://docs.helm.sh/using_helm/#installing-tiller
