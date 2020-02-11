api的功能就是给k8s提供开发接口，默认在8080端口执行REST服务。一般情况下不能直接访问，需要手动启动8080端口。



```
kubectl proxy --port=8080 &

```



这样就运行了8080的代理。



通过curl可以查看api的版本信息：



```
[root@k8s1 ~]# curl localhost:8080/api

{

  "kind": "APIVersions",

  "versions": [

    "v1"

  ],

  "serverAddressByClientCIDRs": [

    {

      "clientCIDR": "0.0.0.0/0",

      "serverAddress": "192.168.89.129:6443"

    }

  ]

}

```



在api下可以查看pods、service和replicationcontrollers（即RC）。



这也就给kube二开做了很好的基础，可以通过调用api获取kube的所有信息。



返回信息是json格式，可以通过python或者java解析。



# API Server的分层结构



## API层



API层是k8s交互的主要层，可以通过API层获取k8s的数据信息。



## 访问控制层



用户访问API的时候，访问控制层对用户进行鉴定，是否有访问权限。



## 注册表层



和windows下的注册表相似，将资源对象存放在注册表里。



## etcd数据库



存放了集群主机的数据



# Proxy API



Proxy是代理的意思，这里就是代理访问pod。



统一API格式



```

/api/v1/namespaces/{namespace-name}/pods/{pod-name}

```



下边是API表：



| API | 功能 |

|--------|--------|

|proxy|创建proxy连接，后接path访问指定path|

|portforward|访问pod的portforward|





## 模块之间通信



node上的kubelet定时向API Server“汇报”当前状态，API Server将数据存放在etcd中。



Node Controller和API Server进行通信，监控node的状态。



