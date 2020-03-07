haproxy是进行四层转发（http转发）和七层转发（TCP转发）的常用工具，常用于负载均衡。

安装：

```bash
yum install haproxy
```

配置文件：

配置文件在/etc/haproxy/haproxy.cnf

参数功能表


| 参数 | 功能 |
|----------|----------|
| log | 全局日志 |
|chroot|工作目录|
|pidfile|pid的储存位置|
|maxconn|最大并发数|
|user|软件用户|
|group|软件用户组|

# 负载均衡

负载均衡的原理就是使用反向代理，利用轮询等算法进行服务器访问。

在配置文件创建listen模块。语法：

```
listen SL_name *:port
    balance roundrobin #均衡模块
    server server_name serverhost check
    server server_name serverhost check
```

# 动静分离

动静分离类似于nginx的动静分离，具体配置如下：

在配置文件创建模块：

```
frontend  main *:80
    acl url_static      path_beg      -i /static /images
    acl url_static      path_end      -i .jpg .gif .png .css .js 
    use_backend static          if url_static 
    default_backend            app 
```

* acl表示负载策略，后边以url的参数是acl策略的名。

* 第三个参数是url的解析策略。path_beg是url开头包含，path_end是url结尾包含。

* -i参数后边跟的是解析策略。

后边的逻辑分析如下：

```
    use_backend static          if url_static  #后边if的意思是探测是否触发了静态的acl
    default_backend            app  #默认的模块
```

* 第一个参数表示模块设定

* 第二个参数表示backend名

* 第三个参数表示backend模块

在这个模块之后定义一下backend。backend定义格式和前边负载均衡定义listen的语法相同。


# 调度算法

## roundrobin

轮询算法，也可以配合使用weight记性权重轮询，但是更推荐使用下边的方法进行权重轮询。

## static-rr

权重轮询，按照配置的权重进行轮询均衡。

## leastconn

查最少连接的主机，转发到最少连接的主机。

## source

根据源IP进行转发。

## uri

根据uri进行请求转发。

## url_param

根据url的请求进行转发。

在设置这个参数的时候，这样写：

```
balance url_param test
```

这样的话，需要请求到哪个主机，直接使用get请求，发送到haproxy服务器。

## hdr

根据http请求头锁定一个请求。

# session共享及一致性问题

如果session一致就会出现刷新页面后，页面的信息发生变化，所以我们就要做session共享。

* 方法一： 使用source，获取用户IP hash。
* 方法二： 使用cookie进行session识别。
* 方法三： 后端服务器产生的session和后端服务器标识存在haproxy中的一张表里