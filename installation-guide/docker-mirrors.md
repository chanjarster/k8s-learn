# Docker镜像清单

## Docker hub镜像

Docker hub有时候会抽风，可以在每个kubelet服务器上配置镜像，有以下选项：

1. [Daocloud][daocloud]，具体用法见网站。
1. [中科大][ustc-docker]，具体用法见网站。

## Linux软件源

### Ubuntu APT

**Docker CE**

使用[阿里云](mirrors.aliyun.com)的，用法见[文档](https://yq.aliyun.com/articles/110806)。

**Docker Project**

使用清华的：

* `https://mirrors.tuna.tsinghua.edu.cn/docker/apt/gpg`
* `https://mirrors.tuna.tsinghua.edu.cn/docker/apt/repo`

用法参考阿里云的[使用apt-get进行安装](https://yq.aliyun.com/articles/110806#4)，注意把阿里云的url替换成清华的url就可以了。

[daocloud]: https://www.daocloud.io/mirror#accelerator-doc
[anjia0532]: https://github.com/anjia0532/gcr.io_mirror
[ustc-docker]: http://mirrors.ustc.edu.cn/help/dockerhub.html
