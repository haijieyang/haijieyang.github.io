---
layout: post
title: k8s 学习篇四
category: k8s
comments: false
---


## k8s 安装monitoring
---
  * 克隆heapster仓库
  * 修改配置文件
  * 启动服务
  * grafana添加datasource

  
---

### 克隆仓库heapster

```
git clone https://github.com/kubernetes/heapster
```
---

### 修改配置文件
配置文件路径 ./heapster/deploy/kube-config/influxdb
###### heapster-deployment.yaml
```
--source=kubernetes:http:
修改成 --source=kubernetes:http://192.168.152.179:8080?inClusterConfig=false
```
###### grafana-deployment.yaml
```
value: /
#修改成 value: /api/v1/proxy/namespaces/kube-system/services/monitoring-grafana/
```

------------------------------------

### 启动服务
###### 创建deployment

```
cd ./heapster/deploy/
./kube.sh start

```
###### 查看服务启动状态
```
启动正常后几分钟就能看到cpu memory的状态了
```
------------------------------------
### grafana添加datasource
###### grafana访问地址
```
http://serverip:8080/api/v1/proxy/namespaces/kube-system/services/monitoring-grafana/
```

###### 点击datasource
```
type InfluxDB
#添加influxdb的一些基本参数
database k8s
User root
Password root
```

