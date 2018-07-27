# k8s docker image加速仓库

## gcr.io/google-containers

使用[中科大的镜像][ustc-gcr]

```bash
docker pull image gcr.io/google-containers/xxx:yyy
或
docker pull image gcr.io/google_containers/xxx:yyy
=>
docker pull image gcr.mirrors.ustc.edu.cn/google-containers/xxx:yyy
```

也可以使用[anjia0532的搬运仓库][anjia0532]

## k8s.gcr.io

k8s.gcr.io等价于gcr.io/google-containers，因此同上可以使用中科大镜像或者anjia0532的搬运仓库。

使用[中科大的镜像][ustc-gcr]

```bash
docker pull image k8s.gcr.io/xxx:yyy
=>
docker pull image gcr.mirrors.ustc.edu.cn/google-containers/xxx:yyy
```

使用[anjia0532的搬运仓库][anjia0532]

## quay.io

使用[中科大的镜像][ustc-quay]

```bash
docker pull image quay.io/xxx/yyy:zzz
=>
docker pull image quay.mirrors.ustc.edu.cn/xxx/yyy:zzz
```

[daocloud]: https://www.daocloud.io/mirror#accelerator-doc
[anjia0532]: https://github.com/anjia0532/gcr.io_mirror
[ustc-gcr]: https://github.com/ustclug/mirrorrequest/issues/187
[ustc-quay]: https://github.com/ustclug/mirrorrequest/issues/135
