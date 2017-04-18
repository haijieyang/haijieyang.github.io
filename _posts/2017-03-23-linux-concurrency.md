---
layout: post
title: k8s 学习篇二
category: k8s
comments: false
---


## k8s 安装dashboad
---
  * 下载kubernetes-dashboard.yaml
  * 修改文件
  * 启动服务
  * 添加认证

  
---

### 下载

```
wget https://rawgit.com/kubernetes/dashboard/master/src/deploy/kubernetes-dashboard.yaml
```
---

### 修改kubernetes-dashboard.yaml

```
# - --apiserver-host=http://my-address:port
修改成  - --apiserver-host=http://192.168.152.179:8080
```
在node节点为docker添加环境变量防止拉取镜像时失败  
cat /opt/kubernetes/cfg/docker
```
HTTP_PROXY=127.0.0.1:8118
```

------------------------------------

### 启动服务
###### 创建服务

```
kubectl create -f  kubernetes-dashboard.yaml
```
###### 查看服务启动状态
```
kubectl  describe pods --namespace=kube-system
```
###### 删除服务
```
kubectl delete -f  kubernetes-dashboard.yaml
```
------------------------------------

