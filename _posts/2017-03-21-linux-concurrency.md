---
layout: post
title: 安装k8s
category: k8s
comments: false
---


#### centos 安装k8s
---
  * 组件关系
  * 克隆kubernetes仓库
  * 使用 build.sh下载必要的组件
  * 安装etcd
  * 安装flannel  
  * 安装master组件
  * 安装node组件
 
  
---

### 组件关系

###### master 和node都需要安装etcd flannel

master | node |   all   
---|---|---
apiserver       | kubelet | etcd
control-manager | proxy   | flannel
scheduler       | 

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

###### 使用脚本安装
```
cd kubernetes/cluster/centos/master/scripts
sudo sh etcd.sh
```
###### 修改etcd.sh

```
ETCD_NAME=${1:-"default"}  #default修改成节点名容易辨识
ETCD_LISTEN_IP=${2:-"0.0.0.0"} #0.0.0.0修改成etho ip地址
ETCD_INITIAL_CLUSTER=${3:-} 

#修改成ETCD_INITIAL_CLUSTER=${3:-"default=http://192.168.152.179:2380,default1=http://192.168.152.180:2380"}

```

------------------------------------

###### 安装flannel

```
sudo sh flannel.sh
```

###### 修改flannel.sh

```
ETCD_SERVERS=${1:-"http://8.8.8.18:4001"} 
#mater修改成ETCD_SERVERS=${1:-"http://192.168.152.179:4001"}
#node 修改成ETCD_SERVERS=${1:-"http://192.168.152.180:4001"}
```


