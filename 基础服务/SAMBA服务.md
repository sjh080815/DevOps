> SAMBA服务可以搭建网上邻居服务器，可共享一些硬件设备。

# 服务端安装

安装必要软件

```bash
yum install samba samba-client samba-common
```

配置一下samba的配置信息，位置/etc/samba/smb.conf

```
[global]
        workgroup = SAMBA
        security = user

        passdb backend = tdbsam

        printing = cups
        printcap name = cups
        load printers = yes
        cups options = raw
[print$]
        comment = Printer Drivers
        path = /var/lib/samba/drivers
        write list = @printadmin root
        force group = @printadmin
        create mask = 0664
        directory mask = 0775
```

# global全局配置模块

workgroup：工作组

log file：日志文件，文件名可用变量。

max log size：最大日志大小。

security：

* share：无密码分享
* user：使用服务器本地的账户密码
* domain：使用外部密码数据库

passdb backend：密码数据库格式。

# 共享文件模块

comment：目录说明。

path：共享目录位置。

browseable：可浏览。

writeable：可写。

# 无密码共享

在samba 4.2.0 版本之后不再兼容3.0 的版本， 该参数无法兼容security=share，所以无法搭建无密码共享。

# 需密码共享

security写user参数，认证密码是本地密码，

在配置文件创建共享目录，用```testparm -v```测试配置文件。没有问题之后启动服务。

进行本地测试,校验本地配置文件正确性。

## 添加用户

首先添加本地账户和密码
```bash
useradd -G users smb1
echo 123456 | passwd --stdin smb1
```

添加samba账户

```bash
pdbedit -a -u smb1
```
会要求设置密码，密码是登录samba的密码。

创建号后重启服务，用本地测试一下连接。

```
bash# smbclient -L //127.0.0.1  -U smb1
Enter SAMBA\smb1's password:

        Sharename       Type      Comment
        ---------       ----      -------
        temp            Disk      test
        IPC$            IPC       IPC Service (Samba 4.9.1)
Reconnecting with SMB1 for workgroup listing.

        Server               Comment
        ---------            -------

        Workgroup            Master
        ---------            -------
```

服务端配置完成。

