#  全局块

user：user是Nginx运行的用户。比如虚拟主机文件放在/root目录，则需要root账户去访问。

work_process：工作线程，应对高并发。

error_log:这是Nginx的错误日志配置，默认会生成以下参数。

```
#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;
```

pid：在Linux中，每运行一个软件都会产生一个pid。这里的参数是pid放置位置。

序列化进程：添加参数accept_mutex

Nginx有时会接受多个网络连接，在全局添加multi_accept参数。

# http块

include：包含配置文件。默认配置中采用mime-type，表示区分资源类别。

default-type：处理前端请求

access_log：访问日志

sendfile：发送文件，默认打开。

keepalive_timeout：连接超时，单位是秒

keepalive_request：限制连接发送请求数

 **以上两个参数都可以在http、server和location中配置**

