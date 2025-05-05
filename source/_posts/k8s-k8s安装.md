---
title: k8s安装方式
date: 2020-04-16 12:20:20
tags:
- Kubernetes
categories:
- Kubernetes
---

## kind安装

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



## minikube安装

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



## kubeadm安装

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



## 二进制安装

适用场景：生产环境。

### 集群信息

kubernetes版本：v1.33

节点信息

| 节点       | IP             |
| ---------- | -------------- |
| master01   | 192.168.10.10  |
| master02   | 192.168.10.11  |
| master03   | 192.168.10.12  |
| master hpa | 192.168.10.100 |
| node01     | 192.168.10.13  |



### 二进制安装包和相关资料下载

1）k8s server binaries下载

a. 打开kubernetes github仓库：https://github.com/kubernetes/kubernetes/tree/master

b. 打开CHANGELOG目录，选择对应版本(1.33)。

c. 下载对应Server Binaries包(kubernetes-server-linux-amd64.tar.gz)。

d. 搜索etcd，确定此版本支持的etcd版本号(v3.5.21)



2）etcd binaries下载

a. 打开etcd github仓库：https://github.com/etcd-io/etcd

b. 进入release页面：https://github.com/etcd-io/etcd/releases

c. 下载etcd包：https://github.com/etcd-io/etcd/releases/download/v3.5.21/etcd-v3.5.21-linux-amd64.tar.gz



### 安装k8s和etcd二进制文件

1）解压server binaries包

```shell
[root@master01 ~]# tar -xf kubernetes-server-linux-amd64.tar.gz --strip-components=3 -C /usr/local/bin kubernetes/server/bin/kube{let,ctl,-apiserver,-controller-manager,-scheduler,-proxy}
```

2）解压etcd包

```shell
[root@master01 ~]# tar -xf etcd-v3.5.21-linux-amd64.tar.gz --strip-components=1 -C /usr/local/bin etcd-v3.5.21-linux-amd64/etcd{,ctl}
```

3）将包发送到其他节点

```shell
# master节点
[root@master01 ~]# MasterNodes='master02,master03'
[root@master01 ~]# for NODE in $MasterNodes; do echo $NODE; scp /usr/local/bin/kube{let,ctl,-apiserver,-controller-manager,-scheduler,-proxy} $NODE:/usr/local/bin/; done
[root@master01 ~]# for NODE in $MasterNodes; do echo $NODE; scp /usr/local/bin/etcdctl $NODE:/usr/local/bin/; done

# node节点
[root@master01 ~]# WorkNodes='node01'
[root@master01 ~]# for NODE in $WorkNodes; do scp /usr/local/bin/kube{let,-proxy} $NODE:/usr/local/bin/; done
```

4）所有节点创建网络插件目录

```shell
# master01&master02&master03&node01
[root@master01 ~]# mkdir -p /opt/cni/bin
```

5）生成etcd证书

```shell
# 下载证书工具
[root@master01 ~]# wget "https://pkg.cfssl.org/R1.2/cfssl_linux-amd64" -O /usr/local/cfssl
[root@master01 ~]# wget "https://pkg.cfssl.org/R1.2/cfssljson_linux-amd64" -O /usr/local/bin/cfssljson
[root@master01 ~]# chmod +x /usr/local/bin/cfssl /usr/local/bin/cfssljson

# 生成etcd证书
# 所有master节点(master01&master02&master03)创建etcd证书目录
[root@maser01 ~]#  mkdir -p /etc/etcd/ssl
# 所有节点(master01&master02&master03&node01)创建kubernetes相关目录
[root@master01 ~]# mkdir -p /etc/kubernetes/pki

# 生成etcd CA证书和CA证书的key(注意hostname要替换成自己的)
[root@master01 ~]# cfssl gencert -initca etcd-ca-csr.json | cfssljson -bare /etc/etcd/ssl/etcd-ca
[root@master01 ~]# cfssl gencert \
  -ca=/etc/etcd/ssl/etcd-ca.pem \
  -ca-key=/etc/etcd/ssl/etcd-ca-key.pem \
  -config=ca-config.json
  -hostname=127.0.0.1,master01,master02,master03,192.168.10.10,192.168.10.11,192.168.10.12 \
  -profile=kubernetes \
  etcd-csr.json | cfssljson -bare /etc/etcd/ssl/etcd
  
# 复制证书到其它master节点
[root@master01 ~]# MasterNodes='master02,master03'
[root@master01 ~]# for NODE in $MasterNodes; do 
	ssh $NODE "mkdir -p /etc/etcd/ssl"
	for FILE in etcd-ca-key.pem etcd-ca.pem etcd-key.pem etcd.pem; do
		scp /etc/etcd/ssl/${FILE} $NODE:/etc/etcd/ssl/${FILE}
	done
done
```

6）生成kubernetes组件证书

a.生成根证书

```shell
# 生成根证书
[root@master01 ~]# cfssl gencert -initca ca-csr.json | cfssljson -bare /etc/kubernetes/pki/ca

# 查看证书
[root@master01 ~]# ls /etc/kubernetes/pki/ca
```

b.生成apiserver证书

```shell
# 生成apiserver证书（master节点ip替换成自己的）
[root@master01 ~]# cfssl gencert -ca=/etc/kubernetes/pki/ca.pem -ca-key=/etc/kubernetes/pki/ca-key.pem -config=ca-config.json -hostname=10.10.0.1,192.168.10.100,127.0.0.1,kubernetes,kubernetes.default,kubernetes.default.svc,kubernetes.default.svc.cluster,kubernetes.default.svc.cluster.local,192.168.10.10,192.168.10.11,192.168.10.12 -profile=kubernetes apiserver-csr.json | cfssljson -bare /etc/kubernetes/pki/apiserver

# 生成apiserver的聚合证书
[root@master01 ~]# cfssl gencert -initca front-proxy-ca-csr.json | cfssljson -bare /etc/kubernetes/pki/front-proxy-ca
[root@master01 ~]# cfssl gencert -ca=/etc/kubernetes/pki/front-proxy-ca.pem -ca-key=/etc/kubernetes/pki/front-proxy-ca-key.pem -profile=kubernetes front-proxy-client-csr.json | cfssljson -bare /etc/kubernetes/pki/front-proxy-client
```

c.生成controller-manager证书

```shell
# 生成controller-manager的证书
[root@master01 ~]# cfssl gencert \
	-ca=/etc/kubernetes/pki/ca.pem \
	-ca-key=/etc/kubernetes/pki/ca-key.pem \
	-config=ca-config.json \
	-profile=kubernetes \
	manager-csr.json | cfssljson -bare /etc/kubernetes/pki/controller-manager
	
# set-cluster:设置一个集群项，相当于我们访问apiserver的配置文件（注意vip）
[root@master01 ~]# kubectl config set-cluster kubernetes \
	--certificate-authority=/etc/kubernetes/pki/ca.pem \
	--embed-certs=true \
	--server=https://192.168.10.100:8443 \
	--kubeconfig=/etc/kubernetes/controller-manager.kubeconfig
	
# 设置一个环境项，一个上下文
[root@master01 ~]# kubectl config set-context system:kube-controller-manager@kubernetes \
	--cluster=kubernetes \
	--user=system:kube-controller-manager \
	--kubeconfig=/etc/kubernetes/controller-manager.kubeconfig

# set-credentials，设置一个用户项
[root@master01 ~]# kubectl config set-credentials system:kube-controller-manager \
	--client-certificate=/etc/kubernetes/pki/controller-manager.pem \
	--client-key=/etc/kubernetes/pki/controller-manager-key.pem \
	--embed-certs=true \
	--kubeconfig=/etc/kubernetes/controller-manager.kubeconfig

# 使用某个环境当做默认环境
[root@master01 ~]# kubectl config use-context system:kube-controller-manager@kubernetes \
	--kubeconfig=/etc/kubernetes/controller-manager.kubeconfig
```

d.生成scheduler证书

```shell
[root@master01 ~]# cfssl gencert \
	-ca=/etc/kubernetes/pki/ca.pem \
	-ca-key=/etc/kubernetes/pki/ca-key.pem \
	-config=ca-config.json \
	-profile=kubernetes \
	scheduler-csr.json | cfssljson -bare /etc/kubernetes/pki/scheduler

# set-cluster:设置一个集群项，相当于我们访问apiserver的配置文件（注意vip）
[root@master01 ~]# kubectl config set-cluster kubernetes \
	--certificate-authority=/etc/kubernetes/pki/ca.pem \
	--embed-certs=true \
	--server=https://192.168.10.100:8443 \
	--kubeconfig=/etc/kubernetes/scheduler.kubeconfig

[root@master01 ~]# kubectl config set-credentials system:kube-scheduler \
	--client-certificate=/etc/kubernetes/pki/scheduler.pem \
	--client-key=/etc/kubernetes/pki/scheduler-key.pem \
	--embed-certs=true \
	--kubeconfig=/etc/kubernetes/scheduler.kubeconfig

[root@master01 ~]# kubectl config set-context system:kube-scheduler@kubernetes \
	--cluster=kubernetes \
	--user=system:kube-scheduler \
	--kubeconfig=/etc/kubernetes/scheduler.kubeconfig

[root@master01 ~]# kubectl config use-context system:kube-scheduler@kubernetes \
	--kubeconfig=/etc/kubernetes/scheduler.kubeconfig	
```

e.生成admin.kubernetes

```shell
[root@master01 ~]# cfssl gencert \
	-ca=/etc/kubernetes/pki/ca.pem \
	-ca-key=/etc/kubernetes/pki/ca-key.pem \
	-config=ca-config.json \
	-profile=kubernetes \
	admin-csr.json | cfssljson -bare /etc/kubernetes/pki/admin

[root@master01 ~]# kubectl config set-cluster kubernetes \
	--certificate-authority=/etc/kubernetes/pki/ca.pem \
	--embed-certs=true \
	--server=https://192.168.10.100:8443 \
	--kubeconfig=/etc/kubernetes/admin.kubeconfig

[root@master01 ~]# kubectl config set-credentials kubernetes-admin \
	--client-certificate=/etc/kubernetes/pki/admin.pem \
	--client-key=/etc/kubernetes/pki/admin-key.pem \
	--embed-certs=true \
	--kubeconfig=/etc/kubernetes/admin.kubeconfig

[root@master01 ~]# kubectl config set-context kubernetes-admin@kubernetes |
	--cluster=kubernetes \
	--user=kubernetes-admin \
	--kubeconfig=/etc/kuernetes/admin.kubeconfig

[root@master01 ~]# kubectl config use-context kubernetes-admin@kubernetes \
	--kubeconfig=/etc/kubernetes/admin.kubeconfig
```

f.创建ServiceAccount key&secret

```shell
[root@master01 ~]# openssl genrsa -out /etc/kubernetes/pki/sa.key 2048
[root@master01 ~]# openssl rsa -in /etc/kubernetes/pki/sa.key -pubout -out /etc/kubernetes/pki/sa.pub
```

g.发送证书至其他节点

```shell
[root@master01 ~]# for NODE in master02,master03; do
	for FILE in $(ls /etc/kubernetes/pki | grep -v etcd); do
		scp /etc/kubernetes/pki/${FILE} ${NODE}:/etc/kubernetes/pki/${FILE};
	done;
	for FILE in admin.kubeconfig controller-manager.kubeconfig scheduler.kubeconfig; do
		scp /etc/kubernetes/${FILE} ${NODE}:/etc/kubernetes/${FILE};
	done;
done

# 查看证书文件
[root@master01 ~]# ls /etc/kubernetes/pki/
```

### etcd集群配置

创建Service服务

```shell
# 所有Master节点创建etcd service并启动和etcd的证书目录
[root@master01 ~]# vim /usr/lib/systemd/system/etcd.service
[Unit]
Description=Etcd Service
Documentation=https://coreos.com/etcd/docs/latest/
After=network.target

[Service]
Type=notify
ExecStart=/usr/local/bin/etcd --config-file=/etc/etcd/etcd.config.yml
Restart=on-failure
RestartSec=10
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
Alias=etcd3.service
```

创建Etcd配置文件

```shell
# 所有master节点创建etcd配置文件(master02&master03节点需要替换ip)
[root@master01 ~]# vim /etc/etcd/etcd.config.yml
name: 'master01'
date-dir: /var/lib/etcd
wal-dir: /var/lib/etcd/wal
snapshot-count: 5000
heartbeat-interval: 100
election-timeout: 1000
quota-backend-bytes: 0
listen-peer-urls: 'https://192.168.10.10:2380'
listen-client-urls: 'https://192.168.10.10:2379,http://127.0.0.1:2379'
max-snapshots: 3
max-wals: 5
cors:
initial-advertise-peer-urls: "https://192.168.10.10:2380"
advertise-client-urls: 'https://192.168.10.10:2379'
discovery:
discovery-fallback: 'proxy'
discovery-proxy:
discovery-srv:
initial-cluster: 'master01=https://192.168.10.10:2380,master02=https://192.168.10.11:2380,master03=https://192.168.10.12:2380'
initial-cluster-token: 'etcd-k8s-cluster'
initial-cluster-state: 'new'
strict-reconfig-check: false
enable-v2: true
enable-pprof: true
proxy: 'off'
proxy-failure-wait: 5000
proxy-refresh-interval: 30000
proxy-dial-timeout: 1000
proxy-write-timeout: 5000
proxy-read-timeout: 0
client-transport-security:
  cert-file: '/etc/kubernetes/pki/etcd/etcd.pem'
  key-file: '/etc/kubernetes/pki/etcd/etcd-key.pem'
  client-cert-auth: true
  trusted-ca-file: '/etc/kubernetes/pki/etcd/etcd-ca.pem'
  auto-tls: true
 peer-transport-security:
   cert-file: '/etc/kubernetes/pki/etcd/etcd.pem'
   key-file: '/etc/kubernetes/pki/etcd/etcd-key.pem'
   peer-client-cert-auth: true
   trusted-ca-file: '/etc/kubernetes/pki/etcd/etcd-ca.pem'
   auto-tls: true
 debug: false
 log-package-levels:
 log-outputs: [default]
 force-new-cluster: false
 
 [root@master01 ~]# mkdir /etc/kubernetes/pki/etcd
 [root@master01 ~]# ln -s /etc/etcd/ssl/* /etc/kubernetes/pki/etcd/
 [root@master01 ~]# systemctl daemon-reload
 [root@master01 ~]# systemctl enable --now etcd
 
 # 查看etcd状态
 [root@master01 ~]# export ETCDCTL_API=3
 [root@master01 ~]# etcdctl --endpoints="192.168.10.10:2379,192.168.10.11:2379,192.168.10.12:2379" --cacert=/etc/kubernetes/pki/etcd/etcd-ca.pem --cert=/etc/kubernetes/pki/etcd/etcd.pem --key=/etc/kubernetes/pki/etcd/etcd-key.pem endpoint status --write-out=table
```



### master高可用

参照前文kubeadm安装里的master高可用进行配置。



### kubernetes组件配置启动服务

所有节点创建相关目录

```shell
[root@master01 ~]# mkdir -p /etc/kubernetes/manifests/ /etc/systemd/system/kubelet.service.d /var/lib/kubelet /var/log/kubernetes
```



a.apiserver

所有master节点创建kube-apiserver service。注意advertise-address各节点不同。

```shell
[root@master01 ~]# vim /usr/lib/systemd/system/kube-apiserver.service
[Unit]
Description=Kubernetes API Server
Documentation=https://github.com/kubernetes/kubernetes
After=network.target

[Service]
ExecStart=/usr/local/bin/kube-apiserver \
	--v=2 \
	--logtostderr=true \
	--allow-privileged=true \
	--bind-address=0.0.0.0 \
	--secure-port=6443 \
	--insecure-port=0 \
	--advertise-address=192.168.10.10 \
	--service-cluster-ip-range=10.10.0.0/16 \
	--service-node-port-range=30000-32767 \
	--etcd-servers=https://192.168.10.10:2379,https://192.168.10.11:2379,https://192.168.10.12:2379 \
	--etcd-cafile=/etc/etcd/ssl/etcd-ca.pem \
	--etcd-certfile=/etc/etcd/ssl/etcd.pem \
	--etcd-keyfile=/etc/etcd/ssl/etcd-key.pem \
	--client-ca-file=/etc/kubernetes/pki/ca.pem \
	--tls-cert-file=/etc/kubernetes/pki/apiserver.pem \
	--tls-private-key-file=/etc/kubernetes/pki/apiserver-key.pem \
	--kubelet-client-certificate=/etc/kubernetes/pki/apiserver.pem \
	--kubelet-client-key=/etc/kubernetes/pki/apiserver-key.pem \
	--service-account-key-file=/etc/kubernetes/pki/sa.pub \
	--service-account-signing-key-file=/etc/kubernetes/pki/sa.key \
	--service-account-issuer=https://kubernetes.default.svc.cluster.local \
	--kubelet-preferred-address-types=InternalIP,ExternalIP, Hostname \
	-- enable-admission-plugins=NamespaceLifecycle,LimitRanger,ServiceAccount,DefaultStorageClass,DefaultTolerationSeconds,NodeRestriction,ResourceQuota \
	--authorization-mode=Node,RBAC \
	--enable-bootstrap-token-auth=true \
	--requestheader-client-ca-file=/etc/kubernetes/pki/front-proxy-ca.pem \
	--proxy-client-cert-file=/etc/kubernetes/pki/front-proxy-client.pem \
	--proxy-client-key-file=/etc/kubernetes/pki/pki/front-proxy-client-key.pem \
	--requestheader-allowed-names=aggregator \
	--requestheader-group-headers=X-Remote-Group \
	--requestheader-extra-headers-prefix=X-Remote-Extra- \
	--requestheader-username-headers=X-Remote-User
	# --token-auth-file=/etc/kubernetes/token.csv
	
Restart=on-failure
RestartSec=10s
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target

# 开启kube-apiserver
[root@master01 ~]# systemctl daemon-reload && systemctl enable --now kube-apiserver

# 检测kuber-apiserver状态
[root@master01 ~]# systemctl status kube-apiserver
```



b.controller-manager配置

所有master节点创建controller-manager service。

```shell
[root@master01 ~]# vim /usr/lib/systemd/system/kube-controller-manager.service
[Unit]
Description=Kubernetes Controller Manager
Documentation=https://github.com/kubernetes/kubernetes
After=network.target

[Service]
ExecStart=/usr/local/bin/kube-controller-manager \
	--v=2 \
	--logtostderr=true \
	--address=127.0.0.1 \
	--root-ca-file=/etc/kubernetes/pki/ca.pem \
	--cluster-signing-cert-file=/etc/kubernetes/pki/ca.pem \
	--cluster-signing-key-file=/etc/kubernetes/pki/ca-key.pem \
	--service-account-private-key-file=/etc/kubernetes/pki/sa.key \
	--kubeconfig=/etc/kubernetes/controller-manager.kubeconfig \
	--leader-elect=true \
	--use-service-account-credentials=true \
	--node-monitor-grace-period=40s \
	--node-monitor-period=5s \
	--pod-eviction-timeout=2m0s \
	--controllers=*,bootstrapsigner,tokencleaner \
	--allocate-node-cidrs=true \
	--cluster-cidr=172.16.0.0/12 \
	--requestheader-client-ca-file=/etc/kubernetes/pki/front-proxy-ca.pem \
	--node-cidr-mask-size=24

Restart=always
RestartSec=10s

[Install]
WantedBy=multi-user.target

# 开启kube-controller-manager
[root@master01 ~]# systemctl daemon-reload && systemctl enable --now kube-controller-manager

# 查看启动状态
[root@master01 ~]# systemctl status kube-controller-manager
```



c.scheduler

所有master节点配置kube-scheduler service。

```shell
[root@master01 ~]# vim /usr/lib/systemd/system/kube-scheduler.service
[Unit]
Description=Kubernetes Scheduler
Documentation=https://github.com/kubernetes/kubernetes
After=network.target

[Service]
ExecStart=/usr/local/bin/kube-scheduler \
	--v=2 \
	--logtostderr=true \
	--address=127.0.0.1 \
	--leader-elect=true \
	--kubeconfig=/etc/kubernetes/scheduler.kubeconfig

Restart=always
RestartSec=10s

[Install]
WantedBy=multi-user.target

# 开启kube-scheduler
[root@master01 ~]# systemctl daemon-reload && systemctl enable --now kube-scheduler

# 查看服务状态
[root@master01 ~]# systemctl status kube-scheduler
```



### 配置bootstrap TLS

master01节点执行即可。



### node节点配置

a. 配置kubelet

```shell
# 复制master01节点证书至node节点
[root@master01 ~]# cd /etc/kubernetes/
[root@master01 ~]# for NODE in master02 master03 node01; do
	ssh $NODE mkdir -p /etc/kubernetes.pki
	for FILE in pki/ca.pem pki/ca-key.pem pki/front-proxy-ca.pem bootstrap-kubelet.kubeconfig; do
		scp /etc/kubernetes/$FILE $NODE:/etc/kubernetes/$FILE
	done
done

# 所有节点创建相关目录
[root@master01 ~]# mkdir -p /var/lib/kubelet /var/log/kubernetes /etc/systemd/system/kubelet.service.d /etc/kubernetes/manifests/

# 所有节点配置kubelet service
[root@master01 ~]# vim /usr/lib/systemd/system/kubelet.service
[Unit]
Description=Kubernetes Kubelet
Documentation=https://github.com/kubernetes/kubernetes
After=docker.service
Requires=docker.service

[Service]
ExecStart=/usr/local/bin/kubelet
Restart=always
StartLimitInterval=0
RestartSec=10

[Install]
WantedBy=multi-user.target


# 使用Runtime为Containerd
[root@master01 ~]# vim /etc/systemd/system/kubelet.service.d/10-kubelet.conf
[Service]
Environment="KUBELET_KUBECONFIG_ARGS=--bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.kubeconfig --kubeconfig=/etc/kubernetes/kubelet.kubeconfig"
Environment="KUBELET_SYSTEM_ARGS=--network-plugin=cni --cni-conf-dir=/etc/cni/net.d --cni-bin-dir=/opt/cni/bin --container-runtime=remote --runtime-request-timeout=15m --container-runtime-endpoint=unix:///run/containerd/containerd.sock --cgroup-driver=systemd"
Environment="KUBELET_CONFIG_ARGS=--config=/etc/kubernetes/kubelet-conf.yml"
Environment="KUBELET_EXTRA_ARGS=--node-labels=node.kubernetes.io/node='' "
ExecStart=
ExecStart=/usr/local/bin/kubelet $KUBELET_KUBECONFIG_ARGS $KUBELET_CONFIG_ARGS $KUBELET_SYSTEM_ARGS $KUBELET_EXTRA_ARGS

# 创建kubelet配置文件
[root@master01 ~]# vim /etc/kubernetes/kubelet-conf.yml
apiVersion: kubelet.config.k8s.io/v1beta1
kind: KubeletConfiguration
address: 0.0.0.0
port: 10250
readOnlyPOrt: 10255
authentication:
  anonymous:
    enabled: false
  webhook:
    cacheTTL: 2m0s
    enabled: true
  x509:
    clientCAFile: /etc/kubernetes/pki/ca.pem
authorization:
  mode: Webhook
  webhook:
    cacheAuthorizatedTTL: 5m0s
    cacheUnauthorizedTTL: 30s
cgroupDriver: systemd
cgroupPerQOS: true
clusterDNS:
- 10.10.0.10      # clusterDNS配置Service网段的第10个地址
clusterDomain: cluster.local
containerLogMaxFiles: 5
containerLogMaxSize: 10Mi
contentType: application/vnd.kubernetes.protobuf
cpuCFSQuota: true
cpuManagerPolicy: none
cpuManagerReconcilePeriod: 10s
enableControllerAttachDetach: true
enableDebuggingHandlers: true
enforceNodeAllocatable:
- pods
eventBurst: 10
eventRecordQPS: 5
evictionHard:
  imagefs.available: 15%
  memory.available: 100Mi
  nodefs.available: 10%
  nodefs.inodesFree: 5%
evictionPressureTransitionPeriod: 5m0s
failSwapOn: true
fileCheckFrequency: 20s
hairpinMode: promiscuous-bridge
healthzBindAddress: 127.0.0.1
healthzPort: 10248
httpCheckFrequency: 20s
imageGCHighThresholdPercent: 85
imageGCLowThresholdPercent: 80
imageMinimumGCAge: 2m0s
iptablesDropBit: 15
iptablesMasqueradeBit: 14
kubeAPIBurst: 10
kubeAPIQPS: 5
makeIPTableUtilChains: true
maxOpenFiles: 1000000
maxPods: 110
nodeStatusUpdateFrequency: 10s
oomScoreAdj: -999
podPidsLimit: -1
registryBurst: 10
registryPullQPS: 5
resolvConf: /etc/resolv.conf
rotateCertificates: true
runtimeRequestTimeout: 2m0s
serializeImagePulls: true
staticPodPath: /etc/kubernetes/manifests
streamingConnectionIdleTimeout: 4h0mos
syscFrequency: 1m0s
volumeStatsAggPeriod: 1m0s

# 开启kubelet
[root@master01 ~]# systemctl daemon-reload && systemctl enable --now kubelet

# 查看kubelet状态
[root@master01 ~]# systemctl status kubelet

# 查看集群状态
[root@master01 ~]# kubectl get node
```



b. 配置kube-proxy

```shell
# master01执行
[root@master01 ~]# kubectl -n kube-system create serviceaccount kube-proxy
[root@master01 ~]# kubectl cerate cluserrolebinding system:kube-proxy --clusterrole system:node-proxier --serviceaccount kube-system:kube-proxy
[root@master01 ~]# SECRET=$(kubectl -n kuber-system get sa/kube-proxy --output=jsonpath='{.secrets[0].name}')
[root@master01 ~]# JWT_TOKEN=$(kubectl -n kube-system get secret/$SECRET --output=jsonpath='{.data.token}' | base64 -d)
[root@master01 ~]# PKI_DIR=/etc/kubernetes/pki
[root@master01 ~]# K8S_DIR=/etc/kubernetes
[root@master01 ~]# kubectl config set-cluster kubernetes \
	--certificate-authority=/etc/kubernetes/pki/ca.pem \
	--embed=certs=true \
	--server=https://192.168.10.100:8443 \
	--kubeconfig=${K8S_DIR}/kube-proxy.kubeconfig
[root@master01 ~]# kubectl config set-credentials kubernetes --token=${JWT_TOKEN} --kubeconfig=/etc/kubernetes/kube-proxy.kubeconfig
[root@master01 ~]# kubectl config set-context kubernetes --cluser=kubernetes --user=kubernetes --kubeconfig=/etc/kubernetes/kube-proxy.kubeconfig
[root@master01 ~]# kubectl config use-context kubernetes --kubeconfig=/etc/kubernetes/kube-proxy.kubeconfig

# 将kubeconfig发送至其他节点
[root@master01 ~]# for NODE in master02 master03 node01; do
	scp /etc/kubernetes/kube-proxy.kubeconfig $NODE:/etc/kubernetes/kube-proxy.kubeconfig
done

# 所有节点添加kube-proxyr的配置和service文件
```

