> 在大型生产环境中，往往都有较高的并发量，一个数据库服务器顶不下来，一般的解决方案就是做一个数据集群。在数据集群中，所有数据库的数据要保持实时同步（总不能同一时间访问数据库，得到两个不同的数据吧）。而且，数据库在读和写的时候，用的资源不一样多，就需要给读和写做一下分离（读的资源需求较小，写的资源需求较大，所以常把读的服务器给的性能可以比写的服务器低一点）。

因为服务器有限，所以我在docker上进行了环境的部署。

# 数据库配置

运行mysql容器，运行两次创建容器代码，创建两个mysql容器

```bash
docker run -itd -e MYSQL_ROOT_PASSWORD=123 mysql:5.6
```

创建运行的mysql-proxy容器，用centos为基础镜像,容器更新yum

```bash
docker run -itd centos yum update
```


查看当前运行的容器

```bash
bash# docker ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
c9459abb3850        centos              "/bin/bash"              39 minutes ago      Up 39 minutes                           loving_wilson
a0f594738d01        e143ed325782        "docker-entrypoint..."   39 minutes ago      Up 30 minutes       3306/tcp            cranky_albattani
6b3d770b701c        e143ed325782        "docker-entrypoint..."   39 minutes ago      Up 30 minutes       3306/tcp            vigilant_bassi
```

容器都运行起来了。

需要注意，在生产环境中容器一般不会用默认名命名，那就在创建容器时添加参数--name= 以定义容器的名称。

**注意：docker运行的主从数据库的UUID默认是相同的，首先咱们先修改一下两个容器MySQL的UUID进行修改**

**为了方便，咱们运行三个容器之后咱们先给容器分别安装wget，vim等软件**

## 主库的配置

进入主库容器

```bash
docker exec -it 6b3d770b701c bash
```

修改容器内mysql的配置，修改mysql的日志和id。配置文件为/etc/mysql/mysql.conf.d/mysqld.cnf，在结尾添加两行

```bash
log-bin=mysql-bin
server-id=1
```

然后退出容器，重启容器。

查看一下容器的IP

```bash
bash# docker inspect 6b3d770b701c

...
  "IPAddress": "172.17.0.2",
...
```

在容器的mysql-cli中查看一下数据库的配置

```bash
mysql> show master status;
+------------------+----------+--------------+------------------+-------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+------------------+----------+--------------+------------------+-------------------+
| mysql-bin.000001 |      120 |              |                  |                   |
+------------------+----------+--------------+------------------+-------------------+
1 row in set (0.00 sec)
```

## 从库的配置

进入从库容器

进行和主库容器相同的配置，但是id必须唯一。

在从库运行mysql-cli

```bash
docker exec -it a0f594738d01 mysql -uroot -p123456
```

配置从库的配置：

```bash
mysql> change master to master_host='172.17.0.2',master_user='root',master_password='123456',master_log_file='mysql-bin.000001',master_log_pos=120;
```

运行slave

```bash
mysql> start slave
```

查看一下从库运行状态

```bash
mysql> show slave status\G;

*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 172.17.0.2
                  Master_User: root
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000001
          Read_Master_Log_Pos: 302
               Relay_Log_File: mysqld-relay-bin.000002
                Relay_Log_Pos: 465
        Relay_Master_Log_File: mysql-bin.000001
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB:
          Replicate_Ignore_DB:
           Replicate_Do_Table:
       Replicate_Ignore_Table:
      Replicate_Wild_Do_Table:
  Replicate_Wild_Ignore_Table:
                   Last_Errno: 0
                   Last_Error:
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 302
              Relay_Log_Space: 639
              Until_Condition: None
               Until_Log_File:
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File:
           Master_SSL_CA_Path:
              Master_SSL_Cert:
            Master_SSL_Cipher:
               Master_SSL_Key:
        Seconds_Behind_Master: 0
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error:
               Last_SQL_Errno: 0
               Last_SQL_Error:
  Replicate_Ignore_Server_Ids:
             Master_Server_Id: 1
                  Master_UUID: f733123f-0f8b-11ea-8d91-0242ac110002
             Master_Info_File: /var/lib/mysql/master.info
                    SQL_Delay: 0
          SQL_Remaining_Delay: NULL
      Slave_SQL_Running_State: Slave has read all relay log; waiting for the slave I/O thread to update it
           Master_Retry_Count: 86400
                  Master_Bind:
      Last_IO_Error_Timestamp:
     Last_SQL_Error_Timestamp:
               Master_SSL_Crl:
           Master_SSL_Crlpath:
           Retrieved_Gtid_Set:
            Executed_Gtid_Set:
                Auto_Position: 0
1 row in set (0.00 sec)
```

上边显示两个Yes就说明成功了，咱们可以在主库写数据试试。

**在复杂的生产环境中，咱们一般会运行多个数据库做数据集群，一般都是多个主库。多主库之间是互相主从复制，从库复制主库数据。**

# 读写分离

进入mysql-proxy容器，进行安装。（为了方便，我就不用dockerfile生成镜像了）

在容器中下载mysql-proxy的文件

```bash
bash# get https://cdn.mysql.com/archives/mysql-proxy/mysql-proxy-0.8.5-linux-glibc2.3-x86-64bit.tar.gz
bash# tar zxvf mysql-proxy-0.8.5-linux-glibc2.3-x86-64bit.tar.gz
bash# mv mysql-proxy-0.8.5-linux-glibc2.3-x86-64bit /usr/local/mysql-proxy
bash# cd /usr/local/mysql-proxy
bash# mkdir lua
bash# cp share/doc/mysql-proxy/rw-splitting.lua ./lua #复制读写分离配置文件
bash# cp share/doc/mysql-proxy/admin-sql.lua ./lua #复制管理脚本
bash# vi /etc/mysql-proxy.cnf   #创建配置文件
```

在配置文件中写入配置

```bash
[mysql-proxy]
user=root #运行mysql-proxy用户
admin-username=root #主从mysql共有的用户
admin-password=123456 #用户的密码
proxy-address=0.0.0.0：3306 #mysql-proxy运行ip和端口，不加端口，默认4040
proxy-read-only-backend-addresses=172.17.0.3 #指定后端从slave读取数据
proxy-backend-addresses=172.17.0.2 #指定后端主master写入数据
proxy-lua-script=/usr/local/mysql-proxy/lua/rw-splitting.lua #指定读写分离配置文件位置
admin-lua-script=/usr/local/mysql-proxy/lua/admin-sql.lua #指定管理脚本
log-file=/usr/local/mysql-proxy/logs/mysql-proxy.log #日志位置
log-level=info #定义log日志级别，由高到低分别有(error|warning|info|message|debug)
daemon=true    #以守护进程方式运行
keepalive=true #mysql-proxy崩溃时，尝试重启
```

设置一下独写分离的条件。在/usr/local/mysql-proxy/lua/rw-splitting.lua，修改
```bash
min_idle_connections = 1, 
max_idle_connections = 1,
```


记得把后边的注释删掉，否则会发生错误。

给配置文件加权限，设置成660就可以了。

好了，咱们运行一下代理软件

```bash
/usr/local/mysql-proxy/bin/mysql-proxy --defaults-file=/etc/mysql-proxy.cnf
```

运行时候可能会出现日志不能访问的报错，咱们只需要手动创建一下日志文件就好了。

不出意外的话，咱们的环境就做好了，可以进行连接试试。这里我就不说了。

另外说一下，docker运行的mysql默认是可以让其他容器访问的，咱们不用单独进行设置。


