---
layout: post
title: docker build image tools
category: docker
comments: false
---

  * packer
  * dockercook
---

### packer 
###### install packer

```
wget https://releases.hashicorp.com/packer/1.0.0/packer_1.0.0_linux_amd64.zip?_ga=1.196288677.802741261.1490696258 -o packer_1.0.0_linux_amd64.zip
unzip packer_1.0.0_linux_amd64.zip
mv packer /usr/bin/
```

###### packer syntax

docker_redis.json

```
{
    "builders": [
        {
            "type": "docker",
            "image": "ubuntu",
            "commit": true
        }
    ],
    "provisioners": [
        {
            "type": "shell",
            "inline": [
                "sleep 30",
                "sudo apt-get install update",
                "sudo apt-get install -y redis-server"
            ]
        }
    ],
    "post-processors": [
        {
            "type": "docker-tag",
            "repository": "redis-test",
            "tag": "packer"
        }
    ]
}

#json文件分成3块 builders provisioners post-processors
# builders时指定type为docker, 而 base image 为ubuntu。"commit":true指定需要运行docker commit， 由container生成image. provisioners将运行一个shell, 安装redis-server。 最后，在post-processors里为image 添加tag redis-test:packer
```

---

###### packer build

```
packer validate docker_redis.json #验证语法正确性
packer build docker_redis.json
```
---

###### note

```
# 需要安装docker
# sleep 30   这句话不能去除ssh进容器是需要时间的
# 只有基础镜像层和其他不能追述历史
```

### dockercook
###### dockercook install

开启docker tcp通信端口
```
/usr/bin/dockerd -H tcp://0.0.0.0:2375
# export DOCKER_HOST="tcp://0.0.0.0:2375"
```

```
git clone https://github.com/factisresearch/dockercook.git && cd dockercook && stack setup && stack install
```

###### dockercook syntax

BranchA repo/package.json
```
{
    "name": "sample-app",
    "version": "0.1.0",
    "dependencies" : {
        "bloomfilter"   :  "0.0.12",
        "express" :  "2.1.x",
        "mongoose" :  "2.2.x",
        "moment": "2.5.x"
    }
}
```


repo/cookfiles/system.cook
```
BASE DOCKER ubuntu:14.04
BEGIN
RUN apt-get update
RUN apt-get install -y nodejs npm
RUN apt-get clean
RUN ln -s /usr/bin/nodejs /usr/bin/node
COMMIT
```

repo/cookfiles/node-pkg.cook
```
BASE COOK system.cook
INCLUDE package.json
UNPACK /app
WORKDIR /app
RUN npm install
```

repo/cookfiles/app.cook
```
BASE COOK node-pkg.cook
INCLUDE *.js
UNPACK /app
CMD node ./app.js
```


###### dockercook build

```
cd repo
dockercook init
git checkout  branchA
dockercook cook cookfiles/app.cook
```
