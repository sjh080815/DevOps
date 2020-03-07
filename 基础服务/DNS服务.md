# DNS服务器原理

DNS服务器是对域名进行解析的服务器。当浏览器访问网页时，首先会给DNS服务器发送一个请求，解析到域名对应的IP上，然后浏览器对IP进行访问。

DNS服务器保存了解析的域名和IP，通常来说，这个解析是一一对应的。

客户端发起请求，首先会在客户端的hosts文件中查询是否有相关的解析，如果没有，将包含有域名的请求头转发给DNS服务器上。当DNS服务器上也查不到相关的解析时，就会将数据转发给根服务器进行数据解析，也可以将数据转发给forward进行解析。

# 权威DNS和非权威DNS

权威DNS是指官方的DNS服务器，这种服务器的解析数量大，且更为稳定。非权威DNS是指自己搭建的DNS服务器，一般是自己或公司使用，使用范围小，可承载的并发量不大的情况。

# 根服务器

目前全球有13个根DNS服务器。九台在美国，还有三台分别在英国，瑞士和日本。还有一台是主跟服务器（相当于根服务器的解析入口，从这被一层一层解析）。

# 安装named服务

可以直接使用yum安装方式安装named

```bash
yum install bind
```

在默认情况下，named的配置文件会被分布到两个位置，一个是/etc下，一个是/var/named下。

对named进行基础参数配置。

编辑named.conf

```bash
bash# vim /etc/named.conf

...
listen-on port 53 { any; };
...
allow-query     { any; };
...
recursion yes;
forward first;
forwarders {
223.5.5.5;
223.6.6.6;
8.8.8.8;
8.8.4.4;
};
```

分析一下上边的字段：

listen-on：监听，配置了监听端口和监听IP，通常使用any（所有网卡都进行监听），如果需要绑定到特定的ip，就直接在这里写。

allow-query：允许访问。一般默认情况这里只会允许localhost进行访问。可以改成any，亦可以写成ip的形式。

# 创建正向解析

观察named.conf文件，会发现有以下一段
```
include "/etc/named.rfc1912.zones";
include "/etc/named.root.key";
```

不难看出，这里引入了两个文件。通常，第一个文件是zone的文件，第二个是用户密钥文件。查看zone文件：

```bash
bash# cat /etc/named.rfc1912.zones

zone "localhost.localdomain" IN {
	type master;
	file "named.localhost";
	allow-update { none; };
};

zone "localhost" IN {
	type master;
	file "named.localhost";
	allow-update { none; };
};

zone "1.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.ip6.arpa" IN {
	type master;
	file "named.loopback";
	allow-update { none; };
};

zone "1.0.0.127.in-addr.arpa" IN {
	type master;
	file "named.loopback";
	allow-update { none; };
};

zone "0.in-addr.arpa" IN {
	type master;
	file "named.empty";
	allow-update { none; };
};
```

可以见得，上述的写法是相同的。都是创建一个zone块，然后对zone块进行参数配置。

type参数：字面意思，就是类型。通常设置master，作为主机。DNS的主从原理后边再说。

file参数：解析的文件，下边再说这个文件的写法和原理。

allow-update参数：这也是权限配置。

# 解析类型及解析文件

查看一下前边的file，看看结构：

```
$TTL 3H
@	IN SOA	@ rname.invalid. (
					0	; serial
					1D	; refresh
					1H	; retry
					1W	; expire
					3H )	; minimum
	NS	@
	A	127.0.0.1
	AAAA	::1
```

这里插入一段前置知识：

| 解析类型 | 意义 |
|--------|--------|
|  A      |   典型的IPV4解析，将域名指向这个IP     |
|CNAME|解析到别名，和rewrite有点像|
|MX|邮箱解析，比如@qq.com这样的|
|TXT|为域名设置说明|
|NS|name server|

**需要注意：**A解析和AAAA解析的区别：A解析和AAAA解析的作用是相同的，但是一个工作在IPV4网络下，一个工作在IPV6网络下。

按照上边的文件创建一个自己的解析文件：

```
$TTL 3H
@	IN SOA	@ hzlslm.org.cn. (
					0	; serial
					1D	; refresh
					1H	; retry
					1W	; expire
					3H )	; minimum
		IN 		NS     dns.test.com.
		IN 		MX  5  mail
www	IN	A	47.112.118.26
```

设置的生存时间是三小时，即三小时后会刷新DNS。

第一行的那一块是创建了这个域名的解析块，按这样写就好了。

最后一行，我创建了一个www的解析，这个解析格式如下：
```
记录名	IN	解析方式	解析值
```

创建完毕后，可以使用named-checkconf进行配置校验。如果没问题的话就可以启动服务了。

# 客户端配置

客户端一般是windows或者linux用户机。

windows：在网络和共享中心中直接修改DNS。

linux：修改resolve.conf文件。


**注意一点，如果要做外网的DNS非权威服务器，最好使用经典网络服务器，直接将IP分配给服务器，尽量不要使用DMZ的方式做内网映射。**