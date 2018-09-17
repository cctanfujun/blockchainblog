---
title: 晓晨EOS开发教程1 -- 搭建开发环境(Docker)
tags:
  - EOS
translate_title: xiaochen-eos-development-tutorial-1-building-a-environment-docker
date: 2018-09-17 15:53:24
---

## 介绍

接下来计划做一个 EOS 应用开发的教程,区块链上的应用开发和我们正常的App开发在开发模式上略有区别,传统的开发模式一般是前端人员开发App，服务端人员开发服务端暴露接口，前端和和服务端进行交互，而区块链应用的开发，一般是前端人员开发app,通过区块链sdk(web3.js、eosjs等)调用智能合约开发者开发的智能合约和区块链交互。如图:

![架构图](https://ws2.sinaimg.cn/large/006tNbRwgy1fvco6hcgrmj31kw16odua.jpg)

## 搭建环境

eos 为了避免环境不一致以及各种依赖问题，提供了使用 docker 镜像方便我们使用。

首先需要安装 docker 各个系统的童鞋请自行下载安装。

eos docker 镜像地址: https://hub.docker.com/r/eosio/eos-dev/tags/

eosio 的库下有几个镜像:
* eos 主网使用，不包含开发依赖
* eos-dev 包含开发环境，供开发使用
* builder 完整的编译环境
* ci 暂不清楚

我们要下载开发环境 eos-dev 执行:

```bash
docker pull eosio/eos-dev:v1.2.5

docker tag eosio/eos-dev:v1.2.5 eosio/eos-dev:latest

```

下载完成镜像,我们接下来准备使用 docker-compose 来启动节点,说一下什么是 docker-compose ,假设我们要启动一个 web 服务,我们可能要启动多个service,甚至4个 service 的启动顺序和参数也要控制，如果我们都使用 `docker run` 就太不方便了，所以把多个服务的参数和关系描述到一个文件中，之后只要执行这个文件即可。docker-compose 在 mac 系统下是随着 docker 一起安装的，如果使用其他系统可能需要额外安装，请自行查询。

我们新建一个 eos 文件夹作为项目文件夹，新建 eos-local-compose.yml 文件,内容如下：

**eos-local-compose.yml**

```
version: "3"

services:
  nodeosd:
    image: eosio/eos-dev:latest
    command: /opt/eosio/bin/nodeosd.sh --data-dir /opt/eosio/bin/data-dir -e --delete-all-blocks --http-validate-host=false #--genesis-json /opt/eosio/bin/data-dir/genesis.json #--contracts-console #-e — Enable block production, even if the chain is stale
    hostname: nodeosd
    ports:
      - 8888:8888
      - 9876:9876
    expose:
      - "8888"
    volumes:
      - local-nodeos-data-volume:/opt/eosio/bin/data-dir
      #- ./config-local-v1.2.3.ini:/opt/eosio/bin/data-dir/config.ini

  keosd:
    image: eosio/eos-dev:latest
    command: /opt/eosio/bin/keosd --wallet-dir /opt/eosio/bin/data-dir --http-server-address=127.0.0.1:8900 --http-validate-host=false
    hostname: keosd
    links:
      - nodeosd
    volumes:
      - local-keosd-data-volume:/opt/eosio/bin/data-dir

volumes:
 local-nodeos-data-volume:
   external: true
 local-keosd-data-volume:
   external: true

```

然后创建2个volume供2个服务nodeos、keosd挂载使用：

```

docker volume create --name=local-nodeos-data-volume

docker volume create --name=local-keosd-data-volume

```

接下来启动节点：

```
docker-compose -f eos-local-compose.yml up -d
```
-f 指定文件名
up 启动容器
-d 放到后台

可以看到命令行提示创建服务成功。

执行查看nodeos的log,如果正在出块，表示测试节点启动成功:

```
docker logs -f eos_nodeosd_1

```

## 配置别名方便使用

因为我们执行的命令在 docker 的容器中,如果我们要执行一些 eos 的命令就需要进入 docker 中,我们设置一个别名方便我们调用。

执行:

```
alias cleos='docker-compose -f eos-local-compose.yml exec keosd /opt/eosio/bin/cleos -u http://nodeosd:8888 --wallet-url http://localhost:8900'
```

注意：调用cleos时候需要在 `eos-local-compose.yml` 文件夹下。

执行 `cleos get info`

会打印出当前信息，如图：

![info]](https://ws2.sinaimg.cn/large/006tNbRwgy1fvco4oiv7rj30wg0i2my2.jpg)



## 常用命令

查看正在运行的容器
```
docker ps
```

停止某个正在运行的容器
```
docker stop name
```

