---
title: 入侵检测
date: 2025-04-09 15:53:29
tags:
- Linux
categories:
- Linux
---

## 入侵检测

### 查看系统日志

#### 查看安全相关日志

1）ssh远程登录失败日志

```shell
[root@localhost ~]# grep -i Failed /var/log/secure
Apr    9 15:58:33 localhost unix_chkpwd[1572]: password check failed for user (root) 
Apr    9 15:58:35 localhost login: FAILED LOGIN 1 FROM tty1 FOR root, Authentication failure
```

2）ssh远程登录成功日志

```shell
[root@localhost ~]# grep -i Accepted /var/log/secure
Apr    6 15:21:19 localhost sshd[1594]: Accepted password for root from 172.20.10.3 port 50205 ssh2
```

3）统计登录成功或登录失败的ip，并进行去重降序排列

```shell
[root@localhost ~]# grep -i Accepted /var/log/secure | awk '{print $(NF-3)}' | grep '^[0-9]' | sort | uniq -c | sort -rn
1 172.20.10.7
1 172.20.10.3
[root@localhost ~]# grep -i Failed /var/log/secure | awk '{print $(NF-3)}' | grep '^[0-9]' | sort | uniq -c | sort -rn
```

#### 查看历史用户登录信息

1）查看最后5条登录信息

```shell
[root@localhost ~]# last -a -5
```



2）查看指定时间之前登录信息

```shell
[root@localhost ~]# last -a -t 20250304123030
# 2025-03-04 12：30：30之前
```



3）查看登录系统的用户相关信息

```shell
[root@localhost ~]# last -a -f /var/log/btmp
Ben      ssh:notty    Wed Apr  9 17:20    gone - no logout  172.20.10.3
Ben      ssh:notty    Wed Apr  9 17:20 - 17:20  (00:00)     172.20.10.3
Ben      ssh:notty    Wed Apr  9 17:20 - 17:20  (00:00)     172.20.10.3
root     tty1         Wed Apr  9 15:58    gone - no logout
root     ssh:notty    Sun Apr  6 14:25 - 17:20 (3+02:54)    172.20.10.7
root     ssh:notty    Sun Apr  6 14:25 - 14:25  (00:00)     172.20.10.7
root     ssh:notty    Sun Apr  6 14:25 - 14:25  (00:00)     172.20.10.7
root     ssh:notty    Sun Apr  6 14:23 - 14:25  (00:01)     172.20.10.7
root     ssh:notty    Sun Apr  6 14:23 - 14:23  (00:00)     172.20.10.7
root     ssh:notty    Sun Apr  6 14:23 - 14:23  (00:00)     172.20.10.7
root     ssh:notty    Sun Apr  6 14:23 - 14:23  (00:00)     172.20.10.7
root     ssh:notty    Sun Apr  6 14:23 - 14:23  (00:00)     172.20.10.7
root     ssh:notty    Sun Apr  6 14:22 - 14:23  (00:00)     172.20.10.7
root     ssh:notty    Sun Apr  6 14:05 - 14:22  (00:17)     172.20.10.7
test1    ssh:notty    Sun Apr  6 14:05 - 14:05  (00:00)     172.20.10.7
test3    ssh:notty    Sun Apr  6 14:05 - 14:05  (00:00)     172.20.10.7
test3    ssh:notty    Sun Apr  6 13:58 - 14:05  (00:06)     172.20.10.7
test3    ssh:notty    Sun Apr  6 13:58 - 13:58  (00:00)     172.20.10.7
test3    ssh:notty    Sun Apr  6 13:58 - 13:58  (00:00)     172.20.10.7
root     ssh:notty    Sat Apr  5 22:17 - 13:58  (15:41)     172.20.10.7
root     ssh:notty    Sat Apr  5 22:17 - 22:17  (00:00)     172.20.10.7
root     ssh:notty    Sat Apr  5 21:54 - 22:17  (00:22)     172.20.10.7
root     ssh:notty    Sat Apr  5 21:54 - 21:54  (00:00)     172.20.10.7
ben      tty1         Sat Apr  5 21:50 - 15:58 (3+18:08)
ben      tty1         Sat Apr  5 20:24 - 21:50  (01:25)
ben      tty1         Sat Apr  5 20:24 - 20:24  (00:00)
ben      tty1         Sat Apr  5 20:24 - 20:24  (00:00)
ben      tty1         Sat Apr  5 20:23 - 20:24  (00:00)
root     ssh:notty    Sat Apr  5 19:59 - 21:54  (01:55)     172.20.10.7
root     ssh:notty    Sat Apr  5 19:59 - 19:59  (00:00)     172.20.10.7
root     ssh:notty    Sat Apr  5 19:58 - 19:59  (00:00)     172.20.10.7
(unknown tty1         Fri Apr  4 12:02 - 20:23 (1+08:20)
```



4）查看记录每个用户最后的登录信息

```shell
[root@localhost ~]# lastlog
root             pts/0    172.20.10.3      三 4月  9 17:23:24 +0800 2025
bin                                        **从未登录过**
daemon                                     **从未登录过**
adm                                        **从未登录过**
lp                                         **从未登录过**
sync                                       **从未登录过**
shutdown                                   **从未登录过**
halt                                       **从未登录过**
mail                                       **从未登录过**
operator                                   **从未登录过**
games                                      **从未登录过**
ftp                                        **从未登录过**
nobody                                     **从未登录过**
systemd-network                            **从未登录过**
dbus                                       **从未登录过**
polkitd                                    **从未登录过**
sshd                                       **从未登录过**
postfix                                    **从未登录过**
chrony                                     **从未登录过**
ben              tty1                      六 4月  5 21:50:06 +0800 2025
owen                                       **从未登录过**
it01             tty1                      六 4月  5 17:44:42 +0800 2025
it02                                       **从未登录过**
test1            pts/0    172.20.10.7      日 4月  6 13:57:05 +0800 2025
test2                                      **从未登录过**
test3            pts/0    172.20.10.7      日 4月  6 13:58:58 +0800 2025
```



#### 统计当前在线状态

```shell
[root@localhost ~]# w
 17:32:18 up 17 min,  3 users,  load average: 0.00, 0.01, 0.05
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
root     tty1                      17:15   16:26   0.03s  0.03s -bash
root     pts/0    172.20.10.3      17:23    2.00s  0.68s  0.00s w
root     pts/1    172.20.10.3      17:21    9:22   0.14s  0.14s -bash
```



#### 查看系统主日志

```shell
[root@localhost ~]# less /var/log/messages
```



#### 查看计划任务

```shell
[root@localhost ~]# less /var/log/cron
[root@localhost ~]# cat /var/spool/cron/*
[root@localhost ~]# less /etc/crontab
[root@localhost ~]# ls /etc/cron.*
```



### 查看异常流量

#### iftop动态查看网卡接口流量

```
[root@localhost ~]# yum -y install epel-release
[root@localhost ~]# yum -y install iftop
[root@localhost ~]# iftop -i ens33
```



#### 流量监控

- Cacti
- Zabbix
- Ganglia
- Prometheus



#### 数据包抓取

- wireshark
- tcpdump
- sniffer



#### tcpdump

1）基本用法

```shell
[root@localhost ~]# tcpdump -i eth0 -nnv
# 抓取100个包
[root@localhost ~]# tcpdump -i eth0 -nnv -c 100
# 抓取包内容写入文件
[root@localhost ~]# tcpdump -i eth0 -nnv -w /file1.tcpdump
# 从文件中读取包，一般不用tcpdump查看，用wireshark可视化查看更清晰
[root@localhost ~]# tcpdump -nnv -r /file1.tcpdump
```

2）条件(port|host|net)

```shell
# 抓取指定端口的包
[root@localhost ~]# tcpdump -i eth0 -nnv not port 80
[root@localhost ~]# tcpdump -i eth0 -nnv port 22
[root@localhost ~]# tcpdump -i eth0 -nnv port 80
# 抓取指定网络的包
[root@localhost ~]# tcpdump -i eth0 -nnv net 192.168.0.0/24
# 抓取指定主机的包
[root@localhost ~]# tcpdump -i eth0 -nnv host 192.168.0.15
[root@localhost ~]# tcpdump -i eth0 -nnv dst port 22
[root@localhost ~]# tcpdump -i eth0 -nnv src port 22
```

3）条件(协议)

```shell
[root@localhost ~]# tcpdump -i eth0 -nnv arp
[root@localhost ~]# tcpdump -i eth0 -nnv icmp
[root@localhost ~]# tcpdump -i eth0 -nnv udp
[root@localhost ~]# tcpdump -i eth0 -nnv tcp
[root@localhost ~]# tcpdump -i eth0 -nnv ip
[root@localhost ~]# tcpdump -i eth0 -nnv vrrp
```

4）多条件(与或非)

```shell
[root@localhost ~]# tcpdump -i eth0 -nnv not net 192.168.0.0/24
[root@localhost ~]# tcpdump -i eth0 -nnv not port 80
[root@localhost ~]# tcpdump -i eth0 -nnv host 192.168.0.15 and port 22
[root@localhost ~]# tcpdump -i eth0 -nnv host 192.168.0.15 and host 192.168.0.33
[root@localhost ~]# tcpdump -i eth0 -nnv host 192.168.0.15 or host 192.168.0.33
[root@localhost ~]# tcpdump -i eth0 -nnv \(host 192.168.0.15 and port 22\) or \(host 192.168.0.33 and port 89 \)

[root@localhost ~]# tcpdump -i eth0 -nnv host 192.168.0.110 and port 22 or port 80
[root@localhost ~]# tcpdump -i eth0 -nnv host 192.168.0.110 and \(port 22 or port 80 \)

[root@localhost ~]# tcpdump -i eth0 -nnv host 192.168.0.110 and port 80
[root@localhost ~]# tcpdump -i eth0 -nnv host 192.168.0.110 and ! port 80
```

5）条件为tcp仅有的SYN标记的

```
tcp包标识位
| C | E | U | A | P | R | S | F |
| 7 | 6 | 5 | 4 | 3 | 2 | 1 | 0 |
```



```shell
# 抓取SYN的包
[root@localhost ~]# tcpdump -i eth0 -nnv tcp[13]==2
[root@localhost ~]# tcpdump -i eth0 -nnv tcp[13]==2 and port 22 -w ssh-conn.tcpdump
# 条件是：TCP仅有SYN/ACK标记的
[root@localhost ~]# tcpdump -i eth0 -nnv tcp[13]==18      # SYN+ACK的包
[root@localhost ~]# tcpdump -i eth0 -nnv tcp[13]==17      # FIN+ACK的包
```



### 检查可疑进程

#### ps

系统进程一般带有“[]”

```shell
[root@localhost ~]# px -aux | less
```



#### pstree

显示每个程序的完全指令，包含路径、参数或是常驻服务标识、列出树状图时特别标注现在执行的程序

```shell
# 显示所有
[root@localhost ~]# pstree -a
[root@localhost ~]# pstree -h
```



#### top

按cpu、内存排序

```shell
[root@localhost ~]# top -d 1
# 按P以CPU使用排序
# 按M以内存使用排序
```



#### netstat

查看网络连接情况

```shell
[root@localhost ~]# netstat -anputl
```



#### ss

查看某个协议或端口的监听状态

```
[root@localhost ~]# ss -an | grep tcp
[root@localhost ~]# ss -an | grep 22
```



#### 根据文件或端口查找进程

1）根据某文件查看正在被某些进程使用

```shell
[root@localhost ~]# lsof /usr/sbin/vsftpd
[root@localhost ~]# fuser /usr/local/nginx/sbin/nginx
```

2）根据某个端口查看对应进程

```shell
[root@localhost ~]# lsof -i TCP:22
[root@localhost ~]# fuser -v 22/tcp
```



### kernel audit 内核审计



### 文件完全性检查

#### 检验RPM包完整性

没有显示说明包没有被修改

```shell
[root@localhost ~]# rpm -V bash
[root@localhost ~]# rpm -V kernel
[root@localhost ~]# rpm -v vsftpd
[root@localhost ~]# rpm -vf /etc/ssh/sshd_config
```

#### md5sum/sha1sum检测

1）获取当前的/etc目录的md5值

```shell
[root@localhost ~]# find /etc -type f -exec md5sum {} \; >/tmp/`date +%F%H%M`-md5.txt
```

2）对比以上md5值获取操作过的文件

```shell
[root@localhost ~]#　diff /tmp/1-md5.txt /tmp/2-md5.txt
```

#### HIDS:AIDE高级入侵检测环境



## 拓展

### arp

```shell
# 查看本机mac地址列表
[root@localhost ~]# arp -a
```

### arping

```shell
# 获取某个IP对应的mac地址
[root@localhost ~]# arping -I eth0 192.168.43.20
```

