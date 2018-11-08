# Rancher 2.0的安装

写本文时Rancher版本2.0.6

本文遵循的是官方文档[High Availability Installation with External Load Balancer (TCP/Layer 4)][ha-server-install]，但是跳过了[Configure DNS][3-configure-dns]步骤。

本文所用到的服务器都是Ubuntu Server 16.04 LTS

**这篇文章只是如何安装Rancher，虽然Rancher安装完毕后也是运行在K8S集群之上，但是这个K8S集群应该仅供Rancher使用，不应该有其他应用运行在这个K8S集群上。**

**在安装好Rancher之后，需将已有的K8S集群纳管进来，或者利用Rancher创建K8S集群。关于如何拿已有的机器创建K8S集群见[这篇文档][rancher-custom-nodes]**

## 1. 准备Linux服务器

根据[1. Provision Linux Hosts][1-provision-linux-hosts]章节准备3台机器，记住，这些服务器是最终作为K8S集群的node的。

给每个机器安装Docker CE，步骤：

1. 参考[Docker镜像清单](../docker-mirrors.md)里提到的方法，设置正确的软件源
1. 运行`sudo apt -y install docker-ce=17.03.2~ce-0~ubuntu-xenial`，参考[Get Docker CE for Ubuntu][docker-ce-ubuntu]
1. 执行`sudo usermod -aG docker $USER`将用户添加到docker用户组里，参考[Post-installation steps for Linux][docker-linux-postinstall]
1. 修改Docker的Storage driver为overlay2，[文档][docker-storage-overlay2]
1. 参考[Docker镜像清单](../docker-mirrors.md)里提到的方法，配置docker hub镜像

## 2. 配置负载均衡

根据[2. Configure Load Balancer][2-configure-load-balancer]，弄一台机器，配置不用很好，2G内存足以。

安装Nginx：

1. 根据[Nginx官方文档][nginx-ubuntu]安装Nginx
2. 修改`/etc/nginx/nginx.conf`文件，内容这样，把下面的IP_NODE_?改成你的服务器的IP：
   ```
   worker_processes 4;
   worker_rlimit_nofile 40000;
   
   events {
       worker_connections 8192;
   }
   
   http {
       server {
           listen         80;
           return 301 https://$host$request_uri;
       }
   }
   
   stream {
       upstream rancher_servers {
           least_conn;
           server IP_NODE_1:443 max_fails=3 fail_timeout=5s;
           server IP_NODE_2:443 max_fails=3 fail_timeout=5s;
           server IP_NODE_3:443 max_fails=3 fail_timeout=5s;
       }
       server {
           listen     443;
           proxy_pass rancher_servers;
       }
   }
   ```

## 3. 配置DNS

跳过[3. Configure DNS][3-configure-dns]这个步骤，但是Rancher依赖于域名而不是IP访问，因此我们要配置`/etc/hosts`。

假设你规划Rancher的域名是`rancher.company.com`，那么在每台服务器的`/etc/hosts`加上一条：`IP_NGINX rancher.company.com`。

## 4. 下载RKE

根据[4. Download RKE][4-download-rke]下载RKE到之前的Nginx服务器上。

把Nginx服务器的公钥通过添加到之前3台服务器的`~/.ssh/authorized_keys`里。

## 5. 下载RKE配置文件模板

使用[5. Download RKE Config File Template][5-download-rke-config-file-template]提到的[自签名证书模板][self-signed-cert-template]。

## 6. Configure Nodes

根据[6. Configure Nodes][6-configure-nodes]配置节点信息：

1. 把之前下载的模板改个名字叫做`rancher-cluster.yaml`
2. 修改`nodes`部分的信息，把之前3台服务器的IP写上去，并且指定登录用户名

## 7. 配置证书

根据[7. Configure Certificates][7-configure-certificates]配置证书，这里用的依然是[自签名证书的方案][option-self-signed-cert]。

1. 参考[这篇文章][self-signed-certs-ca]里“创建私有CA，然后用该 CA 给证书进行签名”章节得到`ca.crt`、`server.crt`、`server.key`
2. 私有证书的Common Name必须是之前定义好的`rancher.company.com`
3. 通过这样`openssl rsa -in server.key -out server.key.nopass`获得一个没有passphrase的key文件
4. 配置`rancher-cluster.yaml`：
   * `cacerts.pem`对应的是`ca.crt`的BASE64
   * `tls.crt`对应的是`server.crt`的BASE64
   * `tls.key`对应的是`server.key.nopass`的BASE64
   * 获得BASE64的[方法][base64]

因为我们是直接创建私有Root CA，然后签发证书，所以不需要考虑文档中提到的中间证书（Intermediate certificates）的问题。	
## 8. 配置FQDN

根据[8. Configure FQDN][8-configure-fqdn]，把`rancher-cluster.yaml`中所有的`<FQDN>`替换成之前你定下的`rancher.company.com`

## 9. 备份RKE配置文件

根据[9. Back Up Your RKE Config File][9-back-up-your-rke-config-file]备份配置文件。

## 10. 运行RKE

根据[10. Run RKE][10-run-rke]运行RKE，我们这里是Ubuntu所以是这样的：`./rke_linux-amd64 up --config rancher-cluster.yml`

## 11. 备份自动生成的配置文件

根据[11. Back Up Auto-Generated Config File][11-back-up-auto-generated-config-file]备份文件，这个文件用在以后给Rancher升级的时候。

## 12. 配置你的电脑的hosts

因为之前我们跳过了DNS配置，所以你得自己的电脑上配置hosts：`IP_NGINX rancher.company.com`。

然后你就可以在浏览器中打开`https://rancher.company.com`使用Rancher了。第一次访问会让你设置admin用户的密码。

## 13. 给cattle-cluster-agent配置hosts

1. 进入Rancher Web UI之后，拉开左上角的`Global -> Cluster: local -> Default`，进入Default项目。
1. 点击上方Toolbar的Namespaces，把`cattle-system`添加到Default项目里来。
1. 点击上方Toolbar的Workloads，看到`cattle-cluster-agent`，会发现它是红色的，处于错误状态，我们编辑它。
1. 进入编辑页面后，点击最右下方的`Show advanced options`，展开更多选项，再展开`Networking`选项，点击`Add Host Alias`，把我们之前定义的`rancher.company.com`的IP配置进去。
1. 点击Upgrade

之后你就能看到cattle-cluster-agent变成绿色的了。

## 14. 设置正确的MTU

见这篇文章[修改MTU](change-mtu.md)。

## 参考文档

* [Get Docker CE for Ubuntu][docker-ce-ubuntu]

[ha-server-install]: https://rancher.com/docs/rancher/v2.x/en/installation/ha-server-install/
[docker-ce-ubuntu]: https://docs.docker.com/install/linux/docker-ce/ubuntu/
[docker-linux-postinstall]: https://docs.docker.com/install/linux/linux-postinstall/
[nginx-ubuntu]: https://www.nginx.com/resources/wiki/start/topics/tutorials/install/#official-debian-ubuntu-packages
[1-provision-linux-hosts]: https://rancher.com/docs/rancher/v2.x/en/installation/ha-server-install/#1-provision-linux-hosts
[2-configure-load-balancer]: https://rancher.com/docs/rancher/v2.x/en/installation/ha-server-install/#2-configure-load-balancer
[3-configure-dns]: https://rancher.com/docs/rancher/v2.x/en/installation/ha-server-install/#3-configure-dns
[4-download-rke]: https://rancher.com/docs/rancher/v2.x/en/installation/ha-server-install/#4-download-rke
[5-download-rke-config-file-template]: https://rancher.com/docs/rancher/v2.x/en/installation/ha-server-install/#5-download-rke-config-file-template
[6-configure-nodes]: https://rancher.com/docs/rancher/v2.x/en/installation/ha-server-install/#6-configure-nodes
[7-configure-certificates]: https://rancher.com/docs/rancher/v2.x/en/installation/ha-server-install/#7-configure-certificates
[8-configure-fqdn]: https://rancher.com/docs/rancher/v2.x/en/installation/ha-server-install/#8-configure-fqdn
[9-back-up-your-rke-config-file]: https://rancher.com/docs/rancher/v2.x/en/installation/ha-server-install/#9-back-up-your-rke-config-file
[10-run-rke]: https://rancher.com/docs/rancher/v2.x/en/installation/ha-server-install/#10-run-rke
[11-back-up-auto-generated-config-file]: https://rancher.com/docs/rancher/v2.x/en/installation/ha-server-install/#11-back-up-auto-generated-config-file
[option-self-signed-cert]: https://rancher.com/docs/rancher/v2.x/en/installation/ha-server-install/#option-a-bring-your-own-certificate-self-signed
[self-signed-cert-template]: https://raw.githubusercontent.com/rancher/rancher/e9d29b3f3b9673421961c68adf0516807d1317eb/rke-templates/3-node-certificate.yml
[base64]: https://rancher.com/docs/rancher/v2.x/en/installation/ha-server-install/#base64
[self-signed-certs-ca]: https://www.jianshu.com/p/e5f46dcf4664
[rancher-custom-nodes]: https://rancher.com/docs/rancher/v2.x/en/cluster-provisioning/rke-clusters/custom-nodes/
[docker-storage-overlay2]: https://docs.docker.com/storage/storagedriver/overlayfs-driver/#configure-docker-with-the-overlay-or-overlay2-storage-driver
