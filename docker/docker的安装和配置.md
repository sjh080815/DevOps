> Docker是目前容器化的常用软件，它可以隔离真是主机环境，且上线环境部署容易，容灾性高。

# 安装环境

docker可以通过yum进行安装，安装方法
```
yum install docker-ce
```

安装后，咱们最好更改一下docker的加速源，修改```/etc/docker/daemon.json```，添加以下内容：

```
{
"registry-mirrors": ["http://hub-mirror.c.163.com"]
}
```

重启以下docker服务就行了。