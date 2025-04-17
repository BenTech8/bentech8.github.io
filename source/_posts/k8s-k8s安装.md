---
title: k8s安装方式
date: 2020-04-16 12:20:20
tags:
- Kubernetes
categories:
- Kubernetes
---

## kind安装方式

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

