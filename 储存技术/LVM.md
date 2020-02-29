LVM是Linux下逻辑磁盘管理器。在使用LVM时，多块磁盘共同构成一个VG，然后VG（类似于k8s中的PV）又被分成多个lv（逻辑卷，类似于k8s中的PVC）

通常情况LVM是默认被安装在系统里的，我们可以通过rpm -qa查看lvm是否被正确的安装。

查看一下由多少块数据盘

```bash
bash# ll /dev | grep sd
```

这样就可以看到服务器上安装的数据盘。

对各个数据盘进行分区：

```bash
# fdisk /etc/sdb
bash# fdisk /dev/sdb
欢迎使用 fdisk (util-linux 2.23.2)。

更改将停留在内存中，直到您决定将更改写入磁盘。
使用写入命令前请三思。


命令(输入 m 获取帮助)： n
```

这里可以不留备用分区，那就一路回车下去，最后输入w完成操作。

通过上边查看数据盘的方法查看数据分区是否创建完毕。

# 创建pv

创建pv就是将刚才创建的分区转换成可以被利用的卷资源。

```bash
pvcreate /dev/sd{b1,c1}
```

# 创建vg

vg就相当与一个资源池，将多个pv有机的结合起来。这里使用

```bash
vgcreate vg_name pv
```

创建了一个名叫vg_name的vg资源池。

激活刚才创建的vg

```bash
vgchange -a y vg_name
```

# 创建LV

lv就是一个逻辑卷，被创建出来后可被挂载。

```bash
lvcreate -L space -n lv_name vg_name
```

创建完毕后我们可以看一下/dev目录

```bash
bash# ll /dev | grep vg_2
drwxr-xr-x. 2 root root          60 2月  20 16:41 vg_2
bash# tree /dev/vg_2/
/dev/vg_2/
└── lv_1 -> ../dm-2

0 directories, 1 file
```

可见，lvm被制作成了一个虚拟的储存设备，我们直接将这个设备进行挂载就可以使用了。