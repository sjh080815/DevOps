查看Nginx的配置目录，发现有很多东西，如下：

```bash
[root@master conf]# ll
总用量 68
-rw-r--r--. 1 root root 1077 12月 26 10:09 fastcgi.conf
-rw-r--r--. 1 root root 1077 12月 26 10:09 fastcgi.conf.default
-rw-r--r--. 1 root root 1007 12月 26 10:09 fastcgi_params
-rw-r--r--. 1 root root 1007 12月 26 10:09 fastcgi_params.default
-rw-r--r--. 1 root root 2837 12月 26 10:09 koi-utf
-rw-r--r--. 1 root root 2223 12月 26 10:09 koi-win
-rw-r--r--. 1 root root 5170 12月 26 10:09 mime.types
-rw-r--r--. 1 root root 5170 12月 26 10:09 mime.types.default
-rw-r--r--. 1 root root 2656 12月 26 10:09 nginx.conf
-rw-r--r--. 1 root root 2656 12月 26 10:09 nginx.conf.default
-rw-r--r--. 1 root root  636 12月 26 10:09 scgi_params
-rw-r--r--. 1 root root  636 12月 26 10:09 scgi_params.default
-rw-r--r--. 1 root root  664 12月 26 10:09 uwsgi_params
-rw-r--r--. 1 root root  664 12月 26 10:09 uwsgi_params.default
-rw-r--r--. 1 root root 3610 12月 26 10:09 win-utf
```

有一些配置文件是暂时不需要动的。我们主要配置的文件是Nginx.conf

```
#user  nobody;
worker_processes  1;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       mime.types;
    default_type  application/octet-stream;

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    #gzip  on;

    server {
        listen       80;
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            root   html;
            index  index.html index.htm;
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

        # proxy the PHP scripts to Apache listening on 127.0.0.1:80
        #
        #location ~ \.php$ {
        #    proxy_pass   http://127.0.0.1;
        #}

        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        #
        #location ~ \.php$ {
        #    root           html;
        #    fastcgi_pass   127.0.0.1:9000;
        #    fastcgi_index  index.php;
        #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
        #    include        fastcgi_params;
        #}

        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        #location ~ /\.ht {
        #    deny  all;
        #}
    }


    # another virtual host using mix of IP-, name-, and port-based configuration
    #
    #server {
    #    listen       8000;
    #    listen       somename:8080;
    #    server_name  somename  alias  another.alias;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}


    # HTTPS server
    #
    #server {
    #    listen       443 ssl;
    #    server_name  localhost;

    #    ssl_certificate      cert.pem;
    #    ssl_certificate_key  cert.key;

    #    ssl_session_cache    shared:SSL:1m;
    #    ssl_session_timeout  5m;

    #    ssl_ciphers  HIGH:!aNULL:!MD5;
    #    ssl_prefer_server_ciphers  on;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}

}
```

下面说一下配置文件的内容：
* 全局块：全局块是定义Nginx全局的模块，是配置文件开头不包括在大括号里的参数。

* event块：配置服务器和客户机之间网络连接

* http块：这是Nginx的主要块，配置Nginx的http服务。

* server块：虚拟主机配置

* location块：server中的一个块，server可包含多个location块

* work process：Nginx进程数，更高的进程数对应更高的并发量。

