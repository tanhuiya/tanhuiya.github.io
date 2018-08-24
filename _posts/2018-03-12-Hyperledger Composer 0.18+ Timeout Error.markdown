---
layout: post
title: Hyperledger Composer 0.18.1 Timeout Error
date: 2018-03-12 15:59:28.000000000 +09:00
---


从`0.16.x` 更新到 `0.18.1 `以后

### 问题

`composer network start ... `这步就卡住了，改过`connection`配置文件，清过缓存，突然成功让人高兴一下，stop 后删除镜像再跑又不行了。 感觉就像摸奖，跑十次可以成功一次😅。由于是`next`版本，网上也没有解决方案，只能自己踩坑了。


`composer network start ... ` 命令，会为每个peer启动一个容器。它的镜像是 `hyperledger/fabric-ccenv`, tag 是`x86_64-1.1.0-rc1`。 container 名称好像是随便去的，这个容器在成功启动 chaincode 容器一段时间后会自己退出。如下,由于有两个peer，这里会启动两个fabric-ccenv 容器，分别是`objective_franklin ` 和 `compassionate_bell `.下面以`compassionate_bell `举例。

```
$ docker ps
CONTAINER ID        IMAGE                                         COMMAND                  CREATED             STATUS              PORTS                                            NAMES
b6f65e8c905a        hyperledger/fabric-ccenv:x86_64-1.1.0-rc1     "/bin/sh -c 'cp -R /…"   10 seconds ago      Up 1 second                                                          objective_franklin
eaffbbb7aef7        hyperledger/fabric-ccenv:x86_64-1.1.0-rc1     "/bin/sh -c 'cp -R /…"   10 seconds ago      Up 1 second                                                          compassionate_bell
38cc14dab6ef        hyperledger/fabric-tools:x86_64-1.1.0-rc1     "/bin/bash -c './scr…"   51 seconds ago      Up 41 seconds                                                        cli
69b30c1cdb02        hyperledger/fabric-ca:x86_64-1.1.0-rc1        "sh -c 'fabric-ca-se…"   53 seconds ago      Up 43 seconds       0.0.0.0:7054->7054/tcp                           ca_peerOrg1
2f99be174cdc        hyperledger/fabric-peer:x86_64-1.1.0-rc1      "peer node start"        53 seconds ago      Up 42 seconds       0.0.0.0:8051->7051/tcp, 0.0.0.0:8053->7053/tcp   peer1.org1.example.com
c3431f79d07c        hyperledger/fabric-orderer:x86_64-1.1.0-rc1   "orderer"                53 seconds ago      Up 42 seconds       0.0.0.0:7050->7050/tcp                           orderer.example.com
7f7c5a5c1a05        hyperledger/fabric-peer:x86_64-1.1.0-rc1      "peer node start"        53 seconds ago      Up 43 seconds       0.0.0.0:7051->7051/tcp, 0.0.0.0:7053->7053/tcp   peer0.org1.example.com
```

在漫长的等待后，得到一个`Timeout error` .

```
Error: Error trying to instantiate composer runtime. Error: No valid responses from any peers.
Response from attempted peer comms was an error: Error: REQUEST_TIMEOUT
```

 `timeout` 值是你在`connection.json` 里设置的， 默认 45s .

给官方提`issue` ， 官方的回复 `you can switch to another internet access point. to create the chaincode image, 0.18.1 will pull the npm modules from remote repository. i have succeeded running composer network start `
好吧，换个网，可能是在中国的原因，所以这招对我来说没什么用。
至少知道了可能是什么问题导致的。

`fabric-ccenv` 构建的容器中，会使用`npm`命令拉取远程仓库，可能是这步卡住了，直接导致了`timeout`。
执行 `docker logs -f compassionate_bell ` 可以看到，确实卡在了`npm `命令上. 所以就想到了解决办法 -- 更换 npm 源, 但是源是在`docker`容器中的...

`hyperledger/fabric-ccenv:x86_64-1.1.0-rc1` 容器的启动是通过`composer network start `命令，所以我不能自己搞个镜像例如`hyperledger/fabric-ccenv:other`然后让composer启动。 唯一可行的就是修改`hyperledger/fabric-ccenv:x86_64-1.1.0-rc1` 镜像，再用同样的名字覆盖它。

### 首先进入容器
```
docker exec -it compassionate_bell /bin/bash 
```
### 更换`npm`源
网上一搜一堆，例如

```
npm config set registry https://registry.npm.taobao.org
```
换好了再`npm config list` 看一下

然后退出容器`exit`。

### 重新提交镜像
```
docker commit -a "your name" -m "change npm source" compassionate_bell hyperledger/fabric-ccenv:x86_64-1.1.0-rc1 
```
注意最后一个参数一定要和你之前的相同，因为composer 只为启动指定image构建的容器。

然后 `docker images` 会发现镜像`hyperledger/fabric-ccenv:x86_64-1.1.0-rc1` 的 `CREATED` 是几秒之前

### 重新启动
```
docker rm -f compassionate_bell（替换成你自己的）
composer network start ...
```

记得把connection.json 里的 `Timeout` 设置长一点，一般 300 + 吧。
OK，大功告成.希望帮助到大家。

