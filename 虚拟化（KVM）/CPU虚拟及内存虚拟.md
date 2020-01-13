# 硬件虚拟化（CPU和内存）

## 常用CPU架构

### SMP技术

这种架构CPU共同访问所有内存，速率比较低，且效率也低。

### NUMA技术

这是最常见的虚拟化技术，CPU直接通信自己所属的内存，直接调用，效率较高。

**这里需要用到numactl插件，需手动安装一下**

看以下代码：
```
bash$ numactl --hardware
available: 1 nodes (0)
node 0 cpus: 0 1 2 3 4 5 6 7
node 0 size: 7850 MB
node 0 free: 4704 MB
node distances:
node   0 
  0:  10 
  ```

从以上代码得，这台电脑是单CPU，是多核的CPU。内存一共7850M，空余4704M。

查看NUMA使用的内存

```
bash# numasat -c qemu-kvm
Per-node numastat info (in MBs):
                Node 0 Total
                ------ -----
Numa_Hit         19365 19365
Numa_Miss            0     0
Numa_Foreign         0     0
Interleave_Hit     175   175
Local_Node       19365 19365
Other_Node           0     0
Local_Node       19365 19365
Other_Node           0     0
```

可以看到内存的占用状况。
# KSM技术：

虚拟主句和宿主主机内存合并。

## 内存气球技术

需要virt balloon插件调节内存。