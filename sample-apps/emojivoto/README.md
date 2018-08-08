# Emojivoto

emojivoto实际上是linkerd2的[demo app](https://linkerd.io/2/getting-started/#step-3-install-the-demo-app)。不过如果你的K8S集群是本地部署的，那么就没有办法访问这个app，原因是官方配置里`Servcie web-svc`的`type=LoadBalancer`。

所以我们把这个类型改成`ClusterIP`，然后再配置一个ingress，这样就能够外部访问了。

```bash
# 用这个命令把Servcie web-svc的类型改掉
kubectl delete -f web-svc-service-patch.yml
kubectl apply -f web-svc-service-patch.yml

# 添加一个ingress
# 不过在这之前先修改web-svc-ingress.yml中的<your-hostname>
kubectl apply -f web-svc-ingress.yml
```

emojivoto这个demo app有点坑，它只能从根路径访问，所以不能配置path，因此为了避免和其他ingress冲突，就必须配置一个hostname。