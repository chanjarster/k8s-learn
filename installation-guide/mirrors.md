# 镜像/加速仓库清单

一般来说K8S docker image或者Ubuntu APT源的地址都是写在各类安装脚本/工具的配置文件中的，这类配置文件一般来说是`*.yaml`或者`*.yml`，因此在执行这些脚本之前请先使用`find`命令替换掉这些仓库地址。

## Docker Image

### gcr.io/google-containers

使用[中科大的镜像][ustc-gcr]

```bash
docker pull image gcr.io/google-containers/xxx:yyy
或
docker pull image gcr.io/google_containers/xxx:yyy
=>
docker pull image gcr.mirrors.ustc.edu.cn/google-containers/xxx:yyy
```

脚本：

```bash
find . -name '*.yml' -or -name '*.yaml' | xargs -n1 -I{} sed -i  's/gcr\.io\/google-containers\//gcr\.mirrors\.ustc\.edu\.cn\/google-containers\//' {}`
find . -name '*.yml' -or -name '*.yaml' | xargs -n1 -I{} sed -i  's/gcr\.io\/google_containers\//gcr\.mirrors\.ustc\.edu\.cn\/google-containers\//' {}`
```

也可以使用[anjia0532的搬运仓库][anjia0532]

脚本：

```bash
find . -name '*.yml' -or -name '*.yaml' | xargs -n1 -I{} sed -i  's/gcr\.io\/google-containers\//anjia0532\/google-containers\./' {}
find . -name '*.yml' -or -name '*.yaml' | xargs -n1 -I{} sed -i  's/gcr\.io\/google_containers\//anjia0532\/google-containers\./' {}
```

### quay.io

使用[中科大的镜像][ustc-quay]

```bash
docker pull image quay.io/xxx/yyy:zzz
=>
docker pull image quay.mirrors.ustc.edu.cn/xxx/yyy:zzz
```

脚本：

```bash
find . -name '*.yml' -or -name '*.yaml' | xargs -n1 -I{} sed -i  's/quay\.io\//quay\.mirrors\.ustc\.edu\.cn\//' {}`
```

### Docker hub镜像

Docker hub有时候会抽风，可以在每个kubelet服务器上配置镜像，有以下选项：

1. [Daocloud][daocloud]，具体用法见网站。
1. [中科大][ustc-docker]，具体用法见网站。

## Linux软件源

某些工具比如Kubespray会利用Docker的软件源来安装K8S组件，但是Docker的软件源经常无法访问，所以要使用国内镜像。

### Ubuntu APT

#### docker ce ubuntu repo

```bash
https://download.docker.com/linux/ubuntu
=> http://mirrors.aliyun.com/docker-ce/linux/ubuntu 

https://download.docker.com/linux/ubuntu/gpg
=> http://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg 
```

脚本：

```bash
find . -name '*.yml' -or -name '*.yaml' | xargs -n1 -I{} sed -i  's/https:\/\/download\.docker\.com\/linux\/ubuntu/http:\/\/mirrors\.aliyun\.com\/docker-ce\/linux\/ubuntu/' {}`
find . -name '*.yml' -or -name '*.yaml' | xargs -n1 -I{} sed -i  's/https:\/\/download\.docker\.com\/linux\/ubuntu\/gpg/http:\/\/mirrors\.aliyun\.com\/docker-ce\/linux\/ubuntu\/gpg/' {}`
```
#### docker project apt repo

```bash
https://apt.dockerproject.org/repo
=> https://mirrors.tuna.tsinghua.edu.cn/docker/apt/repo

https://apt.dockerproject.org/gpg
=> https://mirrors.tuna.tsinghua.edu.cn/docker/apt/gpg
```

脚本：

```
find . -name '*.yml' -or -name '*.yaml' | xargs -n1 -I{} sed -i  's/https:\/\/apt\.dockerproject\.org\/repo/https:\/\/mirrors\.tuna\.tsinghua\.edu\.cn\/docker\/apt\/repo/' {}`
find . -name '*.yml' -or -name '*.yaml' | xargs -n1 -I{} sed -i  's/https:\/\/apt\.dockerproject\.org\/gpg/https:\/\/mirrors\.tuna\.tsinghua\.edu\.cn\/docker\/apt\/gpg/' {}`
```

[daocloud]: https://www.daocloud.io/mirror#accelerator-doc
[anjia0532]: https://github.com/anjia0532/gcr.io_mirror
[ustc-gcr]: https://github.com/ustclug/mirrorrequest/issues/187
[ustc-quay]: https://github.com/ustclug/mirrorrequest/issues/135
[ustc-docker]: http://mirrors.ustc.edu.cn/help/dockerhub.html
