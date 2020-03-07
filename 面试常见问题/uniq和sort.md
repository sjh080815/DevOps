sort和uniq是两个查看文件时候经常会用到的筛选工具。

sort：升序输出，将输入的文本以升序的方式进行输出。

uniq：降重输出。把输入文本里的重复数据剔除然后输出。

# 面试常见问题

查看nginx的PV地址：

nginx一般会配置access.log日志文件，查询这个日志文件。

```bash
bash# cat access.log

111.20.59.9 - - [25/Feb/2020:17:29:04 +0800] "GET /zabbix/favicon.ico HTTP/1.1" 200 32988 "http://39.97.232.17/zabbix/zabbix.php?action=dashboard.view&ddreset=1" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/80.0.3987.1
112
```

分析日志结构，发现第一列是用户的RealIP，也就是这里的PV，后边是http请求头的一些参数。

按要求进行排序并降重：

```bash
bash# cat access.log | awk '{print $1}' | sort | uniq
```

分析命令：第二段使用了awk进行了正则筛选，选中了第一列数据。将选中的数据进行升序和降重。