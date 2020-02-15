GlusterlFS是一种GNU的分布式文件系统，具有高可用，多种方式数据安全保障的数据系统。这种文件系统可以使用数据复制和分布储存的方式进行数据储存。

# 安装GlusterFS

使用yum安装Gluster的yum源

```bash
yum install centos-release-gluster
```

安装Gluster和相关的依赖：

```bash
yum install -y glusterfs glusterfs-server glusterfs-fuse glusterfs-rdma
```

# 数据盘管理

使用分布式数据存储服务的时候，一般不会将数据直接放在宿主机的系统盘上，经常会使用RAID1的磁盘阵列作为数据储存阵列。这里设置一下数据盘：

创建数据盘，即格式化磁盘：

```bash
mkfs.xfs -i size=512 /dev/sdb
```

把创建好的数据盘挂载到集群节点主机上。

```bash
mkdir -p /glu/sdb
mount /glu/sdb /dev/sdb
```

这样就做好了一个数据存储位置。

如果是磁盘阵列主机的话，这里就进行多次操作。

# 创建Gluster集群

先创建一下数据存储集群：

```bash
gluster peer probe host
```

这里使用的是host，所以先要在/etc/hosts中进行主机解析。

可以使用命令检查一下我们刚才添加的集群节点是否成功：

```bash
gluster> peer status
Number of Peers: 1

Hostname: host4
Uuid: 0497f5dc-2d01-4a51-8840-d4d99e7a5330
State: Peer in Cluster (Connected)
```

这就说明添加集群节点成功。

# 创建数据卷

Gluster有多种分布式数据储存方式，下边是几种较为常见的：

## 分布卷

分布卷类似于RAID0磁盘，将数据离散的分布在不同Brick上，这样的结构读写效率更高，但是容灾性能较差。

创建的时候无需添加参数。

```bash
gluster volume create name host:/path
```

## 复制卷

两个Brick上储存相同的数据，这样的的架构数据读写效率较低，但是数据的安全性能更好。该种数据存储方式类似于RAID1，当一个节点挂掉的时候，有另一个节点可用。或一个节点的数据盘产生坏道数据丢失时另一个数据盘有相同的数据。

创建数据卷时添加参数replica。

```bash
gluster volume create name replica num host:/path
```

## 条带卷

条带的意思就是将文件的数据进行拆分开来，分成一条一条的，分别存放在不同的Brick。这样的话，如果有一个Brick发生故障，那么这个文件的其余数据就不能拼接成整个文件，文件数据相当于全部丢失。

创建数据卷时添加参数stripe。

```bash
gluster volume create stripe num name host:/path
```

**需要注意的是，这种情况经常被用作大文件存储。**

大文件需要占用的空间是非常大的。通常情况下，服务器的硬盘都是SAS或者是SATA的，读取效率都不是很高，如果将一个大文件完整的放在一个Brick上，那么读取效率会非常低。如果使用条带卷的话，数据被拆分到多个node的Brick上，读取数据的时候相当于做了多机多线程数据读取，效率较高。

其他高级数据存储方式是以上三种方式的混合使用，这样提高了数据的可用性，同时提高了数据效率。

通常情况下，以上三种方式就够用了。
