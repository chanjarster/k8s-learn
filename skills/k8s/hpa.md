HPA 模板

```yaml
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
  name: <名字>
  namespace: <命名空间>
spec:
  maxReplicas: <最大Pod数量>
  minReplicas: <最小Pod数量>
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: <deployment的名字>
  targetCPUUtilizationPercentage: <目标CPU利用率百分比>
```

例子：

```yaml
apiVersion: v1
items:
- apiVersion: autoscaling/v1
  kind: HorizontalPodAutoscaler
  metadata:
    name: echo-server
    namespace: default
  spec:
    maxReplicas: 3
    minReplicas: 1
    scaleTargetRef:
      apiVersion: apps/v1
      kind: Deployment
      name: echo-server
    targetCPUUtilizationPercentage: 80
kind: List
metadata:
  resourceVersion: ""
  selfLink: ""
```

也可以使用等价的命令：

```bash
kubectl --namespace default autoscale deployment echo-server --min=1 --max=3  --cpu-percent=80
```

查看HPA：

```bash
kubectl --namespace default get hpa
```

相关文档：

* [Horizontal Pod Autoscaler Walkthrough](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/)
* [Pod Autoscaler](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/Horizontal)

