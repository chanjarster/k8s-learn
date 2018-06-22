# Ingress安装指南

本文根据[Ingress Nginx Installation Guide][ingress-nginx-deploy]做了一些变化，下面假设你使用的是Baremetal的方式。

1. 新建一个目录，名字叫做`ingress-install`
1. 下载指南中的yaml文件：`mandatory.yaml`和`service-nodeport.yaml`
1. 根据[镜像/加速仓库清单][mirrors.md]的方法修改仓库地址
1. 修改`mandatory.yaml`添加`--enable-ssl-passthrough`启动参数，位置在这里：
   ```yaml
   ...
   serviceAccountName: nginx-ingress-serviceaccount
   containers:
     - name: nginx-ingress-controller
       image: quay.mirrors.ustc.edu.cn/kubernetes-ingress-controller/nginx-ingress-controller:0.15.0
       args:
         - /nginx-ingress-controller
         ...
         - --enable-ssl-passthrough
   ...
   ```
1. 修改`service-nodeport.yaml`添加`nodePort`参数，位置在这里：
   ```yaml
   ...
   ports:
   - name: http
     port: 80
     targetPort: 80
     protocol: TCP
     nodePort: 30080
   - name: https
     port: 443
     targetPort: 443
     protocol: TCP
     nodePort: 30443
   ```
1. 执行安装：`kubectl apply -f ingress-install`

以后如果你配置了基于path的反向代理，你就可以通过

* http://\<any-node-ip\>:30080/xxx
* https://\<any-node-ip\>:30443/xxx

来访问应用了。

[ingress-nginx-deploy]: https://kubernetes.github.io/ingress-nginx/deploy/
[mirrors.md]: ../../installation-guide/mirrors.md