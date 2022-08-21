# 平台详细设计

[TOC]

## 概述

该平台是一款基于智能合约的完全去中心化的支付通道平台，该平台利用存入智能合约的交易担保金建立买卖双方的共识关系，致力于打造安全链上支付通道，利用人工智能审核支付流水，合约开源

本平台可以交易一切可收货的商品，交易进度由买卖双方推动

## 问题解决

目前线上P2P交易的问题点主要有三个

* 共识问题，在大型的交易所进行P2P交易都是由交易所进行担保，交易所不安全的情况下由第三人进行担保
* 审核问题，目前交易流水的审核都是靠人工审核，作假作弊的可能性大
* 仲裁问题，出现纠纷目前还没有很好的仲裁机制

### 合约担保

通过智能合约进行共识约束，就不需要第三方担保，合约开源，具体逻辑如下：

卖方建立支付通道时向合约存入担保资产V，如果是交易虚拟货币还要向合约存入商品价格资产K，如果是真是商品，那卖方收款以后必须要有发货动作

买方链下支付卖方等价商品价格资产K1，往合约存入担保资产V

卖方发货，买方收货都需要更改

买卖结束以后卖方或者资产为K1+V，买方获得资产为K+V，实现买卖双方K1及K的买卖交易

具体合约实现看**合约分析**

### 智能审核

完全可以通过人工智能审核交易的流水

目前深度学习的银行卡流水训练集的测试通过在87.3%

多次审核不通过的账户考虑加进黑名单

### 仲裁

通过往合约存入担保金，让买卖双方完成交易，取回担保金已经成为买卖双反的共识了

但是潜在交易未知性依旧需要一种仲裁方式，还在设计中

 

## 平台收益

目前平台收益点会考虑两个方面，vip制度和手续费

### VIP制度

用户需要支付一笔VIP费用，进行账户绑定，在合约白名单上的地址才能进行交易

### 手续费抽取

目前计划收取每个交易通道担保金的3% 



## 版本计划

目前平台由三个版本技术

### alpha版

内测版本，详细测试计划会在发版前制定

实现法币USDT交易

### 1.0版

生产版本

* 实现各种主流币种对应交易，在交易前商定好币种已经价格

* 嵌入人工智能审核机制

### 2.0版

生产版本

*  嵌入价格预言机，在交易前确定价格即可



## 功能介绍

该平台的主要是三个模块，买家、卖家和交易记录

### 卖家

![1661062168581](C:\Users\13080\AppData\Roaming\Typora\typora-user-images\1661062168581.png)

### 买家

![1661062185142](C:\Users\13080\AppData\Roaming\Typora\typora-user-images\1661062185142.png)

### 交易记录

![1661062392398](C:\Users\13080\AppData\Roaming\Typora\typora-user-images\1661062392398.png)



## 合约分析

展示部分合约重点接口

* 定义商品状态

```solidity
 enum State { Created, Locked, Funded,Release, Inactive }
// 创建、已支付、已收款、以收货、失效 
```

结合商品的状态已经操作的限制可以一步一步推动商品的交易各个流程

* 权限修饰

```solidity
//条件修饰 判断出价是否正确
modifier condition(bool condition_) {
        require(condition_);
        _;
    }
modifier onlyBuyer() {
        if (msg.sender != buyer)
            revert OnlyBuyer();
        _;
    }

modifier onlySeller() {
        if (msg.sender != seller)
            revert OnlySeller();
        _;
    }

modifier inState(State state_) {
        if (state != state_)
            revert InvalidState();
        _;
    }
```

* 卖方创建

```solidity
//创建支付通道接口
function create()externa {}
```

* 卖方撤回

```solidity
//需要买方没有操作的情况下可以撤回
function abort()externa lonlySeller inState(State.Created){}
```

* 买方交易

```solidity
//需要买方没有操作的情况下可以撤回
function confirmPurchase()
		external 
		inState(State.Created) 
		condition(msg.value == (2 * value))
        payable{}
```

* 卖方发货

```solidity
//买方收款以后进行发货
function out()externa lonlySeller inState(State.Locked){}
```

* 买方确认收货

```solidity
// 买方确认收货
function confirmReceived()
        external
        onlyBuyer
        inState(State.Funded){}
```

* 双方取回担保金

```solidity
//买方收货以后，交易结束，双方取回担保金
function back()externa  inState(State.Release){}
```



## 交易案例

下面用买卖双方交易usdt为案例，如果双方买卖实际商品，那卖方必须要有发货动作，买方要有收货动作

![](E:\pigcharid\PP\pp交易流程.png)