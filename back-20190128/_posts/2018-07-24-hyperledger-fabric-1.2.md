---
layout: post
title: hyperledger-fabric v 1.2
categories: fabric
description: hyperledger-fabric v 1.2
keywords: fabric
catalog:    true
header-img: "img/pexels/triangular.jpeg"
---


> fabric v1.2 , 单机 多节点 手动部署, 所有认证这边暂时未启动 TLS 认证 。
>
> 3个 orderer , 2个 CA,  2个 peer, 3个zookeeper, 4个kafka

# 部署 hyperledger-fabric v1.2

## 官方地址

> 文档以官方文档为主 http://hyperledger-fabric.readthedocs.io/en/release-1.2/prereqs.html

```
# 官网 github
https://github.com/hyperledger/fabric
```

## 环境准备

* 安装 Docker (用于 fabric 服务启动) 
    

```
# 导入 yum 源

# 安装 yum-config-manager

yum -y install yum-utils

# 导入
yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
    
    
    
# 安装 docker

yum -y install docker-ce
    
```


```

# 启动 docker 

systemctl daemon-reload
systemctl start docker
systemctl enable docker

```


```
# 查看 docker 版本

docker version
Client:
 Version:      17.05.0-ce
 API version:  1.29
 Go version:   go1.7.5
 Git commit:   89658be
 Built:        Thu May  4 22:06:25 2017
 OS/Arch:      linux/amd64

Server:
 Version:      17.05.0-ce
 API version:  1.29 (minimum version 1.12)
 Go version:   go1.7.5
 Git commit:   89658be
 Built:        Thu May  4 22:06:25 2017
 OS/Arch:      linux/amd64
 Experimental: false
```




* 安装 Docker-compose (用于 docker 容器服务统一管理 编排)


```
# 安装 pip

curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py

python get-pip.py

```


```
# 安装 docker-compose

pip install docker-compose

```


```
docker-compose version
docker-compose version 1.16.1, build 6d1ac219
docker-py version: 2.5.1
CPython version: 2.7.5
OpenSSL version: OpenSSL 1.0.1e-fips 11 Feb 2013
```


* Golang (用于 fabric cli 服务的调用， ca 服务证书生成 )


```
mkdir -p /opt/golang
mkdir -p /opt/gopath

# 国外地址
curl -O https://storage.googleapis.com/golang/go1.10.linux-amd64.tar.gz

# 国内地址
curl -O https://studygolang.com/dl/golang/go1.10.linux-amd64.tar.gz


# 解压
tar zxvf go1.10.linux-amd64.tar.gz

# 配置环境变量

vi /etc/profile

添加如下

# golang env
export PATH=$PATH:/opt/golang/go/bin
export GOPATH=/opt/gopath

# 生效配置

source /etc/profile


# 查看配置

go version
go version go1.10 linux/amd64

```


## 环境规划

> 相关hostname 必须配置 dns 
>
> 关于 orderer 集群
>
> 当orderer 向peer节点提交Transaction的时候，peer节点会得到或返回一个读写集结果，该结果会发送给orderer节点进行共识和排序，此时如果orderer节点突然down掉，就会使请求服务失效而引发的数据丢失等问题，且目前的sdk对orderer发送的Transaction的回调会占用极长的时间，当大批量数据导入的时候该回调可认为不可用。

|节点标识|hostname|IP|开放端口|系统|
|--------|--------|--|--------|----------|---|--|
|orderer0节点|orderer0.jicki.me|192.168.168.100|7050|CentOS 7 x64|
|orderer1节点|orderer1.jicki.me|192.168.168.100|8050|CentOS 7 x64|
|orderer2节点|orderer2.jicki.me|192.168.168.100|9050|CentOS 7 x64|
|ca0 节点|ca.org1.jicki.me|192.168.168.100|7054|CentOS 7 x64|
|ca1 节点|ca.org2.jicki.me|192.168.168.100|8054|CentOS 7 x64|
|peer0节点|peer0.org1.jicki.me|192.168.168.100|7051, 7052, 7053|CentOS 7 x64|
|peer0节点|peer0.org2.jicki.me|192.168.168.100|8051, 8052, 8053|CentOS 7 x64|



## Hyperledger Fabric 源码


```
# 下载 Fabric 源码, 源码中 import 的路径为github.com/hyperledger/fabric ,所以我们要按照这个路径

mkdir -p /opt/jicki/github.com/hyperledger

cd /opt/jicki/github.com/hyperledger

git clone https://github.com/hyperledger/fabric

mkdir -p /opt/gopath/src

cp -r github.com /opt/gopath/src

# 文件如下:

[root@localhost fabric]# ls -lt
总用量 580
drwxr-xr-x  7 root root    138 6月   5 09:29 vendor
-rw-r--r--  1 root root    301 6月   5 09:29 tox.ini
drwxr-xr-x  2 root root     56 6月   5 09:29 unit-test
drwxr-xr-x  7 root root    134 6月   5 09:29 test
drwxr-xr-x  2 root root   4096 6月   5 09:29 scripts
-rw-r--r--  1 root root    316 6月   5 09:29 settings.gradle
drwxr-xr-x  3 root root     91 6月   5 09:29 sampleconfig
drwxr-xr-x  2 root root   4096 6月   5 09:29 release_notes
drwxr-xr-x 11 root root   4096 6月   5 09:29 protos
drwxr-xr-x  3 root root     30 6月   5 09:29 release
drwxr-xr-x  9 root root    154 6月   5 09:29 peer
drwxr-xr-x  3 root root     23 6月   5 09:29 proposals
drwxr-xr-x  6 root root    126 6月   5 09:29 orderer
drwxr-xr-x  6 root root   4096 6月   5 09:29 msp
drwxr-xr-x  9 root root    130 6月   5 09:29 images
drwxr-xr-x  2 root root     29 6月   5 09:29 gotools
drwxr-xr-x  2 root root   4096 6月   5 09:29 idemix
drwxr-xr-x 15 root root   4096 6月   5 09:29 gossip
drwxr-xr-x 10 root root   4096 6月   5 09:29 examples
drwxr-xr-x  4 root root     48 6月   5 09:29 events
drwxr-xr-x  4 root root    137 6月   5 09:29 docs
drwxr-xr-x  4 root root   4096 6月   5 09:29 devenv
-rw-r--r--  1 root root   3356 6月   5 09:29 docker-env.mk
drwxr-xr-x 20 root root   4096 6月   5 09:29 core
drwxr-xr-x 23 root root   4096 6月   5 09:29 common
drwxr-xr-x 10 root root   4096 6月   5 09:29 bddtests
-rw-r--r--  1 root root     13 6月   5 09:29 ci.properties
drwxr-xr-x  8 root root   4096 6月   5 09:29 bccsp
-rw-r--r--  1 root root  11358 6月   5 09:29 LICENSE
-rwxr-xr-x  1 root root  17068 6月   5 09:29 Makefile
-rw-r--r--  1 root root   5678 6月   5 09:29 README.md
-rw-r--r--  1 root root 475471 6月   5 09:29 CHANGELOG.md
-rw-r--r--  1 root root    597 6月   5 09:29 CODE_OF_CONDUCT.md
-rw-r--r--  1 root root    664 6月   5 09:29 CONTRIBUTING.md


```


## 生成 Hyperledger Fabric 证书


```
# 下载官方证书生成软件(均为二进制文件)
# 官方离线下载地址为 https://nexus.hyperledger.org/content/repositories/releases/org/hyperledger/fabric/hyperledger-fabric/


# 选择相应版本 CentOS 选择 linux-amd64-1.2.0  Mac 选择 darwin-amd64-1.2.0


# 下载地址为: https://nexus.hyperledger.org/content/repositories/releases/org/hyperledger/fabric/hyperledger-fabric/linux-amd64-1.2.0/hyperledger-fabric-linux-amd64-1.2.0.tar.gz

cd /opt/jicki

wget https://nexus.hyperledger.org/content/repositories/releases/org/hyperledger/fabric/hyperledger-fabric/linux-amd64-1.2.0/hyperledger-fabric-linux-amd64-1.2.0.tar.gz

tar zxvf hyperledger-fabric-linux-amd64-1.2.0.tar.gz

# 解压后是 一个 bin 与 一个 config 目录

[root@localhost jicki]# tree bin/
bin/
├── configtxgen
├── configtxlator
├── cryptogen
├── get-docker-images.sh
├── orderer
└── peer

0 directories, 6 files
 
 
# 为方便使用 我们配置一个 环境变量

vi /etc/profile


# fabric env
export PATH=$PATH:/opt/jicki/bin


# 使文件生效
source /etc/profile

```


```
# 拷贝 configtx.yaml  与 crypto-config.yaml 。


cd /opt/jicki/

cp fabric/examples/e2e_cli/crypto-config.yaml .

cp fabric/examples/e2e_cli/configtx.yaml .

```

```
# 这里修改相应 jicki.me 为 jicki.me

sed -i 's/example\.com/jicki\.me/g' *.yaml


# 编辑 crypto-config.yaml 增加 orderer 为三个如下:


OrdererOrgs:
  - Name: Orderer
    Domain: jicki.me
    CA:
        Country: CN
        Province: GuangDong
        Locality: ShenZhen
    Specs:
      - Hostname: orderer0
      - Hostname: orderer1
      - Hostname: orderer2
      
PeerOrgs:
  - Name: Org1
    Domain: org1.jicki.me
    EnableNodeOUs: true
    CA:
        Country: CN
        Province: GuangDong
        Locality: ShenZhen
    Template:
      Count: 2
    Users:
      Count: 1
  - Name: Org2
    Domain: org2.jicki.me
    EnableNodeOUs: true
    CA:
        Country: CN
        Province: GuangDong
        Locality: ShenZhen
    Template:
      Count: 2
    Users:
      Count: 1
      
```




```
# 然后这里使用 cryptogen 软件来生成相应的证书了

[root@localhost jicki]# cryptogen generate --config=./crypto-config.yaml
org1.jicki.me
org2.jicki.me

# 生成一个 crypto-config 证书目录

[root@payment jicki]# tree crypto-config
crypto-config
├── ordererOrganizations
│   └── jicki.me
│       ├── ca
│       │   ├── ca.jicki.me-cert.pem
│       │   └── ea7afe99d56e6c2e839113909651f8b3531a6447e4615eed208c7aa86b211ad0_sk
│       ├── msp
│       │   ├── admincerts
│       │   │   └── Admin@jicki.me-cert.pem
│       │   ├── cacerts
│       │   │   └── ca.jicki.me-cert.pem
│       │   └── tlscacerts
│       │       └── tlsca.jicki.me-cert.pem
│       ├── orderers
│       │   ├── orderer0.jicki.me
│       │   │   ├── msp
│       │   │   │   ├── admincerts
│       │   │   │   │   └── Admin@jicki.me-cert.pem
│       │   │   │   ├── cacerts
│       │   │   │   │   └── ca.jicki.me-cert.pem
│       │   │   │   ├── keystore
│       │   │   │   │   └── c4ff1a8f55af9e4108596a26471a2a748eeb370b00300abddac692d5c5f72f57_sk
│       │   │   │   ├── signcerts
│       │   │   │   │   └── orderer0.jicki.me-cert.pem
│       │   │   │   └── tlscacerts
│       │   │   │       └── tlsca.jicki.me-cert.pem
│       │   │   └── tls
│       │   │       ├── ca.crt
│       │   │       ├── server.crt
│       │   │       └── server.key
│       │   ├── orderer1.jicki.me
│       │   │   ├── msp
│       │   │   │   ├── admincerts
│       │   │   │   │   └── Admin@jicki.me-cert.pem
│       │   │   │   ├── cacerts
│       │   │   │   │   └── ca.jicki.me-cert.pem
│       │   │   │   ├── keystore
│       │   │   │   │   └── f057fd116621213b9d06cf05d1b04b3b53057b4df4338ca5bb3b59e36acf074a_sk
│       │   │   │   ├── signcerts
│       │   │   │   │   └── orderer1.jicki.me-cert.pem
│       │   │   │   └── tlscacerts
│       │   │   │       └── tlsca.jicki.me-cert.pem
│       │   │   └── tls
│       │   │       ├── ca.crt
│       │   │       ├── server.crt
│       │   │       └── server.key
│       │   └── orderer2.jicki.me
│       │       ├── msp
│       │       │   ├── admincerts
│       │       │   │   └── Admin@jicki.me-cert.pem
│       │       │   ├── cacerts
│       │       │   │   └── ca.jicki.me-cert.pem
│       │       │   ├── keystore
│       │       │   │   └── a0395ee7823af3ab3ae56fa4569051a5525324f506cd38b0d0dcaa3e484ae084_sk
│       │       │   ├── signcerts
│       │       │   │   └── orderer2.jicki.me-cert.pem
│       │       │   └── tlscacerts
│       │       │       └── tlsca.jicki.me-cert.pem
│       │       └── tls
│       │           ├── ca.crt
│       │           ├── server.crt
│       │           └── server.key
│       ├── tlsca
│       │   ├── 4164a39aaee758c85e9d4458bae4d51f0d01e58116003cd5782dbbaff68b1545_sk
│       │   └── tlsca.jicki.me-cert.pem
│       └── users
│           └── Admin@jicki.me
│               ├── msp
│               │   ├── admincerts
│               │   │   └── Admin@jicki.me-cert.pem
│               │   ├── cacerts
│               │   │   └── ca.jicki.me-cert.pem
│               │   ├── keystore
│               │   │   └── 30d0b23ab85ede3d4424a336d839067bf7cad70f538fcf87d2a9c5083b5482f4_sk
│               │   ├── signcerts
│               │   │   └── Admin@jicki.me-cert.pem
│               │   └── tlscacerts
│               │       └── tlsca.jicki.me-cert.pem
│               └── tls
│                   ├── ca.crt
│                   ├── client.crt
│                   └── client.key
└── peerOrganizations
    ├── org1.jicki.me
    │   ├── ca
    │   │   ├── 98671ea7c42ccdeb73327d7c67f759e487497f13349b97a135392f660cf6747f_sk
    │   │   └── ca.org1.jicki.me-cert.pem
    │   ├── msp
    │   │   ├── admincerts
    │   │   │   └── Admin@org1.jicki.me-cert.pem
    │   │   ├── cacerts
    │   │   │   └── ca.org1.jicki.me-cert.pem
    │   │   ├── config.yaml
    │   │   └── tlscacerts
    │   │       └── tlsca.org1.jicki.me-cert.pem
    │   ├── peers
    │   │   ├── peer0.org1.jicki.me
    │   │   │   ├── msp
    │   │   │   │   ├── admincerts
    │   │   │   │   │   └── Admin@org1.jicki.me-cert.pem
    │   │   │   │   ├── cacerts
    │   │   │   │   │   └── ca.org1.jicki.me-cert.pem
    │   │   │   │   ├── config.yaml
    │   │   │   │   ├── keystore
    │   │   │   │   │   └── c99b65a414bc877998bb52c7ba80faf7e01d6512e8e50253daccd93e273bbb24_sk
    │   │   │   │   ├── signcerts
    │   │   │   │   │   └── peer0.org1.jicki.me-cert.pem
    │   │   │   │   └── tlscacerts
    │   │   │   │       └── tlsca.org1.jicki.me-cert.pem
    │   │   │   └── tls
    │   │   │       ├── ca.crt
    │   │   │       ├── server.crt
    │   │   │       └── server.key
    │   │   └── peer1.org1.jicki.me
    │   │       ├── msp
    │   │       │   ├── admincerts
    │   │       │   │   └── Admin@org1.jicki.me-cert.pem
    │   │       │   ├── cacerts
    │   │       │   │   └── ca.org1.jicki.me-cert.pem
    │   │       │   ├── config.yaml
    │   │       │   ├── keystore
    │   │       │   │   └── f1a8262cd8c78642d0aa50eb9ac2bae98949ce937063fb8e0d0c03fb7b16e4ce_sk
    │   │       │   ├── signcerts
    │   │       │   │   └── peer1.org1.jicki.me-cert.pem
    │   │       │   └── tlscacerts
    │   │       │       └── tlsca.org1.jicki.me-cert.pem
    │   │       └── tls
    │   │           ├── ca.crt
    │   │           ├── server.crt
    │   │           └── server.key
    │   ├── tlsca
    │   │   ├── c4e8130e739a69e6f9628e1e98fa54326ddfdcc93ef8568ffb72ba521069f3d1_sk
    │   │   └── tlsca.org1.jicki.me-cert.pem
    │   └── users
    │       ├── Admin@org1.jicki.me
    │       │   ├── msp
    │       │   │   ├── admincerts
    │       │   │   │   └── Admin@org1.jicki.me-cert.pem
    │       │   │   ├── cacerts
    │       │   │   │   └── ca.org1.jicki.me-cert.pem
    │       │   │   ├── keystore
    │       │   │   │   └── c8edb858abc7a4589458419d9ca74125830115e1e3544cc85c7a4a91f8b84b52_sk
    │       │   │   ├── signcerts
    │       │   │   │   └── Admin@org1.jicki.me-cert.pem
    │       │   │   └── tlscacerts
    │       │   │       └── tlsca.org1.jicki.me-cert.pem
    │       │   └── tls
    │       │       ├── ca.crt
    │       │       ├── client.crt
    │       │       └── client.key
    │       └── User1@org1.jicki.me
    │           ├── msp
    │           │   ├── admincerts
    │           │   │   └── User1@org1.jicki.me-cert.pem
    │           │   ├── cacerts
    │           │   │   └── ca.org1.jicki.me-cert.pem
    │           │   ├── keystore
    │           │   │   └── c720e53e25331b60bd8b33f2e19642e1ebef092afcc5f4c07f6efab91eceb298_sk
    │           │   ├── signcerts
    │           │   │   └── User1@org1.jicki.me-cert.pem
    │           │   └── tlscacerts
    │           │       └── tlsca.org1.jicki.me-cert.pem
    │           └── tls
    │               ├── ca.crt
    │               ├── client.crt
    │               └── client.key
    └── org2.jicki.me
        ├── ca
        │   ├── 5e398a55ab9e021cc89f60dfc16ae21def5b35495f0621d8aa55a2463ba051f6_sk
        │   └── ca.org2.jicki.me-cert.pem
        ├── msp
        │   ├── admincerts
        │   │   └── Admin@org2.jicki.me-cert.pem
        │   ├── cacerts
        │   │   └── ca.org2.jicki.me-cert.pem
        │   ├── config.yaml
        │   └── tlscacerts
        │       └── tlsca.org2.jicki.me-cert.pem
        ├── peers
        │   ├── peer0.org2.jicki.me
        │   │   ├── msp
        │   │   │   ├── admincerts
        │   │   │   │   └── Admin@org2.jicki.me-cert.pem
        │   │   │   ├── cacerts
        │   │   │   │   └── ca.org2.jicki.me-cert.pem
        │   │   │   ├── config.yaml
        │   │   │   ├── keystore
        │   │   │   │   └── a4eb0cf3769173e73cfcaa74ec938ef2def877b0ae0efe4e95be4a7588621da0_sk
        │   │   │   ├── signcerts
        │   │   │   │   └── peer0.org2.jicki.me-cert.pem
        │   │   │   └── tlscacerts
        │   │   │       └── tlsca.org2.jicki.me-cert.pem
        │   │   └── tls
        │   │       ├── ca.crt
        │   │       ├── server.crt
        │   │       └── server.key
        │   └── peer1.org2.jicki.me
        │       ├── msp
        │       │   ├── admincerts
        │       │   │   └── Admin@org2.jicki.me-cert.pem
        │       │   ├── cacerts
        │       │   │   └── ca.org2.jicki.me-cert.pem
        │       │   ├── config.yaml
        │       │   ├── keystore
        │       │   │   └── 26e3d15b68a9d2b022e6560607b3556e0150df3526826ff4a81c585714d5d41c_sk
        │       │   ├── signcerts
        │       │   │   └── peer1.org2.jicki.me-cert.pem
        │       │   └── tlscacerts
        │       │       └── tlsca.org2.jicki.me-cert.pem
        │       └── tls
        │           ├── ca.crt
        │           ├── server.crt
        │           └── server.key
        ├── tlsca
        │   ├── 0979917773701392b0e7f1924ae711cedfae0dfbb4d82bd5a90c18207855fe8e_sk
        │   └── tlsca.org2.jicki.me-cert.pem
        └── users
            ├── Admin@org2.jicki.me
            │   ├── msp
            │   │   ├── admincerts
            │   │   │   └── Admin@org2.jicki.me-cert.pem
            │   │   ├── cacerts
            │   │   │   └── ca.org2.jicki.me-cert.pem
            │   │   ├── keystore
            │   │   │   └── ba8e549c298b47d1889d51d75e3a01724b8d2b068da369c7319f2cd586c49e70_sk
            │   │   ├── signcerts
            │   │   │   └── Admin@org2.jicki.me-cert.pem
            │   │   └── tlscacerts
            │   │       └── tlsca.org2.jicki.me-cert.pem
            │   └── tls
            │       ├── ca.crt
            │       ├── client.crt
            │       └── client.key
            └── User1@org2.jicki.me
                ├── msp
                │   ├── admincerts
                │   │   └── User1@org2.jicki.me-cert.pem
                │   ├── cacerts
                │   │   └── ca.org2.jicki.me-cert.pem
                │   ├── keystore
                │   │   └── e183353072b166455f40b2b65ec168f52518101ee7097345cd8ff8864a71698e_sk
                │   ├── signcerts
                │   │   └── User1@org2.jicki.me-cert.pem
                │   └── tlscacerts
                │       └── tlsca.org2.jicki.me-cert.pem
                └── tls
                    ├── ca.crt
                    ├── client.crt
                    └── client.key

125 directories, 123 files
```



## 生成 Hyperledger Fabric 创世区块


```
# 这里使用 configtxgen 来创建 创世区块


# 首先需要创建一个文件夹
mkdir -p /opt/jicki/channel-artifacts


# 修改 configtx.yaml 文件增加 orderer 多节点 以及 peer 多节点, 删除了 &Org3 节点

# 完整 configtx.yaml 如下: 
# configtx.yaml 文件格式 请千万注意 空格 与 tab 键 里的缩进，否则会报错。

Organizations:

    - &OrdererOrg
        Name: OrdererOrg
        ID: OrdererMSP
        MSPDir: crypto-config/ordererOrganizations/jicki.me/msp
        Policies:
            Readers:
                Type: Signature
                Rule: "OR('OrdererMSP.member')"
            Writers:
                Type: Signature
                Rule: "OR('OrdererMSP.member')"
            Admins:
                Type: Signature
                Rule: "OR('OrdererMSP.admin')"

    - &Org1
        Name: Org1MSP
        ID: Org1MSP
        MSPDir: crypto-config/peerOrganizations/org1.jicki.me/msp
        AnchorPeers:
            - Host: peer0.org1.jicki.me
              Port: 7051
        Policies:
            Readers:
                Type: Signature
                Rule: "OR('Org1MSP.admin', 'Org1MSP.peer', 'Org1MSP.client')"
            Writers:
                Type: Signature
                Rule: "OR('Org1MSP.admin', 'Org1MSP.client')"
            Admins:
                Type: Signature
                Rule: "OR('Org1MSP.admin')"

    - &Org2
        Name: Org2MSP
        ID: Org2MSP
        MSPDir: crypto-config/peerOrganizations/org2.jicki.me/msp
        AnchorPeers:
            - Host: peer0.org2.jicki.me
              Port: 7051
        Policies:
            Readers:
                Type: Signature
                Rule: "OR('Org2MSP.admin', 'Org2MSP.peer', 'Org2MSP.client')"
            Writers:
                Type: Signature
                Rule: "OR('Org2MSP.admin', 'Org2MSP.client')"
            Admins:
                Type: Signature
                Rule: "OR('Org2MSP.admin')"

Capabilities:
    Global: &ChannelCapabilities
        V1_1: true

    Orderer: &OrdererCapabilities
        V1_1: true

    Application: &ApplicationCapabilities
        V1_2: true

Application: &ApplicationDefaults

    Organizations:

    Policies:
        Readers:
            Type: ImplicitMeta
            Rule: "ANY Readers"
        Writers:
            Type: ImplicitMeta
            Rule: "ANY Writers"
        Admins:
            Type: ImplicitMeta
            Rule: "MAJORITY Admins"
    Capabilities:
        <<: *ApplicationCapabilities

Orderer: &OrdererDefaults
    OrdererType: kafka
    Addresses:
        - orderer0.jicki.me:7050
        - orderer1.jicki.me:7050
        - orderer2.jicki.me:7050

    BatchTimeout: 2s
    BatchSize:
        MaxMessageCount: 10
        # 设置最大的区块大小。每个区块最大有Orderer.AbsoluteMaxBytes个字节
        # 假定这里设置的值为 99MB，记住这个值，这会影响怎样配置Kafka代理。
        AbsoluteMaxBytes: 99 MB
        # 设置每个区块建议的大小。Kafka对于相对小的消息提供更高的吞吐量。
        # 区块大小最好不要超过1MB。
        PreferredMaxBytes: 512 KB
    Kafka:
        Brokers:
        # 这里如果非容器内建议使用 [ip:port]配置kafka 集群。
        # 如下为 单机版本集群内的配置。
        # kafka 集群最少为3节点，4节点为了能容错1台。
            - kafka0:9092
            - kafka1:9092
            - kafka2:9092
            - kafka3:9092
    Organizations:
    
    Policies:
        Readers:
            Type: ImplicitMeta
            Rule: "ANY Readers"
        Writers:
            Type: ImplicitMeta
            Rule: "ANY Writers"
        Admins:
            Type: ImplicitMeta
            Rule: "MAJORITY Admins"
        BlockValidation:
            Type: ImplicitMeta
            Rule: "ANY Writers"
    Capabilities:
        <<: *OrdererCapabilities

Channel: &ChannelDefaults

    Policies:
        Readers:
            Type: ImplicitMeta
            Rule: "ANY Readers"
        Writers:
            Type: ImplicitMeta
            Rule: "ANY Writers"
        Admins:
            Type: ImplicitMeta
            Rule: "MAJORITY Admins"

    Capabilities:
        <<: *ChannelCapabilities

Profiles:

    TwoOrgsOrdererGenesis:
        <<: *ChannelDefaults
        Orderer:
            <<: *OrdererDefaults
            Organizations:
                - *OrdererOrg
        Consortiums:
            SampleConsortium:
                Organizations:
                    - *Org1
                    - *Org2
    TwoOrgsChannel:
        Consortium: SampleConsortium
        Application:
            <<: *ApplicationDefaults
            Organizations:
                - *Org1
                - *Org2

```






```
# 创建 创世区块  TwoOrgsOrdererGenesis 名称为 configtx.yaml 中 Profiles 字段下的

[root@localhost jicki]# configtxgen -profile TwoOrgsOrdererGenesis -outputBlock ./channel-artifacts/genesis.block


configtxgen -profile TwoOrgsOrdererGenesis -outputBlock ./channel-artifacts/genesis.block
2018-07-24 14:17:28.681 CST [common/tools/configtxgen] main -> WARN 001 Omitting the channel ID for configtxgen is deprecated.  Explicitly passing the channel ID will be required in the future, defaulting to 'testchainid'.
2018-07-24 14:17:28.681 CST [common/tools/configtxgen] main -> INFO 002 Loading configuration
2018-07-24 14:17:28.726 CST [msp] getMspConfig -> INFO 003 Loading NodeOUs
2018-07-24 14:17:28.727 CST [msp] getMspConfig -> INFO 004 Loading NodeOUs
2018-07-24 14:17:28.727 CST [common/tools/configtxgen] doOutputBlock -> INFO 005 Generating genesis block
2018-07-24 14:17:28.728 CST [common/tools/configtxgen] doOutputBlock -> INFO 006 Writing genesis block



# 创世区块 是在 orderer 服务中使用

[root@localhost jicki]# ls -lt channel-artifacts/
总用量 16
-rw-r--r-- 1 root root 12484 7月  24 14:17 genesis.block
```


```
# 下面来生成一个 peer 服务 中使用的 tx 文件 TwoOrgsChannel 名称为 configtx.yaml 中 Profiles 字段下的，这里必须指定上面的 channelID


[root@localhost jicki]# configtxgen -profile TwoOrgsChannel -outputCreateChannelTx ./channel-artifacts/channel.tx -channelID mychannel


configtxgen -profile TwoOrgsChannel -outputCreateChannelTx ./channel-artifacts/channel.tx -channelID mychannel
2018-07-24 14:18:02.261 CST [common/tools/configtxgen] main -> INFO 001 Loading configuration
2018-07-24 14:18:02.306 CST [common/tools/configtxgen] doOutputChannelCreateTx -> INFO 002 Generating new channel configtx
2018-07-24 14:18:02.307 CST [msp] getMspConfig -> INFO 003 Loading NodeOUs
2018-07-24 14:18:02.308 CST [msp] getMspConfig -> INFO 004 Loading NodeOUs
2018-07-24 14:18:02.311 CST [common/tools/configtxgen] doOutputChannelCreateTx -> INFO 005 Writing new channel tx



[root@localhost jicki]# ls -lt channel-artifacts/
总用量 20
-rw-r--r-- 1 root root   346 7月  24 14:18 channel.tx
-rw-r--r-- 1 root root 12484 7月  24 14:17 genesis.block

```


```
# 定义组织 生成锚节点更新文件


# Org1MSP

[root@localhost jicki]# configtxgen -profile TwoOrgsChannel -outputAnchorPeersUpdate ./channel-artifacts/Org1MSPanchors.tx -channelID mychannel -asOrg Org1MSP


2018-07-24 14:18:02.261 CST [common/tools/configtxgen] main -> INFO 001 Loading configuration
2018-07-24 14:18:02.306 CST [common/tools/configtxgen] doOutputChannelCreateTx -> INFO 002 Generating new channel configtx
2018-07-24 14:18:02.307 CST [msp] getMspConfig -> INFO 003 Loading NodeOUs
2018-07-24 14:18:02.308 CST [msp] getMspConfig -> INFO 004 Loading NodeOUs
2018-07-24 14:18:02.311 CST [common/tools/configtxgen] doOutputChannelCreateTx -> INFO 005 Writing new channel tx



# Org2MSP


[root@localhost jicki]# configtxgen -profile TwoOrgsChannel -outputAnchorPeersUpdate ./channel-artifacts/Org2MSPanchors.tx -channelID mychannel -asOrg Org2MSP

2018-07-24 14:19:21.947 CST [common/tools/configtxgen] main -> INFO 001 Loading configuration
2018-07-24 14:19:21.993 CST [common/tools/configtxgen] doOutputAnchorPeersUpdate -> INFO 002 Generating anchor peer update
2018-07-24 14:19:21.993 CST [common/tools/configtxgen] doOutputAnchorPeersUpdate -> INFO 003 Writing anchor peer update


```




## 配置 Zookeeper Kafka 集群

> zookeeper 与 kafka 集群 必须在 所有服务器之间启动。


```
vi docker-compose-zk-kafka.yaml


version: '2'
services:
  zookeeper1:
    container_name: zookeeper1
    hostname: zookeeper1
    image: hyperledger/fabric-zookeeper
    restart: always
    environment:
      - ZOO_MY_ID=1
      - ZOO_SERVERS=server.1=zookeeper1:2888:3888 server.2=zookeeper2:2888:3888 server.3=zookeeper3:2888:3888
    #volumes:
    # 存储数据与日志
    #- ./data/zookeeper1/data:/data
    #- ./data/zookeeper1/datalog:/datalog
    networks:
      default:
        aliases:
          - jicki

  zookeeper2:
    container_name: zookeeper2
    hostname: zookeeper2
    image: hyperledger/fabric-zookeeper
    restart: always
    environment:
      - ZOO_MY_ID=2
      - ZOO_SERVERS=server.1=zookeeper1:2888:3888 server.2=zookeeper2:2888:3888 server.3=zookeeper3:2888:3888
    #volumes:
    # 存储数据与日志
    #- ./data/zookeeper2/data:/data
    #- ./data/zookeeper2/datalog:/datalog
    networks:
      default:
        aliases:
          - jicki

  zookeeper3:
    container_name: zookeeper3
    hostname: zookeeper3
    image: hyperledger/fabric-zookeeper
    restart: always
    environment:
      - ZOO_MY_ID=3
      - ZOO_SERVERS=server.1=zookeeper1:2888:3888 server.2=zookeeper2:2888:3888 server.3=zookeeper3:2888:3888
    #volumes:
    # 存储数据与日志
    #- ./data/zookeeper3/data:/data
    #- ./data/zookeeper3/datalog:/datalog
    networks:
      default:
        aliases:
          - jicki


  kafka0:
    container_name: kafka0
    hostname: kafka0
    image: hyperledger/fabric-kafka
    restart: always
    environment:
      - KAFKA_BROKER_ID=1
      # 设置一个M值,数据提交时会写入至少M个副本(这里M=2)（这些数据会被同步并且归属到in-sync 副本集合或ISR）M 必须小于 如下 N 值,并且大于1,既最小为2。
      - KAFKA_MIN_INSYNC_REPLICAS=2
      # 设置一个N值, N代表着每个channel都保存N个副本的数据到Kafka的代理上。N 必须大于如上 M 值, 既 N 值最小值为 3。
      - KAFKA_DEFAULT_REPLICATION_FACTOR=3
      - KAFKA_ZOOKEEPER_CONNECT=zookeeper1:2181,zookeeper2:2181,zookeeper3:2181
      # 如下99为configtx.yaml中会设置最大的区块大小(参考configtx.yaml中AbsoluteMaxBytes参数)
      # 每个区块最大有Orderer.AbsoluteMaxBytes个字节
      # 99 * 1024 * 1024 B
      - KAFKA_MESSAGE_MAX_BYTES=103809024
      # 每个通道获取的消息的字节数 如上一样
      - KAFKA_REPLICA_FETCH_MAX_BYTES=103809024
      # 数据一致性在区块链环境中是至关重要的, 我们不能从in-sync 副本（ISR）集合之外选取channel leader , 否则我们将会面临对于之前的leader产生的offsets覆盖的风险
      - KAFKA_UNCLEAN_LEADER_ELECTION_ENABLE=false
      # 关闭基于时间的日志保留方式并且避免分段到期。
      - KAFKA_LOG_RETENTION_MS=-1
    #volumes:
    # 存储数据与日志.
    #- ./data/kafka1/data:/data
    #- ./data/kafka1/data:/logs
    networks:
      default:
        aliases:
          - jicki
    depends_on:
      - zookeeper1
      - zookeeper2
      - zookeeper3

  kafka1:
    container_name: kafka1
    hostname: kafka1
    image: hyperledger/fabric-kafka
    restart: always
    environment:
      - KAFKA_BROKER_ID=2
      - KAFKA_MIN_INSYNC_REPLICAS=2
      - KAFKA_DEFAULT_REPLICATION_FACTOR=3
      - KAFKA_ZOOKEEPER_CONNECT=zookeeper1:2181,zookeeper2:2181,zookeeper3:2181
      - KAFKA_MESSAGE_MAX_BYTES=103809024
      - KAFKA_REPLICA_FETCH_MAX_BYTES=103809024
      - KAFKA_UNCLEAN_LEADER_ELECTION_ENABLE=false
      - KAFKA_LOG_RETENTION_MS=-1
    #volumes:
    # 存储数据与日志.
    #- ./data/kafka1/data:/data
    #- ./data/kafka1/data:/logs
    networks:
      default:
        aliases:
          - jicki
    depends_on:
      - zookeeper1
      - zookeeper2
      - zookeeper3
          
  kafka2:
    container_name: kafka2
    hostname: kafka2
    image: hyperledger/fabric-kafka
    restart: always
    environment:
      - KAFKA_BROKER_ID=3
      - KAFKA_MIN_INSYNC_REPLICAS=2
      - KAFKA_DEFAULT_REPLICATION_FACTOR=3
      - KAFKA_ZOOKEEPER_CONNECT=zookeeper1:2181,zookeeper2:2181,zookeeper3:2181
      - KAFKA_MESSAGE_MAX_BYTES=103809024
      - KAFKA_REPLICA_FETCH_MAX_BYTES=103809024
      - KAFKA_UNCLEAN_LEADER_ELECTION_ENABLE=false
      # 关闭基于时间的日志保留方式并且避免分段到期。
      - KAFKA_LOG_RETENTION_MS=-1
    #volumes:
    # 存储数据与日志.
    #- ./data/kafka2/data:/data
    #- ./data/kafka2/data:/logs
    networks:
      default:
        aliases:
          - jicki
    depends_on:
      - zookeeper1
      - zookeeper2
      - zookeeper3
          
  kafka3:
    container_name: kafka3
    hostname: kafka3
    image: hyperledger/fabric-kafka
    restart: always
    environment:
      - KAFKA_BROKER_ID=4
      - KAFKA_MIN_INSYNC_REPLICAS=2
      - KAFKA_DEFAULT_REPLICATION_FACTOR=3
      - KAFKA_ZOOKEEPER_CONNECT=zookeeper1:2181,zookeeper2:2181,zookeeper3:2181
      - KAFKA_MESSAGE_MAX_BYTES=103809024
      - KAFKA_REPLICA_FETCH_MAX_BYTES=103809024
      - KAFKA_UNCLEAN_LEADER_ELECTION_ENABLE=false
      - KAFKA_LOG_RETENTION_MS=-1
    #volumes:
    # 存储数据与日志.
    #- ./data/kafka3/data:/data
    #- ./data/kafka3/data:/logs
    networks:
      default:
        aliases:
          - jicki
    depends_on:
      - zookeeper1
      - zookeeper2
      - zookeeper3
          
```




## 配置 Hyperledger Fabric Orderer


```
# 创建文件 docker-compose-orderer.yaml

# 创建于 /opt/jicki 目录下

vi  docker-compose-orderer.yaml


version: '2'
services:
  orderer0.jicki.me:
    container_name: orderer0.jicki.me
    image: hyperledger/fabric-orderer
    environment:
      - CORE_VM_DOCKER_HOSTCONFIG_NETWORKMODE=jicki_default
      # - ORDERER_GENERAL_LOGLEVEL=error
      - ORDERER_GENERAL_LOGLEVEL=debug
      - ORDERER_GENERAL_LISTENADDRESS=0.0.0.0
      - ORDERER_GENERAL_LISTENPORT=7050
      #- ORDERER_GENERAL_GENESISPROFILE=AntiMothOrdererGenesis
      - ORDERER_GENERAL_GENESISMETHOD=file
      - ORDERER_GENERAL_GENESISFILE=/var/hyperledger/orderer/orderer.genesis.block
      - ORDERER_GENERAL_LOCALMSPID=OrdererMSP
      - ORDERER_GENERAL_LOCALMSPDIR=/var/hyperledger/orderer/msp
      #- ORDERER_GENERAL_LEDGERTYPE=ram
      #- ORDERER_GENERAL_LEDGERTYPE=file
      # enabled TLS
      - ORDERER_GENERAL_TLS_ENABLED=false
      - ORDERER_GENERAL_TLS_PRIVATEKEY=/var/hyperledger/orderer/tls/server.key
      - ORDERER_GENERAL_TLS_CERTIFICATE=/var/hyperledger/orderer/tls/server.crt
      - ORDERER_GENERAL_TLS_ROOTCAS=[/var/hyperledger/orderer/tls/ca.crt]

      # KAFKA
      - ORDERER_KAFKA_RETRY_LONGINTERVAL=10s
      - ORDERER_KAFKA_RETRY_LONGTOTAL=100s
      - ORDERER_KAFKA_RETRY_SHORTINTERVAL=1s
      - ORDERER_KAFKA_RETRY_SHORTTOTAL=30s
      - ORDERER_KAFKA_VERBOSE=true
      - ORDERER_KAFKA_BROKERS=[kafka0:9092,kafka1:9092,kafka2:9092,kafka3:9092]
    working_dir: /opt/gopath/src/github.com/hyperledger/fabric
    command: orderer
    volumes:
    - ./channel-artifacts/genesis.block:/var/hyperledger/orderer/orderer.genesis.block
    - ./crypto-config/ordererOrganizations/jicki.me/orderers/orderer0.jicki.me/msp:/var/hyperledger/orderer/msp
    - ./crypto-config/ordererOrganizations/jicki.me/orderers/orderer0.jicki.me/tls/:/var/hyperledger/orderer/tls
    networks:
      default:
        aliases:
          - jicki
    ports:
      - 7050:7050
    depends_on:
      - kafka0
      - kafka1
      - kafka2
      - kafka3

  orderer1.jicki.me:
    container_name: orderer1.jicki.me
    image: hyperledger/fabric-orderer
    environment:
      - CORE_VM_DOCKER_HOSTCONFIG_NETWORKMODE=jicki_default
      # - ORDERER_GENERAL_LOGLEVEL=error
      - ORDERER_GENERAL_LOGLEVEL=debug
      - ORDERER_GENERAL_LISTENADDRESS=0.0.0.0
      - ORDERER_GENERAL_LISTENPORT=7050
      #- ORDERER_GENERAL_GENESISPROFILE=AntiMothOrdererGenesis
      - ORDERER_GENERAL_GENESISMETHOD=file
      - ORDERER_GENERAL_GENESISFILE=/var/hyperledger/orderer/orderer.genesis.block
      - ORDERER_GENERAL_LOCALMSPID=OrdererMSP
      - ORDERER_GENERAL_LOCALMSPDIR=/var/hyperledger/orderer/msp
      #- ORDERER_GENERAL_LEDGERTYPE=ram
      #- ORDERER_GENERAL_LEDGERTYPE=file
      # enabled TLS
      - ORDERER_GENERAL_TLS_ENABLED=false
      - ORDERER_GENERAL_TLS_PRIVATEKEY=/var/hyperledger/orderer/tls/server.key
      - ORDERER_GENERAL_TLS_CERTIFICATE=/var/hyperledger/orderer/tls/server.crt
      - ORDERER_GENERAL_TLS_ROOTCAS=[/var/hyperledger/orderer/tls/ca.crt]
      
      # KAFKA
      - ORDERER_KAFKA_RETRY_LONGINTERVAL=10s
      - ORDERER_KAFKA_RETRY_LONGTOTAL=100s
      - ORDERER_KAFKA_RETRY_SHORTINTERVAL=1s
      - ORDERER_KAFKA_RETRY_SHORTTOTAL=30s
      - ORDERER_KAFKA_VERBOSE=true
      - ORDERER_KAFKA_BROKERS=[kafka0:9092,kafka1:9092,kafka2:9092,kafka3:9092]
    working_dir: /opt/gopath/src/github.com/hyperledger/fabric
    command: orderer
    volumes:
    - ./channel-artifacts/genesis.block:/var/hyperledger/orderer/orderer.genesis.block
    - ./crypto-config/ordererOrganizations/jicki.me/orderers/orderer1.jicki.me/msp:/var/hyperledger/orderer/msp
    - ./crypto-config/ordererOrganizations/jicki.me/orderers/orderer1.jicki.me/tls/:/var/hyperledger/orderer/tls
    networks:
      default:
        aliases:
          - jicki
    ports:
      - 8050:7050
    depends_on:
      - kafka0
      - kafka1
      - kafka2
      - kafka3
      
  orderer2.jicki.me:
    container_name: orderer2.jicki.me
    image: hyperledger/fabric-orderer
    environment:
      - CORE_VM_DOCKER_HOSTCONFIG_NETWORKMODE=jicki_default
      # - ORDERER_GENERAL_LOGLEVEL=error
      - ORDERER_GENERAL_LOGLEVEL=debug
      - ORDERER_GENERAL_LISTENADDRESS=0.0.0.0
      - ORDERER_GENERAL_LISTENPORT=7050
      #- ORDERER_GENERAL_GENESISPROFILE=AntiMothOrdererGenesis
      - ORDERER_GENERAL_GENESISMETHOD=file
      - ORDERER_GENERAL_GENESISFILE=/var/hyperledger/orderer/orderer.genesis.block
      - ORDERER_GENERAL_LOCALMSPID=OrdererMSP
      - ORDERER_GENERAL_LOCALMSPDIR=/var/hyperledger/orderer/msp
      #- ORDERER_GENERAL_LEDGERTYPE=ram
      #- ORDERER_GENERAL_LEDGERTYPE=file
      # enabled TLS
      - ORDERER_GENERAL_TLS_ENABLED=false
      - ORDERER_GENERAL_TLS_PRIVATEKEY=/var/hyperledger/orderer/tls/server.key
      - ORDERER_GENERAL_TLS_CERTIFICATE=/var/hyperledger/orderer/tls/server.crt
      - ORDERER_GENERAL_TLS_ROOTCAS=[/var/hyperledger/orderer/tls/ca.crt]
      
      # KAFKA
      - ORDERER_KAFKA_RETRY_LONGINTERVAL=10s
      - ORDERER_KAFKA_RETRY_LONGTOTAL=100s
      - ORDERER_KAFKA_RETRY_SHORTINTERVAL=1s
      - ORDERER_KAFKA_RETRY_SHORTTOTAL=30s
      - ORDERER_KAFKA_VERBOSE=true
      - ORDERER_KAFKA_BROKERS=[kafka0:9092,kafka1:9092,kafka2:9092,kafka3:9092]
    working_dir: /opt/gopath/src/github.com/hyperledger/fabric
    command: orderer
    volumes:
    - ./channel-artifacts/genesis.block:/var/hyperledger/orderer/orderer.genesis.block
    - ./crypto-config/ordererOrganizations/jicki.me/orderers/orderer2.jicki.me/msp:/var/hyperledger/orderer/msp
    - ./crypto-config/ordererOrganizations/jicki.me/orderers/orderer2.jicki.me/tls/:/var/hyperledger/orderer/tls
    networks:
      default:
        aliases:
          - jicki
    ports:
      - 9050:7050
    depends_on:
      - kafka0
      - kafka1
      - kafka2
      - kafka3
      
```




## 配置 Hyperledger Fabric ca


```

# 创建于 /opt/jicki 目录下

# 注: 如下目录中相关的 ca 证书请替换为 各自生成的, 本文目录为 /opt/jicki/crypto-config/peerOrganizations/org1.jicki.me/ca
/opt/jicki/crypto-config/peerOrganizations/org2.jicki.me/ca

```




```
vi  docker-compose-ca.yaml


version: '2'
services:
  ca.org1.jicki.me:
    container_name: ca.org1.jicki.me
    image: hyperledger/fabric-ca
    environment:
      - FABRIC_CA_HOME=/etc/hyperledger/fabric-ca-server
      - FABRIC_CA_SERVER_CA_NAME=ca-org1
      - FABRIC_CA_SERVER_TLS_ENABLED=false
      - FABRIC_CA_SERVER_TLS_CERTFILE=/etc/hyperledger/fabric-ca-server-config/ca.org1.jicki.me-cert.pem
      - FABRIC_CA_SERVER_TLS_KEYFILE=/etc/hyperledger/fabric-ca-server-config/98671ea7c42ccdeb73327d7c67f759e487497f13349b97a135392f660cf6747f_sk
    ports:
      - "7054:7054"
    command: sh -c 'fabric-ca-server start --ca.certfile /etc/hyperledger/fabric-ca-server-config/ca.org1.jicki.me-cert.pem --ca.keyfile /etc/hyperledger/fabric-ca-server-config/98671ea7c42ccdeb73327d7c67f759e487497f13349b97a135392f660cf6747f_sk -b admin:adminpw -d'
    volumes:
      - ./crypto-config/peerOrganizations/org1.jicki.me/ca/:/etc/hyperledger/fabric-ca-server-config
    depends_on:
      - orderer0.jicki.me
      - orderer1.jicki.me
      - orderer2.jicki.me
      

  ca.org2.jicki.me:
    container_name: ca.org2.jicki.me
    image: hyperledger/fabric-ca
    environment:
      - FABRIC_CA_HOME=/etc/hyperledger/fabric-ca-server
      - FABRIC_CA_SERVER_CA_NAME=ca-org2
      - FABRIC_CA_SERVER_TLS_ENABLED=false
      - FABRIC_CA_SERVER_TLS_CERTFILE=/etc/hyperledger/fabric-ca-server-config/ca.org2.jicki.me-cert.pem
      - FABRIC_CA_SERVER_TLS_KEYFILE=/etc/hyperledger/fabric-ca-server-config/5e398a55ab9e021cc89f60dfc16ae21def5b35495f0621d8aa55a2463ba051f6_sk
    ports:
      - "8054:7054"
    command: sh -c 'fabric-ca-server start --ca.certfile /etc/hyperledger/fabric-ca-server-config/ca.org2.jicki.me-cert.pem --ca.keyfile /etc/hyperledger/fabric-ca-server-config/5e398a55ab9e021cc89f60dfc16ae21def5b35495f0621d8aa55a2463ba051f6_sk -b admin:adminpw -d'
    volumes:
      - ./crypto-config/peerOrganizations/org2.jicki.me/ca/:/etc/hyperledger/fabric-ca-server-config
    depends_on:
      - orderer0.jicki.me
      - orderer1.jicki.me
      - orderer2.jicki.me
 
```
 
 
 
## 配置 Hyperledger Fabric peer
 

 

 
```      
vi  docker-compose-peer.yaml

version: '2'
services:

  couchdb0:
    container_name: couchdb0
    image: hyperledger/fabric-couchdb
    environment:
      - COUCHDB_USER=
      - COUCHDB_PASSWORD=
    ports:
      - "5984:5984"
    volumes:
      # 数据持久化，用于存储链码值
      - ./data/couchdb0/data:/opt/couchdb/data
    networks:
      default:
        aliases:
          - jicki

  couchdb1:
    container_name: couchdb1
    image: hyperledger/fabric-couchdb
    environment:
      - COUCHDB_USER=
      - COUCHDB_PASSWORD=
    ports:
      - "6984:5984"
    volumes:
      # 数据持久化，用于存储链码值
      - ./data/couchdb1/data:/opt/couchdb/data
    networks:
      default:
        aliases:
          - jicki
          
  peer0.org1.jicki.me:
    container_name: peer0.org1.jicki.me
    image: hyperledger/fabric-peer
    environment:
      - CORE_LEDGER_STATE_STATEDATABASE=CouchDB
      - CORE_LEDGER_STATE_COUCHDBCONFIG_COUCHDBADDRESS=couchdb0:5984
      
      - CORE_PEER_ID=peer0.org1.jicki.me
      - CORE_PEER_NETWORKID=jicki
      - CORE_PEER_ADDRESS=peer0.org1.jicki.me:7051
      - CORE_PEER_CHAINCODELISTENADDRESS=peer0.org1.jicki.me:7052
      - CORE_PEER_GOSSIP_EXTERNALENDPOINT=peer0.org1.jicki.me:7051
      - CORE_PEER_LOCALMSPID=Org1MSP

      - CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
      - CORE_VM_DOCKER_HOSTCONFIG_NETWORKMODE=jicki
      # - CORE_LOGGING_LEVEL=ERROR
      - CORE_LOGGING_LEVEL=DEBUG
      - CORE_VM_DOCKER_HOSTCONFIG_NETWORKMODE=jicki_default
      - CORE_PEER_GOSSIP_SKIPHANDSHAKE=true
      - CORE_PEER_GOSSIP_USELEADERELECTION=true
      - CORE_PEER_GOSSIP_ORGLEADER=false
      - CORE_PEER_PROFILE_ENABLED=false
      - CORE_PEER_TLS_ENABLED=false
      - CORE_PEER_TLS_CERT_FILE=/etc/hyperledger/fabric/tls/server.crt
      - CORE_PEER_TLS_KEY_FILE=/etc/hyperledger/fabric/tls/server.key
      - CORE_PEER_TLS_ROOTCERT_FILE=/etc/hyperledger/fabric/tls/ca.crt
    volumes:
        - /var/run/:/host/var/run/
        - ./crypto-config/peerOrganizations/org1.jicki.me/peers/peer0.org1.jicki.me/msp:/etc/hyperledger/fabric/msp
        - ./crypto-config/peerOrganizations/org1.jicki.me/peers/peer0.org1.jicki.me/tls:/etc/hyperledger/fabric/tls
        # 数据持久化, 存储安装，以及实例化智能合约的数据
        - ./data/peer0org1:/var/hyperledger/production
    working_dir: /opt/gopath/src/github.com/hyperledger/fabric/peer
    command: peer node start
    ports:
      - 7051:7051
      - 7052:7052
      - 7053:7053
    networks:
      default:
        aliases:
          - jicki
    depends_on:
      - couchdb0
      - ca.org1.jicki.me
      - ca.org2.jicki.me
      - orderer0.jicki.me
      - orderer1.jicki.me
      - orderer2.jicki.me

  peer0.org2.jicki.me:
    container_name: peer0.org2.jicki.me
    image: hyperledger/fabric-peer
    environment:
      - CORE_LEDGER_STATE_STATEDATABASE=CouchDB
      - CORE_LEDGER_STATE_COUCHDBCONFIG_COUCHDBADDRESS=couchdb1:5984
      
      - CORE_PEER_ID=peer0.org2.jicki.me
      - CORE_PEER_NETWORKID=jicki
      - CORE_PEER_ADDRESS=peer0.org2.jicki.me:7051
      - CORE_PEER_CHAINCODELISTENADDRESS=peer0.org2.jicki.me:7052
      - CORE_PEER_GOSSIP_EXTERNALENDPOINT=peer0.org2.jicki.me:7051
      - CORE_PEER_LOCALMSPID=Org2MSP

      - CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
      - CORE_VM_DOCKER_HOSTCONFIG_NETWORKMODE=jicki
      # - CORE_LOGGING_LEVEL=ERROR
      - CORE_LOGGING_LEVEL=DEBUG
      - CORE_VM_DOCKER_HOSTCONFIG_NETWORKMODE=jicki_default
      - CORE_PEER_GOSSIP_SKIPHANDSHAKE=true
      - CORE_PEER_GOSSIP_USELEADERELECTION=true
      - CORE_PEER_GOSSIP_ORGLEADER=false
      - CORE_PEER_PROFILE_ENABLED=false
      - CORE_PEER_TLS_ENABLED=false
      - CORE_PEER_TLS_CERT_FILE=/etc/hyperledger/fabric/tls/server.crt
      - CORE_PEER_TLS_KEY_FILE=/etc/hyperledger/fabric/tls/server.key
      - CORE_PEER_TLS_ROOTCERT_FILE=/etc/hyperledger/fabric/tls/ca.crt
    volumes:
        - /var/run/:/host/var/run/
        - ./crypto-config/peerOrganizations/org2.jicki.me/peers/peer0.org2.jicki.me/msp:/etc/hyperledger/fabric/msp
        - ./crypto-config/peerOrganizations/org2.jicki.me/peers/peer0.org2.jicki.me/tls:/etc/hyperledger/fabric/tls
        # 数据持久化, 存储安装，以及实例化智能合约的数据
        - ./data/peer0org2:/var/hyperledger/production
    working_dir: /opt/gopath/src/github.com/hyperledger/fabric/peer
    command: peer node start
    ports:
      - 8051:7051
      - 8052:7052
      - 8053:7053
    networks:
      default:
        aliases:
          - jicki
    depends_on:
      - couchdb1
      - ca.org1.jicki.me
      - ca.org2.jicki.me
      - orderer0.jicki.me
      - orderer1.jicki.me
      - orderer2.jicki.me

  cli:
    container_name: cli
    image: hyperledger/fabric-tools
    tty: true
    environment:
      - GOPATH=/opt/gopath
      - CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
      # - CORE_LOGGING_LEVEL=ERROR
      - CORE_LOGGING_LEVEL=DEBUG
      - CORE_PEER_ID=cli
      - CORE_PEER_ADDRESS=peer0.org1.jicki.me:7051
      - CORE_PEER_LOCALMSPID=Org1MSP
      - CORE_PEER_TLS_ENABLED=false
      - CORE_PEER_TLS_CERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.jicki.me/peers/peer0.org1.jicki.me/tls/server.crt
      - CORE_PEER_TLS_KEY_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.jicki.me/peers/peer0.org1.jicki.me/tls/server.key
      - CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.jicki.me/peers/peer0.org1.jicki.me/tls/ca.crt
      - CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.jicki.me/users/Admin@org1.jicki.me/msp
    working_dir: /opt/gopath/src/github.com/hyperledger/fabric/peer
    volumes:
        - /var/run/:/host/var/run/
        - /opt/golang/go:/opt/go
        - /opt/gopath:/opt/gopath
        - ./peer:/opt/gopath/src/github.com/hyperledger/fabric/peer
        - ./chaincode/go/:/opt/gopath/src/github.com/hyperledger/fabric/jicki/chaincode/go
        - ./crypto-config:/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/
        - ./channel-artifacts:/opt/gopath/src/github.com/hyperledger/fabric/peer/channel-artifacts
    networks:
      default:
        aliases:
          - jicki
```



## 启动 Hyperledger Fabric 服务


```
# 启动 orderer 服务

# 首先启动 zk 与 kafka
[root@localhost jicki]# docker-compose -f docker-compose-zk-kafka.yaml up -d
[root@localhost jicki]# docker-compose -f docker-compose-orderer.yaml up -d
[root@localhost jicki]# docker-compose -f docker-compose-ca.yaml up -d
[root@localhost jicki]# docker-compose -f docker-compose-peer.yaml up -d

```



```
# 下载的镜像

[root@payment jicki]# docker images
REPOSITORY                                  TAG                   IMAGE ID            CREATED             SIZE
hyperledger/fabric-ca                       latest                66cc132bd09c        2 weeks ago         252MB
hyperledger/fabric-tools                    latest                379602873003        2 weeks ago         1.51GB
hyperledger/fabric-orderer                  latest                4baf7789a8ec        2 weeks ago         152MB
hyperledger/fabric-peer                     latest                82c262e65984        2 weeks ago         159MB
hyperledger/fabric-zookeeper                latest                2b51158f3898        3 weeks ago         1.44GB
hyperledger/fabric-kafka                    latest                936aef6db0e6        3 weeks ago         1.45GB
hyperledger/fabric-couchdb                  latest                35228d48a25a        3 months ago        1.56GB

```




```
# 查看启动的服务

[root@localhost jicki]# docker ps -a
CONTAINER ID        IMAGE                                             COMMAND                  CREATED             STATUS                PORTS                                                                                                                           NAMES
eda050252f3d        hyperledger/fabric-peer                           "peer node start"        37 minutes ago      Up 37 minutes               0.0.0.0:7051-7053->7051-7053/tcp                                                                                                peer0.org1.jicki.me
b70279217b98        hyperledger/fabric-peer                           "peer node start"        37 minutes ago      Up 37 minutes               0.0.0.0:8051->7051/tcp, 0.0.0.0:8052->7052/tcp, 0.0.0.0:8053->7053/tcp                                                          peer0.org2.jicki.me
b5ad5fad6234        hyperledger/fabric-ca                             "sh -c 'fabric-ca-..."   37 minutes ago      Up 37 minutes               0.0.0.0:7054->7054/tcp                                                                                                          ca.org1.jicki.me
87f44a68e3a6        hyperledger/fabric-ca                             "sh -c 'fabric-ca-..."   37 minutes ago      Up 37 minutes               0.0.0.0:8054->7054/tcp                                                                                                          ca.org2.jicki.me
58ac578bbb6c        hyperledger/fabric-orderer                        "orderer"                37 minutes ago      Up 37 minutes               0.0.0.0:7050->7050/tcp                                                                                                          orderer0.jicki.me
4d93f59a340f        hyperledger/fabric-orderer                        "orderer"                37 minutes ago      Up 37 minutes               0.0.0.0:9050->7050/tcp                                                                                                          orderer2.jicki.me
c301dd0226c3        hyperledger/fabric-orderer                        "orderer"                37 minutes ago      Up 37 minutes               0.0.0.0:8050->7050/tcp                                                                                                          orderer1.jicki.me
caa696519020        hyperledger/fabric-kafka                          "/docker-entrypoin..."   37 minutes ago      Up 37 minutes               9092-9093/tcp                                                                                                                   kafka2
34f93813953e        hyperledger/fabric-kafka                          "/docker-entrypoin..."   37 minutes ago      Up 37 minutes               9092-9093/tcp                                                                                                                   kafka3
f11d9d98251d        hyperledger/fabric-kafka                          "/docker-entrypoin..."   37 minutes ago      Up 37 minutes               9092-9093/tcp                                                                                                                   kafka0
84d1f2dee0dd        hyperledger/fabric-kafka                          "/docker-entrypoin..."   37 minutes ago      Up 37 minutes               9092-9093/tcp                                                                                                                   kafka1
eb59be8473bd        hyperledger/fabric-zookeeper                      "/docker-entrypoin..."   37 minutes ago      Up 37 minutes               2181/tcp, 2888/tcp, 3888/tcp                                                                                                    zookeeper3
64092529ff38        hyperledger/fabric-tools                          "/bin/bash"              37 minutes ago      Up 37 minutes                                                                                                                                               cli
2186332120e8        hyperledger/fabric-zookeeper                      "/docker-entrypoin..."   37 minutes ago      Up 37 minutes               2181/tcp, 2888/tcp, 3888/tcp                                                                                                    zookeeper2
cc4622306c77        hyperledger/fabric-couchdb                        "tini -- /docker-e..."   37 minutes ago      Up 37 minutes               4369/tcp, 9100/tcp, 0.0.0.0:6984->5984/tcp                                                                                      couchdb1
c44296629e1d        hyperledger/fabric-couchdb                        "tini -- /docker-e..."   37 minutes ago      Up 37 minutes               4369/tcp, 9100/tcp, 0.0.0.0:5984->5984/tcp                                                                                      couchdb0
9531875f430d        hyperledger/fabric-zookeeper                      "/docker-entrypoin..."   37 minutes ago      Up 37 minutes               2181/tcp, 2888/tcp, 3888/tcp                                                                                                    zookeeper1

```


## Hyperledger Fabric 创建 Channel



```
# 上面我们创建了 cli 容器，我们可以直接进入 容器里操作


[root@localhost jicki]# docker exec -it cli bash
root@77962643125a:/opt/gopath/src/github.com/hyperledger/fabric/peer# 


# 执行 创建命令 (未启动 认证)

peer channel create -c mychannel -f ./channel-artifacts/channel.tx --orderer orderer0.jicki.me:7050


# 以下为启用认证

peer channel create -o orderer0.jicki.me:7050 -c mychannel -f ./channel-artifacts/channel.tx --tls $CORE_PEER_TLS_ENABLED --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/jicki.me/orderers/orderer0.jicki.me/msp/tlscacerts/tlsca.jicki.me-cert.pem




# 输出如下:

2018-07-24 06:22:23.897 UTC [viperutil] getKeysRecursively -> DEBU 001 Found map[string]interface{} value for peer.BCCSP
2018-07-24 06:22:23.897 UTC [viperutil] unmarshalJSON -> DEBU 002 Unmarshal JSON: value cannot be unmarshalled: invalid character 'S' looking for beginning of value
2018-07-24 06:22:23.897 UTC [viperutil] getKeysRecursively -> DEBU 003 Found real value for peer.BCCSP.Default setting to string SW
2018-07-24 06:22:23.897 UTC [viperutil] getKeysRecursively -> DEBU 004 Found map[string]interface{} value for peer.BCCSP.SW
2018-07-24 06:22:23.897 UTC [viperutil] unmarshalJSON -> DEBU 005 Unmarshal JSON: value cannot be unmarshalled: invalid character 'S' looking for beginning of value
2018-07-24 06:22:23.897 UTC [viperutil] getKeysRecursively -> DEBU 006 Found real value for peer.BCCSP.SW.Hash setting to string SHA2
2018-07-24 06:22:23.898 UTC [viperutil] unmarshalJSON -> DEBU 007 Unmarshal JSON: value is not a string: 256
2018-07-24 06:22:23.898 UTC [viperutil] getKeysRecursively -> DEBU 008 Found real value for peer.BCCSP.SW.Security setting to int 256
2018-07-24 06:22:23.898 UTC [viperutil] getKeysRecursively -> DEBU 009 Found map[string]interface{} value for peer.BCCSP.SW.FileKeyStore
2018-07-24 06:22:23.898 UTC [viperutil] unmarshalJSON -> DEBU 00a Unmarshal JSON: value cannot be unmarshalled: unexpected end of JSON input
2018-07-24 06:22:23.898 UTC [viperutil] getKeysRecursively -> DEBU 00b Found real value for peer.BCCSP.SW.FileKeyStore.KeyStore setting to string 
2018-07-24 06:22:23.898 UTC [viperutil] getKeysRecursively -> DEBU 00c Found map[string]interface{} value for peer.BCCSP.PKCS11
2018-07-24 06:22:23.898 UTC [viperutil] unmarshalJSON -> DEBU 00d Unmarshal JSON: value is not a string: <nil>
2018-07-24 06:22:23.898 UTC [viperutil] getKeysRecursively -> DEBU 00e Found real value for peer.BCCSP.PKCS11.Security setting to <nil> <nil>
2018-07-24 06:22:23.899 UTC [viperutil] getKeysRecursively -> DEBU 00f Found map[string]interface{} value for peer.BCCSP.PKCS11.FileKeyStore
2018-07-24 06:22:23.899 UTC [viperutil] unmarshalJSON -> DEBU 010 Unmarshal JSON: value is not a string: <nil>
2018-07-24 06:22:23.899 UTC [viperutil] getKeysRecursively -> DEBU 011 Found real value for peer.BCCSP.PKCS11.FileKeyStore.KeyStore setting to <nil> <nil>
2018-07-24 06:22:23.899 UTC [viperutil] unmarshalJSON -> DEBU 012 Unmarshal JSON: value is not a string: <nil>
2018-07-24 06:22:23.899 UTC [viperutil] getKeysRecursively -> DEBU 013 Found real value for peer.BCCSP.PKCS11.Library setting to <nil> <nil>
2018-07-24 06:22:23.900 UTC [viperutil] unmarshalJSON -> DEBU 014 Unmarshal JSON: value is not a string: <nil>
2018-07-24 06:22:23.900 UTC [viperutil] getKeysRecursively -> DEBU 015 Found real value for peer.BCCSP.PKCS11.Label setting to <nil> <nil>
2018-07-24 06:22:23.900 UTC [viperutil] unmarshalJSON -> DEBU 016 Unmarshal JSON: value is not a string: <nil>
2018-07-24 06:22:23.900 UTC [viperutil] getKeysRecursively -> DEBU 017 Found real value for peer.BCCSP.PKCS11.Pin setting to <nil> <nil>
2018-07-24 06:22:23.900 UTC [viperutil] unmarshalJSON -> DEBU 018 Unmarshal JSON: value is not a string: <nil>
2018-07-24 06:22:23.900 UTC [viperutil] getKeysRecursively -> DEBU 019 Found real value for peer.BCCSP.PKCS11.Hash setting to <nil> <nil>
2018-07-24 06:22:23.901 UTC [viperutil] EnhancedExactUnmarshalKey -> DEBU 01a map[peer.BCCSP:map[Default:SW SW:map[Hash:SHA2 Security:256 FileKeyStore:map[KeyStore:]] PKCS11:map[Security:<nil> FileKeyStore:map[KeyStore:<nil>] Library:<nil> Label:<nil> Pin:<nil> Hash:<nil>]]]
2018-07-24 06:22:23.901 UTC [bccsp_sw] openKeyStore -> DEBU 01b KeyStore opened at [/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.jicki.me/users/Admin@org1.jicki.me/msp/keystore]...done
2018-07-24 06:22:23.901 UTC [bccsp] initBCCSP -> DEBU 01c Initialize BCCSP [SW]
2018-07-24 06:22:23.901 UTC [msp] getPemMaterialFromDir -> DEBU 01d Reading directory /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.jicki.me/users/Admin@org1.jicki.me/msp/signcerts
2018-07-24 06:22:23.901 UTC [msp] getPemMaterialFromDir -> DEBU 01e Inspecting file /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.jicki.me/users/Admin@org1.jicki.me/msp/signcerts/Admin@org1.jicki.me-cert.pem
2018-07-24 06:22:23.901 UTC [msp] getPemMaterialFromDir -> DEBU 01f Reading directory /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.jicki.me/users/Admin@org1.jicki.me/msp/cacerts
2018-07-24 06:22:23.901 UTC [msp] getPemMaterialFromDir -> DEBU 020 Inspecting file /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.jicki.me/users/Admin@org1.jicki.me/msp/cacerts/ca.org1.jicki.me-cert.pem
2018-07-24 06:22:23.901 UTC [msp] getPemMaterialFromDir -> DEBU 021 Reading directory /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.jicki.me/users/Admin@org1.jicki.me/msp/admincerts
2018-07-24 06:22:23.901 UTC [msp] getPemMaterialFromDir -> DEBU 022 Inspecting file /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.jicki.me/users/Admin@org1.jicki.me/msp/admincerts/Admin@org1.jicki.me-cert.pem
2018-07-24 06:22:23.901 UTC [msp] getPemMaterialFromDir -> DEBU 023 Reading directory /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.jicki.me/users/Admin@org1.jicki.me/msp/intermediatecerts
2018-07-24 06:22:23.901 UTC [msp] getMspConfig -> DEBU 024 Intermediate certs folder not found at [/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.jicki.me/users/Admin@org1.jicki.me/msp/intermediatecerts]. Skipping. [stat /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.jicki.me/users/Admin@org1.jicki.me/msp/intermediatecerts: no such file or directory]
2018-07-24 06:22:23.901 UTC [msp] getPemMaterialFromDir -> DEBU 025 Reading directory /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.jicki.me/users/Admin@org1.jicki.me/msp/tlscacerts
2018-07-24 06:22:23.901 UTC [msp] getPemMaterialFromDir -> DEBU 026 Inspecting file /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.jicki.me/users/Admin@org1.jicki.me/msp/tlscacerts/tlsca.org1.jicki.me-cert.pem
2018-07-24 06:22:23.901 UTC [msp] getPemMaterialFromDir -> DEBU 027 Reading directory /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.jicki.me/users/Admin@org1.jicki.me/msp/tlsintermediatecerts
2018-07-24 06:22:23.902 UTC [msp] getMspConfig -> DEBU 028 TLS intermediate certs folder not found at [/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.jicki.me/users/Admin@org1.jicki.me/msp/tlsintermediatecerts]. Skipping. [stat /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.jicki.me/users/Admin@org1.jicki.me/msp/tlsintermediatecerts: no such file or directory]
2018-07-24 06:22:23.902 UTC [msp] getPemMaterialFromDir -> DEBU 029 Reading directory /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.jicki.me/users/Admin@org1.jicki.me/msp/crls
2018-07-24 06:22:23.902 UTC [msp] getMspConfig -> DEBU 02a crls folder not found at [/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.jicki.me/users/Admin@org1.jicki.me/msp/crls]. Skipping. [stat /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.jicki.me/users/Admin@org1.jicki.me/msp/crls: no such file or directory]
2018-07-24 06:22:23.902 UTC [msp] getMspConfig -> DEBU 02b MSP configuration file not found at [/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.jicki.me/users/Admin@org1.jicki.me/msp/config.yaml]: [stat /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.jicki.me/users/Admin@org1.jicki.me/msp/config.yaml: no such file or directory]
2018-07-24 06:22:23.902 UTC [msp] newBccspMsp -> DEBU 02c Creating BCCSP-based MSP instance
2018-07-24 06:22:23.902 UTC [msp] New -> DEBU 02d Creating Cache-MSP instance
2018-07-24 06:22:23.902 UTC [msp] loadLocaMSP -> DEBU 02e Created new local MSP
2018-07-24 06:22:23.902 UTC [msp] Setup -> DEBU 02f Setting up MSP instance Org1MSP
2018-07-24 06:22:23.904 UTC [msp/identity] newIdentity -> DEBU 030 Creating identity instance for cert -----BEGIN CERTIFICATE-----
MIICKzCCAdGgAwIBAgIQZYNt6uyySFyj/TRGAVhZUDAKBggqhkjOPQQDAjBnMQsw
CQYDVQQGEwJDTjESMBAGA1UECBMJR3VhbmdEb25nMREwDwYDVQQHEwhTaGVuWmhl
bjEWMBQGA1UEChMNb3JnMS5qaWNraS5tZTEZMBcGA1UEAxMQY2Eub3JnMS5qaWNr
aS5tZTAeFw0xODA3MjQwNjA5NTJaFw0yODA3MjEwNjA5NTJaMGcxCzAJBgNVBAYT
AkNOMRIwEAYDVQQIEwlHdWFuZ0RvbmcxETAPBgNVBAcTCFNoZW5aaGVuMRYwFAYD
VQQKEw1vcmcxLmppY2tpLm1lMRkwFwYDVQQDExBjYS5vcmcxLmppY2tpLm1lMFkw
EwYHKoZIzj0CAQYIKoZIzj0DAQcDQgAEkiL5lK9S/BogKSPwOusjZO2OjVN4LaJF
k+MRPGoLS0qHQazcesKOad6OuMn6iI1g5ZHy+to9wW00EAfF9ZLLRKNfMF0wDgYD
VR0PAQH/BAQDAgGmMA8GA1UdJQQIMAYGBFUdJQAwDwYDVR0TAQH/BAUwAwEB/zAp
BgNVHQ4EIgQgmGcep8QszetzMn18Z/dZ5IdJfxM0m5ehNTkvZgz2dH8wCgYIKoZI
zj0EAwIDSAAwRQIhAPdhlQHaiwUJMLQ7AGJtWREjWnrawY1SNRL5D4iVj04oAiAi
Qo9YAIn9iEaB/sIrFrRrh254B8JziVFHuMzC4CYR9g==
-----END CERTIFICATE-----
2018-07-24 06:22:23.904 UTC [msp/identity] newIdentity -> DEBU 031 Creating identity instance for cert -----BEGIN CERTIFICATE-----
MIICFTCCAbygAwIBAgIRAOfRh4uSC37m50yLeNgNWIowCgYIKoZIzj0EAwIwZzEL
MAkGA1UEBhMCQ04xEjAQBgNVBAgTCUd1YW5nRG9uZzERMA8GA1UEBxMIU2hlblpo
ZW4xFjAUBgNVBAoTDW9yZzEuamlja2kubWUxGTAXBgNVBAMTEGNhLm9yZzEuamlj
a2kubWUwHhcNMTgwNzI0MDYwOTUyWhcNMjgwNzIxMDYwOTUyWjBjMQswCQYDVQQG
EwJDTjESMBAGA1UECBMJR3VhbmdEb25nMREwDwYDVQQHEwhTaGVuWmhlbjEPMA0G
A1UECxMGY2xpZW50MRwwGgYDVQQDDBNBZG1pbkBvcmcxLmppY2tpLm1lMFkwEwYH
KoZIzj0CAQYIKoZIzj0DAQcDQgAElDYNkzvDd5FvK7VWs9TLwNX2jbq4q7MV0Rwf
J/RLmGY3jdD70rmSUYV4xUUy3OMuPWoeCK0K63qUhGAFt25qGaNNMEswDgYDVR0P
AQH/BAQDAgeAMAwGA1UdEwEB/wQCMAAwKwYDVR0jBCQwIoAgmGcep8QszetzMn18
Z/dZ5IdJfxM0m5ehNTkvZgz2dH8wCgYIKoZIzj0EAwIDRwAwRAIgZbJaWDCZypxp
JxdwoN4mp/NQE5O00ZYWcYkiXW1zzPwCIGRZTxyaLtHgO38k7DZxTrjMda2v84Ra
hPsLuLRBfhWs
-----END CERTIFICATE-----
2018-07-24 06:22:23.938 UTC [msp/identity] newIdentity -> DEBU 032 Creating identity instance for cert -----BEGIN CERTIFICATE-----
MIICFTCCAbygAwIBAgIRAOfRh4uSC37m50yLeNgNWIowCgYIKoZIzj0EAwIwZzEL
MAkGA1UEBhMCQ04xEjAQBgNVBAgTCUd1YW5nRG9uZzERMA8GA1UEBxMIU2hlblpo
ZW4xFjAUBgNVBAoTDW9yZzEuamlja2kubWUxGTAXBgNVBAMTEGNhLm9yZzEuamlj
a2kubWUwHhcNMTgwNzI0MDYwOTUyWhcNMjgwNzIxMDYwOTUyWjBjMQswCQYDVQQG
EwJDTjESMBAGA1UECBMJR3VhbmdEb25nMREwDwYDVQQHEwhTaGVuWmhlbjEPMA0G
A1UECxMGY2xpZW50MRwwGgYDVQQDDBNBZG1pbkBvcmcxLmppY2tpLm1lMFkwEwYH
KoZIzj0CAQYIKoZIzj0DAQcDQgAElDYNkzvDd5FvK7VWs9TLwNX2jbq4q7MV0Rwf
J/RLmGY3jdD70rmSUYV4xUUy3OMuPWoeCK0K63qUhGAFt25qGaNNMEswDgYDVR0P
AQH/BAQDAgeAMAwGA1UdEwEB/wQCMAAwKwYDVR0jBCQwIoAgmGcep8QszetzMn18
Z/dZ5IdJfxM0m5ehNTkvZgz2dH8wCgYIKoZIzj0EAwIDRwAwRAIgZbJaWDCZypxp
JxdwoN4mp/NQE5O00ZYWcYkiXW1zzPwCIGRZTxyaLtHgO38k7DZxTrjMda2v84Ra
hPsLuLRBfhWs
-----END CERTIFICATE-----
2018-07-24 06:22:23.939 UTC [bccsp_sw] loadPrivateKey -> DEBU 033 Loading private key [c8edb858abc7a4589458419d9ca74125830115e1e3544cc85c7a4a91f8b84b52] at [/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.jicki.me/users/Admin@org1.jicki.me/msp/keystore/c8edb858abc7a4589458419d9ca74125830115e1e3544cc85c7a4a91f8b84b52_sk]...
2018-07-24 06:22:23.939 UTC [msp/identity] newIdentity -> DEBU 034 Creating identity instance for cert -----BEGIN CERTIFICATE-----
MIICFTCCAbygAwIBAgIRAOfRh4uSC37m50yLeNgNWIowCgYIKoZIzj0EAwIwZzEL
MAkGA1UEBhMCQ04xEjAQBgNVBAgTCUd1YW5nRG9uZzERMA8GA1UEBxMIU2hlblpo
ZW4xFjAUBgNVBAoTDW9yZzEuamlja2kubWUxGTAXBgNVBAMTEGNhLm9yZzEuamlj
a2kubWUwHhcNMTgwNzI0MDYwOTUyWhcNMjgwNzIxMDYwOTUyWjBjMQswCQYDVQQG
EwJDTjESMBAGA1UECBMJR3VhbmdEb25nMREwDwYDVQQHEwhTaGVuWmhlbjEPMA0G
A1UECxMGY2xpZW50MRwwGgYDVQQDDBNBZG1pbkBvcmcxLmppY2tpLm1lMFkwEwYH
KoZIzj0CAQYIKoZIzj0DAQcDQgAElDYNkzvDd5FvK7VWs9TLwNX2jbq4q7MV0Rwf
J/RLmGY3jdD70rmSUYV4xUUy3OMuPWoeCK0K63qUhGAFt25qGaNNMEswDgYDVR0P
AQH/BAQDAgeAMAwGA1UdEwEB/wQCMAAwKwYDVR0jBCQwIoAgmGcep8QszetzMn18
Z/dZ5IdJfxM0m5ehNTkvZgz2dH8wCgYIKoZIzj0EAwIDRwAwRAIgZbJaWDCZypxp
JxdwoN4mp/NQE5O00ZYWcYkiXW1zzPwCIGRZTxyaLtHgO38k7DZxTrjMda2v84Ra
hPsLuLRBfhWs
-----END CERTIFICATE-----
2018-07-24 06:22:23.939 UTC [msp] setupSigningIdentity -> DEBU 035 Signing identity expires at 2028-07-21 06:09:52 +0000 UTC
2018-07-24 06:22:23.940 UTC [msp] Validate -> DEBU 036 MSP Org1MSP validating identity
2018-07-24 06:22:23.940 UTC [msp] GetDefaultSigningIdentity -> DEBU 037 Obtaining default signing identity
2018-07-24 06:22:23.940 UTC [grpc] Printf -> DEBU 038 parsed scheme: ""
2018-07-24 06:22:23.940 UTC [grpc] Printf -> DEBU 039 scheme "" not registered, fallback to default scheme
2018-07-24 06:22:23.941 UTC [grpc] Printf -> DEBU 03a ccResolverWrapper: sending new addresses to cc: [{orderer0.jicki.me:7050 0  <nil>}]
2018-07-24 06:22:23.941 UTC [grpc] Printf -> DEBU 03b ClientConn switching balancer to "pick_first"
2018-07-24 06:22:23.941 UTC [grpc] Printf -> DEBU 03c pickfirstBalancer: HandleSubConnStateChange: 0xc42029faa0, CONNECTING
2018-07-24 06:22:23.942 UTC [grpc] Printf -> DEBU 03d pickfirstBalancer: HandleSubConnStateChange: 0xc42029faa0, READY
2018-07-24 06:22:23.942 UTC [channelCmd] InitCmdFactory -> INFO 03e Endorser and orderer connections initialized
2018-07-24 06:22:23.943 UTC [msp] GetDefaultSigningIdentity -> DEBU 03f Obtaining default signing identity
2018-07-24 06:22:23.943 UTC [msp] GetDefaultSigningIdentity -> DEBU 040 Obtaining default signing identity
2018-07-24 06:22:23.943 UTC [msp/identity] Sign -> DEBU 041 Sign: plaintext: 0A9A060A074F7267314D5350128E062D...53616D706C65436F6E736F727469756D 
2018-07-24 06:22:23.943 UTC [msp/identity] Sign -> DEBU 042 Sign: digest: 05D85C87A943B7019BC29F514DAD53B82402CCCD52A4A361FEED8CFA61161672 
2018-07-24 06:22:23.943 UTC [msp] GetDefaultSigningIdentity -> DEBU 043 Obtaining default signing identity
2018-07-24 06:22:23.943 UTC [msp] GetDefaultSigningIdentity -> DEBU 044 Obtaining default signing identity
2018-07-24 06:22:23.943 UTC [msp/identity] Sign -> DEBU 045 Sign: plaintext: 0AD1060A1508021A06089F8BDBDA0522...5D91B85DABA6FA89A54E2320533B5E9A 
2018-07-24 06:22:23.943 UTC [msp/identity] Sign -> DEBU 046 Sign: digest: 291307330D6AF0081F47EB02FEC1257C5990E12AAB57E4E7FFA40EBA6591E30F 
2018-07-24 06:22:23.943 UTC [grpc] Printf -> DEBU 047 parsed scheme: ""
2018-07-24 06:22:23.943 UTC [grpc] Printf -> DEBU 048 scheme "" not registered, fallback to default scheme
2018-07-24 06:22:23.943 UTC [grpc] Printf -> DEBU 049 ccResolverWrapper: sending new addresses to cc: [{orderer0.jicki.me:7050 0  <nil>}]
2018-07-24 06:22:23.943 UTC [grpc] Printf -> DEBU 04a ClientConn switching balancer to "pick_first"
2018-07-24 06:22:23.943 UTC [grpc] Printf -> DEBU 04b pickfirstBalancer: HandleSubConnStateChange: 0xc4201b6770, CONNECTING
2018-07-24 06:22:23.944 UTC [grpc] Printf -> DEBU 04c pickfirstBalancer: HandleSubConnStateChange: 0xc4201b6770, READY
2018-07-24 06:22:23.996 UTC [msp] GetDefaultSigningIdentity -> DEBU 04d Obtaining default signing identity
2018-07-24 06:22:23.997 UTC [msp] GetDefaultSigningIdentity -> DEBU 04e Obtaining default signing identity
2018-07-24 06:22:23.997 UTC [msp/identity] Sign -> DEBU 04f Sign: plaintext: 0AD1060A1508051A06089F8BDBDA0522...474EE8FC2E3C12080A021A0012021A00 
2018-07-24 06:22:23.997 UTC [msp/identity] Sign -> DEBU 050 Sign: digest: 2CBE60CBF1B0EF9601CD0DF2E6218F133DCB19F89AF6B21C9438592CFF4F4439 
2018-07-24 06:22:23.997 UTC [cli/common] readBlock -> INFO 051 Got status: &{NOT_FOUND}
2018-07-24 06:22:23.997 UTC [msp] GetDefaultSigningIdentity -> DEBU 052 Obtaining default signing identity
2018-07-24 06:22:23.998 UTC [grpc] Printf -> DEBU 053 parsed scheme: ""
2018-07-24 06:22:23.998 UTC [grpc] Printf -> DEBU 054 scheme "" not registered, fallback to default scheme
2018-07-24 06:22:23.998 UTC [grpc] Printf -> DEBU 055 ccResolverWrapper: sending new addresses to cc: [{orderer0.jicki.me:7050 0  <nil>}]
2018-07-24 06:22:23.998 UTC [grpc] Printf -> DEBU 056 ClientConn switching balancer to "pick_first"
2018-07-24 06:22:23.998 UTC [grpc] Printf -> DEBU 057 pickfirstBalancer: HandleSubConnStateChange: 0xc4200b2da0, CONNECTING
2018-07-24 06:22:23.999 UTC [grpc] Printf -> DEBU 058 pickfirstBalancer: HandleSubConnStateChange: 0xc4200b2da0, READY
2018-07-24 06:22:23.999 UTC [channelCmd] InitCmdFactory -> INFO 059 Endorser and orderer connections initialized
2018-07-24 06:22:24.199 UTC [msp] GetDefaultSigningIdentity -> DEBU 05a Obtaining default signing identity
2018-07-24 06:22:24.199 UTC [msp] GetDefaultSigningIdentity -> DEBU 05b Obtaining default signing identity
2018-07-24 06:22:24.199 UTC [msp/identity] Sign -> DEBU 05c Sign: plaintext: 0AD1060A1508051A0608A08BDBDA0522...B1BE874B350112080A021A0012021A00 
2018-07-24 06:22:24.199 UTC [msp/identity] Sign -> DEBU 05d Sign: digest: 6E6A1993487D83337812C286F1E8DCCD1873C3C49A25F0275455B27873C9B278 
2018-07-24 06:22:24.200 UTC [cli/common] readBlock -> INFO 05e Got status: &{SERVICE_UNAVAILABLE}
2018-07-24 06:22:24.200 UTC [msp] GetDefaultSigningIdentity -> DEBU 05f Obtaining default signing identity
2018-07-24 06:22:24.200 UTC [grpc] Printf -> DEBU 060 parsed scheme: ""
2018-07-24 06:22:24.200 UTC [grpc] Printf -> DEBU 061 scheme "" not registered, fallback to default scheme
2018-07-24 06:22:24.200 UTC [grpc] Printf -> DEBU 062 ccResolverWrapper: sending new addresses to cc: [{orderer0.jicki.me:7050 0  <nil>}]
2018-07-24 06:22:24.200 UTC [grpc] Printf -> DEBU 063 ClientConn switching balancer to "pick_first"
2018-07-24 06:22:24.200 UTC [grpc] Printf -> DEBU 064 pickfirstBalancer: HandleSubConnStateChange: 0xc42036a340, CONNECTING
2018-07-24 06:22:24.201 UTC [grpc] Printf -> DEBU 065 pickfirstBalancer: HandleSubConnStateChange: 0xc42036a340, READY
2018-07-24 06:22:24.201 UTC [channelCmd] InitCmdFactory -> INFO 066 Endorser and orderer connections initialized
2018-07-24 06:22:24.402 UTC [msp] GetDefaultSigningIdentity -> DEBU 067 Obtaining default signing identity
2018-07-24 06:22:24.402 UTC [msp] GetDefaultSigningIdentity -> DEBU 068 Obtaining default signing identity
2018-07-24 06:22:24.402 UTC [msp/identity] Sign -> DEBU 069 Sign: plaintext: 0AD1060A1508051A0608A08BDBDA0522...F4359830947712080A021A0012021A00 
2018-07-24 06:22:24.402 UTC [msp/identity] Sign -> DEBU 06a Sign: digest: D787F07FD1BEC80AE6FF0138D0B9817382BFF08098E187F82863B88EA72B30CA 
2018-07-24 06:22:24.403 UTC [cli/common] readBlock -> INFO 06b Got status: &{SERVICE_UNAVAILABLE}
2018-07-24 06:22:24.403 UTC [msp] GetDefaultSigningIdentity -> DEBU 06c Obtaining default signing identity
2018-07-24 06:22:24.403 UTC [grpc] Printf -> DEBU 06d parsed scheme: ""
2018-07-24 06:22:24.403 UTC [grpc] Printf -> DEBU 06e scheme "" not registered, fallback to default scheme
2018-07-24 06:22:24.403 UTC [grpc] Printf -> DEBU 06f ccResolverWrapper: sending new addresses to cc: [{orderer0.jicki.me:7050 0  <nil>}]
2018-07-24 06:22:24.403 UTC [grpc] Printf -> DEBU 070 ClientConn switching balancer to "pick_first"
2018-07-24 06:22:24.403 UTC [grpc] Printf -> DEBU 071 pickfirstBalancer: HandleSubConnStateChange: 0xc42036aa00, CONNECTING
2018-07-24 06:22:24.404 UTC [grpc] Printf -> DEBU 072 pickfirstBalancer: HandleSubConnStateChange: 0xc42036aa00, READY
2018-07-24 06:22:24.404 UTC [channelCmd] InitCmdFactory -> INFO 073 Endorser and orderer connections initialized
2018-07-24 06:22:24.604 UTC [msp] GetDefaultSigningIdentity -> DEBU 074 Obtaining default signing identity
2018-07-24 06:22:24.605 UTC [msp] GetDefaultSigningIdentity -> DEBU 075 Obtaining default signing identity
2018-07-24 06:22:24.605 UTC [msp/identity] Sign -> DEBU 076 Sign: plaintext: 0AD1060A1508051A0608A08BDBDA0522...13EE91D7C10B12080A021A0012021A00 
2018-07-24 06:22:24.605 UTC [msp/identity] Sign -> DEBU 077 Sign: digest: 3DB8D072AACD58ADCF8317E0051F060C092AA61CDBF789C6B286F0108EB6175F 
2018-07-24 06:22:24.609 UTC [cli/common] readBlock -> INFO 078 Received block: 0





# 创建以后生成文件 mychannel.block

# ls -lt
total 16
-rw-r--r-- 1 root root 15417 Jul 24 06:22 mychannel.block
drwxr-xr-x 2 root root   111 Jul 24 06:19 channel-artifacts
drwxr-xr-x 4 root root    69 Jul 24 06:14 crypto

```



## Hyperledger Fabric 加入 Channel


> 我们这边有2个 peer 所以需要分别加入, 后续有多少个 peer 都需要加入到 Channel 中

```
# peer0.org1.jicki.me 加入 此 channel 中，首先需要查看如下 环境变量


echo $CORE_PEER_LOCALMSPID
echo $CORE_PEER_ADDRESS
echo $CORE_PEER_MSPCONFIGPATH
echo $CORE_PEER_TLS_ROOTCERT_FILE


# 加入 channel

peer channel join -b mychannel.block


# 输出如下: 

2018-07-24 06:36:54.007 UTC [msp] setupSigningIdentity -> DEBU 035 Signing identity expires at 2028-07-21 06:09:52 +0000 UTC
2018-07-24 06:36:54.007 UTC [msp] Validate -> DEBU 036 MSP Org1MSP validating identity
2018-07-24 06:36:54.008 UTC [msp] GetDefaultSigningIdentity -> DEBU 037 Obtaining default signing identity
2018-07-24 06:36:54.008 UTC [grpc] Printf -> DEBU 038 parsed scheme: ""
2018-07-24 06:36:54.008 UTC [grpc] Printf -> DEBU 039 scheme "" not registered, fallback to default scheme
2018-07-24 06:36:54.008 UTC [grpc] Printf -> DEBU 03a ccResolverWrapper: sending new addresses to cc: [{peer0.org1.jicki.me:7051 0  <nil>}]
2018-07-24 06:36:54.008 UTC [grpc] Printf -> DEBU 03b ClientConn switching balancer to "pick_first"
2018-07-24 06:36:54.009 UTC [grpc] Printf -> DEBU 03c pickfirstBalancer: HandleSubConnStateChange: 0xc4200b2b30, CONNECTING
2018-07-24 06:36:54.010 UTC [grpc] Printf -> DEBU 03d pickfirstBalancer: HandleSubConnStateChange: 0xc4200b2b30, READY
2018-07-24 06:36:54.010 UTC [channelCmd] InitCmdFactory -> INFO 03e Endorser and orderer connections initialized
2018-07-24 06:36:54.010 UTC [msp/identity] Sign -> DEBU 03f Sign: plaintext: 0A97070A5B08011A0B088692DBDA0510...3E43F46E31BD1A080A000A000A000A00 
2018-07-24 06:36:54.010 UTC [msp/identity] Sign -> DEBU 040 Sign: digest: 3B18C9F6B95106DEBA0F1D3FE94440AC3B1CCE13BA1C8D09BC455B04D47B0148 
2018-07-24 06:36:54.093 UTC [channelCmd] executeJoin -> INFO 041 Successfully submitted proposal to join channel


```


```
# peer1.org2.jicki.me 加入 此 channel 中，这里配置一下环境变量

export CORE_PEER_LOCALMSPID="Org2MSP"
export CORE_PEER_ADDRESS=peer0.org2.jicki.me:7051
export CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.jicki.me/peers/peer0.org2.jicki.me/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.jicki.me/users/Admin@org2.jicki.me/msp


# 加入 channel

peer channel join -b mychannel.block



# 输入如下:

2018-07-24 06:45:40.044 UTC [msp] setupSigningIdentity -> DEBU 035 Signing identity expires at 2028-07-21 06:09:52 +0000 UTC
2018-07-24 06:45:40.044 UTC [msp] Validate -> DEBU 036 MSP Org2MSP validating identity
2018-07-24 06:45:40.045 UTC [msp] GetDefaultSigningIdentity -> DEBU 037 Obtaining default signing identity
2018-07-24 06:45:40.046 UTC [grpc] Printf -> DEBU 038 parsed scheme: ""
2018-07-24 06:45:40.046 UTC [grpc] Printf -> DEBU 039 scheme "" not registered, fallback to default scheme
2018-07-24 06:45:40.046 UTC [grpc] Printf -> DEBU 03a ccResolverWrapper: sending new addresses to cc: [{peer0.org2.jicki.me:7051 0  <nil>}]
2018-07-24 06:45:40.046 UTC [grpc] Printf -> DEBU 03b ClientConn switching balancer to "pick_first"
2018-07-24 06:45:40.046 UTC [grpc] Printf -> DEBU 03c pickfirstBalancer: HandleSubConnStateChange: 0xc4201b08e0, CONNECTING
2018-07-24 06:45:40.047 UTC [grpc] Printf -> DEBU 03d pickfirstBalancer: HandleSubConnStateChange: 0xc4201b08e0, READY
2018-07-24 06:45:40.047 UTC [channelCmd] InitCmdFactory -> INFO 03e Endorser and orderer connections initialized
2018-07-24 06:45:40.048 UTC [msp/identity] Sign -> DEBU 03f Sign: plaintext: 0A9B070A5B08011A0B089496DBDA0510...3E43F46E31BD1A080A000A000A000A00 
2018-07-24 06:45:40.048 UTC [msp/identity] Sign -> DEBU 040 Sign: digest: 3A9F8A7C7E1CDF74651E9094F35AC3218DFFF6112D6049039EAA84B3A2F38954 
2018-07-24 06:45:40.137 UTC [channelCmd] executeJoin -> INFO 041 Successfully submitted proposal to join channel

```


## Hyperledger Fabric 锚节点

>  锚节点通过广播的方式通知有新节点加入


```
# 使用Org1的管理员身份更新锚节点配置 

# 同样需要先配置变量

export CORE_PEER_LOCALMSPID="Org1MSP"
export CORE_PEER_ADDRESS=peer0.org1.jicki.me:7051
export CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.jicki.me/peers/peer0.org1.jicki.me/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.jicki.me/users/Admin@org1.jicki.me/msp



# 未开启认证的方式

peer channel update -o orderer0.jicki.me:7050 -c mychannel -f ./channel-artifacts/Org1MSPanchors.tx



# 开启认证的方式

peer channel update -o orderer0.jicki.me:7050 -c mychannel -f ./channel-artifacts/Org1MSPanchors.tx --tls $CORE_PEER_TLS_ENABLED --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/jicki.me/orderers/orderer0.jicki.me/msp/tlscacerts/tlsca.jicki.me-cert.pem



# 输出如下:

2018-07-24 06:51:39.351 UTC [msp] setupSigningIdentity -> DEBU 035 Signing identity expires at 2028-07-21 06:09:52 +0000 UTC
2018-07-24 06:51:39.351 UTC [msp] Validate -> DEBU 036 MSP Org1MSP validating identity
2018-07-24 06:51:39.352 UTC [msp] GetDefaultSigningIdentity -> DEBU 037 Obtaining default signing identity
2018-07-24 06:51:39.352 UTC [grpc] Printf -> DEBU 038 parsed scheme: ""
2018-07-24 06:51:39.352 UTC [grpc] Printf -> DEBU 039 scheme "" not registered, fallback to default scheme
2018-07-24 06:51:39.352 UTC [grpc] Printf -> DEBU 03a ccResolverWrapper: sending new addresses to cc: [{orderer0.jicki.me:7050 0  <nil>}]
2018-07-24 06:51:39.352 UTC [grpc] Printf -> DEBU 03b ClientConn switching balancer to "pick_first"
2018-07-24 06:51:39.352 UTC [grpc] Printf -> DEBU 03c pickfirstBalancer: HandleSubConnStateChange: 0xc4201bcfe0, CONNECTING
2018-07-24 06:51:39.353 UTC [grpc] Printf -> DEBU 03d pickfirstBalancer: HandleSubConnStateChange: 0xc4201bcfe0, READY
2018-07-24 06:51:39.353 UTC [channelCmd] InitCmdFactory -> INFO 03e Endorser and orderer connections initialized
2018-07-24 06:51:39.354 UTC [msp] GetDefaultSigningIdentity -> DEBU 03f Obtaining default signing identity
2018-07-24 06:51:39.354 UTC [msp] GetDefaultSigningIdentity -> DEBU 040 Obtaining default signing identity
2018-07-24 06:51:39.354 UTC [msp/identity] Sign -> DEBU 041 Sign: plaintext: 0A9A060A074F7267314D5350128E062D...2A0641646D696E732A0641646D696E73 
2018-07-24 06:51:39.354 UTC [msp/identity] Sign -> DEBU 042 Sign: digest: E7C240D43411691160518739EF34E45622B7B3BBD27997A77D23A3EB6D1B49D8 
2018-07-24 06:51:39.354 UTC [msp] GetDefaultSigningIdentity -> DEBU 043 Obtaining default signing identity
2018-07-24 06:51:39.354 UTC [msp] GetDefaultSigningIdentity -> DEBU 044 Obtaining default signing identity
2018-07-24 06:51:39.354 UTC [msp/identity] Sign -> DEBU 045 Sign: plaintext: 0AD1060A1508021A0608FB98DBDA0522...5F7DFD468DE7ECB274BF2498F128217E 
2018-07-24 06:51:39.354 UTC [msp/identity] Sign -> DEBU 046 Sign: digest: D95D1BD9A824CE13EBB62D6C9D7CF3A95959227F440AA3E9C8622B77CF27B48C 
2018-07-24 06:51:39.354 UTC [grpc] Printf -> DEBU 047 parsed scheme: ""
2018-07-24 06:51:39.354 UTC [grpc] Printf -> DEBU 048 scheme "" not registered, fallback to default scheme
2018-07-24 06:51:39.354 UTC [grpc] Printf -> DEBU 049 ccResolverWrapper: sending new addresses to cc: [{orderer0.jicki.me:7050 0  <nil>}]
2018-07-24 06:51:39.354 UTC [grpc] Printf -> DEBU 04a ClientConn switching balancer to "pick_first"
2018-07-24 06:51:39.354 UTC [grpc] Printf -> DEBU 04b pickfirstBalancer: HandleSubConnStateChange: 0xc4201a87f0, CONNECTING
2018-07-24 06:51:39.355 UTC [grpc] Printf -> DEBU 04c pickfirstBalancer: HandleSubConnStateChange: 0xc4201a87f0, READY
2018-07-24 06:51:39.495 UTC [channelCmd] update -> INFO 04d Successfully submitted channel update

```


```
# 使用Org2的管理员身份更新锚节点配置 

# 同样需要先配置变量

export CORE_PEER_LOCALMSPID="Org2MSP"
export CORE_PEER_ADDRESS=peer0.org2.jicki.me:7051
export CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.jicki.me/peers/peer0.org2.jicki.me/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.jicki.me/users/Admin@org2.jicki.me/msp


# 未开启认证的方式

peer channel update -o orderer0.jicki.me:7050 -c mychannel -f ./channel-artifacts/Org2MSPanchors.tx



# 开启认证的方式

peer channel update -o orderer0.jicki.me:7050 -c mychannel -f ./channel-artifacts/Org2MSPanchors.tx --tls $CORE_PEER_TLS_ENABLED --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/jicki.me/orderers/orderer0.jicki.me/msp/tlscacerts/tlsca.jicki.me-cert.pem


# 输出如下:

2018-07-24 06:52:49.095 UTC [msp] setupSigningIdentity -> DEBU 035 Signing identity expires at 2028-07-21 06:09:52 +0000 UTC
2018-07-24 06:52:49.096 UTC [msp] Validate -> DEBU 036 MSP Org2MSP validating identity
2018-07-24 06:52:49.096 UTC [msp] GetDefaultSigningIdentity -> DEBU 037 Obtaining default signing identity
2018-07-24 06:52:49.096 UTC [grpc] Printf -> DEBU 038 parsed scheme: ""
2018-07-24 06:52:49.096 UTC [grpc] Printf -> DEBU 039 scheme "" not registered, fallback to default scheme
2018-07-24 06:52:49.096 UTC [grpc] Printf -> DEBU 03a ccResolverWrapper: sending new addresses to cc: [{orderer0.jicki.me:7050 0  <nil>}]
2018-07-24 06:52:49.096 UTC [grpc] Printf -> DEBU 03b ClientConn switching balancer to "pick_first"
2018-07-24 06:52:49.097 UTC [grpc] Printf -> DEBU 03c pickfirstBalancer: HandleSubConnStateChange: 0xc420344e10, CONNECTING
2018-07-24 06:52:49.098 UTC [grpc] Printf -> DEBU 03d pickfirstBalancer: HandleSubConnStateChange: 0xc420344e10, READY
2018-07-24 06:52:49.098 UTC [channelCmd] InitCmdFactory -> INFO 03e Endorser and orderer connections initialized
2018-07-24 06:52:49.098 UTC [msp] GetDefaultSigningIdentity -> DEBU 03f Obtaining default signing identity
2018-07-24 06:52:49.098 UTC [msp] GetDefaultSigningIdentity -> DEBU 040 Obtaining default signing identity
2018-07-24 06:52:49.098 UTC [msp/identity] Sign -> DEBU 041 Sign: plaintext: 0A9E060A074F7267324D53501292062D...2A0641646D696E732A0641646D696E73 
2018-07-24 06:52:49.098 UTC [msp/identity] Sign -> DEBU 042 Sign: digest: C04065ABF13DC508AD382B3F4D80DC18C5A0F93DD3B3863BFD82E0A434D8CD0F 
2018-07-24 06:52:49.099 UTC [msp] GetDefaultSigningIdentity -> DEBU 043 Obtaining default signing identity
2018-07-24 06:52:49.099 UTC [msp] GetDefaultSigningIdentity -> DEBU 044 Obtaining default signing identity
2018-07-24 06:52:49.099 UTC [msp/identity] Sign -> DEBU 045 Sign: plaintext: 0AD5060A1508021A0608C199DBDA0522...CF2890AD30502AECABB28E83EF8BB57E 
2018-07-24 06:52:49.099 UTC [msp/identity] Sign -> DEBU 046 Sign: digest: FDD8F602C72188C951699B01504E2953BE957C1541B8EC793841C15DC6C0C1CD 
2018-07-24 06:52:49.099 UTC [grpc] Printf -> DEBU 047 parsed scheme: ""
2018-07-24 06:52:49.099 UTC [grpc] Printf -> DEBU 048 scheme "" not registered, fallback to default scheme
2018-07-24 06:52:49.099 UTC [grpc] Printf -> DEBU 049 ccResolverWrapper: sending new addresses to cc: [{orderer0.jicki.me:7050 0  <nil>}]
2018-07-24 06:52:49.099 UTC [grpc] Printf -> DEBU 04a ClientConn switching balancer to "pick_first"
2018-07-24 06:52:49.099 UTC [grpc] Printf -> DEBU 04b pickfirstBalancer: HandleSubConnStateChange: 0xc4201b67d0, CONNECTING
2018-07-24 06:52:49.100 UTC [grpc] Printf -> DEBU 04c pickfirstBalancer: HandleSubConnStateChange: 0xc4201b67d0, READY
2018-07-24 06:52:49.142 UTC [channelCmd] update -> INFO 04d Successfully submitted channel update

```







## Hyperledger Fabric 实例化测试

>  在上面我们已经拷贝了官方的例子，在 chaincode 下, 下面我们来测试一下


### 安装智能合约


```
# cli 部分  ./chaincode/go/:/opt/gopath/src/github.com/hyperledger/fabric/jicki/chaincode/go
# 为 智能合约的目录 我们约定为这个目录 需要预先创建 

mkdir -p /opt/jicki/chaincode/go

cd /opt/jicki/chaincode/go

# 创建以后~我们拷贝官方的 例子进来，方便后面进行合约测试

cp -r /opt/jicki/fabric/examples/chaincode/go/example0* /opt/jicki/chaincode/go/


# 官方这里有5个例子

[root@localhost jicki]# ls -lt chaincode/go/
总用量 0
drwxr-xr-x 3 root root 75 7月  20 15:22 example04
drwxr-xr-x 3 root root 75 7月  20 15:22 example05
drwxr-xr-x 3 root root 47 7月  20 15:22 example01
drwxr-xr-x 3 root root 75 7月  20 15:22 example02
drwxr-xr-x 3 root root 75 7月  20 15:22 example03



# 如上我们挂载的地址为 github.com/hyperledger/fabric/jicki/chaincode/go


# 注: 这里面的 example02 的 package 为 example02 会报错

Error: could not assemble transaction, err Proposal response was not successful, error code 500, msg failed to execute transaction 819b581ce88604e9b6651764324876f2ca7a47d7aeb7ee307f273af867a4a134: error starting container: error starting container: API error (404): oci runtime error: container_linux.go:247: starting container process caused "exec: \"chaincode\": executable file not found in $PATH"


# 将 chaincode.go  chaincode_test.go 中  package 修改成 main 然后在最下面增加 main()函数

func main() {
        err := shim.Start(new(SimpleChaincode))
        if err != nil {
                fmt.Printf("Error starting Simple chaincode: %s", err)
        }
}

```



```
# 安装指定合约到 所有的 peer 节点中，每个节点都必须安装一次

# 同样需要先配置变量

export CORE_PEER_LOCALMSPID="Org1MSP"
export CORE_PEER_ADDRESS=peer0.org1.jicki.me:7051
export CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.jicki.me/peers/peer0.org1.jicki.me/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.jicki.me/users/Admin@org1.jicki.me/msp



# 安装 合约

peer chaincode install -n example2 -p github.com/hyperledger/fabric/jicki/chaincode/go/example02 -v 1.0   



# 输出如下:


2018-07-24 07:14:12.106 UTC [msp] setupSigningIdentity -> DEBU 035 Signing identity expires at 2028-07-21 06:09:52 +0000 UTC
2018-07-24 07:14:12.106 UTC [msp] Validate -> DEBU 036 MSP Org1MSP validating identity
2018-07-24 07:14:12.108 UTC [grpc] Printf -> DEBU 037 parsed scheme: ""
2018-07-24 07:14:12.108 UTC [grpc] Printf -> DEBU 038 scheme "" not registered, fallback to default scheme
2018-07-24 07:14:12.108 UTC [grpc] Printf -> DEBU 039 ccResolverWrapper: sending new addresses to cc: [{peer0.org1.jicki.me:7051 0  <nil>}]
2018-07-24 07:14:12.108 UTC [grpc] Printf -> DEBU 03a ClientConn switching balancer to "pick_first"
2018-07-24 07:14:12.108 UTC [grpc] Printf -> DEBU 03b pickfirstBalancer: HandleSubConnStateChange: 0xc4201a9f30, CONNECTING
2018-07-24 07:14:12.109 UTC [grpc] Printf -> DEBU 03c pickfirstBalancer: HandleSubConnStateChange: 0xc4201a9f30, READY
2018-07-24 07:14:12.110 UTC [grpc] Printf -> DEBU 03d parsed scheme: ""
2018-07-24 07:14:12.110 UTC [grpc] Printf -> DEBU 03e scheme "" not registered, fallback to default scheme
2018-07-24 07:14:12.110 UTC [grpc] Printf -> DEBU 03f ccResolverWrapper: sending new addresses to cc: [{peer0.org1.jicki.me:7051 0  <nil>}]
2018-07-24 07:14:12.110 UTC [grpc] Printf -> DEBU 040 ClientConn switching balancer to "pick_first"
2018-07-24 07:14:12.110 UTC [grpc] Printf -> DEBU 041 pickfirstBalancer: HandleSubConnStateChange: 0xc4203c34d0, CONNECTING
2018-07-24 07:14:12.111 UTC [grpc] Printf -> DEBU 042 pickfirstBalancer: HandleSubConnStateChange: 0xc4203c34d0, READY
2018-07-24 07:14:12.112 UTC [msp] GetDefaultSigningIdentity -> DEBU 043 Obtaining default signing identity
2018-07-24 07:14:12.112 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 044 Using default escc
2018-07-24 07:14:12.112 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 045 Using default vscc
2018-07-24 07:14:12.112 UTC [chaincodeCmd] getChaincodeSpec -> DEBU 046 java chaincode disabled
2018-07-24 07:14:12.150 UTC [golang-platform] getCodeFromFS -> DEBU 047 getCodeFromFS github.com/hyperledger/fabric/jicki/chaincode/go/example02
2018-07-24 07:14:12.289 UTC [golang-platform] func1 -> DEBU 048 Discarding GOROOT package fmt
2018-07-24 07:14:12.289 UTC [golang-platform] func1 -> DEBU 049 Discarding provided package github.com/hyperledger/fabric/core/chaincode/shim
2018-07-24 07:14:12.289 UTC [golang-platform] func1 -> DEBU 04a Discarding provided package github.com/hyperledger/fabric/protos/peer
2018-07-24 07:14:12.289 UTC [golang-platform] func1 -> DEBU 04b Discarding GOROOT package strconv
2018-07-24 07:14:12.290 UTC [golang-platform] func1 -> DEBU 04c skipping dir: /opt/gopath/src/github.com/hyperledger/fabric/jicki/chaincode/go/example02/cmd
2018-07-24 07:14:12.290 UTC [golang-platform] GetDeploymentPayload -> DEBU 04d done
2018-07-24 07:14:12.290 UTC [container] WriteFileToPackage -> DEBU 04e Writing file to tarball: src/github.com/hyperledger/fabric/jicki/chaincode/go/example02/chaincode.go
2018-07-24 07:14:12.292 UTC [container] WriteFileToPackage -> DEBU 04f Writing file to tarball: src/github.com/hyperledger/fabric/jicki/chaincode/go/example02/chaincode_test.go
2018-07-24 07:14:12.293 UTC [msp/identity] Sign -> DEBU 050 Sign: plaintext: 0A98070A5C08031A0C08C4A3DBDA0510...CB8CFF040000FFFFFC7B154200260000 
2018-07-24 07:14:12.293 UTC [msp/identity] Sign -> DEBU 051 Sign: digest: 0C8ECCEA95DE1AAEA9214686FACB2767CF84E1753467EE81AE43FD02E5BA2EF7 
2018-07-24 07:14:12.310 UTC [chaincodeCmd] install -> INFO 052 Installed remotely response:<status:200 payload:"OK" > 

```

```
# 安装指定合约到 所有的 peer 节点中，每个节点都必须安装一次

# 同样需要先配置变量

export CORE_PEER_LOCALMSPID="Org2MSP"
export CORE_PEER_ADDRESS=peer0.org2.jicki.me:7051
export CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.jicki.me/peers/peer0.org2.jicki.me/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.jicki.me/users/Admin@org2.jicki.me/msp



# 安装 合约

peer chaincode install -n example2 -p github.com/hyperledger/fabric/jicki/chaincode/go/example02 -v 1.0  



# 输出如下:

2018-07-24 07:16:05.058 UTC [msp] setupSigningIdentity -> DEBU 035 Signing identity expires at 2028-07-21 06:09:52 +0000 UTC
2018-07-24 07:16:05.058 UTC [msp] Validate -> DEBU 036 MSP Org2MSP validating identity
2018-07-24 07:16:05.059 UTC [grpc] Printf -> DEBU 037 parsed scheme: ""
2018-07-24 07:16:05.059 UTC [grpc] Printf -> DEBU 038 scheme "" not registered, fallback to default scheme
2018-07-24 07:16:05.059 UTC [grpc] Printf -> DEBU 039 ccResolverWrapper: sending new addresses to cc: [{peer0.org2.jicki.me:7051 0  <nil>}]
2018-07-24 07:16:05.059 UTC [grpc] Printf -> DEBU 03a ClientConn switching balancer to "pick_first"
2018-07-24 07:16:05.059 UTC [grpc] Printf -> DEBU 03b pickfirstBalancer: HandleSubConnStateChange: 0xc4203ba070, CONNECTING
2018-07-24 07:16:05.060 UTC [grpc] Printf -> DEBU 03c pickfirstBalancer: HandleSubConnStateChange: 0xc4203ba070, READY
2018-07-24 07:16:05.061 UTC [grpc] Printf -> DEBU 03d parsed scheme: ""
2018-07-24 07:16:05.061 UTC [grpc] Printf -> DEBU 03e scheme "" not registered, fallback to default scheme
2018-07-24 07:16:05.061 UTC [grpc] Printf -> DEBU 03f ccResolverWrapper: sending new addresses to cc: [{peer0.org2.jicki.me:7051 0  <nil>}]
2018-07-24 07:16:05.061 UTC [grpc] Printf -> DEBU 040 ClientConn switching balancer to "pick_first"
2018-07-24 07:16:05.061 UTC [grpc] Printf -> DEBU 041 pickfirstBalancer: HandleSubConnStateChange: 0xc420397e50, CONNECTING
2018-07-24 07:16:05.062 UTC [grpc] Printf -> DEBU 042 pickfirstBalancer: HandleSubConnStateChange: 0xc420397e50, READY
2018-07-24 07:16:05.063 UTC [msp] GetDefaultSigningIdentity -> DEBU 043 Obtaining default signing identity
2018-07-24 07:16:05.063 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 044 Using default escc
2018-07-24 07:16:05.063 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 045 Using default vscc
2018-07-24 07:16:05.063 UTC [chaincodeCmd] getChaincodeSpec -> DEBU 046 java chaincode disabled
2018-07-24 07:16:05.114 UTC [golang-platform] getCodeFromFS -> DEBU 047 getCodeFromFS github.com/hyperledger/fabric/jicki/chaincode/go/example02
2018-07-24 07:16:05.245 UTC [golang-platform] func1 -> DEBU 048 Discarding GOROOT package fmt
2018-07-24 07:16:05.245 UTC [golang-platform] func1 -> DEBU 049 Discarding provided package github.com/hyperledger/fabric/core/chaincode/shim
2018-07-24 07:16:05.245 UTC [golang-platform] func1 -> DEBU 04a Discarding provided package github.com/hyperledger/fabric/protos/peer
2018-07-24 07:16:05.245 UTC [golang-platform] func1 -> DEBU 04b Discarding GOROOT package strconv
2018-07-24 07:16:05.245 UTC [golang-platform] func1 -> DEBU 04c skipping dir: /opt/gopath/src/github.com/hyperledger/fabric/jicki/chaincode/go/example02/cmd
2018-07-24 07:16:05.245 UTC [golang-platform] GetDeploymentPayload -> DEBU 04d done
2018-07-24 07:16:05.245 UTC [container] WriteFileToPackage -> DEBU 04e Writing file to tarball: src/github.com/hyperledger/fabric/jicki/chaincode/go/example02/chaincode.go
2018-07-24 07:16:05.249 UTC [container] WriteFileToPackage -> DEBU 04f Writing file to tarball: src/github.com/hyperledger/fabric/jicki/chaincode/go/example02/chaincode_test.go
2018-07-24 07:16:05.250 UTC [msp/identity] Sign -> DEBU 050 Sign: plaintext: 0A9B070A5B08031A0B08B5A4DBDA0510...CB8CFF040000FFFFFC7B154200260000 
2018-07-24 07:16:05.250 UTC [msp/identity] Sign -> DEBU 051 Sign: digest: 9C4BD67CAC8EDCDC12BE6F8A0BD1309C6A41039013D30BB22593D0E00478D433 
2018-07-24 07:16:05.267 UTC [chaincodeCmd] install -> INFO 052 Installed remotely response:<status:200 payload:"OK" > 

```

### 实例化 Chaincode


> 这里无论多少个 peer 节点, 实例化只需要实例化一次，就可以。


```
# 实例化合约 (未认证)
peer chaincode instantiate -o orderer0.jicki.me:7050 -C mychannel -n example2 -c '{"Args":["init","A","100","B","50"]}' -P "OR ('Org1MSP.member','Org2MSP.member')" -v 1.0



# 实例化合约 (已认证)
peer chaincode instantiate -o orderer0.jicki.me:7050 --tls $CORE_PEER_TLS_ENABLED --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/jicki.me/orderers/orderer0.jicki.me/msp/tlscacerts/tlsca.jicki.me-cert.pem -C mychannel -n example2 -c '{"Args":["init","A","100","B","50"]}' -P "OR ('Org1MSP.member','Org2MSP.member')" -v 1.0




# 输出如下:

2018-07-24 08:08:18.109 UTC [msp] setupSigningIdentity -> DEBU 035 Signing identity expires at 2028-07-21 06:09:52 +0000 UTC
2018-07-24 08:08:18.109 UTC [msp] Validate -> DEBU 036 MSP Org1MSP validating identity
2018-07-24 08:08:18.110 UTC [grpc] Printf -> DEBU 037 parsed scheme: ""
2018-07-24 08:08:18.110 UTC [grpc] Printf -> DEBU 038 scheme "" not registered, fallback to default scheme
2018-07-24 08:08:18.110 UTC [grpc] Printf -> DEBU 039 ccResolverWrapper: sending new addresses to cc: [{peer0.org1.jicki.me:7051 0  <nil>}]
2018-07-24 08:08:18.110 UTC [grpc] Printf -> DEBU 03a ClientConn switching balancer to "pick_first"
2018-07-24 08:08:18.110 UTC [grpc] Printf -> DEBU 03b pickfirstBalancer: HandleSubConnStateChange: 0xc4201b42f0, CONNECTING
2018-07-24 08:08:18.112 UTC [grpc] Printf -> DEBU 03c pickfirstBalancer: HandleSubConnStateChange: 0xc4201b42f0, READY
2018-07-24 08:08:18.112 UTC [grpc] Printf -> DEBU 03d parsed scheme: ""
2018-07-24 08:08:18.113 UTC [grpc] Printf -> DEBU 03e scheme "" not registered, fallback to default scheme
2018-07-24 08:08:18.113 UTC [grpc] Printf -> DEBU 03f ccResolverWrapper: sending new addresses to cc: [{peer0.org1.jicki.me:7051 0  <nil>}]
2018-07-24 08:08:18.113 UTC [grpc] Printf -> DEBU 040 ClientConn switching balancer to "pick_first"
2018-07-24 08:08:18.113 UTC [grpc] Printf -> DEBU 041 pickfirstBalancer: HandleSubConnStateChange: 0xc4204d17c0, CONNECTING
2018-07-24 08:08:18.114 UTC [grpc] Printf -> DEBU 042 pickfirstBalancer: HandleSubConnStateChange: 0xc4204d17c0, READY
2018-07-24 08:08:18.114 UTC [msp] GetDefaultSigningIdentity -> DEBU 043 Obtaining default signing identity
2018-07-24 08:08:18.114 UTC [grpc] Printf -> DEBU 044 parsed scheme: ""
2018-07-24 08:08:18.114 UTC [grpc] Printf -> DEBU 045 scheme "" not registered, fallback to default scheme
2018-07-24 08:08:18.115 UTC [grpc] Printf -> DEBU 046 ccResolverWrapper: sending new addresses to cc: [{orderer0.jicki.me:7050 0  <nil>}]
2018-07-24 08:08:18.115 UTC [grpc] Printf -> DEBU 047 ClientConn switching balancer to "pick_first"
2018-07-24 08:08:18.115 UTC [grpc] Printf -> DEBU 048 pickfirstBalancer: HandleSubConnStateChange: 0xc42038cc80, CONNECTING
2018-07-24 08:08:18.115 UTC [grpc] Printf -> DEBU 049 pickfirstBalancer: HandleSubConnStateChange: 0xc42038cc80, READY
2018-07-24 08:08:18.116 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 04a Using default escc
2018-07-24 08:08:18.116 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 04b Using default vscc
2018-07-24 08:08:18.116 UTC [chaincodeCmd] getChaincodeSpec -> DEBU 04c java chaincode disabled
2018-07-24 08:08:18.116 UTC [msp/identity] Sign -> DEBU 04d Sign: plaintext: 0AA2070A6608031A0B08F2BCDBDA0510...324D53500A04657363630A0476736363 
2018-07-24 08:08:18.117 UTC [msp/identity] Sign -> DEBU 04e Sign: digest: 1EFD554F4267BE9DB8B4CE96805B5DBBE754AB029709B44CE53ACDFE8AC12199 
2018-07-24 08:08:28.419 UTC [msp/identity] Sign -> DEBU 04f Sign: plaintext: 0AA2070A6608031A0B08F2BCDBDA0510...1BCC206523F19FEE0660CF2638EEF42F 
2018-07-24 08:08:28.419 UTC [msp/identity] Sign -> DEBU 050 Sign: digest: D40A8051D08456F6E17CE0027445EBE96AC80795550739F8877116F1386D44A8 

```


### 操作智能合约


```
# query 查询方法


# 查询 A 账户里的余额

peer chaincode query -C mychannel -n example2 -c '{"Args":["query","A"]}'


# 输出如下:
2018-07-24 08:10:18.229 UTC [msp] setupSigningIdentity -> DEBU 035 Signing identity expires at 2028-07-21 06:09:52 +0000 UTC
2018-07-24 08:10:18.229 UTC [msp] Validate -> DEBU 036 MSP Org1MSP validating identity
2018-07-24 08:10:18.231 UTC [grpc] Printf -> DEBU 037 parsed scheme: ""
2018-07-24 08:10:18.231 UTC [grpc] Printf -> DEBU 038 scheme "" not registered, fallback to default scheme
2018-07-24 08:10:18.231 UTC [grpc] Printf -> DEBU 039 ccResolverWrapper: sending new addresses to cc: [{peer0.org1.jicki.me:7051 0  <nil>}]
2018-07-24 08:10:18.231 UTC [grpc] Printf -> DEBU 03a ClientConn switching balancer to "pick_first"
2018-07-24 08:10:18.231 UTC [grpc] Printf -> DEBU 03b pickfirstBalancer: HandleSubConnStateChange: 0xc4201b5930, CONNECTING
2018-07-24 08:10:18.232 UTC [grpc] Printf -> DEBU 03c pickfirstBalancer: HandleSubConnStateChange: 0xc4201b5930, READY
2018-07-24 08:10:18.233 UTC [grpc] Printf -> DEBU 03d parsed scheme: ""
2018-07-24 08:10:18.233 UTC [grpc] Printf -> DEBU 03e scheme "" not registered, fallback to default scheme
2018-07-24 08:10:18.233 UTC [grpc] Printf -> DEBU 03f ccResolverWrapper: sending new addresses to cc: [{peer0.org1.jicki.me:7051 0  <nil>}]
2018-07-24 08:10:18.233 UTC [grpc] Printf -> DEBU 040 ClientConn switching balancer to "pick_first"
2018-07-24 08:10:18.233 UTC [grpc] Printf -> DEBU 041 pickfirstBalancer: HandleSubConnStateChange: 0xc4205c5ae0, CONNECTING
2018-07-24 08:10:18.234 UTC [grpc] Printf -> DEBU 042 pickfirstBalancer: HandleSubConnStateChange: 0xc4205c5ae0, READY
2018-07-24 08:10:18.235 UTC [msp] GetDefaultSigningIdentity -> DEBU 043 Obtaining default signing identity
2018-07-24 08:10:18.235 UTC [chaincodeCmd] getChaincodeSpec -> DEBU 044 java chaincode disabled
2018-07-24 08:10:18.235 UTC [msp/identity] Sign -> DEBU 045 Sign: plaintext: 0AA6070A6A08031A0B08EABDDBDA0510...706C65321A0A0A0571756572790A0141 
2018-07-24 08:10:18.235 UTC [msp/identity] Sign -> DEBU 046 Sign: digest: 616DDDAA2E73BFF060EADC26C675A82CA83C21364A5A0351BAABD1B06E21F019 
100


# 可以看到 返回 100




# 查询 B 账户里的余额

peer chaincode query -C mychannel -n example2 -c '{"Args":["query","B"]}'


# 输出如下:

2018-07-24 08:11:18.858 UTC [msp] setupSigningIdentity -> DEBU 035 Signing identity expires at 2028-07-21 06:09:52 +0000 UTC
2018-07-24 08:11:18.858 UTC [msp] Validate -> DEBU 036 MSP Org1MSP validating identity
2018-07-24 08:11:18.859 UTC [grpc] Printf -> DEBU 037 parsed scheme: ""
2018-07-24 08:11:18.859 UTC [grpc] Printf -> DEBU 038 scheme "" not registered, fallback to default scheme
2018-07-24 08:11:18.859 UTC [grpc] Printf -> DEBU 039 ccResolverWrapper: sending new addresses to cc: [{peer0.org1.jicki.me:7051 0  <nil>}]
2018-07-24 08:11:18.860 UTC [grpc] Printf -> DEBU 03a ClientConn switching balancer to "pick_first"
2018-07-24 08:11:18.860 UTC [grpc] Printf -> DEBU 03b pickfirstBalancer: HandleSubConnStateChange: 0xc4201baea0, CONNECTING
2018-07-24 08:11:18.861 UTC [grpc] Printf -> DEBU 03c pickfirstBalancer: HandleSubConnStateChange: 0xc4201baea0, READY
2018-07-24 08:11:18.862 UTC [grpc] Printf -> DEBU 03d parsed scheme: ""
2018-07-24 08:11:18.862 UTC [grpc] Printf -> DEBU 03e scheme "" not registered, fallback to default scheme
2018-07-24 08:11:18.862 UTC [grpc] Printf -> DEBU 03f ccResolverWrapper: sending new addresses to cc: [{peer0.org1.jicki.me:7051 0  <nil>}]
2018-07-24 08:11:18.862 UTC [grpc] Printf -> DEBU 040 ClientConn switching balancer to "pick_first"
2018-07-24 08:11:18.862 UTC [grpc] Printf -> DEBU 041 pickfirstBalancer: HandleSubConnStateChange: 0xc42037e0e0, CONNECTING
2018-07-24 08:11:18.863 UTC [grpc] Printf -> DEBU 042 pickfirstBalancer: HandleSubConnStateChange: 0xc42037e0e0, READY
2018-07-24 08:11:18.864 UTC [msp] GetDefaultSigningIdentity -> DEBU 043 Obtaining default signing identity
2018-07-24 08:11:18.864 UTC [chaincodeCmd] getChaincodeSpec -> DEBU 044 java chaincode disabled
2018-07-24 08:11:18.864 UTC [msp/identity] Sign -> DEBU 045 Sign: plaintext: 0AA7070A6B08031A0C08A6BEDBDA0510...706C65321A0A0A0571756572790A0142 
2018-07-24 08:11:18.864 UTC [msp/identity] Sign -> DEBU 046 Sign: digest: 6B8C88BB7715C9FE0AA6BC4CE3E2509EEA32F0A62EDB9AE8CB270E841FD7F22F 
50


# 可以看到 返回 50

```


```
# invoke 转账方法


# 从A账户 转账 20 个币 到 B 账户


peer chaincode invoke -C mychannel -n example2 -c '{"Args":["invoke", "A", "B", "20"]}'


# 输出如下:

2018-07-24 08:13:59.922 UTC [grpc] Printf -> DEBU 0bf parsed scheme: ""
2018-07-24 08:13:59.922 UTC [grpc] Printf -> DEBU 0c0 scheme "" not registered, fallback to default scheme
2018-07-24 08:13:59.922 UTC [grpc] Printf -> DEBU 0c1 ccResolverWrapper: sending new addresses to cc: [{orderer0.jicki.me:7050 0  <nil>}]
2018-07-24 08:13:59.922 UTC [grpc] Printf -> DEBU 0c2 ClientConn switching balancer to "pick_first"
2018-07-24 08:13:59.922 UTC [grpc] Printf -> DEBU 0c3 pickfirstBalancer: HandleSubConnStateChange: 0xc4201a8a30, CONNECTING
2018-07-24 08:13:59.923 UTC [grpc] Printf -> DEBU 0c4 pickfirstBalancer: HandleSubConnStateChange: 0xc4201a8a30, READY
2018-07-24 08:13:59.923 UTC [chaincodeCmd] getChaincodeSpec -> DEBU 0c5 java chaincode disabled
2018-07-24 08:13:59.923 UTC [msp/identity] Sign -> DEBU 0c6 Sign: plaintext: 0AA7070A6B08031A0C08C7BFDBDA0510...696E766F6B650A01410A01420A023230 
2018-07-24 08:13:59.923 UTC [msp/identity] Sign -> DEBU 0c7 Sign: digest: F19E299C5EA1A7E8BAFD2577CF2A6F1706B601D26113F3D9DA58D3091115AC22 
2018-07-24 08:13:59.941 UTC [msp/identity] Sign -> DEBU 0c8 Sign: plaintext: 0AA7070A6B08031A0C08C7BFDBDA0510...9C352B250413082E544EA398B29A4403 
2018-07-24 08:13:59.941 UTC [msp/identity] Sign -> DEBU 0c9 Sign: digest: B61BAFEF000043B77CD940650767E329375BB21BDB273C4A64A0605162054810 
2018-07-24 08:13:59.949 UTC [chaincodeCmd] chaincodeInvokeOrQuery -> DEBU 0ca ESCC invoke result: version:1 response:<status:200 > payload:"\n (L\245\376\241l\201<7\002$\202|\2201\251\2449h\214`\257\340c\320[\302\221\242\322\234\247\022d\nL\0220\n\010example2\022$\n\007\n\001A\022\002\010\003\n\007\n\001B\022\002\010\003\032\007\n\001A\032\00280\032\007\n\001B\032\00270\022\030\n\004lscc\022\020\n\016\n\010example2\022\002\010\003\032\003\010\310\001\"\017\022\010example2\032\0031.3" endorsement:<endorser:"\n\007Org1MSP\022\216\006-----BEGIN CERTIFICATE-----\nMIICFDCCAbqgAwIBAgIRAKypz4bFhLrqVGurT7/SfNswCgYIKoZIzj0EAwIwZzEL\nMAkGA1UEBhMCQ04xEjAQBgNVBAgTCUd1YW5nRG9uZzERMA8GA1UEBxMIU2hlblpo\nZW4xFjAUBgNVBAoTDW9yZzEuamlja2kubWUxGTAXBgNVBAMTEGNhLm9yZzEuamlj\na2kubWUwHhcNMTgwNzI0MDYwOTUyWhcNMjgwNzIxMDYwOTUyWjBhMQswCQYDVQQG\nEwJDTjESMBAGA1UECBMJR3VhbmdEb25nMREwDwYDVQQHEwhTaGVuWmhlbjENMAsG\nA1UECxMEcGVlcjEcMBoGA1UEAxMTcGVlcjAub3JnMS5qaWNraS5tZTBZMBMGByqG\nSM49AgEGCCqGSM49AwEHA0IABGk1RT3xLER/rnCemp5w83SHRv0fn84EcMnw51xz\nK8y3iVLkyDeHrj++/Dav1N4e6ZUiDCQMBA3kD7Hc50UvfhWjTTBLMA4GA1UdDwEB\n/wQEAwIHgDAMBgNVHRMBAf8EAjAAMCsGA1UdIwQkMCKAIJhnHqfELM3rczJ9fGf3\nWeSHSX8TNJuXoTU5L2YM9nR/MAoGCCqGSM49BAMCA0gAMEUCIQDPPkcazwaBFWAs\nv68iBBgFDg6/PwjCqBRU2SxFVUVw7QIgXa34u0TkYGE3l87AGbvyFauq5YwNsV9L\n04rEtu4NQEU=\n-----END CERTIFICATE-----\n" signature:"0D\002 <\202\266\210\016\254\372{G\235!\303\200\255\361\014i\340\225s4\357\023{\363/\342{\034\005\013-\002 O\034\2336\253\003\261P\317\337{In\245\323\222\2345+%\004\023\010.TN\243\230\262\232D\003" > 
2018-07-24 08:13:59.950 UTC [chaincodeCmd] chaincodeInvokeOrQuery -> INFO 0cb Chaincode invoke successful. result: status:200 



# 可以看到返回 invoke successful. result: status:200 成功


# 这里再查询 A 与 B 的账户

# A 账户余额 

peer chaincode query -C mychannel -n example2 -c '{"Args":["query","A"]}'  


# 输出如下:

2018-07-24 08:37:18.866 UTC [msp] setupSigningIdentity -> DEBU 035 Signing identity expires at 2028-07-21 06:09:52 +0000 UTC
2018-07-24 08:37:18.867 UTC [msp] Validate -> DEBU 036 MSP Org1MSP validating identity
2018-07-24 08:37:18.868 UTC [grpc] Printf -> DEBU 037 parsed scheme: ""
2018-07-24 08:37:18.868 UTC [grpc] Printf -> DEBU 038 scheme "" not registered, fallback to default scheme
2018-07-24 08:37:18.868 UTC [grpc] Printf -> DEBU 039 ccResolverWrapper: sending new addresses to cc: [{peer0.org1.jicki.me:7051 0  <nil>}]
2018-07-24 08:37:18.868 UTC [grpc] Printf -> DEBU 03a ClientConn switching balancer to "pick_first"
2018-07-24 08:37:18.868 UTC [grpc] Printf -> DEBU 03b pickfirstBalancer: HandleSubConnStateChange: 0xc4200b2d50, CONNECTING
2018-07-24 08:37:18.870 UTC [grpc] Printf -> DEBU 03c pickfirstBalancer: HandleSubConnStateChange: 0xc4200b2d50, READY
2018-07-24 08:37:18.870 UTC [grpc] Printf -> DEBU 03d parsed scheme: ""
2018-07-24 08:37:18.871 UTC [grpc] Printf -> DEBU 03e scheme "" not registered, fallback to default scheme
2018-07-24 08:37:18.871 UTC [grpc] Printf -> DEBU 03f ccResolverWrapper: sending new addresses to cc: [{peer0.org1.jicki.me:7051 0  <nil>}]
2018-07-24 08:37:18.871 UTC [grpc] Printf -> DEBU 040 ClientConn switching balancer to "pick_first"
2018-07-24 08:37:18.871 UTC [grpc] Printf -> DEBU 041 pickfirstBalancer: HandleSubConnStateChange: 0xc4203cd4d0, CONNECTING
2018-07-24 08:37:18.872 UTC [grpc] Printf -> DEBU 042 pickfirstBalancer: HandleSubConnStateChange: 0xc4203cd4d0, READY
2018-07-24 08:37:18.872 UTC [msp] GetDefaultSigningIdentity -> DEBU 043 Obtaining default signing identity
2018-07-24 08:37:18.872 UTC [chaincodeCmd] getChaincodeSpec -> DEBU 044 java chaincode disabled
2018-07-24 08:37:18.873 UTC [msp/identity] Sign -> DEBU 045 Sign: plaintext: 0AA7070A6B08031A0C08BECADBDA0510...706C65321A0A0A0571756572790A0141 
2018-07-24 08:37:18.873 UTC [msp/identity] Sign -> DEBU 046 Sign: digest: F6129F8CE11959DFD14315C9FC82C79E43B1A1BB7DE526582B1A6EFDFAAA7F64 
80




# B 账户余额 

peer chaincode query -C mychannel -n example2 -c '{"Args":["query","B"]}'   



# 输出如下:

2018-07-24 08:37:40.993 UTC [msp] setupSigningIdentity -> DEBU 035 Signing identity expires at 2028-07-21 06:09:52 +0000 UTC
2018-07-24 08:37:40.993 UTC [msp] Validate -> DEBU 036 MSP Org1MSP validating identity
2018-07-24 08:37:40.994 UTC [grpc] Printf -> DEBU 037 parsed scheme: ""
2018-07-24 08:37:40.994 UTC [grpc] Printf -> DEBU 038 scheme "" not registered, fallback to default scheme
2018-07-24 08:37:40.994 UTC [grpc] Printf -> DEBU 039 ccResolverWrapper: sending new addresses to cc: [{peer0.org1.jicki.me:7051 0  <nil>}]
2018-07-24 08:37:40.994 UTC [grpc] Printf -> DEBU 03a ClientConn switching balancer to "pick_first"
2018-07-24 08:37:40.994 UTC [grpc] Printf -> DEBU 03b pickfirstBalancer: HandleSubConnStateChange: 0xc42034cfa0, CONNECTING
2018-07-24 08:37:40.996 UTC [grpc] Printf -> DEBU 03c pickfirstBalancer: HandleSubConnStateChange: 0xc42034cfa0, READY
2018-07-24 08:37:40.996 UTC [grpc] Printf -> DEBU 03d parsed scheme: ""
2018-07-24 08:37:40.997 UTC [grpc] Printf -> DEBU 03e scheme "" not registered, fallback to default scheme
2018-07-24 08:37:40.997 UTC [grpc] Printf -> DEBU 03f ccResolverWrapper: sending new addresses to cc: [{peer0.org1.jicki.me:7051 0  <nil>}]
2018-07-24 08:37:40.997 UTC [grpc] Printf -> DEBU 040 ClientConn switching balancer to "pick_first"
2018-07-24 08:37:40.997 UTC [grpc] Printf -> DEBU 041 pickfirstBalancer: HandleSubConnStateChange: 0xc4205eb4d0, CONNECTING
2018-07-24 08:37:40.998 UTC [grpc] Printf -> DEBU 042 pickfirstBalancer: HandleSubConnStateChange: 0xc4205eb4d0, READY
2018-07-24 08:37:40.998 UTC [msp] GetDefaultSigningIdentity -> DEBU 043 Obtaining default signing identity
2018-07-24 08:37:40.999 UTC [chaincodeCmd] getChaincodeSpec -> DEBU 044 java chaincode disabled
2018-07-24 08:37:40.999 UTC [msp/identity] Sign -> DEBU 045 Sign: plaintext: 0AA7070A6B08031A0C08D4CADBDA0510...706C65321A0A0A0571756572790A0142 
2018-07-24 08:37:40.999 UTC [msp/identity] Sign -> DEBU 046 Sign: digest: AEEED3E4DEF160EE0B2C2DB7D829405E1D2DD659F82D1275BECFE1F30FA7F449 
70

```

```
# 查看 生成的容器

[root@localhost jicki]# docker ps -a
CONTAINER ID        IMAGE                                                                                                     COMMAND                  CREATED             STATUS                      PORTS                                                                                                                           NAMES
e34441de67e9        jicki-peer0.org1.jicki.me-example2-1.0-136869e4973254fe503787a319d571949b2c252ce271019b8e1040e65d454c47   "chaincode -peer.a..."   About an hour ago   Up About an hour                                                                                                                                            jicki-peer0.org1.jicki.me-example2-1.0

```




## Hyperledger Fabric 操作命令


### peer 命令


```
peer chaincode          # 对链进行操作
peer channel            # channel相关操作
peer logging            # 设置日志级别
peer node               # 启动、管理节点
peer version            # 查看版本信息

```

> upgrade 更新合约 更新合约相当于将合约重新实例化，并带有一个新的版本号。
>
> 更新合约之前，需要在所有的 peer节点 上安装(install)最新的合约，并使用新的版本号。

```
# 更新合约

# 首先安装(install)新的合约, 以本文为例, chaincode_example02, 初次安装版本号为 1.0 

peer chaincode install -n example2 -p github.com/hyperledger/fabric/jicki/chaincode/go/chaincode_example02 -v 1.1


# 更新版本为 1.1 的合约
peer chaincode upgrade -o orderer0.jicki.me:7050 -C mychannel -n example2 -c '{"Args":["init","A","100","B","50"]}' -P "OR ('Org1MSP.member','Org2MSP.member')" -v 1.1


# 旧版本的合约, 目前，fabric不支持合约的启动与暂停。要暂停或删除合约，只能到peer上手动删除容器。

```






```
# 查看 已经创建的 通道 (channel)

peer channel  list


# 查看通道(channel) 的状态 -c(小写) 加 通道名称

peer channel getinfo -c mychannel


# 查看已经 安装的 智能合约(chincode)

peer chaincode  list --installed


# 查看已经 实例化的 智能合约(chincode) 需要使用 -C(大写) 加通道名称

peer chaincode -C mychannel list --instantiated

```

