saltstack是一种自动化运维工具，主要体现出自动化就是使用剧本自动部署。下边引入几个概念：

1. file_root 这表示文件根的地址。这个目录下存放工具使用的剧本文件。
2. state模块 这是salt的状态模块，可以用来安装服务和获取服务状态。
3. highstate 高级模式。通常在file_root下存放的是以目录存放的剧本，使用一个top.sls文件引导剧本的入口。

# 编写一个简单的剧本

在编写第一个剧本之前，先修改和创建一个剧本存放目录：

在master的配置文件中，将file_root的配置字段取消注释。注意，这里一定要包含一个bash字段。这个此段表示文件基础地址。

创建剧本目录。这个目录多半是不会自动创建的，需要手动创建。权限不需要特别的去设置，默认权限就可以了。

创建剧本的时候，最好使用目录分类保存，这样更便于剧本管理。

在剧本目录中创建一个新的剧本：

```yaml
tomcat-install:
  pkg.installed:
    - names: 
      - tomcat

tomcat-service:
  service.running:
    - name: tomcat
    - enable: True
```

这个剧本在集群中自动安装了tomcat服务，并自动启动了tomcat，并把tomcat设置了自动启动。

使用salt分发剧本并执行：

```bash
salt '*' state.sls web.tomcat
```

等待执行完毕会回复成功。

## 常见故障

如果剧本在编写过程中写错了单词，那么salt肯定是不会执行剧本的，master会显示minion无返回的报错。这种情况仔细检查一下剧本是否有错误。

# 使用高级模式

在base目录下创建一个top.sls文件，编写主机执行选择：

```yaml
base:
  '*':
    web.tomcat
```

这样指定了在base环境下，所有主机都执行web.tomcat的剧本。

使用高级模式进行剧本执行：

```bash
salt '*' state.highstate
```

和前边一样，剧本执行完毕后会返回相关的参数。