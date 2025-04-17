---
title: k8s安装方式
date: 2020-04-16 12:20:20
tags:
- Kubernetes
categories:
- Kubernetes
---

## kind安装方式

适用场景：开发环境。

### kind安装

```shell
[root@localhost ~]# curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.27.0/kind-linux-amd64
[root@localhost ~]# chmod +x ./kind
[root@localhost ~]# sudo mv ./kind /usr/local/bin/kind
```



### 集群管理

1）快速创建集群

```shell
# 默认name创建集群
[root@localhost ~]# kind create cluster

# 指定name创建集群
[root@localhost ~]# kind create cluster --name kind-2

# 查看集群
[root@localhost ~]# kind get clusters
kind
kind-2

# kubectl指定context
[root@localhost ~]# kubectl cluster-info --context kind-kind
```



2）创建高可用集群

```shell
[root@localhost ~]# more config.yaml
kind: Cluster
apiVersion: kind.x-k8s-io/v1alpha4
nodes:
- role: control-plane
- role: control-plane
- role: control-plane
- role: worker
- role: worker
- role: worker

[root@localhost ~]# kind create cluster --config config.yaml
```



3）删除集群

```shell
# 删除name为kind集群
[root@localhost ~]# kind delete cluster 

# 删除name为kind-2集群
[root@localhost ~]# kind delete cluster --name kind-2
```



### 集群测试

1）load nginx镜像至集群节点

```shell
# 向节点导入镜像
[root@localhost ~]# kind load docker-image docker.io/library/nginx:latest

# 查看节点镜像是否导入成功
[root@localhost ~]# docker exec -it kind-worker crictl images
```

2）创建deployment

```shell
# 创建deployment
[root@localhost ~]# kubectl create deployment nginx --image=docker.io/library/nginx:latest
# 扩容
[root@localhost ~]# kubectl scale deployment/nginx --replicas=3
```

3）暴露服务

```shell
# 实质是创建一个service
[root@localhost ~]# kubectl expose deployment nginx --port=8000 --target-port=80
# port-forward至本机端口
[root@localhost ~]# kubectl port-forward service/nginx 7000:8000
# 访问nginx
[root@localhost ~]# curl http://localhost:7000
```

### 自定义参数集群配置

```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
# 节点配置
node:
- role: control-plane
  # 将control-plane容器的80端口映射至主机8080端口，这样可不必使用port-forward
  extraPortMappings:
  - containerPort: 80
    hostPort: 8080
    listenAddress: "127.0.0.1"
    protocol: TCP
  # 挂载
  extraMounts:
  - hostPath: /path/to/my/files
    containerPath: /files
- role: control-plane
- role: control-plane
- role: worker
- role: worker
- role: worker
# 网络配置
networking:
  # apiServer服务暴露端口
  apiServerPort: 6443
  # pod IP地址范围
  podSubnet: "10.244.0.0/16"
  # service IP地址范围
  serviceSubnet: "10.96.0.0/12"
  # kubeProxy工作模式
  kubeProxyMode: "ipvs"
```



## minikube安装方式

适用场景：开发环境。

### minikube安装

```shell
[root@localhost ~]# curl -LO https://github.com/kubernetes/minikube/releases/latest/download/minikube-linux-amd64
[root@localhost ~]# sudo install minikube-linux-amd64 /usr/local/bin/minikube && rm minikube-linux-amd64
```



### 集群管理

1）创建集群

```shell
[root@localhost ~]# minikube start
```

2）删除集群

```shell
# 停止minikube
[root@localhost ~]# minikube stop

# 删除集群(不再使用时操作)
[root@localhost ~]# minikube delete --all
```



3）minikube常用操作命令

```shell
# 查看minikube自带插件
[root@localhost ~]# minikube addons list
```



### 集群测试

1）创建deployment

```shell
[root@localhost ~]# kubectl create deployment nginx --image=docker.io/library/nginx:latest
```

 

2）创建service

```shell
[root@localhost ~]# kubectl expose deployment nginx --port=8000 --target-port=80
```



3）暴露服务

```shell
[root@localhost ~]# minikube service nginx
```



## kubeadm安装方式

适用场景：测试或生产环境。

### 准备工作

1）内核升级

```shell
# 下载内核文件
https://elrepo.org/linux/kernel/el7/x86_64/RPMS/
https://mirrors.aliyun.com/elrepo/kernel/el7/x86_64/RPMS/

# 安装内核新版本
[root@localhost ~]# yum localinstall -y kernel-ml*

# 修改新安装的内核为默认内核
[root@localhost ~}# rub2-set-default 0 && grub2-mkconfig -o /etc/grub2.cfg
[root@localhost ~]# grubby --args="user_namespace.enable=1" --update-kernel="$(grubby --default-kernel)"

# 查看系统可用内核,显示为最新安装的内核
[root@localhost ~]# grubby --default-kernel

# 重启linux
[root@localhost ~]# reboot
```



2）yum源配置

```shell
[root@localhost ~]# yum -y install yum-utils
# 替换阿里源
[root@localhost ~]# sed -i -e '/mirrors.cloud.aliyuncs.com/d' -e '/mirrors.aliyuncs.com/d' /etc/yum.repos.d/CentOS-Base.repo
# k8s源
[root@localhost ~]# cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package
EOF

# 升级软件
[root@localhost ~]# yum update -y --exclude=kernel*
```



3）基本软件安装与配置

```shell
# 安装工具
[root@localhost ~]# yum install -y yum-utils device-mapper-persistent-data lvm2 ipvsadn ipset sysstat conntrack libseccomp wget jq psmisc vim net-tools telnet yum-utils device-mapper-persistent-data lvm2 git

# 配置ipvs模块
[root@localhost ~]# modprobe -- ip_vs
[root@localhost ~]# modprobe -- ip_vs_rr
[root@localhost ~]# modprobe -- ip_vs_wrr
[root@localhost ~]# modprobe -- ip_vs_sh
[root@localhost ~]# modprobe -- nf_conntrack
[root@localhost ~]# vim /etc/modules-load.d/ipvs.conf
# 加入以下内容
ip_vs
ip_vs_lc
ip_vs_wlc
ip_vs_rr
ip_vs_wrr
ip_vs_lblc
ip_vs_lblcr
ip_vs_dh
ip_vs_sh
ip_vs_nq
ip_vs_sed
ip_vs_ftp
ip_vs_sh
nf_conntrack
ip_tables
ip_set
xt_set
ipt_set
ipt_rpfilter
ipt_REJECT
ipip
[root@localhost ~]# systemctl enable --now systemd-modules-load.service
```



4）配置hosts

```shell
[root@localhost ~]# vim /etc/hosts
172.20.10.6 master1
172.20.10.7 master2
172.20.10.8 master3
172.20.10.9 k8s-vip
172.20.10.10 node01
172.20.10.11 node02
```



5）关闭服务

```shell
# 关闭防火墙
[root@localhost ~]# systemctl disable --now firewalld
# 关闭域名解析服务
[root@Localhost ~]# systemctl disable --now dnsmasq
# 关闭网络服务
[root@localhost ~]# systemctl disable --now NetworkManager
# 关闭selinux
[root@localhost ~]# setenforce 0
[root@localhost ~]# sed -i 's#SELINUX=enforcing#SELINUX=disabled#g' /etc/sysconfig/selinux
[root@localhost ~]# sed -i 's#SELINUX=enforcing#SELINUX=disabled#g' /etc/selinux/config
# 关闭swap
[root@localhost ~]# swapoff -a && sysctl -w vm.swappiness=0
[root@localhost ~]# sed -ri '/^[^#]*swap/s@^@#@' /etc/fstab
```



6）配置时间同步

```shell
[root@localhost ~]# ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
[root@localhost ~]# yum install ntpdate -y
[root@localhost ~]# ntpdate time2.aliyun.com

# 添加定时任务
[root@localhost ~]# vim /etc/crontab
*/5 * * * * /usr/sbin/ntpdate time2.aliyun.com
```



7）配置limit

```shell
[root@localhost ~]# ulimit -SHn 65535
[root@localhost ~]# vim /etc/security/limits.conf
# 文件最后添加下面内容
* soft nofile 65535
* hard nofile 131072
* sofr nproc 65535
* hard nproc 655350
* soft memlock unlimited
* hard memlock unlimited
```



8）k8s内核参数修改

```shell
[root@localhost ~]# cat <<EOF > /etc/sysctl.d/k8s.conf
net.ipv4.ip_forward = 1
net.bridge.bridge-nf-call-iptables =1
net.bridge.bridge-nf-call-ip6tables = 1
fs.may_detach_mounts =1
net.ipv4.conf.all.route_localnet = 1
vm.overcommit_memory=1
vm.panic_on_oom=0
fs.inotify.max_user_watches=89100
fs.file-max=52706963
fs.nr_open=52706963
net.netfilter.nf_conntrack_max=2310720

net.ipv4.tcp_keepalive_time = 600
net.ipv4.tcp_keepalive_probes = 3
net.ipv4.tcp_keepalive_intvl = 15
net.ipv4.tcp_max_tw_buckets = 36000
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_max_orphans = 327680
net.ipv4.tcp_orphans_retries = 3
net.ipv4.tcp_syncookies = 1
net.ipv4.tcp_max_syn_backlog = 16384
net.ipv4.ip_conntrack_max = 65536
net.ipv4.tcp_max_syn_backlog = 16384
net.ipv4.tcp_timestamps = 0
net.core.somaxconn = 16384
EOF
[root@Localhost ~]# sysctl --system

# 重启计算机
[root@localhost ~]# reboot
# 查看是否修改成功
[root@localhost ~]# lsmod
[root@localhost ~]# lsmod | grep nf_conntrack
```



9）containerd安装

```shell
# 配置相关模块
[root@localhost ~]# cat <<EOF | sudo tee /etc/modules-load.d/containerd.conf
overlay
br_netfilter
EOF

# 加载模块
[root@localhost ~]# modprobe -- overlay
[root@localhost ~]# modprobe -- br_netfilter

# 配置内核 
[root@localhost ~]# cat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF
# 加载
[root@localhost ~]# sysctl --system

# 安装containerd
[root@localhost ~]# yum install -y containerd

# 配置containerd
[root@localhost ~]# cd /etc/containerd
[root@localhost ~]# mv config.toml config.toml.bak
[root@localhost ~]# containerd config default > config.toml
# 更改pause镜像仓库为阿里云仓库
[root@localhost ~]# sed -i "s#k8s.gcr.io#registry.cn-hangzhou.aliyuncs.com/google_container#g" /etc/containerd/config.toml
# 更改SystemdCgroup=true
[root@localhost ~]# sed -i '/containerd.runtimes.runc.options/a\ \ \ \ \ \ \ \ \ \ \ \ SystemdCgroup = true' /etc/containerd/config.toml
# 更新配置
[root@localhost ~]# systemctl daemon-reload
# containerd开机自启动
[root@localhost ~]# systemctl enable --now containerd
```



### kubeadm安装

```shell
# 安装kubeadm到所有节点
[root@localhost ~]# yum install kubeadm-1.32* kubelet-1.32* kubectl-1.23* -y

# 更改kubelet的配置使用Containerd作为Runtime
[root@localhost ~]# cat > /etc/sysconfig/kubelet <<EOF
KUBELET_KUBEADM_ARGS="--container-runtime=remote --runtime-request-timeout=15m --container-runtime-endpoint=unix:///run/containerd/containerd.sock"
EOF

# 重新加载，开启自启动
[root@localhost ~]# systemctl daemon-reload
[root@localhost ~]# systemctl enable --now kubelet
```



### master高可用

所有master节点安装HAProxy和KeepAlived，使得master节点高可用。(生产用专用负载均衡器，生产一般不自己搭建k8s集群，可选择公有云集群)

```shell
# 安装keepalived和haproxy
[root@localhost ~]# yum install -y keepalived haproxy

# 每个Master节点配置HAProxy
[root@localhost ~]# mkdir /etc/haproxy
[root@master01 ~]# vim /etc/haproxy/haproxy.cfg
global
  maxconn  2000
  ulimit-n 16384
  log 127.0.0.1 local0 err
  stats timeout 30s
 
defaults
  log global
  mode http
  option httplog
  timeout connect 5000
  timeout client  50000
  timeout server  50000
  timeout http-request 15s
  timeout http-keep-alive 15s

frontend monitor-in
  bind *:33305
  mode http
  option httplog
  monitor-uri /monitor
  
frontend k8s-master
  bind 0.0.0.0:16443
  bind 127.0.0.1:16443
  mode tcp
  option tcplog
  tcp-request inspect-delay 5s
  default_backend master

backend master
  mode tcp
  option tcplog
  option tcp-check
  balance roundrobin
  default-server inter 10s downinter 5s rise 2 fall 2 slowstart 60s maxconn 250 maxqueue 256 weight 100
  server master1      172.20.10.6:6443  check
  server master2      172.20.10.7:6443  check
  server master3      172.20.10.8:6443  check
  
 
# 配置keepalived
[root@master1 ~]# mkdir /etc/keepalived
# Master节点的配置，每个节点配置不同：注意下面的ip地址要修改成自己的ip地址，vip地址要是一个没有被占用的地址
[root@master1 ~]# vim /etc/keepalived/keepalived.conf
global_defs {
    router_id LVS_DEVEL
script_user root
    enable_script_security
}
vrrp_script chk_apiserver {
    script "/etc/keepalived/check_apiserver.sh"
    interval 5
    weight -5
    fall 2
rsie 1
}
vrrp_instance VI_1 {
    state MASTER
    interface ens33               # master 网卡
    mcast_src_ip 172.20.10.6      # master ip
    virtual_router_id 51
    priority 101
    advert_int 2
    authentication {
        auth_type PASS
        auth_pass K8SHA_KA_AUTH
    }
    virtual_ipaddress {
        172.20.10.9          # vip
    }
    track_script {
        chk_apiserver
    }
}

# 所有master节点配置keepalived健康检查文件：下载健康检查文件check_apiserver.sh
[root@localhost ~]# chmod +x /etc/keepalived/check_apiserver.sh

# 启动所有服务
[root@master1 ~]# systemctl daemon-reload
[root@master1 ~]# systemctl enable --now haproxy
[root@master1 ~]# systemctl enable --now keepalived

# 检查vip是否可用，一般在master1上面执行ifconfig/ip a 可以看见绑定的ip
[root@master1 ~]# ifconfig
[root@master1 ~]# ping 172.20.10.9

# master节点免密登录
[root@master1 ~]# ssh-keygen -t rsa
[root@master1 ~]# cd /root && ssh-copy-id .ssh/id_rsa.pub root@所有节点主机名字
```



### 初始化集群

在master1上初始化，其他节点加入集群的方式操作。

```shell
# 查看kubeadm config默认初始化配置文件
[root@master1 ~]# kubeadm config print init-defaults
[root@master1 ~]# vim kubeadm-config.yaml
apiVersion: kubeadm.k8s.io/v1beta4
bootstrapTokens:
- groups:
  - system:bootstrappers:kubeadm:default-node-token
  token: abcdef.0123456789abcdef
  ttl: 24h0m0s
  usages:
  - signing
  - authentication
kind: InitConfiguration
localAPIEndpoint:
  advertiseAddress: 172.20.10.6       # 本机IP
  bindPort: 6443
nodeRegistration:
  criSocket: unix:///var/run/containerd/containerd.sock
  imagePullPolicy: IfNotPresent
  imagePullSerial: true
  name: master1              # 本机名
  taints:
  - effect: NoSchedule                     
    key: node-role.kubernetes.io/master      
timeouts:
  controlPlaneComponentHealthCheck: 4m0s
  discovery: 5m0s
  etcdAPICall: 2m0s
  kubeletHealthCheck: 4m0s
  kubernetesAPICall: 1m0s
  tlsBootstrap: 5m0s
  upgradeManifests: 5m0s
---
apiServer: {}
apiVersion: kubeadm.k8s.io/v1beta4
caCertificateValidityPeriod: 87600h0m0s
certificateValidityPeriod: 8760h0m0s
certificatesDir: /etc/kubernetes/pki
clusterName: kubernetes
controlPlaneEndpoint： "172.20.10.9:16443"         # 虚拟IP和haproxy端口
controllerManager: {}
dns: {}
encryptionAlgorithm: RSA-2048
etcd:
  local:
    dataDir: /var/lib/etcd
imageRepository: registry.cn-hangzhou.aliyuncs.com/google_containers    # 镜像仓库
kind: ClusterConfiguration
kubernetesVersion: 1.32.0        # k8s版本
networking:
  dnsDomain: cluster.local
  serviceSubnet: 10.96.0.0/12    # service网段
  podSubnet: 10.244.0.0/16       # pod网段
proxy: {}
scheduler: {}
---
apiVersion: kubeproxy.config.k8s.io/v1alpha1
kind: KubeProxyConfiguration
mode: ipvs

# 更新kubeadm，使其符合最新版本规范
[root@master1 ~]# kubeadm config migrate --old-config kubeadm-config.yaml --new-config new-config.yaml

# 将new-config.yaml文件复制到其他master节点
[root@master1 ~]# for i in master2 master3; do scp new-config.yaml $i/root/; done

# 拉取镜像,每个master都要执行
[root@master1 ~]# kubeadm config images pull --config /root/new-config.yaml

# 初始化集群,仅在master1上执行
[root@master1 ~]# kubeadm init --config /root/new-config.yaml --upload-certs

# master2,master3加入集群，在master2,master3上执行
[root@masterx ~]# kubeadm join xxxxxxxxxxxxxxxxxx

# node节点加入集群
[root@nodexx ~]# kubeadm join xxxxxxxxxxxxxxxxxx

# 查看node状态
[root@master1 ~]# kubectl get nodes
```



### 安装网络插件

```shell
# 获取calico.yaml
[root@master1 ~]# curl https://raw.githubusercontent.com/projectcalico/calico/v3.29.3/manifests/calico-etcd.yaml -o calico.yaml

# 修改pod_cidr
[root@master1 ~]# POD_SUBNET=`cat /etc/kubernetes/manifests/kube-controller-manager.yaml | grep cluster-cidr= | awk -F='{print $NF}'`
[root@master1 ~]# sed -i "s#POD_SUBNET_CIDR#${POD_SUBNET}#g" calico.yaml

# 部署calico
[root@master1 ~]# kubectl apply -f calico.yaml

# 再次查看集群节点状态，确定集群节点处于ready状态
[root@master1 ~]# kubectl get nodes
```



### 集群校验

```shell
# 节点可用性
[root@master1 ~]# kubectl get nodes

# k8s服务及dns是否正常
[root@master1 ~]# kubectl get svc
[root@master1 ~]#kubectl get svc -n kube-system
# 测试服务端口可用性
[root@master1 ~]# telnet 10.10.0.1 443

# 验证pod与节点通信是正常,使用busybox工具镜像分别ping不同节点上面的pod地址
[root@master1 ~]# kubectl run -i -t busybox --image=busybox --restart=Never
/ # ping 172.20.10.6                 # pod与节点通信是否正常

# 验证service是否正常
[root@master1 ~]# kubectl create deployment nginx --image=docker.io/library/nginx:latest
[root@master1 ~]# kubectl expose deployment nginx --port=8000 --target-port=80
[root@master1 ~]# curl http://<svc_ip>/

# 验证pod与service通信是否正常
[root@master1 ~]# kubectl run -i -t busybox --image=busybox --restart=Never
/ # nslookup nginx        # 域名解析是否正常
/ # curl http://nginx/    # 访问service
```

