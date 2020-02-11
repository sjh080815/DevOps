# 安装jenkins

jenkins需要java的环境，所以我们先安装java环境。这里可以直接使用openjdk。

```bash
yum install java-1.8.0-openjdk
```

安装完成后，可以检查是否安装完成：

```bash
bash# java -version
openjdk version "1.8.0_242"
OpenJDK Runtime Environment (build 1.8.0_242-b08)
OpenJDK 64-Bit Server VM (build 25.242-b08, mixed mode)
```

这样就说明安装完成了。

# 安装jenkins

向服务器上传jenkins的rpm文件，使用rpm安装

```bash
bash# rpm -ivh jenkins-2.210-1.1.noarch.rpm
```

安装完成后，运行服务```systemctl start jenkins```。

## 修改jenkins配置

修改文件```/var/lib/jenkins/hudson.model.UpdateCenter.xml```，将文件中的url标签内的参数改成http://mirror.xmission.com/jenkins/updates/update-center.json。

稍等一会，会要要求输入密码，输入密码后，修改模块更新参数。修改文件```http://mirror.xmission.com/jenkins/updates/update-center.json```。将updates.jenkins-ci.org/download全部改成mirrors.tuna.tsinghua.edu.cn/jenkins，将www.google.com改成www.baidu.com。

重启服务，重新选择推荐插件，稍等就会完成。

# 进行jenkins系统配置

在jenkins面板中，进入系统管理=>系统配置，配置GitHub Service


![1.png](.\img\1.png)

在github上创建一个token：


![2.png](.\img\2.png)

在创建github服务器那块创建凭证：


![3.png](.\img\3.png)

创建一个自由风格项目，名称随意。


![4.png](.\img\4.png)

![5.png](.\img\5.png)

![6.png](.\img\6.png)

![7.png](.\img\7.png)

这样就创建完成一个自动部署任务了。我们手动进行一下部署，部署速度可能会有点慢，多等一会。

咱们对github上的项目进行更新，会发现jenkins上的项目也发生了变化，执行了那个shell脚本。

这个shell也可以写到主机上，然后再jenkins上写sh执行脚本就ok了。