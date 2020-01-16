# Pod基本用法

Pod时一个根容器，多个容器同时存在于Pod内。在定义Pod时候定义业务容器内容，语法如下：

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp
  labels:
    name: myapp
spec:
  containers:
  - name: myapp
    image: <Image>
    resources:
      limits:
        memory: "128Mi"
        cpu: "500m"
    ports:
      - containerPort: <Port>
```

上边时创建Pod的模板。

上述代码中，```metadata.name```和```metadata.labels```是Pod的参数。

```spec.containers```是容器的参数，即以下的容器运行到这个Pod中。

注意看代码：
```
containers:
    - name: frontend
      image: kubeguide/guestbook-php-fortend:localredis
      ports:
        - containerPort: 80
    - name: redis
      image: kubeguide/redis-master
      ports:
        - containerPort: 6379
```

这里定义了两个容器，两个容器被放到一个Pod里，那么，他们有共同的Pod IP。两个容器都可以通过localhost:port进行相互访问。

# 静态Pod
 
静态Pod就是直接用yaml创建的Pod，通常不会被这么使用。这样的Pod有以下这些特点：
 
 1. 直接通过kubelet，运行在特定Node
 2. 不和RC、deployment、DeamonSet关联，
 3. kubelet无法对这样的Pod进行健康检查

静态Pod由kubelet创建，随kubelet一起启动。

这里经常使用配置文件形式启动。使用```kubelet --config=```指定配置文件，启动后查看docker可见。
 

# 数据卷Volume
 
和docker的数据卷概念相似，数据卷可以共享。Kubernetes中的数据卷通常只是容器间进行数据共享用的。

同Pod多容器使用Pod级的Volume，即可进行容器间数据卷共享。
 
**过程**

创建Volume：

```yaml
volume:
    - name:
      emptyDir:{}
```

Containers中应用：

```yaml
volumeMounts:
    - name: 
      mountPath: 
```

这里使用Volume的name进行挂载选择，mountPath是挂载到容器的地址。

# 配置方案ConfigMap

使用ConfigMap进行容器集群环境的配置。

通常ConfigMap的用法：

1. 生成环境变量
2. 设置容器启动命令
3. 设置、挂载volume

ConfigMap使用key：value的方式进行配置。

语法：

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: test
data:
  key1: test1
  key2: test2
```

通常ConfigMap都是进行单文件定义的。当然也可以使用---在一个yaml文件下进行生命。

ConfigMap也可以用文件创建。

语法：

```bash
kubectl create configmap NAME --from_file=filename
```

## Pod调用

### 环境变量

调用ConfigMap导入到env中，语法：

```yaml
env:
    - name: ttt
      valueFrom:
        configMapKeyRef:
            name: test
            key: test1
```

使用```docker exec```命令。可以看到容器内的环境变量：

```bash
root@nginx:/# echo $ttt  
ttt
```

所以我们就可以在容器内进行变量调用了。

### 挂载调用

可以通过key:value的方式指向文件。

创建Volume数据卷：

```yaml
volumes:
    - name:teee
      configMap:
        name: ConfigMap_Name
        items:
          - key:
            path: 
```

在containers中调用数据卷Volume:

```yaml
volumeMounts:
    - name: teee
      mountPath: /root
```

以上配置文件中，configMap关键字加载了ConfigMap，在name中加载了这个ConfigMap，在item中加载了key。一定记得加path，path是将key加载到path中。

# 需要注意

1. ConfigMap必须在Pod之前定义，因为Pod需要调用，那么一定要先定义。
2. ConfigMap必须是同一个namespace才能调用
