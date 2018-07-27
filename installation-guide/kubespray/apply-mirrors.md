# 替换配置文件中的下载地址

根据[Docker镜像清单](../docker-mirrors.md)和[K8S image仓库镜像](../k8s-image-repo-mirrors.md)里列出的地址，替换掉Kubespray配置文件中的下载地址。

## Docker Image

### gcr.io/google-containers

如果要使用[中科大的镜像][ustc-gcr]，脚本：

```bash
find . -name '*.yml' -or -name '*.yaml' | xargs -n1 -I{} sed -i  's/gcr\.io\/google-containers\//gcr\.mirrors\.ustc\.edu\.cn\/google-containers\//' {}
find . -name '*.yml' -or -name '*.yaml' | xargs -n1 -I{} sed -i  's/gcr\.io\/google_containers\//gcr\.mirrors\.ustc\.edu\.cn\/google-containers\//' {}
```

如果要使用[anjia0532的搬运仓库][anjia0532]，脚本：

```bash
find . -name '*.yml' -or -name '*.yaml' | xargs -n1 -I{} sed -i  's/gcr\.io\/google-containers\//anjia0532\/google-containers\./' {}
find . -name '*.yml' -or -name '*.yaml' | xargs -n1 -I{} sed -i  's/gcr\.io\/google_containers\//anjia0532\/google-containers\./' {}
```

### k8s.gcr.io

如果要使用[中科大的镜像][ustc-gcr]，脚本：

```bash
find . -name '*.yml' -or -name '*.yaml' | xargs -n1 -I{} sed -i  's/k8s\.gcr\.io\//gcr\.mirrors\.ustc\.edu\.cn\/google-containers\//' {}
```

如果要使用[anjia0532的搬运仓库][anjia0532]，脚本：

```bash
find . -name '*.yml' -or -name '*.yaml' | xargs -n1 -I{} sed -i  's/k8s\.gcr\.io\//anjia0532\/google-containers\./' {}
```

### quay.io

如果只能使用[中科大的镜像][ustc-quay]，脚本：

```bash
find . -name '*.yml' -or -name '*.yaml' | xargs -n1 -I{} sed -i  's/quay\.io\//quay\.mirrors\.ustc\.edu\.cn\//' {}
```

## Linux软件源

### Ubuntu APT

#### docker ce ubuntu repo

使用aliyun的，脚本：

```bash
find . -name '*.yml' -or -name '*.yaml' | xargs -n1 -I{} sed -i  's/https:\/\/download\.docker\.com\/linux\/ubuntu/http:\/\/mirrors\.aliyun\.com\/docker-ce\/linux\/ubuntu/' {}`
find . -name '*.yml' -or -name '*.yaml' | xargs -n1 -I{} sed -i  's/https:\/\/download\.docker\.com\/linux\/ubuntu\/gpg/http:\/\/mirrors\.aliyun\.com\/docker-ce\/linux\/ubuntu\/gpg/' {}`
```

#### docker project apt repo

使用清华大学的，脚本：

```
find . -name '*.yml' -or -name '*.yaml' | xargs -n1 -I{} sed -i  's/https:\/\/apt\.dockerproject\.org\/repo/https:\/\/mirrors\.tuna\.tsinghua\.edu\.cn\/docker\/apt\/repo/' {}`
find . -name '*.yml' -or -name '*.yaml' | xargs -n1 -I{} sed -i  's/https:\/\/apt\.dockerproject\.org\/gpg/https:\/\/mirrors\.tuna\.tsinghua\.edu\.cn\/docker\/apt\/gpg/' {}`
```

[anjia0532]: https://github.com/anjia0532/gcr.io_mirror
[ustc-gcr]: https://github.com/ustclug/mirrorrequest/issues/187
[ustc-quay]: https://github.com/ustclug/mirrorrequest/issues/135
[ustc-docker]: http://mirrors.ustc.edu.cn/help/dockerhub.html
