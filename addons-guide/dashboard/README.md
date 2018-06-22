# Dashboard配置Ingress

执行本项目提供的`dashboard-ingress.yaml`：

```bash
kubectl apply -f dashboard-ingress.yaml
```

然后就可以通过 https://\<any-node-ip\>:30443/k8s-dashboard 访问dashboard了