Apache是一种很常见的静态web服务器，常常被用作静态页面服务器。

# 安装

一般生产环境中都使用编译安装或者二进制安装，这里使用编译安装。

首先安装一下依赖环境：

```bash
yum groupinstall -y "Development Tools" "Development Libraries"
```

下载需要编译安装的依赖（在官网上都有，下载安装即可）

下载httpd安装包，进行解压安装：

```bash
wget http://mirror.bit.edu.cn/apache//httpd/httpd-2.4.41.tar.gz
tar -zxvf httpd-2.4.41.tar.gz
cd httpd-2.4.41
./configure
make && make install
```

这样就安装完成了。

# 第一次启动

Apache第一次启动需要设置一些参数，否则会报错。

在httpd.conf中设置ServerName，这是设置域名解析的字段。

启动服务：

```bash
/usr/local/apache2/bin/httpd
```

# 发布静态页面

从配置文件可以看出静态文件的地址是/usr/local/apache2/htdocs，那么我们可以进这个目录去上传html静态页面。