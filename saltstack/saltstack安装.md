# Saltstack安装

## 安装

Saltstack使用python3开发，所以我们首先需要安装python3环境。

一般情况下，Centos会安装python2，因为yum是由python2开发的。

### 更新python

安装依赖

```bash
yum install -y epel-release
```
安装python3.7

```bash
yum install -y python36 python36-devel python36-libs
```

安装号侯，python的安装地址为```bash /usr/bin/python3 ```,添加软连接

```bash
ln -sf /usr/bin/python3 /usr/bin/python
```

在/usr/bin/yum和/usr/libexec/urlgrabber-ext-down的第一行将```bash #! /usr/bin/python ```改成```bash #! /usr/bin/python2 ```,运行yum测试是否正常。

### 安装Saltstack

分别在主从机上添加RPM

```bash
sudo yum install https://repo.saltstack.com/py3/redhat/salt-py3-repo-latest.el7.noarch.rpm
```

设置主从机的hostname

```bash
hostnamectl set-hostname saltstack 服务端
hostnamectl set-hostname client1     客户端
```

关闭firewalld和SELinux

分别编辑主从机的```bash /etc/hosts ```文件，添加主从机的解析

```bash
vim /etc/hosts
192.168.1.131  saltstack
192.168.1.137  client1
```

在主机安装salt-master和salt-minion软件，在从机安装salt-minion。

```bash
服务端安装
yum -y install salt-master salt-minion

客户端安装
yum -y install salt-minion
```

#### 主机配置

在主机上进行配置，让主机的minion识别主机。

```bash
vi /etc/salt/minion

master: saltstack
id: saltstack
```
在从机上设置，让从机的minion识别主机。

```bash
vi /etc/salt/minion

master: saltstack    
id： client1
```

上述的配置中可以将master的hostname直接换成主机的IP，但是携程hostname更易于管理。

到这里已完成了Saltstack的安装。

## 测试Saltstack

首先看一下Saltstack的原理：

> minion上线后先与master端联系，把自己的pub key发过去，这时master端通过salt-key -L命令就会看到minion的key，接受该minion-key后，也就是master与minion已经互信

由此可得，在minion启动时，会将pub key发送给master，我们可以在主机上通过```bash salt-key -L```查看发来的pub key。

```bash
salt-key -L

Accepted Keys:
Denied Keys:
Unacceptrd Keys:
salstack
client1
Rejected Keys:
```
说明master和从机的minion都启动成功了，我们可以通过```bash salt-key -A```对搜友发来的pub key进行授权。至此Saltstack的安装和测试过程完成。




