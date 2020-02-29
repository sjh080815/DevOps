LVS全称是Linux Virtual Server，即Linux虚拟服务器。这是一个具有三种工作模式，十种调度算法的代理服务器软件。

这是一种常见的反向代理案例

![](http://www.zsythink.net/wp-content/uploads/2017/07/070617_0124_1.png)

这是LVS组成了一个负载均衡集群，LVS服务器将接收到的请求转发给了群内主机上。因为LVS在这里进行了数据的分发，像一个中转，所以我们叫做“调度器”。

在生产环境中，服务器一般都是要发布在公网的。因为backend的特殊性不便于直接暴露在公网上，那么LVS服务器就起了很大的作用。

LVS被配置了一个公网IP，这个IP将被客户进行访问，这个IP通常称为VIP。LVS服务器一般都是使用了双网卡，另一张网卡要连接到内网，那么这张内网网卡的IP就叫做DIP，群内主机的内网IP就叫做RIP。

# NAT定义

NAT的意思就行网络地址转换。了解虚拟化的话就会知道，VMware虚拟机默认运行在NAT的网络模式下，这个网络模式下，群内主机使用一个特殊的IP，与NAT服务器的与外网连接的IP不同。外部访问VIP的时候，请求的VIP会被LVS服务器转换成RIP，再由DIP进行转发，发送到RIP上。返回报文的原理也一样。

定义方式：

创建NAT映射：

```bash
ipvsadm -A -t VIP:port -s rr
ipvsadm -a -t VIP:PORT -r RIP:port -m
```

这样创建出来的NAT不能被localhost访问，原因如下：

NAT被直接绑定到了VIP，并没有绑定到主机本身，那么主机的localhost环路是无法访问到环路外的内容的。

