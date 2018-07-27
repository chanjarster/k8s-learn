# 使用Kubespray安装k8s集群

在使用Kubespray之前，请[应用镜像地址](apply-mirrors.md)

## 准备Kubespray

**请不要使用Kubespray的master分支，请使用最新tag来安装K8S集群**

1. 找一台机器，用来执行Kubespray，Kubespray的原理是通过ssh连接到各个target nodes执行命令安装k8s集群。
1. 在这个机器上安装[bash-git-prompt][bash-git-prompt]，到时候你就能知道自己在哪个分支上了
1. 在这个机器上安装pip
1. 到Kubespray[项目仓库][kubespray-repo]下载代码：
   `git clone https://github.com/kubernetes-incubator/kubespray.git`
1. checkout到最新tag
1. 根据[镜像/加速仓库清单][mirrors.md]中的方法替换仓库地址

## 准备K8S集群机器

1. 准备好几台服务器，假设你准备了3台。
1. 给每个node安装好操作系统，Ubuntu 16.04 Server LTS或者CentOS 7，并且有一个可以sudo的用户或者直接有root用户。
1. 确保每个node都安装了python 2.7
1. 在每个node上开启IPv4 Forwarding，修改`/etc/sysctl.conf`，然后重启。可参考[这篇文章][enable-ipv4-forwarding]
   ```
   net.ipv4.ip_forward = 1
   ```
1. 给每个node都安装ntpd，关于ntpd的说明见[这里][ntpd]
1. 关闭node上的防火墙（似乎不是必须的）
1. 将Kubespray机器的`.ssh/id_rsa.pub`上传到每个K8S集群机器上：
   `ssh-copy-id user@target-node-host`   

## 执行Kubespray 

1. 到Kubespray机器
1. 观察K8S集群机器网卡的MTU：`ifconfig`：
   ```
   ens3      Link encap:Ethernet  HWaddr fa:16:3e:67:b7:23
   inet addr:192.168.1.11  Bcast:192.168.1.255  Mask:255.255.255.0
   inet6 addr: fe80::f816:3eff:fe67:b723/64 Scope:Link
   UP BROADCAST RUNNING MULTICAST  MTU:1450  Metric:1
   ...
   ```
   这个例子中的MTU是1450，记住这个数字
1. 修改`roles/network_plugin/calico/defaults/main.yml`文件里的`calico_mtu`参数，根据[官方文档][calico-mtu]给每个服务器设置MTU。简单来说就是kubespray默认为calico启用了IP-in-IP模式，那么它的MTU应该是网卡MTU-20。
1. 根据项目仓库的指南执行命令，**但是不要执行最后的`ansible-playbook ...`**
1. 修改`inventory/mycluster/group_vars/k8s-cluster.yml`文件：
   * `kubeconfig_localhost: true`
   * `kubectl_localhost: true`
1. 执行`ansible-playbook ...`


### Troubleshooting

#### 提示Permission denied之类的错误

执行Ansible playbook的时候，ssh到target node执行某些命令缺少root权限。

在教程的最后一步`ansible-playbook -i inventory/mycluster/hosts.ini cluster.yml`，根据情况添加`-b --become-user --become-method`等参数。

写本文时target node是ubuntu cloud image，所以添加`-b -u ubuntu`。其他Linux请自行摸索。

#### 提示unable to resolve host

这是因为每个target node有一个hostname，但是在`/etc/hosts`下没有配置造成的：

1. 执行`hostname`获得hostname
1. 修改`/etc/hosts`：
   ```
   127.0.0.1 localhost <hostname>
   ```

#### 提示FAILED! ip in ansible\_all\_ipv4\_addresses

这种错误出现在云环境中，target node有两个IP，一个是内部IP（外部不能访问），一个是外部IP（在OpenStack环境下就是一个是Project network IP，一个是Floating IP）。

这个时候需要修改`inventory/mycluster/hosts.ini`，把node的IP属性改成内部IP，比如下面这种：

```
[all]
node1 ansible_host=172.50.10.2 ip=192.168.1.4
node2 ansible_host=172.50.10.13 ip=192.168.1.8
node3 ansible_host=172.50.10.15 ip=192.168.1.9
...
```

## 用kubectl访问

到这里，你应该已经安装完毕K8S了，然后你就可以在你自己的电脑上利用kubectl来管理K8S集群了：

1. [Install and Set Up kubectl][install-kubectl]
1. 到Kubespray机器找到`inventory/mycluster/artifacts/admin.conf`文件，copy到你自己电脑的`~/.kube/config`文件
1. 运行`kubectl -n kube-system get pod`来验证

注意：在OpenStack环境下，每个node会被分配一个Floating IP，会导致你`kubectl`无法使用，这个时候需要你这样做：

* 注释`.clusters.cluster.certificate-authority-data`
* 添加`.clusters.cluster.insecure-skip-tls-verify: true`
* 修改`.clusters.cluster.server`的IP为Floating IP

结果比如这样：

```
apiVersion: v1
clusters:
- cluster:
    # certificate-authority-data: <...>
    insecure-skip-tls-verify: true
    server: https://<master-floating-ip>:6443
  name: cluster.local
....
```

如果你不想每次使用`kubectl`都加上`-n`参数，或者你有多个k8s cluster，那么请参考[Configure Access to Multiple Clusters][kubectl-multi-cluster]。这样你要切换context的时候就只需要`kubectl config use-context <context-name>`。

## 访问Dashboard

如果都安装成功，那么你可以访问k8s dashboard来看看安装结果。打开浏览器，访问`https://{某个master的IP}:6443/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/`，但是你无法登录，你需要

1. 创建一个用户，分配权限
   ```bash
   kubectl -n kube-system apply -f admin-user.yaml
   kubectl -n kube-system apply -f admin-user-role.yaml
   ```
1. 获得token
   ```
   kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep admin-user | awk '{print $1}')
   ```

详情见[这篇guide][k8s-dashboard-create-user]。

然后你就可以使用token登录了，但是token很长，用起来很麻烦，你可以使用本项目提供的`config-4-dashboard-login`文件，修改里面的`<master-ip`和`<token>`为你项目的信息，然后使用Kubeconfig登录。


[mirrors.md]: ../mirrors.md
[install-kubectl]: https://kubernetes.io/docs/tasks/tools/install-kubectl/
[calico-mtu]: https://docs.projectcalico.org/v2.2/usage/configuration/mtu
[kubespray-repo]: https://github.com/kubernetes-incubator/kubespray
[bash-git-prompt]: https://github.com/magicmonty/bash-git-prompt
[enable-ipv4-forwarding]: http://www.ducea.com/2006/08/01/how-to-enable-ip-forwarding-in-linux/
[k8s-dashboard-create-user]: https://github.com/kubernetes/dashboard/wiki/Creating-sample-user
[anjia0532-mirror]: https://github.com/anjia0532/gcr.io_mirror
[ustc-mirror]: https://github.com/ustclug/mirrorrequest
[kubectl-multi-cluster]: https://kubernetes.io/docs/tasks/access-application-cluster/configure-access-multiple-clusters/
[ntpd]: https://wiki.dovecot.org/TimeMovedBackwards#Time_synchronization