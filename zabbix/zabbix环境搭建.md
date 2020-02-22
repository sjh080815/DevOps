zabbix是一种常用的监控平台。监控在集群中是很有必要的，监控的存在能及时通知相关的工程师进行技术处理。

zabbix分为三部分。第一部分是server，第二部分是web，第三部分是agent。分别是系统中的服务端，dashbroad和客户端，这些部分都是缺一不可的。

# 部署服务端

首先在服务器上进行yum配置，添加阿里云的镜像源：

```bash
[root@localhost yum.repos.d]# cat zabbix.repo 

[zabbix]
name=Zabbix Official Repository - \$basearch
baseurl=https://mirrors.aliyun.com/zabbix/zabbix/4.0/rhel/7/\$basearch/
enabled=1
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-ZABBIX-A14FE591
 
[zabbix-non-supported]
name=Zabbix Official Repository non-supported - \$basearch 
baseurl=https://mirrors.aliyun.com/zabbix/non-supported/rhel/7/\$basearch/
enabled=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-ZABBIX
gpgcheck=1
```

添加yum源的公钥：

```bash
bash# curl https://mirrors.aliyun.com/zabbix/RPM-GPG-KEY-ZABBIX-A14FE591 -o /etc/pki/rpm-gpg/RPM-GPG-KEY-ZABBIX-A14FE591
bash# curl https://mirrors.aliyun.com/zabbix/RPM-GPG-KEY-ZABBIX -o /etc/pki/rpm-gpg/RPM-GPG-KEY-ZABBIX
```

通过域名进行安装：

```bash
bash# yum install -y zabbix-server-mysql zabbix-web-mysql
```

这里一般情况下是没有安装mysql数据库的，我们需要手动进行安装。注意，咱们不要安装mysql8数据库，mysql8数据库的数据库引擎发生了改变，不能通用。不建议使用mysql，可以使用mariadb数据库，这是目前数据库最多也是最优的解决方案。

```bash
bash# yum install mariadb mariadb-server mariadb-devel
```

启动数据库并进行数据库初始化：

```bash
bash# systemctl start mariadb
bash# mysql_secure_installation
```

创建数据库，并创建用户授权给用户：

```bash
mysql> create database zabbix character set utf8 collate utf8_bin;
mysql> grant all privileges on zabbix.* to 'zabbix'@'localhost' identified by 'zabbix';
mysql> flush privileges;
```

导入数据库：

```bash
bash# zcat /usr/share/doc/zabbix-server-mysql-3.0.12/create.sql.gz | mysql zabbix -uzabbix -pzabbix
```

修改相关配置：

```bash
bash# vi zcat /usr/share/doc/zabbix-server-mysql-3.0.12/create.sql.gz | mysql zabbix -uzabbix -pzabbix

DBHost=localhost
DBName=zabbix
DBUser=zabbix
DBPassword=zabbix

bash# /etc/httpd/conf.d/zabbix.conf

php_value date.timezone Asia/Shanghai
```

服务安装完成了，启动相关服务：

```bash
bash# systemctl start httpd
bash# systemctl enable httpd
bash# systemctl start zabbix-server
bash# systemctl enable zabbix-server
```

使用浏览器访问主机的ip，直接进行配置。和普通的cms配置方法相同，这里就不读赘述了。

# 安装监控端

在被监控节点上安装agent：

```bash
yum install zabbix-agent
```

安装完成后配置相关的参数：

```bash
bash# vi /etc/zabbix/zabbix_agentd.conf

Server=192.168.89.150
ServerActive=192.168.89.150
Hostname=node1
```

Hostname这个参数是必须有的，在整个集群内需要唯一，否则服务端那边注册不了数据。

在服务端手动添加一下监控端：


![1.png](.\img\1.png)

这里需要注意，这个主机名称要和刚才设置的Hostname相同。

在监控端启动服务：

```bash
bash# systemctl start zabbix-agent
```

我们可以使用zabbix-get进行一下集群内主机的测试：

```bash
[root@localhost yum.repos.d]# zabbix_get -s 192.168.89.151 -p 10050 -k "agent.hostname"
node1
```

这就是说明主机安装好了。

**注意：这里没有添加监控项，ZBX的符号是灰色的，并不是没有安装好。**