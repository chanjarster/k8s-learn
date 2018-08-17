# Echo Server(istio版）

我们已经知道Echo Server是一个很方便的可以用来观察HTTP请求的简易程序，我们这里提供Istio版的部署：

* 在这之前你确保你部署了[Istio](../../addons-guide/istio)

使用本项目的提供的yaml文件，在执行之前修改`<your domain name>`：

```bash
kubectl apply -f echo-server.yaml
```

上面这个命令同时部署了几种模式，以方便让你对比它们的区别：


| Entry         | Pod Mode              | Url                                                  |
|---------------|-----------------------|------------------------------------------------------|
| Ingress Nginx | Istio not injected    | `http://ingress-test.<your domain name>/echo/`       |
| Ingress Nginx | Istio injected        | `http://ingress-test.<your domain name>/echo-istio/` |
| Istio Gateway | Istio not injected    | `http://istio-test.<your domain name>/echo/`         |
| Istio Gateway | Istio injected        | `http://istio-test.<your domain name>/echo-istio/`   |

如果你做了SSL Termination，则是：

| Entry         | Pod Mode              | Url                                                   |
|---------------|-----------------------|-------------------------------------------------------|
| Ingress Nginx | Istio not injected    | `https://ingress-test.<your domain name>/echo/`       |
| Ingress Nginx | Istio injected        | `https://ingress-test.<your domain name>/echo-istio/` |
| Istio Gateway | Istio not injected    | `https://istio-test.<your domain name>/echo/`         |
| Istio Gateway | Istio injected        | `https://istio-test.<your domain name>/echo-istio/`   |