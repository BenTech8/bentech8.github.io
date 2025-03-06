---
title: Linux操作系统性能调优
date: 2025-03-06 23:13:48
tags:
categories: Linux
---

## CPU调优

目标：最大化CPU利用率，减少上下文切换和延迟。

### 调整CPU频率策略

将CPU频率设置为'performance'模式，确保CPU始终以最高频率运行。

```shell
# 安装cpufrequtils工具
sudo apt install cpufrequtils
# 设置为performance模式
sudo cpufreq-set -g performance
```



### 绑定进程到特定CPU核心

使用'taskset'将关键进程绑定到特定CPU核心，减少上下文切换。

```shell
taskset -cp 0,1 <pid>
```



### 优化中断处理

将中断处理分散到多个CPU核心，避免单个CPU过载。

```shell
echo 2 | sudo tree /proc/irq/<irq_number>/smp_affinity
```



## 内存调优

目标：减少内存碎片化，降低交换分区使用频率。



### 调整Swappiness

降低swappiness值，减少系统使用交换分区（swap）的频率。

```shell
# 临时生效
sudo sysctl vm.swappiness=10
# 永久生效,在/etc/sysctl.conf下添加
vm.swappiness=10
```

在读取swap交换分区中的数据时，由于数据需要从磁盘中读取，因此可能会比物理内存中读取慢得多。

Linux中的swap交换分区是类似于Windows的虚拟内存，它的作用是在物理内存使用完之后，将磁盘空间(也就是swap分区)虚拟成内存来使用。它的功能就是在内存不够的情况下，操作系统先把内存中暂时不用的数据，存到硬盘的交换空间，腾出来让别的程序运行。



### 调整内存分配策略

设置vm.overcommit_memory为1，允许系统超额分配内存。

```shell
sudo sysctl vm.overcommit_memory=1
```

可选值：

- 0：表示内核将检查是否有足够的可用内存供应用进程使用；如果有足够的可用内存，内存申请允许；否则，内存申请失败，并把错误返回给应用进程。
- 1：表示内核允许分配所有的物理内存，而不管当前的内存状态如何。
- 2：表示内核允许分配超过所有物理内存和交换空间总和的内存。

Linux对大部分申请的内存的请求都回复“yes”，以便能跑更多更大的程序。因为申请内存后，并不会马上使用内存。这种技术叫做overcommit。当Linux发现内存不足时，会发生OOM killer(OOM=out-of-memory)。它会选择杀死一些进程(用户态进程，不是内核线程)，以便释放内存。



### 调整透明大页(THP)

对于某些工作负载(如数据库)，禁用透明大页可能更有利。

```shell
echo never | sudo tree /sys/kernel/mm/transparent_hugepage/enabled
```



## 磁盘I/O调优

目标：优化磁盘读写性能，减少I/O延迟。



### 选择合适的I/O调度器

对于SSD，建议使用noop或deadline调度器

```shell
echo noop | sudo tree /sys/block/sdX/queue/scheduler
```

对于HDD，建议使用deadline调度器

```shell
echo deadline | sudo tree /sys/block/sdX/queue/scheduler
```



### 调整文件系统挂载选项

对于ext4文件系统，启用noatime和data=writeback选项。

```shell
sudo mount -o remount,noatime,data=writeback /
```

默认的方式下linux会把文件访问的时间(atime)做记录，因为系统运行的时候要访问大量文件，如果能减少一些动作(比如减少时间戳的记录次数等)将会显著提高磁盘IO效率、提升文件系统的性能。

data=ordered模式是ext4文件系统默认日志格式。在data=writeback模式下，当元数据提交到日志后，data可以直接被提交到磁盘。即会做元数据日志，数据不做日志，并且不保证数据比元数据先落盘。writeback是ext4提供的性能最好的模式。



### 优化磁盘队列深度

增加磁盘队列深度以提高I/O吞吐量。

```shell
echo 256 | sudo tree /sys/block/sdX/queue/nr_requests
```

队列深度决定了给块设备写I/O的最大并发数，对于Linux系统，默认值为128。一般情况下不建议用户修改此参数。当对系统进行极限性能测试时，为了增大主机写I/O的压力及I/O在队列中被合并的概率，可以适当的增大此参数。



## 网络调优

目标：提高网络吞吐量，降低延迟。



### 调整TCP缓冲区大小

增加TCP接收和发送缓冲区大小。

```
sudo sysctl -w net.core.rmem_max=16777216
sudo sysctl -w net.core.wmem_max=16777216
sudo sysctl -w net.ipv4.tcp_rmem="4096 87380 16777216"
sudo sysctl -w net.ipv4.tcp_wmem="4096 65536 16777216"
```



### 启用TCP快速打开

减少TCP连接的建立时间。

```shell
sudo sysctl -w net.ipv4.tcp_fastopen=3
```



### 调整连接队列大小

增加连接队列大小以应对高并发连接。

```shell
sudo sysctl -w net.core.somaxconn=65535
sudo sysctl -w net.ipv4.tcp_max_syn_backlog=65535
```



## 文件系统调优

目标：提高文件系统性能，减少元数据操作开销。



### 调整文件描述符限制

增加系统允许打开的文件描述符数量。

```shell
sudo sysctl -w fs.file-max=100000
ulimit -n 100000
```



### 优化inode缓存

增加inode缓存大小以提高文件系统性能。

```
sudo sysctl -w vm.vfs_cache_pressure=50
```



## 内核参数调优

目标：优化内核行为，提升系统整体性能。



### 调整进程调度策略

对于实时性要求高的任务，使用SCHED_FIFO或SCHED_RR调度策略。

```shell
chrt -f -p 99 <pid>
```



### 优化虚拟内存管理

调整vm.dirty_ratio和vm.dirty_background_ratio，控制脏页写回行为。

```
sudo sysctl -w vm.dirty_ratio=10
sudo sysctl -w vm.dirty_background_ratio=5
```



## 监控与维护

目标：持续监控系统性能，及时发现瓶颈。



### 使用性能监控工具

- top、htop：监控CPU和内存的使用情况。
- vmstat：监控虚拟内存、CPU和I/O状态。
- iostat：监控磁盘I/O性能。
- netstat、ss：监控网络连接状态。



### 定期清理系统

- 清理不必要的日志文件和缓存。
- 使用logrotate管理日志文件大小。



## 硬件优化

目标：充分利用硬件资源



### 升级硬件

- 使用SSD替代HDD以提高I/O性能。
- 增加内存容量以减少交换分区的使用。
- 使用多核CPU以提高并发处理能力。



## 持久化调优配置

将调优参数写入/etc/sysctl.conf，确保重启后生效。

```shell
vm.swappiness=10
vm.overcommit_memory=1
net.core.somaxconn=65535
net.ipv4.tcp_max_syn_backlog=65535
```

