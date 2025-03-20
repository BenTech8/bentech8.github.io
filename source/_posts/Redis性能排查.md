---
title: Redis性能排查
date: 2025-03-20 20:39:21
tags:
- Redis
categories:
- Redis
---

Redis作为内存数据库，具有非常高的性能，单个实例的OPS能够达到10W左右。

## 确认是否Redis本身响应慢

如果你发现你的业务服务API响应延迟变长，首先你需要先排查服务内部，究竟是哪个环境拖慢了整个服务。比较高效的做法是在服务内部集成链路追踪。

如果发现确实是操作Redis的这条链路耗时变长了，那么此刻需要把焦点关注在业务服务到Redis这条链路上。

从你的业务服务到Redis这条链路变慢的原因可能也有2个：

- 业务服务器到Redis服务器之间的网络存在问题，例如网络线路质量不佳，网络数据包在传输时存在延迟、丢包等情况。
- Redis本身存在问题，需要进一步排查是什么原因导致Redis变慢。



## Redis基准性能测试

从Redis角度来排查，是否存在导致变慢的场景，以及都有哪些因素会导致Redis的延迟增加，然后针对性的进行优化。

排除网络原因，确认你的Redis是否真的变慢了，首先需要对Redis进行基准性能测试，了解你的Redis在生产环境服务器上的基准性能。

什么是基准性能？

基准性能就是指Redis在一台负载正常的机器上，其最大的响应延迟和平均响应延迟分别是怎样的。

Redis在不同的软硬件环境下，它的性能是不同的，比如我的机器配置比较低，当延迟为2ms时，我就认为Redis变慢了，但是如果你的硬件配置比较高，那么在你的运行环境下，可能延迟在0.5ms时就可以认为Redis变慢了。所以，只有在了解你的Redis在生产环境服务器上的基准性能，才能进一步评估，当其延迟达到什么程度时，才认为Redis确实变慢了。

具体如何做？

为避免业务服务器到Redis服务器之间的网络延迟，你需要直接在Redis服务器上测试实例的响应延迟情况。

执行以下命令，就可以测试出这个实例60s内的最大响应延迟：

```shell
$ redis-cli -h 127.0.0.1 -p 6379 --intrinsic-latency 60
Max latency so far: 1 microseconds.
Max latency so far: 15 microseconds.
Max latency so far: 17 microseconds.
Max latency so far: 18 microseconds.
Max latency so far: 31 microseconds.
Max latency so far: 32 microseconds.
Max latency so far: 59 microseconds.
Max latency so far: 72 microseconds.

1428669267 total runs (avg latency: 0.0420 microseconds / 42.00 nanoseconds per run).
Worst run took 1429x longer than the average latency.
```

从输出结果可以看到，这60s内的最大响应延迟为72微秒。

还可以执行以下命令，查看一段时间内Redis的最小、最大、平均访问延迟：

```shell
$ redis-cli -h 127.0.0.1 -p 6379 --latency-history -i 1
min: 0, max: 1, avg: 0.13 (100 samples) -- 1.01 seconds range
min: 0, max: 1, avg: 0.12 (99 samples) -- 1.01 seconds range
min: 0, max: 1, avg: 0.13 (99 samples) -- 1.01 seconds range
min: 0, max: 1, avg: 0.10 (99 samples) -- 1.01 seconds range
min: 0, max: 1, avg: 0.13 (98 samples) -- 1.00 seconds range
min: 0, max: 1, avg: 0.08 (99 samples) -- 1.01 seconds range
...
```

以上输出结果为每间隔1s，采样Redis的平均操作耗时，其结果分布在0.08~0.13ms之间。

基准性能测试方法

了解了基准性能测试方法，就可以按照以下几步，来判断你的Redis是否真的变慢了：

- 在相同配置的服务器上，测试一个正常Redis实例的基准性能。
- 找到你认为可能变慢的Redis实例，测试这个实例的基准性能。
- 如果观察到，这个可能变慢的Redis实例的运行延迟是正常Redis基准性能的2倍以上，即可认为这个Redis实例确实变慢了。



## Redis变慢的因素

### 使用复杂度过高的命令

如何发现耗时长的命令？

Redis提供了慢日志(slowlog)命令统计功能，它记录了有哪些命令在执行时耗时比较久。

查看Redis慢日志前，你需要设置慢日志的阈值。如设置慢日志的阈值为5ms，并且保留最近500条慢日志记录：

```shell
# 命令执行耗时超过5ms，记录慢日志
CONFIG SET slowlog-log-slower-than 5000
# 只保留最近500条慢日志
CONFIG SET slowlog-max-len 500
```

设置完成后，所有执行的命令如果操作耗时超过了5ms，都会被Redis记录下来。

此时，你执行以下命令，就可以查询到最近记录的慢日志：

```shell
127.0.0.1:6379> SLOWLOG get 5
1) 1) (integer) 32693       # 慢日志ID
   2) (integer) 1593763337  # 执行时间戳
   3) (integer) 5299        # 执行耗时(微秒)
   4) 1) "LRANGE"           # 具体执行的命令和参数
      2) "user_list:2000"
      3) "0"
      4) "-1"
2) 1) (integer) 32692
   2) (integer) 1593763337
   3) (integer) 5044
   4) 1) "GET"
      2) "user_info:1000"
...
```

通过查看慢日志，就可以知道在什么时间间，执行了哪些命令比较耗时。

使用复杂度过高命令的场景有哪些？

如果你的应用程序执行的Redis命令有以下特点，那么可能会导致操作延迟变大：

- 经常使用O(N)以上复杂度的命令，例如SORT、SUNION、ZUNIONSTORE聚合类命令
- 使用O(N)复杂度的命令，但N的值非常大

第一种情况导致变慢的原因在于，Redis在操作内存数据时，时间复杂度过高，要花费更多的CPU资源。

第二种情况导致变慢的原因在于，Redis一次需要返回给客户端的数据过多，更多时间花费在数据协议的组装和网络传输过程中。

另外，我们还可以从资源使用率层面来分析，如果你的应用程序操作Redis的OPS不是很大，但Redis实例的CPU使用率却很高，那么很有可能是使用了复杂度过高的命令导致的。

除此之外，我们都知道，Redis是单线程处理客户端请求的，如果你经常使用以上命令，那么当Redis处理客户端请求时，一旦前面某个命令发生耗时，就会导致后面的请求发生排队，对于客户端来说，响应延迟也会变长。

如何解决？

- 尽量不使用O(N)以上复杂度过高的命令，对于数据的聚合操作，放在客户端做。
- 执行O(N)命令，保证N尽量小(推荐N<=300)，每次获取尽量少的数据，让Redis可以及时处理返回。



