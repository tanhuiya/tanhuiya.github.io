---
layout: post
title: Ubuntu 运行 Hyperledger Fabric  网络失败 -- 阿里云服务器
date: 2018-03-18 15:59:28.000000000 +09:00
---

### _环境_:

*  `System Version: Ubuntu Linux 14.04 / 16.04 LTS`
*  `Hyperledger Composer version: 0.19.1`
 
### _步骤_:

```
mkdir ~/fabric-dev-servers && cd ~/fabric-dev-servers

curl -O https://raw.githubusercontent.com/hyperledger/composer-tools/master/packages/fabric-dev-servers/fabric-dev-servers.tar.gz

tar -xvf fabric-dev-servers.tar.gz
	
cd ~/fabric-dev-servers&&./downloadFabric.sh
	
cd ~/fabric-dev-servers && ./createPeerAdminCard.sh && ./startFabric.sh
    
```

### _问题_:



_查看该容器 `log` ，发现有条警告信息：_
	
> **2018-05-06 01:32:11.118 UTC [couchdb] handleRequest -> WARN 016 Retrying couchdb request in 125ms. Attempt:1  Error:Get http://couchdb:5984/: dial tcp 172.18.0.4:5984: getsockopt: connection refused**


看错误，是`peer0`容器没有能连上`couchdb`。 
 
经过一番搜寻，`stackoverflow` 上有解决方案说 `dns` 解析问题，在 `peer0.org1.example.com` 中加入`dns_search: .`, ***But It does not work*** 😡😡😡  

那好，我尝试不用`couchdb`，用他默认的`levelDB`。注释掉 `docker-composer.yaml` 里 `peer0.org1.com` 连接 `couchdb` 的一些依赖以及环境变量
重新执行 `./startFabric.sh`.  

这次peer0还是不能正常启动，依旧出现了报错。报错信息如下
> **fatal error: unexpected signal during runtime execution [signal SIGSEGV: segmentation violation code=0x1 addr=0x63 ...**

经过很长时间的查询，发现最新的阿里云服务器中 `golang` 使用`cgo resolver`解析dns(宿主机ECS的配置文件变了), 过去OK 的版本使用的是 `pure go resolver`.

### _解决_:

_解决方法:_

在`docker-compose.yaml`里对**`peer、orderer、ca、couchdb`**的环境变量加入**`GODEBUG=netdns=go`**

再次尝试 `./startFabric.sh` ,终于  OK ！！！  
✌️✌️✌️