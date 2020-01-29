Pod并非已成不变的，在应用场景中会经常进行容器添加、删除和调度。调度是一个很繁琐的活，不可能全由人工进行操作，所以有RC、Deployment和ReplicaSet，控制副本调度。

# 自动调度

使用deployment或RC。

创建一个基于deployment的自动调度：

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
```
deployment和RC控制器的写法大致相同，用途、用法也大致相同。

```bash
[root@k8s1 ~]# kubectl get rs
NAME               DESIRED   CURRENT   READY   AGE
nginx-7bffc778db   3         3         3       106s
```
可以从上述的命令看到刚才deployment创建的rs

# 定向调度

有时候某些特定的服务需要运行在特定的node上，这就需要使用定向调度。

创建node的label

```bash
kubectl labels nodes node_name key=value
```

创建RC时使用nodeSelector指定节点。

```yaml
nodeSelector:
    im: master
```

# Node亲和度调度

