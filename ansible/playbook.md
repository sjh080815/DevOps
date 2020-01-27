playbook就是和saltstack类似的yaml部署描述文件。一个playbooks内包含一个或多个play，下面对play进行一下解析：

下边是一个例子：

```
---
- hosts: webservers
  vars:
    http_port: 80
    max_clients: 200
  remote_user: root
  tasks:
  - name: ensure apache is at the latest version
    yum: pkg=httpd state=latest
  - name: write the apache config file
    template: src=/srv/httpd.j2 dest=/etc/httpd.conf
    notify:
    - restart apache
  - name: ensure apache is running
    service: name=httpd state=started
  handlers:
    - name: restart apache
      service: name=httpd state=restarted
```

# 指定机器

在yaml的前半段是对机器、用户和组的配置。

* hosts：指定执行的机器或者组
* remote_user：运行的用户

** 这里可以添加sudo: yes语句，表示使用sudo执行操作**

* vars：创建变量表

# tasks 表

tasks表就是任务表，每个数据元素就是一个任务。

这是一个简单的nginx自动构建playbook。

```
- hosts: test
  vars:
    httpport: 80
  remote_user: root
  tasks:
    - name: install nginx
      yum: pkg=nginx state=latest
    - name: start nginx
      service: name=nginx state=started
      ```

这个文件包含了两个task。

第一个是安装nginx，直接调用yum，用pkg key指定安装nginx，用state指定安装的版本。

第二个是服务状态控制。安装完成后nginx的状态是关闭的，需要进行手动启动。这里使用了service模块进行了自动启动。在name中指定服务名，在state中指定服务状态。


这里要注意，service指定的state必须是reload，started或者stopped，其他状态都是无效的。

