---
title: "迁移博客到IPFS"
date: 2017-09-24T21:18:15+08:00
draft: false
type: post
author: htfy96
categories:
  - 代码
tags:
  - ipfs
  - 运维
  - 区块链
---

[IPFS](https://ipfs.io/)是最近非常热门的一个分布式网络系统，其主要支持以下功能：

- `ipfs add`，将本地的一个文件（夹）上传到一个去中心化的web中，返回一个和文件夹内容有关的Hash，记为`fsHash`。之后用户既可以通过`ipfs`客户端根据`fsHash`来获取内容；也可以通过官方提供的https gateway，用`https://ipfs.io/ipfs/QmdPtC3T7Kcu9iJg6hYzLBWR5XCDcYMY7HV685E3kH3EcS/2015/09/15/hosting-a-website-on-ipfs/`之类的链接直接访问获取
- `ipfs name publish`。每次上传新的文件夹后，得到的`Hash`均不同，这给访问带来了很大的不便。为了解决这个问题，我们在持有私钥的情况下可以把`fsHash`publish到一个和公钥相关的地址`keyHash`，这样，用户就可以用`https://ipfs.io/ipns/keyHash`来访问不同的内容，而不用担心内容版本的变化
- `ipfs dns`。因为`keyHash`非常长，`ipfs`提供了一种能够通过`https://ipfs.io/ipns/intmainreturn0.com`方式访问`https://ipfs.io/ipns/keyHash`的方法：配置`intmainreturn0.com`的DNS，添加`TXT`项，内容是`dnslink=/ipns/keyHash`即可

希望达到的效果是：

- push Hugo source到github后自动编译成HTML
- 自动上传到IPFS，自动`name publish`
- 用户可以用 https://intmainreturn0.com 访问这个博客
- 不依赖于`https://ipfs.io`的可用性

## 自动编译 & 自动上传/publish
这一块都依赖于Travis-CI的功能。但这也要求我们把私钥交给第三方……

首先我们需要生成一对公钥私钥：
```sh
ipfs key gen -t ed25519 blog
# 这里会输出keyHash
base64 ~/.ipfs/keystore/blog #把输出的私钥复制成一行
```

然后运行：
```sh
travis encrypt SECKEY_BASE64=私钥
```

输出：
```
secure: "BLAHBLAH"
```

之后我们就可以编写`.travis.yml`了：
```yaml
sudo: required
dist: trusty

services:
    - docker

env:
  global:
    - secure: "BLAHBLAH"
branches:
  only:
    - master

before_install:
  - git submodule update --init --recursive
  - mkdir -p ipfs/staging ipfs/data
  - docker run -d --name ipfs_host -v $(pwd)/ipfs/staging:/export -v $(pwd)/ipfs/data:/data/ipfs -p 18080:8080 -p 14001:4001 -p 15001:5001 ipfs/go-ipfs:latest # 启动daemon
  - sleep 3 # 等它创建好FS
  - ls -l ipfs/data
  - echo $UID
  - sudo chmod -R 777 ipfs/data/keystore
  - echo $SECKEY_BASE64 | base64 -d > ipfs/data/keystore/blog

install:
  - rm -rf public || exit 0
  - wget https://github.com/gohugoio/hugo/releases/download/v0.27.1/hugo_0.27.1_Linux-64bit.tar.gz && tar xzf hugo_0.27.1_Linux-64bit.tar.gz
script:
  - ./hugo -v
after_success:
  - cp -r ./public ipfs/staging
  - export DATA_HASH=$(docker exec ipfs_host ipfs add -r /export/public/ | tail -1 | awk '{print $2}') # 提取出fsHash
  - echo "DATA_HASH = $DATA_HASH"
  - docker exec ipfs_host ipfs name publish --key=blog $DATA_HASH # 用blog key上传
  - sleep 5 # Wait for a while
```

这样，编译完成的东西就会自动上传到IPFS中并注册ipns了。这个时候，应该可以通过`https://ipfs.io/ipns/keyHash`来访问了

## 在`intmainreturn0.com`反代IPFS访问
大多数人很难记住长长的`https://ipfs.io/ipns/fooooooooobbbbbbaaaaaaarrrrrrr`，而且`ipfs.io`随时有可能被墙，因此我们需要反代一下其的访问。

反代需要我们本地跑一个`ipfs`和一个`caddy`，目录结构：
```
.
|-- caddy
|   `-- Caddyfile
|-- docker-compose.yml
`-- ipfs
    |-- data # 空文件夹
    `-- staging # 空文件夹
```

`docker-compose.yml`:
```
version: '2'

services:

  ipfs:
    image: jbenet/go-ipfs
    restart: always
    volumes:
    - /home/htfy96/ipfs-stack/ipfs/staging:/export
    - /home/htfy96/ipfs-stack/ipfs/data:/data/ipfs

  caddy:
    image: abiosoft/caddy
    restart: always
    volumes: 
    - /home/htfy96/ipfs-stack/caddy/Caddyfile:/etc/Caddyfile
    depends_on:
    - ipfs
    ports:
    - 443:443
```

`Caddyfile`:
```
gzip
log stdout
rewrite {
	r (.*)
	to /ipns/keyHash/{1}
}

proxy /ipns/keyHash ipfs:8080
```

最后直接`docker-compose up -d`就可以了，`Caddy`会自动解决TLS证书获取等问题。

## 总结
目前来看，去中心化、效率与便利似乎三者中最多只能满足两者。要想让一般人也能用浏览器访问就必须要一个中心化的gateway，要想让作者随时随地上传就需要一个中心化的auto build/deploy系统。
总体来看，转到了IPFS之后，控制权从最先的完全VPS owned，变成了Github / TravisCI / IPFS net / VPS 四方面共同拥有。好处是只要IPFS net不挂，历史上已经上传的东西都能找到；坏处是Github/Travis CI可以篡改内容，或者VPS挂了就不能反代访问。有利有弊需要权衡。