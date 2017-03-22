---
layout: post
title: k8s 学习篇一
category: k8s
comments: false
---


## k8s 集群安装
---
  * 组件关系
  * 克隆kubernetes仓库
  * 使用 build.sh下载必要的组件
  * 安装etcd
  * 安装flannel  
  * 安装master组件
  * 安装node组件
  * 制作证书
  * master 组件安装
  * slave 组件安装
  
---

### 组件关系

```
master 需安装 apiserver control-manager scheduler
node   需安装 kubelet proxy 
all    需要装 etcd flannel
```
---

### 克隆kubernetes仓库

```
git clone https://github.com/kubernetes/kubernetes.git

```

------------------------------------

### 使用 build.sh 安装

###### 下载必要组件

```
cd kubernetes/cluster/centos
./build.sh download  
./build.sh unpack
```

###### 创建工作目录
```
sudo mkdir -p /opt/kubernetes/{bin,cfg}
cp binary/master/bin/* /opt/kubernetes/bin/
```
------------------------------------

### 安装etcd

###### 修改etcd.sh

```
ETCD_NAME=${1:-"default"}  #default修改成节点名容易辨识
ETCD_LISTEN_IP=${2:-"0.0.0.0"} #0.0.0.0修改成etho ip地址
ETCD_INITIAL_CLUSTER=${3:-} 

#修改成ETCD_INITIAL_CLUSTER=${3:-"default=http://192.168.152.179:2380,default1=http://192.168.152.180:2380"}

```


###### 使用脚本安装
```
cd kubernetes/cluster/centos/master/scripts
sudo sh etcd.sh
```

------------------------------------

### 安装flannel
###### 修改flannel.sh

```
ETCD_SERVERS=${1:-"http://8.8.8.18:4001"} 
#mater修改成ETCD_SERVERS=${1:-"http://192.168.152.179:4001"}
#node 修改成ETCD_SERVERS=${1:-"http://192.168.152.180:4001"}
```
###### 执行脚本flannel.sh
```
sudo sh flannel.sh
```
---
### 制作证书
```
mkdir -p /srv/kubernetes/ && cd /srv/kubernetes/
openssl genrsa -out ca.key 2048
openssl req -x509 -new -nodes -key ca.key -subj "/CN=${MASTER_IP}" -days 10000 -out ca.crt
openssl genrsa -out server.key 2048
openssl req -new -key server.key -subj "/CN=${MASTER_IP}" -out server.csr
openssl x509 -req -in server.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out server.crt -days 10000
openssl x509 -noout -text -in ./server.crt
```
---
### master 节点安装组件
###### 执行脚本安装
```
sudo sh apiserver.sh
sudo sh controller-manager.sh
sudo sh scheduler.sh
# 需要修改 masterip 
```
---
### slave 节点安装组件
###### 执行脚本安装
```
sudo sh docker.sh
sudo sh kubelet.sh
sudo sh proxy.sh
# 执行脚本前需执行以下步骤
cp -rf ./bin/docker* /usr/bin/
```

