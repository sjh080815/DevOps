ansible是一个基于SSH协议的自动化运维工具，通常使用的时候需要给其他主机配置免密登录，且只需要安装控制端。

# 安装

在控制端安装依赖及主程序

```
yum install epel-release
yum install ansible
```

也可以使用pip安装

```
pip install ansible
```

安装好后进行主机配置和远程配置。

在控制端配置/etc/ansible/hosts文件，在这个文件里可以使用ip，也可以使用hostname（那就需要使用hosts文件对主机进行解析或者DNS能进行解析）。


![1.png](.\img\1.png)

生成SSH公钥，将公钥信息下发给被控制主机。

```
ssh-keygen -t rsa
```

可以看到，生成的rsa公钥被存放在~/.ssh/id_rsa.pub中，那么久需要将这个文件下发：

```
ssh-copy-id -i ~/.ssh/id_rsa.pub root@192.168.89.129
```

这样就配置好了SSH免密登录。

对配置好的集群进行验证：

```shell
bash# ansible all -m ping
192.168.89.130 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "ping": "pong"
}
192.168.89.129 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "ping": "pong"
}
```

这条命令和saltstack的test.ping的用途相同，就是尝试进行服务器间的通信测试。这里的两条success表示测试正常。
