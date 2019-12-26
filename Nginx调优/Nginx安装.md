安装Nginx的依赖库

```bash
yum install gcc gcc-c++ automake pcre pcre-devel zlib zlib-devel open open-ssl
```

这里的依赖库是Nginx的所有依赖，有部分是非必要的，所以可以进行部分安装，但是openssl、gcc和gcc-c++都是必要的。

下载Nginx安装包：

```bash
wget http://nginx.org/download/nginx-1.14.2.tar.gz
```

下载完成后解压文件。解压完成后进入目录进行编译。

生成配置：

```bash
./configure --prefix=/Nginx
```

这里生成配置是将Nginx编译安装到/Nginx的目录下。

生成完成后开始编译安装

```bash
make && make install
```

安装完成，看一下Nginx的文件系统结构

```bash
[root@master Nginx]# ll
总用量 4
drwxr-xr-x. 2 root root 4096 12月 26 10:09 conf
drwxr-xr-x. 2 root root   40 12月 26 10:09 html
drwxr-xr-x. 2 root root    6 12月 26 10:09 logs
drwxr-xr-x. 2 root root   19 12月 26 10:09 sbin
```

* conf目录：存放Nginx的配置文件。
* html目录：静态文件目录。
* logs目录：日志文件目录。
* sbin目录：软件目录。

启动Nginx：进入sbin目录，执行./nginx启动nginx。

访问Nginx服务器的IP，看到如下返回：

![](/img/QQ浏览器截图20191226101744.png)