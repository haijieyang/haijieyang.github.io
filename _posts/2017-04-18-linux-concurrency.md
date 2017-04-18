---
layout: post
title: coreos install by pxe
category: coreos
comments: false
---

### Install coreos by PXE

---

  * installation
  * vmware workstation config
  
---

### 1. installation
###### 1.1 yum install base rpm

```
sudo yum install tftp-server dhcp syslinux xinetd -y

```

###### 1.2 tftp config

vi /etc/xinetd.d/tftp

```
disable = no

```

base  syslinux image
```
su -
mkdir -p /tftpboot
cd /tftpboot
cp /usr/share/syslinux/pxelinux.0 /tftpboot
cp /usr/share/syslinux/menu.c32 /tftpboot
cp /usr/share/syslinux/memdisk /tftpboot
cp /usr/share/syslinux/mboot.c32 /tftpboot
cp /usr/share/syslinux/chain.c32 /tftpboot

/sbin/service dhcpd start
/sbin/service xinetd start
/sbin/chkconfig tftp on
```

Setup default boot menu

```
mkdir /tftpboot/pxelinux.cfg
touch /tftpboot/pxelinux.cfg/default
```


###### 1.3 Adding CoreOS to PXE

```
MY_TFTPROOT_DIR=/tftpboot
mkdir -p $MY_TFTPROOT_DIR/images/coreos/
cd $MY_TFTPROOT_DIR/images/coreos/
wget http://stable.release.core-os.net/amd64-usr/current/coreos_production_pxe.vmlinuz
wget http://stable.release.core-os.net/amd64-usr/current/coreos_production_pxe.vmlinuz.sig
wget http://stable.release.core-os.net/amd64-usr/current/coreos_production_pxe_image.cpio.gz
wget http://stable.release.core-os.net/amd64-usr/current/coreos_production_pxe_image.cpio.gz.sig
gpg --verify coreos_production_pxe.vmlinuz.sig
gpg --verify coreos_production_pxe_image.cpio.gz.sig
```

vi /tftpboot/pxelinux.cfg/default

```
default menu.c32
prompt 0
timeout 300
ONTIMEOUT local
display boot.msg

MENU TITLE Main Menu

LABEL local
        MENU LABEL Boot local hard drive
        LOCALBOOT 0

MENU BEGIN CoreOS Menu

    LABEL coreos-master
        MENU LABEL CoreOS Master
        KERNEL images/coreos/coreos_production_pxe.vmlinuz
        APPEND initrd=images/coreos/coreos_production_pxe_image.cpio.gz cloud-config-url=http://192.168.152.10/coreos/pxe-cloud-config-master.yml

    LABEL coreos-slave
        MENU LABEL CoreOS Slave
        KERNEL images/coreos/coreos_production_pxe.vmlinuz
        APPEND initrd=images/coreos/coreos_production_pxe_image.cpio.gz cloud-config-url=http://192.168.152.10/coreos/pxe-cloud-config-slave.yml

MENU END
```

vi /tftpboot/pxelinux.cfg/coreos-node-master

```
default coreos
prompt 1
timeout 15

display boot.msg

label coreos
    menu default
    kernel images/coreos/coreos_production_pxe.vmlinuz
    append initrd=images/coreos/coreos_production_pxe_image.cpio.gz cloud-config-url=http://192.168.152.10/coreos/pxe-cloud-config-master.yml console=tty0 console=ttyS0 coreos.autologin=tty1 coreos.autologin=ttyS0

```

vi /tftpboot/pxelinux.cfg/oreos-node-slave

```
default coreos
prompt 1
timeout 15

display boot.msg

label coreos
    menu default
    kernel images/coreos/coreos_production_pxe.vmlinuz
    append initrd=images/coreos/coreos_production_pxe_image.cpio.gz cloud-config-url=http://192.168.152.10/coreos/pxe-cloud-config-slave.yml console=tty0 console=ttyS0 coreos.autologin=tty1 coreos.autologin=ttyS0
```



---

###  2. DHCP configuration

###### 2.1 Add the filename to the host or subnet sections

```
# dhcpd.conf

option domain-name-servers 114.114.114.114;

default-lease-time 600;
max-lease-time 7200;
allow booting;
allow bootp;

log-facility local7;

subnet 192.168.152.0 netmask 255.255.255.0 {
  range 192.168.152.200  192.168.152.240;
  option routers 192.168.152.2;
  filename "pxelinux.0";
}
        host core_os_master {
                hardware ethernet 00:0c:29:c4:3a:2b;
                option routers 192.168.152.2;
                fixed-address 192.168.152.201;
                option domain-name-servers 114.114.114.114;
                filename "pxelinux.0";
        }
        host core_os_slave {
                hardware ethernet d0:00:67:13:0d:01;
                option routers 192.168.152.2;
                fixed-address 192.168.152.202;
                option domain-name-servers 114.114.114.114;
                filename "/tftpboot/pxelinux.0";
        }
        host core_os_slave2 {
                hardware ethernet d0:00:67:13:0d:02;
                option routers 192.168.152.2;
                fixed-address 192.168.152.203;
                option domain-name-servers 114.114.114.114;
                filename "/tftpboot/pxelinux.0";
        }

```

###  3. nginx

###### 3.1 install  nginx

```  yum install nginx -y```

###### 3.2 nginx conf
```
{
    server {
        listen       80;
        server_name  192.168.152.10;
        root         /www/work;


        location / {
                autoindex on;
        }

    }
}
```

### 4. node  config yaml

###### 4.1 node-master


```
mkdir -p /www/work/coreos
touch pxe-cloud-config-master.yml
touch pxe-cloud-config-slave.yml
```
pxe-cloud-config-master.yml
 ```
 #cloud-config
ssh_authorized_keys:
    - ssh-rsa AAAAB3NzaC1yc2EAAA....
 
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


