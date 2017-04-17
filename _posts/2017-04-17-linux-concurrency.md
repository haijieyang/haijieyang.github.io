---
layout: post
title: coreos install by Ipxe
category: coreos
comments: false
---

  * installation
  * vmware config
  
---

### 1. installation
###### 1.1 downlaod coroes-ipxe-sever

```
curl -L https://github.com/kelseyhightower/coreos-ipxe-server/releases/download/v0.3.0/coreos-ipxe-server-0.3.0-linux-amd64 -o coreos-ipxe-server

```

###### 1.2 download coreos pxe image

```
mkdir -p $COREOS_IPXE_SERVER_DATA_DIR/images/amd64-usr/310.1.0
cd $COREOS_IPXE_SERVER_DATA_DIR/images/amd64-usr/310.1.0
wget http://storage.core-os.net/coreos/amd64-usr/310.1.0/coreos_production_pxe_image.cpio.gz
wget http://storage.core-os.net/coreos/amd64-usr/310.1.0/coreos_production_pxe.vmlinuz
```

###### 1.3 Add an SSH Public Key

$COREOS_IPXE_SERVER_DATA_DIR/sshkeys/coreos.pub

```
ssh-rsa AAAAB3Nza...
```

###### 1.4 Add a Cloud Config File

$COREOS_IPXE_SERVER_DATA_DIR/configs/development.yml

```
#cloud-config

ssh_authorized_keys:
    - ssh-rsa AAAAB3Nza...
coreos:
  etcd:
    addr: $private_ipv4:4001
    peer-addr: $private_ipv4:7001
  units:
    - name: etcd.service
      command: start
    - name: fleet.service
      command: start
    - name: docker.socket
      command: start
  oem:
    id: coreos
    name: CoreOS Custom
    version-id: 310.1.0
    home-url: https://coreos.com
```

###### 1.5 Add an iPXE Profile

$COREOS_IPXE_SERVER_DATA_DIR/profiles/development.json

```
{
	"cloud_config": "development",
	"rootfstype": "btrfs",
	"sshkey": "coreos",
	"version": "310.1.0"
}
```

---


###### 1.6 ipxe-server  layout

![ipxe-layout](https://raw.githubusercontent.com/haijieyang/haijieyang.github.io/master/images/ipxe.png)

---


### 2. vmware workstation config

######  2.1 download ipxe iso for vmware

```
wget http://boot.ipxe.org/ipxe.iso -o ipxe.iso
```

###### 2.2 ipxe install

```
iPXE> dhcp
iPXE> chain http://192.168.152.134:4777
```
