# redis安装

**一定记得，安装运行redis的时候一定要放行相关的端口，或者直接关闭防火墙，否则会发生各种奇奇怪怪的错误，度娘那都问不到。**

我这里所有服务器都使用了Centos的操作系统，这个软件对发行版的要求比较低，所以可以随意。

要做主从复制，咱们肯定要在主机和从机上安装redis。所以，一下操作，所有redis集群内的主机都要进行。

下载redis。我用的是redis 5.0.7，下载地址http://download.redis.io/releases/redis-5.0.7.tar.gz，使用wget进行下载。下载完成后，进行解压，使用tar -zxvf。

解压完成后，我们可以看到如下的目录

```bash
bash# ll
总用量 1948
-rw-------. 1 root root    1241 11月 10 10:57 anaconda-ks.cfg
drwxrwxr-x. 6 root root    4096 11月 20 01:05 redis-5.0.7
-rw-r--r--. 1 root root 1984203 11月 20 01:06 redis-5.0.7.tar.gz
```

进入redis的目录，直接进行编译

```bash
make && make install
```

编译过程需要gcc的支持，如果服务器没有安装这个东西，咱们先进行安装

```bash
bash# yum install -y gcc
```

然后运行安装，在utils文件夹下有脚本install_server.sh，像执行普通脚本一样执行

```bash
bash# ./install_server.sh
```

安装过程中会出现一些选项，如果没有特殊的需求，直接一直回车就好了。

**注意：redis在安装好以后是默认开机运行的，所以咱们就不用手动设置了**

一般情况下，刚安装好的redis的地址绑定都是127.0.0.1，咱们需要对这一项以及其他参数进行配置。

## 对于主机

修改/etc/redis/6379.conf,修改如下

```bash
bind 0.0.0.0
databases 16
min-replicas-to-write 1
```

## 对于从机

修改相同的文件，修改如下：

```bash
bind 0.0.0.0
slaveof 192.168.139.149 6379
```

这里的第二条参数一般情况默认配置文件不存在，需要自己添加。这条参数设置了主机的ip和主机的服务端口。

设置完成后主机和从机直接重启服务。

```bash
/etc/init.d/redis_6379  restart
```

ok，进行测试。

**主机**

```bash
bash# redis-cli
127.0.0.1:6379> set tt 1
OK
```

**从机**

```bash
bash# redis-cli
127.0.0.1:6379> get tt
"1"
```

主从复制实现。

# 哨兵

> redis集群往往有多台主从服务器，有时候主服务器也会出现问题。为了保证服务的高可用性，常常会使用redis哨兵对多库进行监控，当主库发生宕机的时候，集群自动选举出一个服务器作为新的master，这样大大的提高了服务的可用性。

哨兵在三台和三台以上的服务器中才能表现出它的效果，所以我们按slave的配置方法进行安装配置redis服务。

分别在三台服务器上修改redis-5.03/sentinel.conf，并将文件分别复制到服务器的/etc/redis目录下。修改以下参数：

```bash
protected-mode no   
sentinel monitor mymaster 172.25.78.12 6379 2  
sentinel down-after-milliseconds mymaster 10000 
sentinel parallel-syncs mymaster 1 
sentinel failover-timeout mymaster 180000
```

在三台服务器上分别运行

```bash
redis-server /etc/redis/sentinel.conf --sentinel
```

监控master机，当master机宕机时，自动将一台slave转换为master。