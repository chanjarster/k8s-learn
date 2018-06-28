# Http Redirect端口丢失问题

近日发现一个问题：应用程序在返回Http Redirect的时候丢失了原先访问的端口。比如，我们这样访问`http://IP-A:Port-A/app/delete`，这个url会响应302，但是它返回的Response header `Location`里丢失了端口，正确的结果应该是这样：`http://IP-A:Port-A/app/index`，但返回的却是：`http://IP-A/app/index`，把端口丢失了。

## 基本情况

我们的部署情况是这样的：

* 部署了[Nginx Ingress][ingress-nginx]，并使用NodePort的方式把Nginx Ingress Service暴露出来
* 配置了App的Ingress


服务器信息：

| Server Name | NAT Server  | K8S Node | Nginx Ingress Svc  | Nginx Ingress Pod | App Svc               | App Pod         |
|:-----------:|:-----------:|:--------:|:-----------------------:|:----------------------:|:---------------------:|:---------------:|
| IP          | IP-A        | IP-B     | IP-C(Cluster IP/VIP)   | IP-D(Cluster IP)       | IP-E(Cluster IP/VIP) | IP-F(ClusterIP) |
| Port        | Port-A      | Port-B(Nginx Ingress Svc's NodePort) | Port-C                  | 80(Container Port)     | Port-E                | Port-F   |


其实以上也不全是服务器，其中有两个K8S Service不是服务器，它们是VIP，关于这个请看[K8S - Using Source IP][k8s-doc-using-source-ip]一文，当访问`http://IP-A:Port-A/app/delete`的时候，这个请求从左到右贯穿了这些服务器。

顺便一提上面的NAT Server是一台普通的服务器，我们用它做了[PAT][wiki-pat]使我们的Nginx Ingress能够被外网访问到。

## 观察

我们使用之前提到过的[Echo Server][sample-echo-server]来观察透过Ingress访问Echo Server时传递给Echo Server的Request header：`http://IP-A:Port-A/echo-server`，得到了这些有趣的Request header：

```
host=IP-A:Port-A
x-original-uri=/echo-server
x-forwarded-for=IP-B
x-forwarded-host=IP-A:Port-A
x-forwarded-port=80
x-forwarded-proto=http
```

然后直接访问Echo Server Svc，发现是没有上面提到的`x-*`Request header的。于是怀疑问题出在这几个header上。

## 名词解释

来讲一下这些头各自代表什么意思。

* [x-forwarded-for][x-forwarded-for]，client访问proxy的时候，client的ip。
   在这里之所以是K8S Node的IP，是因为在Nginx Ingress看来请求是来自K8S Node的（好好看看之前提到的[K8S - Using Source IP][k8s-doc-using-source-ip]一文），在这之前的NAT它是不知道的。
* [x-forwarded-host][x-forwarded-host]，client访问proxy的时候，访问的原始host。
* [x-forwarded-proto][x-forwarded-proto]，client访问proxy的时候，访问的原始http scheme。
* [x-forwarded-port][x-forwarded-port]，client访问proxy的时候，访问的port。
* x-original-uri，查不到权威资料。

注意，前三个是事实标准，MDN有收录，`x-forwarded-port`和`x-original-uri`似乎是私有扩展。

## 实验

找一个趁手的Http Request工具（我用的是Postman），记得把Follow redirect关掉，然后模拟Nginx请求的方式（就是把上面提到的`x-*` header带上/去掉/修改值）**直接请求**App Svc。

结果发现`x-forwarded-port`是Response header `Location`的关键，即如果`x-forwarded-port=Port-A`的话，`Location`就会带上正确的端口。

## 分析

### Redirect url是如何构造的

可以推测，App利用了`host`和`x-forwarded-*`这些header来构造redirect url。

在Java Servlet API中，在描述[HttpServletResponse#sendRedirect][javadoc-send-redirect]的时候提到，其返回的URL必须是Absolute URL。

Tomcat的`org.apache.catalina.connector.Response`的[toAbsolute方法][src-tomcat85-connector-response-to-absolute]负责构造Absolute URL。

那么它又是如何知道选用什么Port的呢？这个和[RemoteIPValve][remote-ip-valve]([javadoc][javadoc-remote-ip-valve])有关，有兴趣的话你可以查阅相关文档。

上面只是讲了Tomcat是如何构造redirect url的，但这个方法不是标准的，不同的容器有各自的实现，毕竟Java Servlet API也没有规定如何构造Absolute URL。

我之前也写过一篇相关话题的文章《[反向代理使用https协议，后台tomcat使用http，redirect时使用错误协议的解决办法][segmentfault-article]》，你可以看一看。

### 为何x-forwarded-port是80

那么问题来了，我明明访问的是`IP-A:Port-A`，为何Nginx取到的值是80？

这是因为在整个请求链路的前段：NAT Server > K8S Node > Nginx Ingress Svc 都是在第4层工作的，可以认为它们干的事情都是NAT，Nginx Ingress Pod是不知道这些服务器/网络节点的端口，因此它只能把自己的端口80（容器内Port）给x-forwarded-port。

关于这个逻辑你可以查看Nginx Ingress的配置文件就能够知道了：

```
kubectl -n kube-system exec -it <nginx-ingress-controller-pod-name> -- cat /etc/nginx/nginx.conf
```

## 解决办法

### 请求时带上x-forwarded-port(不靠谱)

查看Nginx Ingress配置文件发现如果最初请求的时候带上`x-forwarded-port`的话，就能够改变它传递到后面的值，但是这有两个问题：

1. 通过浏览器访问时，你没有办法加上这个header
2. 这个header一般都是反向代理加的，也就是在我们的Nginx Ingress之前还得有一个反向代理

所以这个方法不好。

### 修改tomcat的代码(不靠谱)

虽然可以通过修改tomcat的代码，让它从`x-forward-host/host` header来取port，但是这个不现实。

### 修改NAT Server的端口为80(靠谱)

这个方法比较靠谱，只要将NAT Server的端口改成80就没有问题了。

事实上，如果你直接访问K8S Node的话（NodePort方式），也是要将NodePort设置为80，记得前面说的吗？Nginx Ingress无法知道上层NAT的端口。

总而言之，就是你最初请求的URL不能是80之外的端口，必须是`http://some-ip/app`才可以。

### 使用Nginx Ingress Annotations(靠谱)

使用Nginx Ingress提供的[Proxy redirect annotations][nginx-proxy-redirect-annotation]，将`Location`的值做文本替换。

[ingress-nginx]: https://kubernetes.github.io/ingress-nginx/
[k8s-doc-using-source-ip]: https://kubernetes.io/docs/tutorials/services/source-ip/
[sample-echo-server]: ../../sample-apps/echo-server
[wiki-pat]: https://zh.wikipedia.org/wiki/网络地址转换#网络地址端口转换（NAPT）
[javadoc-send-redirect]: https://docs.oracle.com/javaee/7/api/javax/servlet/http/HttpServletResponse.html#sendRedirect-java.lang.String-
[src-tomcat85-connector-response-to-absolute]: https://github.com/apache/tomcat85/blob/trunk/java/org/apache/catalina/connector/Response.java#L1646
[remote-ip-valve]: https://tomcat.apache.org/tomcat-8.5-doc/config/valve.html#Remote_IP_Valve
[javadoc-remote-ip-valve]: http://tomcat.apache.org/tomcat-8.5-doc/api/org/apache/catalina/valves/RemoteIpValve.html
[segmentfault-article]: https://segmentfault.com/a/1190000006206083
[x-forwarded-for]: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Forwarded-For
[x-forwarded-proto]: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Forwarded-Proto
[x-forwarded-host]: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Forwarded-Host
[x-forwarded-port]: https://docs.aws.amazon.com/elasticloadbalancing/latest/classic/x-forwarded-headers.html#x-forwarded-port
[nginx-proxy-redirect-annotation]: https://kubernetes.github.io/ingress-nginx/user-guide/nginx-configuration/annotations/#proxy-redirect