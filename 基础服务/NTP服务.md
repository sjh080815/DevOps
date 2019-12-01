> 通常情况下，服务器的授时服务都是连接至网络上的授时服务器的。但是有的时候，服务器进行异地容灾，特别在国外的时候，需要生成日志，日志时间要求是主服务的时间基准，那么就需要架设NTP服务进行授时了。

# 服务端安装

安装必要的软件

```bash 
yum install ntp tzdata
```

下来进行文件配置。ntp的配置文件只有一个，/etc/ntp.conf。我们看一下这个文件

```
# For more information about this file, see the man pages
# ntp.conf(5), ntp_acc(5), ntp_auth(5), ntp_clock(5), ntp_misc(5), ntp_mon(5).

driftfile /var/lib/ntp/drift

# Permit time synchronization with our time source, but do not
# permit the source to query or modify the service on this system.
restrict default nomodify notrap nopeer noquery

# Permit all access over the loopback interface.  This could
# be tightened as well, but to do so would effect some of
# the administrative functions.
restrict 127.0.0.1
restrict ::1

# Hosts on local network are less restricted.
restrict 192.168.139.0 mask 255.255.255.0 nomodify notrap

# Use public servers from the pool.ntp.org project.
# Please consider joining the pool (http://www.pool.ntp.org/join.html).
server 0.centos.pool.ntp.org iburst
server 1.centos.pool.ntp.org iburst
server 2.centos.pool.ntp.org iburst
server 3.centos.pool.ntp.org iburst

#broadcast 192.168.1.255 autokey        # broadcast server
#broadcastclient                        # broadcast client
#broadcast 224.0.1.1 autokey            # multicast server
#multicastclient 224.0.1.1              # multicast client
#manycastserver 239.255.254.254         # manycast server
#manycastclient 239.255.254.254 autokey # manycast client

# Enable public key cryptography.
#crypto

includefile /etc/ntp/crypto/pw

# Key file containing the keys and key identifiers used when operating
# with symmetric key cryptography.
keys /etc/ntp/keys

# Specify the key identifiers which are trusted.
#trustedkey 4 8 42
# Specify the key identifier to use with the ntpdc utility.
#requestkey 8

# Specify the key identifier to use with the ntpq utility.
#controlkey 8

# Enable writing of statistics records.
#statistics clockstats cryptostats loopstats peerstats

# Disable the monitoring facility to prevent amplification attacks using ntpdc
# monlist command when default restrict does not include the noquery flag. See
# CVE-2013-5211 for more details.
# Note: Monitoring will not be disabled with the limited restriction flag.
disable monitor
```

咱们只需要配置一下授权IP就好。授权IP参数是restrict字段。

观察这个配置文件，发现里边有许多server。其实NTP服务也是依托与网络NTP服务，只是让服务端获取时间，然后将时间分发给服务器。

**注意：没有绝对准确的时钟，目前相对准确的时钟是美国NIST锶原子时钟，咱们可以把上边的server换成这个授时源。**

启动服务

```bash
systemctl start ntpd
```

服务端设置完成。

# 客户端使用

在客户端使用ntpdate就可以访问到ntp授时。

```bash
bash# ntpdate 192.168.139.152
 1 Dec 22:35:56 ntpdate[1511]: the NTP socket is in use, exiting
 ```

我是在服务端运行的，所以没有进行时间调整。在其他服务器上运行这个命令时，会返回服务器调整的时间。

**补充知识：服务器和PC机一样，平时都是使用BIOS上的时间，偶尔向NTP服务器获取准确时间以校正本地时间，所以我们获得授时之后最好将时间写入BIOS。**