---
layout: post
title: linux 正向代理
category: proxy
comments: false
---


#### centos 安装正向代理 
---
  * 安装使用 nginx
  * 配置 正向代理
  * 使用正向代理
  

  
---

### 安装 nginx
sudo apt-get update  
sudo apt-get install nginx

### 配置正向代理
    server {
    resolver 8.8.8.8 114.114.114.114; #指定DNS服务器IP地址
    listen 8083;
    location / {
        proxy_pass http://$http_host$request_uri; #设定代理服务器的协议和地址
        proxy_buffers 256 4k;
        proxy_max_temp_file_size 0k;
        proxy_connect_timeout 30;
        proxy_send_timeout 60;
        proxy_read_timeout 60;
    }

    server {
    resolver 8.8.8.8 114.114.114.114; #指定DNS服务器IP地址
    listen 8084;
    location / {
        proxy_pass http://$http_host$request_uri; #设定代理服务器的协议和地址
        proxy_buffers 256 4k;
        proxy_max_temp_file_size 0k;
        proxy_connect_timeout 30;
        proxy_send_timeout 60;
        proxy_read_timeout 60;
    }


------------------------------------

#### 使用代理

### 测试
```
    curl  --proxy 127.0.0.1:8083 http://www.52os.net/
```
### https

```
    curl  --proxy 127.0.0.1:8084  http://www.google.com
```

------------------------------------
