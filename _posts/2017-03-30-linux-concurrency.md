---
layout: post
title: k8s 学习篇五
category: k8s
comments: false
---


## k8s 手动安装calico
---
  * 运行 calico/node & 配置节点 
  * 下载配置calico cni组件
  * 安装calico network policy controller
  * 修改kubelet 启动参数
  * 修改 Kube-Proxy
  
  
---

### 运行 calico/node & 配置节点

###### 下载并启动calicoctl

```
# Download and install `calicoctl`
wget https://github.com/projectcalico/calico-containers/releases/download/v0.22.0/calicoctl 
sudo chmod +x calicoctl

# Run the calico/node container
sudo ETCD_ENDPOINTS=http://<ETCD_IP>:<ETCD_PORT> ./calicoctl node
```

###### calico-node.service

```
[Unit]
Description=calicoctl node
After=docker.service
Requires=docker.service

[Service]
User=root
Environment=ETCD_ENDPOINTS=http://<ETCD_IP>:<ETCD_PORT>
PermissionsStartOnly=true
ExecStartPre=/usr/bin/wget -N -P /opt/bin https://github.com/projectcalico/calico-containers/releases/download/v0.22.0/calicoctl
ExecStartPre=/usr/bin/chmod +x /opt/bin/calicoctl
ExecStart=/opt/bin/calicoctl node --detach=false
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
```

```
把<ETCD_IP>:<ETCD_PORT>替换成你的etcd 配置
```
---

### 下载配置calico cni组件
###### 下载二进制文件并添加可执行权限
```
wget -N -P /opt/cni/bin https://github.com/projectcalico/calico-cni/releases/download/v1.4.2/calico
wget -N -P /opt/cni/bin https://github.com/projectcalico/calico-cni/releases/download/v1.4.2/calico-ipam
chmod +x /opt/cni/bin/calico /opt/cni/bin/calico-ipam
```

cni插件需要标准cni配置文件,policy只是咋部署kube-policy-controller NetworkPolicy需要

```
mkdir -p /etc/cni/net.d
cat >/etc/cni/net.d/10-calico.conf <<EOF
{
    "name": "calico-k8s-network",
    "type": "calico",
    "etcd_endpoints": "http://<ETCD_IP>:<ETCD_PORT>",
    "log_level": "info",
    "ipam": {
        "type": "calico-ipam"
    },
    "policy": {
        "type": "k8s"
    }
}
EOF
```

------------------------------------

### 安装calico network policy controller
###### 下载policy controller manifest
```
wget http://docs.projectcalico.org/v1.5/getting-started/kubernetes/installation/policy-controller.yaml
```
######  修改ETCD_ENDPOINTS


###### 通过kubectl启动policy-controller
```
kubectl create -f policy-controller.yaml
```

###### 查看pod运行状态
```
kubectl get pods --namespace=kube-system
```

---
### 修改kubelet 启动参数
```
# kubelet 启动参数
--network-plugin=cni
--network-plugin-dir=/etc/cni/net.d
```

/opt/kubernetes/cfg/kubelet
```
KUBELET_ARGS="--network-plugin-dir=/etc/cni/net.d  --network-plugin=cni"
```

/lib/systemd/system/kubelet.service

```
[Unit]
Description=Kubernetes Kubelet
After=docker.service
Requires=docker.service

[Service]
EnvironmentFile=-/opt/kubernetes/cfg/kubelet
ExecStart=/opt/kubernetes/bin/kubelet    ${KUBE_LOGTOSTDERR}     \
                    ${KUBE_LOG_LEVEL}       \
                    ${NODE_ADDRESS}         \
                    ${NODE_PORT}            \
                    ${NODE_HOSTNAME}        \
                    ${KUBELET_API_SERVER}   \
                    ${KUBE_ALLOW_PRIV}      \
                    ${KUBELET__DNS_IP}      \
                    ${KUBELET_DNS_DOMAIN}      \
                    ${KUBELET_ARGS}
Restart=on-failure
KillMode=process

[Install]
WantedBy=multi-user.target
```

---
### 修改kube-proxy 启动参数

/opt/kubernetes/cfg/kube-proxy

```
# 增加以下参数
KUBE_PROXY_ARGS=" --proxy-mode=iptables"
```

/lib/systemd/system/kube-proxy.service

```
[Unit]
Description=Kubernetes Proxy
After=network.target

[Service]
EnvironmentFile=-/opt/kubernetes/cfg/kube-proxy
ExecStart=/opt/kubernetes/bin/kube-proxy    ${KUBE_LOGTOSTDERR} \
                    ${KUBE_LOG_LEVEL}   \
		    ${KUBE_PROXY_ARGS}  \
                    ${NODE_HOSTNAME}    \
                    ${KUBE_MASTER}
Restart=on-failure

[Install]
WantedBy=multi-user.target
```
