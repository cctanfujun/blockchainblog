---
title: zeppelinos 初探
category: ETH
tags:
  - ETH
  - zeppelin
translate_title: preliminary-study-of-zeppelinos
date: 2018-09-11 14:56:41
---

之前在开发以太坊合约的时候一直会使用openzepplin的库,今天看见推出了ZeeplinOS这个项目,研究了一下。

> https://zeppelinos.org/
> ZeppelinOS 是专为智能合约设计的操作系统，它不仅提供链上可升级的程序编码库，而且还提供保持程序持续升级、修补的奖励机制。

个人理解本质上是对 Truffle 进行了整合,然后把很多代码库做成了库的模式方便开发者使用，在过程中还引入了激励机制让开发者一起维护。

如果是有 Truffle 使用经验的人的话应该是比较容易上手的。

## 创建项目

首先安装 `zos`：

```bash
npm install --global zos
```

然后初始化项目：

```bash
mkdir my-app && cd my-app
npm init
zos init my-app
```
这里会生成 `zos.json` 这个文件中会记录应用的信息。

安装 `zos-lib` zos库:

```bash
npm install zos-lib
```

进行合约编写 MyContract.sol 放在 contracts/ 目录下面:

```bash

pragma solidity ^0.4.21;
import "zos-lib/contracts/migrations/Migratable.sol";

contract MyContract is Migratable {
  uint256 public x;

  function initialize(uint256 _x) isInitializer("MyContract", "0") public {
    x = _x;
  }
}

```

这里的合约规则和 solidity 原生的规则略有差别，使用 intialize 作为构造函数，zos 的要求。

进行合约编译:

```
zos add MyContract
```

接下来启动区块链网络，这样我没有按照官网来启动 `ganache-cli` 而是打开 `Ganache` 然后修改 `truffle-config.js` 中的端口号

执行

```bash

zos push --network local

```

推送你的应用到区块链网络。

我理解push命令是推送一个应用到区块链网络，zeppelin应该是认为应用由多个合约组成，合约是可以升级的。

创建合约的实例

```bash
zos create MyContract --init initialize --args 42 --network local

```
## 升级合约

如果你发现合约有问题要升级合约,比如官方例子给合约增加了一个函数:

```bash
import "zos-lib/contracts/migrations/Migratable.sol";

contract MyContract is Migratable {
  uint256 public x;

  function initialize(uint256 _x) isInitializer("MyContract", "0") public {
    x = _x;
  }

  function increment() public {
    x += 1;  
  }
}
```

把新的代码推送到网络：
```bash
zos push --network local

```

执行更新命令：
```bash
zos update MyContract --network local

```

## 使用标准库

官方提供了标准库方便开发者使用 

```bash
zos link openzeppelin-zos
```
使用这些库可以简化智能合约的开发。

## 原理推测

我还没有详细研究 zeepelin 辅助实现合约升级的原理，初步猜想应该是通过代理模式，首先部署一个合约,然后通过这个合约来调用其他合约的方法，变更的时候就是调用地址的转向。估计实际的实现应该考虑了更多场景，更多的方案。

关于合约升级，推荐大家阅读2篇文章。

[智能合约升级模式介绍 — 入门篇](https://segmentfault.com/a/1190000015732881)
[深度剖析智能合约升级——inherited storage](https://segmentfault.com/a/1190000015732950)

Zeppelin 的具体实现之后再研究。个人觉得能快速帮助开发者实现合约编写是很好的,如果能有 Truffle box 那种直接生成前端框架的命令就更好了,现在这种情况，,对于编写应用的开发者还做不到开箱即用。