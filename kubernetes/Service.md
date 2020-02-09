# Service、Pod、RC和Deployment之间关系

RC和Deployment批量自动生成Pod，且自动分配Pod IP，那么Pod IP就是随机的。在常规的生产环境中，容器的数量并非是一个定值，经常会删除或者增加Pod，那么Pod IP肯定就会发生改变。现在要外网访问Pod无外乎两种方法：

1. 使用HaProxy或者nginx一类的反向代理工具进行反向代理
2. 使用Service绑定RC或者Deployment

以上方法，第一种，无论是haproxy还是nginx，都需要手动定义反向代理的IP，不具备灵活性，且不能满足HA，所以这种方式基本没有什么可行性。

**注意一点：如果单纯使用了Pod，没有用RC和Deployment生成Pod，那么，只要不删除Pod，Pod一般情况是不会改变IP的。**

第二种方式：将Service绑定到RC或者Deployment上，当Pod发生调度时，始终会保持着labels，能被selector选择到。Service会被分配一个固定IP，将POD业务端口也带到这个IP上，这样就可以用固定的Service访问了。但是还存在一个问题，Service是集群内的一个虚拟网络，不能被外网访问，这时候就需要使用haproxy或者nginx做反向代理了，当然也可以使用nodePort进行端口穿透，但是这个业务端口分配的范围基本都是30000以上，不是常规业务端口，所以最好的方法还是使用反向代理。

**Service的一大特色：负载均衡，数据分发**

# Service基本要素

spec.ports.port: 绑定到Service的port

spec.ports.targetPort: Pod的业务端口

type：映射类型，常用NodePort

# Headless Service

这个Service只对Pod进行了选择，但是没分配clusterIP，可以根据客户的要求进行操作。

# 外网映射

有两种方式，一种是在生成Pod时候定义，一个是通过Service定义。

第一种：

在定义Pod的时候，在spec.containers.ports下添加参数hostPort,定义映射到主机的端口。

第二种：

在创建完RC或者Deployment之后，创建一个Service，这里要给Service添加一个spec.ports.nodePort，这就把Service的端口映射到了主机上。

# Kubernrtes集群DNS原理解析

看一下k8s集群默认会跑哪些容器：

```
[root@k8s1 ~]# kubectl get pods -n kube-system
NAME                                   READY   STATUS    RESTARTS   AGE
coredns-5c98db65d4-p5t4x               1/1     Running   12         4d5h
coredns-5c98db65d4-shrx5               1/1     Running   12         4d5h
etcd-k8s1                              1/1     Running   12         4d5h
kube-apiserver-k8s1                    1/1     Running   13         4d5h
kube-controller-manager-k8s1           1/1     Running   14         4d5h
kube-flannel-ds-amd64-5mcq4            1/1     Running   16         4d5h
kube-flannel-ds-amd64-crgkf            1/1     Running   16         4d5h
kube-proxy-bn64f                       1/1     Running   12         4d5h
kube-proxy-zc4s9                       1/1     Running   12         4d5h
kube-scheduler-k8s1                    1/1     Running   14         4d5h
kubernetes-dashboard-5c7687cf8-f9pjd   1/1     Running   16         4d5h
```

这些容器可分为etcd、DNS、apiversion、controller、flannel、scheduler和proxy。这些容器一起组成了一个集群，看一下这些容器的工作方式（没有相同的图，只有类似图）。

![2.png](.\img\2.png)

Master就是我们广义上的控制端，控制端可以创建Service。

首先看一下Service的原理。Service选择一些特定的容器，将容器的端口映射到Service上，那么这里就必然也会有IP的转移，或者说是解析。

创建Service时，生成一个ClusterIP，当访问Cluster的时候会访问到Pod。

从图上看，kube2sky将生成的Service的DNS记录存放在了etcd上，同时也“通知”healthz开始进行健康检查。这里的etcd就充当了一个DNS数据库的角色，存放Cluster到IP的解析。

当访问Service的时候，etcd给出相关的解析，skyDNS（也就是上边代码中的DNS容器）进行了解析，访问到Pod。

图中有个proxy容器，proxy是代理容器，就是转发请求或者数据。