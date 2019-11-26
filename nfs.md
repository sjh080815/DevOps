服务器列表


| 服务器 | IP |
|----------|----------|
| master | 192.168.139.146 |
| slave | 192.168.139.131 |

首先，记得防火墙放行111端口。我这里直接关闭了防火墙，所以就不多赘述。

安装NFS服务的几个必要软件：rpmbind和nfs-utils

```bash
yum install rpcbind nfs-utils
```

启动服务

```bash
systemctl start rpcbind
systemctl start nfs
systemctl enable rpcbind
systemctl enable nfs
```
咱们可以查看一下服务端口，看看服务是不是已经在运行了

```bash
bash# rpcinfo -p localhost

   program vers proto   port  service
    100000    4   tcp    111  portmapper
    100000    3   tcp    111  portmapper
    100000    2   tcp    111  portmapper
    100000    4   udp    111  portmapper
    100000    3   udp    111  portmapper
    100000    2   udp    111  portmapper
```

111端口正常监听，说明服务已经在运行了。

定义一下服务共享的目录，在/etc/exports中设置：

```bash
bash# vi /etc/exports

/data 192.168.139.0/24(rw)
```

第一个参数是共享的目录，第二个是允许访问的网段，用IP/mask的格式定义。括号里写权限。

这里记得，nfs服务在安装时会创建一个nfsnobody的用户和用户组，共享的目录的所有者和所有组定义为nfsnobody。然后重新启动服务。

OK，咱们在服务端的配置已经完成了，下来配置客户端。

首先在客户机上扫描服务端的共享状态：

```bash
bash# owmount -e 192.168.139.146

Export list for 192.168.139.146:
/data 192.168.139.0/24
```

共享状态正常，下来将nfs挂载到客户机：

在客户机上创建一下挂载点，然后将nfs挂载上去。

```bash
bash# mount -t  nfs 172.16.1.31:/data/ /mnt
```

nfs的基本功能已经可以实现了。