---
layout: post
title: k8s 三skydns
category: k8s
comments: false
---


## k8s 安装skydns
---
  * 声明环境变量
  * 修改配置文件
  * 启动服务
  * 测试服务

  
---

### 声明环境变量

```
export MASTER_IP=192.168.152.179
export DNS_DOMAIN="cluster.local"
export KUBE_APISERVER_URL=http://$MASTER_IP:8080
```
---

### 修改配置文件
文件路径 kubernetes/cluster/addons/dns
###### skydns-rc.yaml.base skydns-svc.yaml.base
```
sed -i "s/__PILLAR__DNS__DOMAIN__/${DNS_DOMAIN}/g" skydns-rc.yaml.base
sed -i "s~__PILLAR__FEDERATIONS__DOMAIN__MAP__~- --kube-master-url=${KUBE_APISERVER_URL}~g" skydns-rc.yaml.base
```

------------------------------------

### 启动服务
###### 创建rc svc
```
kubectl create -f skydns-rc.yaml.base 
kubectl create -f skydns-svc.yaml.base 
```
---

###### 测试dns 
创建一个curl Pod

```
apiVersion: V1
kind: Pod
metadata:
  name: curl-util
spec:
  container:
  - name: curl-util
  command:
  - sh
  - -c
  - while true; do sleep 1;done
```

在curl-util Pod中通过Service名称访问 my-nginx:80

```
kubectl exec curl-util -- curl -s my-nginx:80
```
