# 安装

使用pip安装。直接执行
```bash
pip install psutil
```

# CPU信息

**使用方法：cpu_times()**

cpu信息以元组的方式传出。可以分别访问元素。


# 内存和swap信息

**使用方法：virtual_memory()和swap_memory()**

返回内存和swap的信息。返回的参数都是以B为单位的。

# 磁盘信息

**使用方法：disk_*()**

psutil.disk_partitions():返回所有磁盘。在windows中返回的是硬盘分区。

psutil.disk_usage()：返回硬盘的信息。参数为硬盘的使用量和容量。

psutil.disk_io_counters()：返回磁盘的数据吞吐量。

# 网络信息

**使用方法：psutil.net_io_counters(pernic=True)**

返回网络的数据吞吐量。返回值都是B为单位的。pernic=True是显示每个网卡的信息。不写默认是所有信息。

# 用户信息

**使用方法：users()**

返回所有用户的信息。
