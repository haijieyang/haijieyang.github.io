---
layout: post
title: docker swarm 集群实例
category: swarm
comments: true
---


#### docker编排 
---
  * 概念
  * 实例
  * 安装
  * 运行
  * 缺点  
  
---

#### 概念 

### 什么是docker编排?
docker编排是指容器调度、集群管理和可能其他主机供应配置。

### docker编排优点 
能够让安装和配置主机服务器的程序实现自动化

### docker compose mode
natively managing a cluster of Docker Engines called a swarm

------------------------------------

#### 编排工具 

### 安装 
安装1.13版本的docker

```
curl -fsSL https://get.docker.com/| sh
```

### swarm集群配置

docker swarm init


    $ docker swarm init
    Swarm initialized: current node (dxn1zf6l61qsb1josjja83ngz) is now a manager.
    To add a worker to this swarm, run the following command:
       docker swarm join \
         --token SWMTKN-1-49nj1cmql0jkz5s954yi3oex3nedyz0fb0xx14ie39trti4wxv-8vxv8rssmk743ojnwacrr2e7c \
       192.168.99.100:2377
    To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.


------------------------------------

#### swarm deploy app

docker-compose.yml

    version: '3'
    services:
      nginx:
          image: "lnmp-nginx:latest"
          ports:
              - "80:80"
          networks:
              - frontend
          depends_on:
              - php
      php:
          image: "lnmp-php:latest"
          networks:
              - frontend
              - backend
          environment:
              MYSQL_PASSWORD: Passw0rd
          depends_on:
              - mysql
      mysql:
          image: "mysql:5.7"
          volumes:
              - mysql-data:/var/lib/mysql
          environment:
              TZ: 'Asia/Shanghai'
              MYSQL_ROOT_PASSWORD: Passw0rd
          command: ['mysqld', '--character-set-server=utf8']
          networks:
              - backend
      volumes:
         mysql-data:
      networks:
         frontend:
         backend:

#### 详解
 

------------------------------------


#### 运行 

### 创建 

    docker stack deploy --compose-file  docker-compose.yml lnmp

### 滚动升级 

    docker service update --update-parallelism  --update-delay --image tomcat:7 tomcat00

### 版本回滚 

    docker service update  --rollback --update-delay 0s  my_web

------------------------------------

### 缺点 

查看日志很麻烦,docker service ps 查看各个容器都在哪些节点上，然后再一个个进去先 docker ps 找到容器 ID，然后在 docker logs <容器ID> 查看具体日志。非常繁琐

