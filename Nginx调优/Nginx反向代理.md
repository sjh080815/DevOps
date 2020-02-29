Nginx的有一个主要功能就是反向代理。反向代理的原理就是将Nginx服务器收到的请求数据转发给其他服务器。反向代理就是将数据反向发给更多的机器，从而做到负载均衡。

# 集群创建

这里为了节省内存，运行两个docker容器作为站点服务器。这个后端服务器可以是Nginx，也可以是Apache，也可以是Tomcat。Nginx作为一个七层的代理服务器，只能转发HTTP协议的请求信息。

# 代理服务器配置

配置代理服务器就是在服务器上配置一个后端服务器的集群。

在nginx.conf文件中，在server块中创建一个upstream块。这是一个后端服务器的集群块。在这个块写：

```conf
upstream group{
    server 172.17.0.2;
	server 172.17.0.3;
}

```

这样就组成了一个upstream群。upstream群的名字叫group。这个名字是只能被nginx认定的。

我们对这个upstream进行调用。在location块中使用代理：

```
location / {
	proxy_pass http://group;
}
```

这样就组成了一个简单的反向代理服务器系统。

# 多样的需求

## 不同的权重

在一个upstream群中，有多个服务器，这些服务器的性能可能也不进行同，那么不同的服务器可以承担的并发也不是一个量级的，这时候就需要进行权重配比了。

在创建server的时候添加一个参数weigth，数字越大表示权重越大，如果不定义则表示weigth=1。

## 需要保留的需求

有时候，某些后端服务器需要获取到客户机的ip即其他一些和参数，那么就需要在代理的时候设置一下转发规则。

在location块代理后边添加请求头重定义：

```
proxy_set_header Host $host;
proxy_set_header X-Real-IP $remote_addr;
proxy_set_header X-Forwarded-For $proxy_add_x_forward_for;
```

第一段，转发了客户机的Host请求，即主机名。

第二段，转发了客户机的真实IP。

第三段，转发了代理地址。