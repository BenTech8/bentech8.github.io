---
title: pod核心对象
date: 2020-06-01 16:01:49
tags:
- Kubernetes
categories:
- Kubernetes
---



### Pod联网

每个Pod在每个地址族中获得一个唯一的IP地址。Pod中的每个容器共享网络名字空间，包括IP地址和网络端口。Pod内的容器可以使用localhost互相通信。当Pod中的容器与Pod之外的实体通信时，它们必须协调如何使用共享的网络资源(例如端口)。

Pod使用主机网络

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-use-hostnetwork
spec:
  containers:
  - name: myapp
    image: ikubernetes/myapp:v1
  hostNetwork: true
```



### Pod管理多个容器

Pod被设计成支持形成内聚服务单元的多个协作过程(形式为容器)。Pod中的容器被自动安排到集群中的同一物理机或虚拟机上，并可以一起进行调度。容器之间可以共享资源和依赖、彼此通信、协调何时以及何种方式终止自身。

```yaml
 6apiVersion: v1
kind: Pod
metadata:
  name: mc1
spec:
  volumes:
  - name: html
    emptyDir: {}
  containers:
  - name: 1st
    image: nginx
    volumeMounts:
    - name: html
      mountPath: /usr/share/nginx/html
  - name: 2nd
    image: debian
    volumeMounts:
    - name: html
      mountPath: /html
    command: ["/bin/bash", "-c"]
    args:
      - while true; do
          date >> /html/index.html;
          sleep 1;
        done
```



### 静态Pod

静态Pod（Static Pod）直接由特定节点上的kubelet守护进程管理，不需要API服务器看到它们。尽管大多数Pod都是通过控制面（例如Deployment）来管理的，对于静态Pod而言，kubelet直接监控每个Pod，并在其失效时重启之。

静态Pod通常绑定在某个节点上的kubelet。其主要用途是运行自托管的控制面。在自托管场景中，使用kubelet来管理各个独立的控制面组件。

kubelet自动尝试为每个静态Pod在Kubernetes API服务器上创建一个镜像Pod。这意味着在节点上运行的Pod在API服务器上是可见的，但不可以通过API服务器来控制。

配置文件就是放在特定目录下的标准的JSON或YAML格式的pod定义文件。用kubelet --pod-manifest-path=<the directory\>来启动kubelet进程，kubelet定期的去扫描这个目录，根据这个目录下出现或消失的YAML/JSON文件来创建或删除静态Pod。

比如我们在node01这个节点上用静态pod的方式来启动一个nginx的服务。我们登录到node01节点上面，可以通过下面命令找到kubelet对应的启动配置文件:

```shell
# 查看kubelet状态
[root@node01 ~]# systemctl status kubelet
```

可以看到，kubelet配置文件位置为：/etc/systemd/system/kubelet.service.d/10-kubelet.conf。打开这个文件我们可以看到其中一条如下的环境变量配置：Environment="KUBELET_SYSTEM_PODS_ARGS=--pod-manifest-path=/etc/kubernetes/manifests --allow-privileged=true"。

如果你的kubelet启动参数中没有配置上面的--pod-manifest-path参数的话，那么添加上这个参数然后重启kubelet即可。



### 配置Pod使用卷存储

只要容器存在，容器的文件系统就会存在，因此当一个容器终止并重新启动，对该容器的文件系统改动将丢失。对于独立于容器的持久化存储，你可以使用卷。这对有状态应用程序尤为重要，例如键值存储（如Redis）和数据库存。

#### emptyDir卷

在整个Pod生命周期中一直存在，即使Pod的容器被终止和重启。

```shell
[root@master01 ~]# cat redis.yaml
apiVersion: v1
kind: Pod
metadata: redis
spec:
  containers:
  - name: redis
    image: redis
    volumeMounts:
    - name: redis-storage
      mountPath: /data/redis
  volumes:
  - name: redis-storage
    emptyDir: {}
# 创建Pod
[root@master01 ~]# kubectl apply -f redis.yaml
# 监控Pod变化
[root@master01 ~]# kubectl get pod redis --watch
# 另开一个终端，进入redis容器
[root@master01 ~]# kubectl exec -it redis /bin/bash
# 进入持久化目录
[root@redis:data]# cd /data/redis/
# 创建新文件
[root@redis:/data/redis]# echo "test file" > test-file
# 查看redis进程pid
[root@redis:/data/redis]# ps aux
# kill redis进程，此时关注另一个终端，可以观察到Pod在重启
[root@redis:/data/redis]# kill <pid>
# 重启进入redis容器,查看持久化文件是否存在
[root@master01 ~]# kubectl exec -it redis /bin/bash
# 查看持久化文件是否存在
[root@redis:/data]# ls ./redis
```



### 容器的特权模式

在Linux中，Pod中的任何容器都可以使用容器规约中的安全性上下文的privileged（Linux）参数启用特权模式。这对于想要使用操作系统管理权能（Capabilities，如操纵网络堆栈和访问设备）的容器很有用。

为Pod设置安全性上下文要为Pod设置安全性设置，可在Pod规约中包含securityContext字段。securityContext字段值是一个PodSecurityContext对象。你为Pod所设置的安全性配置会应用到Pod中所有Container上。下面是一个Pod的配置文件，该Pod定义了securityContext和一个emptyDir卷：

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: security-context-demo
spec:
  securityContext:
    runAsUser: 1000
    runAsGroup: 3000
    fsGroup: 2000
  volumes:
  - name: sec-ctx-vol
    emptyDir: {}
  containers:
  - name: sec-ctx-demo
    image: busybox
    command: ["sh", "-c", "sleep 1h"]
    volumeMounts:
    - name: sec-ctx-vol
      mountPath: /data/demo
    securityContext:
      allowPrivilegeEscalation: false
```

在配置文件中，runAsUser字段指定Pod中的所有容器内的进程都使用用户ID 1000来运行。runAsGroup字段指定所有容器中的进程都以主组ID 3000来运行。如果忽略此字段，则容器的主组ID将是root（0）。当runAsGroup被设置时，所有创建的文件也会划归用户1000和组3000。由于fsGroup被设置，容器中所有进程也会是附组ID 2000的一部分。卷/data/demo及在该卷中创建的任何文件的属主都会是组ID 2000。

```shell
[root@master1 ~]# kubectl exec -it security-context-demo sh
/ $ ps
# 输出显示进程以用户1000运行，即runAsUser所设置的值：
PID    USER    TIME    COMMAND
1      1000    0:00    sleep 1h
6      1000    0:00     sh
...

/ $ cd /
/data $ ls -l
drwxrwxrwx 2 root 2000 4096 Jun 6 20:08 demo
# 输出显示/data/demo目录的组ID为2000，即fsGroup的设置值

/data $ cd demo
/data/demo $ echo hello > testfile
/data/demo $ ls -l
-rw-r--r-- 1 1000 2000 6 Jun 6 20:08 testfile
# 输出显示testfile的组ID为2000，也就是fsGroup所设置的值

/data/demo $ id
uid=1000 gid=3000 groups=2000
```



### Pod服务质量

​	Kubernetes创建Pod时就给它指定了下列一种QoS类：

- Guaranteed

  Pod中的每个容器都必须指定内存限制和内存请求。对于Pod中的每个容器，内存限制必须等于内存请求。

  Pod中的每个容器都必须指定CPU限制和CPU请求。对于Pod中的每个容器，CPU限制必须等于CPU请求。

  这些限制同样适用于初始化容器和应用程序容器。

  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: qos-demo
    namespace: qos-example
  spec:
    containers:
    - name: qos-demo-ctr
      image: nginx
      resources:
        limits:
          memory: "200Mi"
          cpu: "700m"
        requests:
          memory: "200Mi"
          cpu: "700m"
  ```

  查看Pod详情：

  ```yaml
  [root@master1 ~]# kubectl get pod qos-demo -n qos-example --output=yaml
  # 结果表明Kubernetes为Pod配置的QoS类为Guaranteed。
  ```

- Burstable

  Pod不符合Guaranteed QoS类的标准。

  Pod中至少一个容器具有内存或CPU请求。

  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: qos-demo-2
    namespace: qos-example
  spec:
    containers:
    - name: qos-demo-2-ctr
      image: nginx
      resources:
        limits:
          memory: "200Mi"
        requests:
          memory: "100Mi"
  ```

- BestEffort

  Pod中的容器必须没有设置内存和CPU限制和请求。

  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: qos-demo-3
    namespace: qos-example
  spec:
    containers:
    - name: qos-demo-3-ctr
      image: nginx
  ```




### Pod生命周期

- Pending

  Pod已被Kubernetes系统接受，但有一个或者多个容器尚未创建亦未运行。此阶段包括等待Pod被调度的时间和通过网络下载镜像的时间。

- Running

  Pod已经绑定到某个节点，Pod中所有的容器都已创建。至少有一个容器仍在运行，或者正处于启动或重启状态。

- Succeeded

  Pod中的所有容器都已成功终止，并且不会再重启。

- Failed

  Pod中的所有容器都已终止，并且至少有一个容器是因为失败终止。也就是说，容器以非0状态退出或者被系统终止。

- Unknown

  因为某些原因无法取得Pod的状态。这种情况通常是因为与Pod所在主机通信失败。

Pod在其生命周期中只会被调度一次，一旦Pod被调度（分派）到某个节点，Pod会一直在该节点运行，直到Pod停止或者被终止。

容器状态

kubernetes会跟踪Pod中每个容器的状态，就像它跟踪Pod总体上的阶段一样。你可以使用容器生命周期回调来在容器生命周期中的特定时间点触发事件。

一旦调度器将Pod分派给某个节点，kubelet就通过容器运行时开始为Pod创建容器。容器的状态有三种：Waiting（等待）、Running（运行中）和Terminated（已终止）。

要检查Pod中容器的状态，你可以使用kubectl describe pod <pod名称>。其输出中包含Pod中每个容器的状态：

- Waiting（等待）

  如果容器并不处在Running或Terminated状态之一，它就处在Waiting状态。处于Waiting状态的容器仍在运行它完成启动所需要的操作：例如，从某个容器镜像仓库拉取镜像，或者向容器应用Secret数据等等。当你使用kubectl来查询包含Waiting状态的容器的Pod时，你也会看到一人Reason字段，其中给出了容器处于等待状态的原因。

- Running（运行中）

  Running状态表明容器正在执行状态并且没有问题发生。如果配置了postStart回调，那么该回调已经执行且已完成。如果你使用kubectl来查询包含Running状态的容器的Pod时，你也会看到关于容器进入Running状态的信息。

- Terminated（已终止）

  处于Terminated状态的容器已经开始执行并且或者正常结束或者因为某些原因失败。如果你使用kubectl来查询包含Terminated状态的容器的Pod时，你会看到容器进入此状态的原因、退出代码以及容器执行期的起止时间。

容器重启策略

Pod的spec中包含一个restartPolicy字段，其可能取值包括Always、Onfailure和Never。默认值是Always。

restartPolicy适用于Pod中的所有容器。restartPolicy仅针对同一节点上kubelet的容器重启动作。当Pod中的容器退出时，kubelet会按指数回退方式计算重启的延迟（10s、20s、40s、...），其最长延迟为5分钟。一旦某容器执行了10分钟并且没有出现问题，kubelet对该容器的重启回退计时器执行重置操作。

钩子函数

Kubernetes支持postStart和preStop事件。当一个容器启动后，Kubernetes将立即发送postStart事件。

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: lifecycle-demo
spec:
  containers:
  - name: lifecycle-demo-container
    image: nginx
    lifecycle:
      postStart:
        exec:
          command: ["/bin/sh", "-c", "echo Hello from the postStart handler > /usr/share/message"]
      preStop:
        exec:
          command: ["/bin/sh", "-c", "nginx -s quit; while killall -0 nginx; do sleep 1; done"]
```

在上述配置文件中，可以看到postStart命令在容器的/usr/share目录下写入文件message。命令preStop负责优雅地终止nginx服务。

[**注意**]

kubernetes在容器创建后立即发送postStart事件。然而，postStart处理函数的调用不保证早于容器的入口点（entrypoint）的执行。

postStart处理函数与容器的代码是异步执行的，但kubernetes的容器管理逻辑会一直等待postStart处理函数执行完毕。只有postStart处理函数执行完毕，容器的状态才会变成RUNNING。
