---
layout: post
title: linux 使用代理
category: proxy
comments: false
---


#### centos 下使用sockt5 
---
  * 安装 shadowsocks
  * 客户端配置
  * 安装 proxychains-ng
  * 配置 proxychains  
  * 使用代理下载
  * 安装Privoxy
  * sock5代理映射为http代理
  
---

### 安装 ss
```
sudo pip install shadowsocks
```
### 添加配置 config.json
    {
     "server": serverip,
     "server_port":port,
     "local_address": "127.0.0.1",
     "local_port":1080,
     "password":"password",
     "timeout":300,
     "method":"aes-256-cfb"
    }

### 启动socket代理
```
sslocal -c config.json  -d start
```
------------------------------------

#### 安装proxychains 

####  install proxychains-ng

```
git  clone https://github.com/rofl0r/proxychains-ng.git
cd proxychains-ng
./configure
make
make install
```

### 修改配置文件
```
cp ./src/proxychains.conf  /usr/local/etc
```
```
 [ProxyList]
socks5 127.0.0.1  1080
```
------------------------------------

#### 使用代理

### 测试
```
    proxychains4  curl https://www.google.com
```
### 使用代理

```
    proxychains4 wget https://artifacts.elastic.co/downloads/logstash/logstash-5.2.2.tar.gz
```

------------------------------------

### 安装Privoxy

```
sudo apt-get install privoxy
```

### 修改配置文件
```
/etc/privoxy/config
```
```
#listen-address  localhost:8118
forward-socks5 / 127.0.0.1:1080 .
listen-address 127.0.0.1:8118
```

### 启动Privoxy
```
sudo service privoxy restart
```

### 使用privoxy
```
export http_proxy='http://localhost:8118'
export https_proxy='http://localhost:8118'
```
