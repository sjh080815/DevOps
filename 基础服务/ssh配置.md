SSH服务是Linux的远程命令行服务。一般情况下，Linux系统默认会安装这个服务，我们只需要修改相关的配置。

# 多网卡设置

服务器通常不止有一个网卡，每个网卡的业务不同，所以就需要指定一个网卡做ssh服务。

在/etc/ssh/下，修改sshd_config服务配置文件。

添加ListenAddress，绑定到网卡的IP上。

# 业务端口绑定

在上述文件中添加Port参数。

在默认情况下这个文件这个文件大部分是被注释掉的，所以我们需要手动进行配置。

# 免密登录

免密登录就是使用密钥进行登录。

首先创建一下public_key
```bash
[root@localhost ssh]# ssh-keygen 
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa): 
Created directory '/root/.ssh'.
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /root/.ssh/id_rsa.
Your public key has been saved in /root/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:63gG1/fyrxc7qJBNKAZ1QPUAgqB7fMX9nr85reoPt8I root@localhost.localdomain
The key's randomart image is:
+---[RSA 2048]----+
| .. ...=++       |
|.  . .o.. o      |
|.    .o .  .     |
| o   ..  ..      |
|. o .  oSo..     |
| . .  o oo=..  . |
|       o.++o.o. o|
|       oo E+o+oo.|
|      .o..o=O*o+o|
+----[SHA256]-----+
```

将密钥分发到要连接的服务器上：

```bash
[root@localhost ssh]# ssh-copy-id root@0.0.0.0
```

这样就直接使用ssh连接到服务器了。

# SCP协议

这里会经常使用SCP协议，传输文件数据。使用的通道也是SSH的通道。使用

```bash
scp file root@0.0.0.0:path
```

这样就将数据传输到指定的服务器上。

# 生产环境案例

通常情况下，生产环境是会禁用掉远程ssh连接root账户的（处于安全考虑），既是使用root远程ssh也会使用跳板机。这里说一下ssh的配置。

防止被扫描端口后进行端口直接攻击，需要改变ssh的端口号：

```
Port 222
```

限制连接数和密码尝试次数
```
MaxAuthTries 3 #最大尝试次数
MaxSessions 5 #最大连接数
```

禁用ssh远程登录root
```
PermitRootLogin no
```

这样就基本保了服务器的ssh安全。

# ssh同步

在某些情况下，需要进行ssh协同工作，有时候需要一起在一个终端上进行操作，那么这里就用到了screen工具。

使用yum安装。

创建一个screen

```bash
screen -S linux
```

在另一个终端直接使用screen -x就可以直接接入这个共享的连接上了。