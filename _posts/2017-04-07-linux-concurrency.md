---
layout: post
title: k8s 六安全认证
category: k8s
comments: false
---

### k8s ssl auth

---

  * ssl
  * k8s basic auth
  
---

### 1. ssl 
###### 1.1 Create a Cluster Root CA

```
openssl genrsa -out ca.key 2048
openssl req -x509 -new -nodes -key ca.key -days 10000 -out ca.crt -subj "/CN=test.com.cn"
```

###### 1.2 apiserver keypair
openssl.conf

```
[req]
req_extensions = v3_req
distinguished_name = req_distinguished_name
[req_distinguished_name]
[ v3_req ]
basicConstraints = CA:FALSE
keyUsage = nonRepudiation, digitalSignature, keyEncipherment
subjectAltName = @alt_names
[alt_names]
DNS.1 = kubernetes
DNS.2 = kubernetes.default
DNS.3 = kubernetes.default.svc
DNS.4 = kubernetes.default.svc.cluster.local
DNS.5 = k8s-master
IP.1 = 10.10.10.1 
IP.2 = 192.168.152.179
```

###### 1.3 generate apiserver keypair


```
openssl genrsa -out server.key 2048
openssl req -new -key server.key -out server.csr -subj "/CN=k8s-master" -config openssl.conf
openssl x509 -req -in server.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out server.crt -days 365 -extensions v3_req -extfile openssl.conf
```

###### 1.4 generate node keypair

```
openssl genrsa -out kubelet_client.key 2048
openssl req -new -key kubelet_client.key -out kubelet_client.csr -subj "/CN=k8s-master" 
openssl x509 -req -in kubelet_client.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out kubelet_client.crt -days 365 
```

---

### 2. k8s basic auth

###### 2.1 kube-apiserver.service

```
# 添加启动参数
--secure-port=443
--basic-auth-file=/etc/k8s/basic_auth_file
```

###### 2.2 create basic_auth_file

```
admin,admin,1
system,system,2
```
