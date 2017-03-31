---
layout: post
title: nginx 四层负载均衡
category: nginx
comments: false
---

---
  * 编译nginx 
  * stream指令
  * upstream配置
  * server配置

---

### 编译nginx 

```
./configure --prefix=/usr/servers --with-stream
```
---

### stream指令

** stream在nginx.conf中是跟http同一级的 ** 

```
stream {
    upstream mysql_backend {
        ……
    }
    server {
        ……
    }
}

```

---

### upsteam配置

```
upstream mysql_backend {
       server 192.168.0.10:3306 max_fails=2 fail_timeout=10s weight=1;
       server 192.168.0.11:3306 max_fails=2 fail_timeout=10s weight=1;
       least_conn;
}

```

---

### server 配置

```
server {
    #监听端口
    listen 3308;
    #失败重试
   proxy_next_upstream on;
   proxy_next_upstream_timeout 0;
   proxy_next_upstream_tries 0;
    #超时配置
   proxy_connect_timeout 1s;
   proxy_timeout 1m;
    #限速配置
   proxy_upload_rate 0;
   proxy_download_rate 0;
    #上游服务器
   proxy_pass mysql_backend;
}
```




