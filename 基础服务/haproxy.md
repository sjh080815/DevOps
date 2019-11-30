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



