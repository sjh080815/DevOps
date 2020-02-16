MooseFS是一种支持容错的分布式文件系统，将多台文件服务器联系在一起（数据分布），提供统一的接口供给客户端挂载使用。

# 服务拓扑分析

一个MooseFS存储系统内包含了Master，Log和Storage三个角色的主机，这三个角色的主机分别是控制、日志和存储节点。这种系统和其他系统不同点在于该种系统必须有一个专门的服务器记录相关的日志。

# 准备工作

在所有节点安装依赖：

```bash
yum -y install zlib-devel gcc-c++ lrzsz
```

创建mfs账户：

```bash
useradd mfs -s /sbin/nologin
```

下载并解压源码文件，进入目录：

```bash
wget https://github.com/moosefs/moosefs/archive/v3.0.101.tar.gz
tar -xf v3.0.101.tar.gz && cd moosefs-3.0.101/
```

进行编译安装：

```bash
./configure --prefix=/usr/local/mfs --with-default-user=mfs --with-default-group=mfs --disable-mfschunkserver --disable-mfsmount
make && make install
```

# Master机

将配置文件拷贝到```/usr/local/mfs/etc/mfs/```下

```bash
cp /usr/local/mfs/etc/mfs/mfsmaster.cfg.sample /usr/local/mfs/etc/mfs/mfsmaster.cfg
cp /usr/local/mfs/etc/mfs/mfsexports.cfg.sample /usr/local/mfs/etc/mfs/mfsexports.cfg
cp /usr/local/mfs/etc/mfs/mfstopology.cfg.sample /usr/local/mfs/etc/mfs/mfstopology.cfg
cp /usr/local/mfs/var/mfs/metadata.mfs.empty /usr/local/mfs/var/mfs/metadata.mfs
```

这样的配置文件是默认的，可以被所有主机访问，也就是可以识别到所有主机。这样的话对于Master节点可能会造成一定的威胁。那么我们就需要对他进行授权管理。

对当前网段进行授权限制。

在mfsexports.cfg中添加字段：

```bash
10.20.0.0/24 / rw,alldir/maproot=0
```

启动Master服务：

```bash
/usr/local/mfs/sbin/mfsmaster start
```

通常情况下，Master是主节点，对全局进行控制，那么就需要一个控制平台。通常使用自带的dashbroad。

```bash
/usr/local/mfs/sbin/mfscgiserv
```

# Log机

拷贝配置文件：

```bash
cp /usr/local/mfs/etc/mfs/mfsmetalogger.cfg.sample  /usr/local/mfs/etc/mfs/mfsmetalogger.cfg
```

在```/usr/local/mfs/etc/mfs/mfsmetalogger.cfg```中添加字段```MASTER_HOST=```

启动服务

```bash
/usr/local/mfs/sbin/mfsmetalogger start
```

# Storage机

拷贝配置文件：

```bash
cp /usr/local/mfs/etc/mfs/mfschunkserver.cfg.sample /usr/local/mfs/etc/mfs/mfschunkserver.cfg
cp /usr/local/mfs/etc/mfs/mfshdd.cfg.sample /usr/local/mfs/etc/mfs/mfshdd.cfg
```

一般情况储存机是使用的多磁盘，这里对磁盘进行格式化和挂载：

```bash
mkfs.xfs -i size=512 /dev/sdb
mkdir /mfs
mount /dev/sdb /mfs/
```

修改配置文件：

* 在mfschunkserver.cfg中添加MASTER_HOST字段。
* 在mfshdd.cfg中写刚才挂载的目录。

启动服务

```bash
/usr/local/mfs/sbin/mfschunkserver start
```

这样，MooseFS的储存架构完成了。

# 客户端挂载

该种文件系统的挂载使用和普通的文件系统不同，需要用mfs专用的挂在工具。那么我们就同样需要安装。

创建一个目录作为挂载点，挂载点使用```/usr/local/mfs/bin/mfsmount```进行挂载。

