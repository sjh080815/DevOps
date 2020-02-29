RAID是linux中常用的一种冗余储存方案。目前主要的RAID技术由RAID0、RAID1、RAID5和RAID10。下边对这些冗余方案做一下总结：

# RAID0

是一种条带化的储存冗余。将数据离散的分布到多个磁盘上。这样的优点在于性能高，读取速度快，高IO特性。但是，如果某一块磁盘发生了故障，那么对整个文件系统将会是灾难性的。
![RAID0](http://mmbiz.qpic.cn/mmbiz/W9DqKgFsc6ibRYQnESPlr3XiaVG1Il4NK656wk6CJb4LkEgwpGMOaMmV3pN5vnmzWWa2TqM0HtZP4kess35eD0kQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

# RAID1

是一种复制卷存储冗余。在磁盘阵列内，两块或多块磁盘进行文件复制，那么这几块磁盘的数据是相同的。这样的有点是数据的可用性更高（高容错），但是同时，磁盘阵列的容量下降，读性能由显著提高，写性能较低。适合做高可用数据阵列。

![RAID1](http://mmbiz.qpic.cn/mmbiz/W9DqKgFsc6ibRYQnESPlr3XiaVG1Il4NK6Tj2k3iacS64YfXS3wJCvnjc8ybGhbsaOuO3ufoMh12btsnib112zPE9g/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

# RAID5

四个磁盘组成一个RAID5阵列，每个磁盘被分为四块，每个磁盘都有一个块是奇偶校验位。当某一个磁盘数据发生损坏时，利用剩下的数据和奇偶校验位恢复数据。那么，由这么推得，每个磁盘只有3/4的空间可以用。

这样的磁盘阵列性能较好，容灾性能也较好。但是这样将会由一个磁盘的空间进行奇偶校验。

![RAID5](http://mmbiz.qpic.cn/mmbiz/W9DqKgFsc6ibRYQnESPlr3XiaVG1Il4NK6gZKttcmqibdVbl3O92RYibU6I77bc7NRD7BsT6htGyiblF7vCruJIiaYTg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

# RAID10

RAID10是RAID0和RAID1结合的一种方案。四个磁盘组成一个储存阵列，两个磁盘为一个储存单元，这两个磁盘互为镜像。同时，这两个储存单元像RAID0一样进行数据分块储存。

这样的磁盘阵列读写性能较好，由你较高的容错性能。但是储存空间也会损耗一半。

![RAID10](http://mmbiz.qpic.cn/mmbiz/W9DqKgFsc6ibRYQnESPlr3XiaVG1Il4NK6o2pUHZN4Swibo4Fopn0pJrF92JowAphia7W45QlZKSiczlAZRbmYH44IQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)