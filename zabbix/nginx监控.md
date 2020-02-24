nginx通常会留一个api接口，这个接口可以获取到nginx目前的工作状态。通常情况下，一台服务器的连接池资源是有限的，所以我们就需要用nginx做一下性能的监控，防止服务器因为访问量过大导致的崩溃。

# 配置nginx api

在nginx.conf的server块中添加一个location，如下：

```
 location ~ /nginx_status {
                stub_status on;
                access_log off;
}
```

这样就启动了nginx的api，我们可以使用浏览器进行访问查看这个api是否启用成功。

![6.png](.\img\6.png)

如果返回这样的参数就说明启动成功了。

# api参数解析

这个api中列出了nginx服务器的http信息。

Active connections表示当前活动的连接。

第三行的三个51分别表示请求数，被允许的请求数和响应数。

第四行，Reading表示正在读的连接数。Writing表示正在写的连接数，Waiting表示正在等待连接数。

# 创建api解析脚本

在linux中，常用curl获取网页返回值。这里也是这样。我们一般只对Active进行解析，那么，脚本应该写成这样：

```bash
curl localhost/nginx_status 2>/dev/null | grep Active | awk '{print $3}'
```

这样就获取到了活动连接数。

# 创建自定义监控key

上边创建了检测脚本，这个脚本是被zabbix账户执行的，那么我们就需要给这个脚本修改所有者并修改权限：

```bash
[root@localhost zabbix_script]# chmod 777 nginx.sh 
[root@localhost zabbix_script]# chown zabbix:zabbix nginx.sh 
```

创建zabbix监控键。先观察一下zabbix_agent.conf。

有两个参数需要修改

* UnsafeUserParameters=1 允许添加key
* EnableRemoteCommands=1 允许执行命令

观察后边的配置，我们会发现一个配置是这样的：

```
Include=/etc/zabbix/zabbix_agentd.d/*.conf
```

这表示包含了配置文件，那么我们就可以在这个目录下添加相关配置。

创建nginx.conf

```
UserParameter=nginx.active[*],/etc/zabbix/zabbix_script/nginx.sh
```

这样就创建了一个键。这个键里有个*，表示接受所有参数。如果写成参数的话，后边脚本使用参数就使用$

# 创建监控

按前边说过的方法创建监控项。监控项表达式直接写```nginx.active[]```就ok了。

这样就定义了一个nginx监控脚本。

**注意：我们会发现监控返回的值，哪怕当前没有访问也是1，那是因为当前脚本会访问nginx，可以根据需求在shell脚本里减1**