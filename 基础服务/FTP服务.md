# FTP介绍

在生产环境的架设中，常常会有开发部门上传一些页面文件到服务器。有些开发的程序员可能Linux基础不是特别好，所以咱们就需要给他们留一个传输文件的接口。通常使用sftp和ftp协议。

# 安装

需要的软件：vsftpd

安装方法：sudo apt install vsftpd

启动方法：systemctl start vsftpd

# 配置文件解析

乌班图下，配置文件在/etc/vsftpd.conf

配置功能表：


| 参数 | 功能 |
|----------|----------|
| listen | 监听 |
|anonymous_enable|匿名登录|
|anon_root|匿名登陆目录|
|local_enable|允许本地用户登录|

# 添加用户和用户目录

可以用userlist和本地账户的方法进行配置。这里说本地账户的方法。

在本地创建一下账户。

```bash 
useradd username -s user_dir
```

这里的-s参数是指定用户目录。如果要给用户添加指定目录，则加参数-d。

创建完用户后，设置密码

```bash
passwd user
```

连续输入两次密码以后则创建成功，可以用ftp工具使用FTP服务。