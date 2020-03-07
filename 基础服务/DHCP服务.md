> 在家用的路由器上，一般都会集成DHCP这个功能。DHCP服务，负责给网络内的主机分配IP和netmask，更便于主机的联网操作。

既然咱们要搭建DHCP服务，那么，咱们就不能让网络内有相关的服务存在，原因如下：

1. 主机不知道该向谁获取IP地址

2. DHCP服务器会发生冲突

DHCP服务，常用在大型网络环境下（如果三四台主机，那就没必要了，手动一下就行了）。但是切记，**在服务器网络不要用DHCP服务，因为DHCP是由租约期的，租约期到会重新分配IP，有可能发生IP改变。服务器需要稳定的IP支持，才能保证用户的正常访问，所以为了避免服务突然没法访问。DHCP一般不会用到这里。**

# 服务端配置

配置DHCP的前提就是网络环境内没有这个配置，所以服务端就没有分配IP，就不会有网，所以咱们先设置一下静态IP。

一般情况下，网卡配置文件在```/etc/sysconfig/network-scripts```,咱们编辑一下这个文件：

```
TYPE="Ethernet"
PROXY_METHOD="none"
BROWSER_ONLY="no"
BOOTPROTO="static" #IP获取方式
DEFROUTE="yes"
IPV4_FAILURE_FATAL="no"
IPV6INIT="yes"
IPV6_AUTOCONF="yes"
IPV6_DEFROUTE="yes"
IPV6_FAILURE_FATAL="no"
IPV6_ADDR_GEN_MODE="stable-privacy"
NAME="ens33"
UUID="46a79d3e-fc47-4372-a2d5-359d2e0545dd"
DEVICE="ens33"
ONBOOT="yes"
IPADDR=192.168.139.152 #本机IP
GATEWAY=192.168.139.2 #网关
NETMASK=255.255.255.0 #子网掩码
DNS1=114.114.114.114 #DNS服务器，记得一定要添加

```

设置好侯，我们重新启动一下网络

```bash
systemctl restart network
```

咱们再查看一下IP

```bash
bash# ip add
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:0c:29:7d:97:dc brd ff:ff:ff:ff:ff:ff
    inet 192.168.139.152/24 brd 192.168.139.255 scope global noprefixroute ens33
       valid_lft forever preferred_lft forever
    inet6 fe80::1811:571e:bbd8:4a30/64 scope link noprefixroute
       valid_lft forever preferred_lft forever
```

IP被绑定了，也可能没网，所以咱们ping一下外网网站。

网络一切准备就绪，开始安装服务。

在服务端，直接进行yum安装

```bash
yum install dhcp
```

dhcpd软件的配置文件在/etc/dhcp/dhcpd.conf，且默认是没有任何参数的。dhcpd在安装时为用户生成了配置的模板文件，咱们拷贝过来。

```bash
cp /usr/share/doc/dhcp*/dhcpd.conf.example /etc/dhcp/dhcpd.conf
```

开始编辑必要参数。参数的功能表如下：


| 字段 | 功能 |
|----------|----------|
| default-lease-time | 默认租约时间 |
|max-lease-time|最大租约时间|
|log-facility|日志|

以上是这个软件的最简配置参数，下来看一下我的配置文件：

```


default-lease-time 600;
max-lease-time 7200;
log-facility local7;

subnet 192.168.139.0 netmask 255.255.255.0 {
        range  192.168.139.100  192.168.139.200;
}

```

这里解释一下：

subnet是网段，就是根据netmask解析的网络IP号

netmask是子网掩码

range是IP分配的取值域

OK，可以运行服务了

```bash
systemctl start dhcpd
```

# 客户端

一般情况下只需要将获取IP方式改成DHCP就好了。

# 查看租约

DHCP的租约时间被保存在```/var/lib/dhcpd/dhcpd.leases```，查看文件。

```
# The format of this file is documented in the dhcpd.leases(5) manual page.
# This lease file was written by isc-dhcp-4.2.5

server-duid "\000\001\000\001%vq\346\000\014)}\227\334";

lease 192.168.139.131 {
  starts 0 2019/12/01 12:46:21;
  ends 0 2019/12/01 12:56:21;
  cltt 0 2019/12/01 12:46:21;
  binding state active;
  next binding state free;
  rewind binding state free;
  hardware ethernet 00:0c:29:46:2c:87;
  uid "\001\000\014)F,\207";
}
lease 192.168.139.131 {
  starts 0 2019/12/01 12:51:20;
  ends 0 2019/12/01 13:01:20;
  cltt 0 2019/12/01 12:51:20;
  binding state active;
  next binding state free;
  rewind binding state free;
  hardware ethernet 00:0c:29:46:2c:87;
  uid "\001\000\014)F,\207";
}
lease 192.168.139.131 {
  starts 0 2019/12/01 12:56:19;
  ends 0 2019/12/01 13:06:19;
  cltt 0 2019/12/01 12:56:19;
  binding state active;
  next binding state free;
  rewind binding state free;
  hardware ethernet 00:0c:29:46:2c:87;
  uid "\001\000\014)F,\207";
}
lease 192.168.139.131 {
  starts 0 2019/12/01 13:01:18;
  ends 0 2019/12/01 13:11:18;
  cltt 0 2019/12/01 13:01:18;
  binding state active;
  next binding state free;
  rewind binding state free;
  hardware ethernet 00:0c:29:46:2c:87;
  uid "\001\000\014)F,\207";
}
lease 192.168.139.131 {
  starts 0 2019/12/01 13:06:18;
  ends 0 2019/12/01 13:16:18;
  cltt 0 2019/12/01 13:06:18;
  binding state active;
  next binding state free;
  rewind binding state free;
  hardware ethernet 00:0c:29:46:2c:87;
  uid "\001\000\014)F,\207";
}
lease 192.168.139.131 {
  starts 0 2019/12/01 13:11:18;
  ends 0 2019/12/01 13:21:18;
  cltt 0 2019/12/01 13:11:18;
  binding state active;
  next binding state free;
  rewind binding state free;
  hardware ethernet 00:0c:29:46:2c:87;
  uid "\001\000\014)F,\207";
}
lease 192.168.139.131 {
  starts 0 2019/12/01 13:16:18;
  ends 0 2019/12/01 13:26:18;
  cltt 0 2019/12/01 13:16:18;
  binding state active;
  next binding state free;
  rewind binding state free;
  hardware ethernet 00:0c:29:46:2c:87;
  uid "\001\000\014)F,\207";
}
lease 192.168.139.131 {
  starts 0 2019/12/01 13:21:17;
  ends 0 2019/12/01 13:31:17;
  cltt 0 2019/12/01 13:21:17;
  binding state active;
  next binding state free;
  rewind binding state free;
  hardware ethernet 00:0c:29:46:2c:87;
  uid "\001\000\014)F,\207";
}
```

可以看出客户端多次向服务端进行注册，且能看出租约的时间。