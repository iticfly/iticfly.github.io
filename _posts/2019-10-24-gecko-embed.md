---
layout: post
title: 支付系统账户体系可行性分析
categories: Gecko
description: 账户体系设计与分析
keywords: 资金
---

# 支付系统账户体系可行性分析

## 前言

**百度百科**中是这么定义支付的： 支付又称付出、付给，多指[付款](https://baike.baidu.com/item/付款)，是发生在购买者和[销售](https://baike.baidu.com/item/销售/239410)者之间的金融交换，是社会经济活动所引起的货币[债权](https://baike.baidu.com/item/债权/6754177)转移的过程。支付包括交易、[清算](https://baike.baidu.com/item/清算/7171204)和[结算](https://baike.baidu.com/item/结算/1459851)。 

在我看来，一条支付指令中应包含**交易方**、**对手方**、以及**订单信息**(订单编码、交易金额、交收方式、货物详情)。如何有效维护交易双方的支付信息，是支付系统需要考虑的核心设计之一。

## 回顾

在现有资金清算系统中，主要由付款单和付款单流水两部分组成。其中，交易双方维护信息如下:

| 字段名              | 字段描述              |
| ------------------- | --------------------- |
| payerSysUserId      | 交易方平台userId      |
| payerSysUserAccount | 交易方平台userAccount |
| payerSysUserName    | 交易方平台userName    |
| payerBankAccName    | 交易方银行账户名      |
| payerBankCardNum    | 交易方银行卡号        |
| payerBankName       | 交易方银行名称        |
| payerBankNo         | 交易方银行编码        |
| payerExBankAccName  | 交易方虚拟账户名      |
| payerExBankAccNo    | 交易方虚拟账户号      |

在平台付款单中，需要耦合交易双方的支付信息，以上支付信息，实际上是用户在不同渠道下开通的虚拟账户或平台内部账号信息。同上，在账单模块、以及业务订单模块，也都会维护多种账户耦合信息，这都是费心费力的事情。那么为何不在资金平台内维护用户的账户信息，在清算系统、结算系统、账单系统以及用户对账系统复用账户。



## 账户关联性

### 支付指令与支付账户的关联性

#### 渠道支付指令

![]( https://magpie-pic.oss-cn-shenzhen.aliyuncs.com/f9a5758eb1ceaf02f83a9d252cdb4e06.png )

如上所示，账户是支付指令的基础，将账户信息维护在平台内部统一管理，可以保证交易订单有帐可查、有帐可对、有帐可控。

### 用户与支付账户的关联性

#### 收银台账户管理

![]( https://magpie-pic.oss-cn-shenzhen.aliyuncs.com/c8860de81fbf23cc0ec079dda468e8bf.jpg )

如上所示，支付方式中会展示出满足要求的所有账户列表，这些账户都维护在平台账户模块，可以在账户的控制属性(是否允许支付、是否允许代付、是否冻结)下进行权限分配等信息。

### 对账与支付账户的关联性

#### 货款账户内部对账

随着货款线上化出入金的推进，账户对账是资金安全的一道重要保障。账户对账的是在账户交易流水上建立的双边对账，主要对账户的日终余额和日交易进行对账处理，这些都是建立在账户体系下实现的。

![]( https://magpie-pic.oss-cn-shenzhen.aliyuncs.com/fda66943aa3a2615bde78e0d87b5fd1a.jpg )

### 账单与支付账户的关联性

#### 交易明细来源追踪

目前账单信息直接与平台用户信息、付款账号关联，关联数据较单一，且存在耦合字段较多，不利于业务和代码维护。让账单信息直接与账户关联，通过多表关联查询，可以有效解决这一问题。

![]( https://magpie-pic.oss-cn-shenzhen.aliyuncs.com/e1b5c9af9682a396cfe934300e11ad5c.jpg )

## 小结

这是一个没有小结的小结。