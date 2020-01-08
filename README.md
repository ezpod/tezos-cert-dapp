# 基于Tezos区块链的教育证书DApp

本文将介绍Tezos Dapp的基本架构，以及如何使用Archetype、
Taquito、Tezbridge等语言工具开发一个基于Tezos区块链的
教育去中心化应用，并在教程最后提供源码下载。

> 相关推荐：[基于BigchainDB的毕业证查验系统](http://sc.hubwiz.com/codebag/certchain/)

## 1、证书token

如何利用区块链记录教育中的证书信息已经探讨很多，最近的趋势
是考虑将区块链技术应用到终身学习中。

我们要开发的这个dapp提出了一种与终身学习效果关联的通证/token：
一个学习者在通过职业认证后可以获得token，学习者累积的token可以
作为学习效果的指标，dapp的智能合约则作为跨越机构边界的认证信息
数据库。

为了计算token的数量，dapp的智能合约注册学习者的证书。证书主要
包含以下数据：

- 证书ID
- 学习者ID
- 所属机构ID
- 认证机构ID
- 日期

Tezos账户在DApp中作为用户的标识ID。

学习者所属机构指的是学习者学习或工作的大学或企业，它与认证机构的
区别在于是否可以签发认证证书。当学生从其学习的大学获得毕业证书时，
大学即是所属机构也是认证机构。

当证书被注册后，相应的token被发放给学习者和机构。机构累积的token
可以作为该机构为提升其人员能力所作努力的衡量。

## 2、Dapp的角色划分与交互

学习者、所属机构或认证机构以不同的身份登录进入DApp，Tezos账户作为
DApp的用户标识。

![](tezos-dapp-tutorial/landing.png)

选中一个tezos账户（使用tezbridge）：

![](tezos-dapp-tutorial/select-account.png)

如果还没有注册，则选择注册：

![](tezos-dapp-tutorial/regsiter.png)

学习者可以查看token数量以及所获得的证书：

![](tezos-dapp-tutorial/cert-list.png)

机构也可以查看其获得的token并注册机构中参加学习的人员：

![](tezos-dapp-tutorial/institution.png)

认证机构页面：

![](tezos-dapp-tutorial/certifier.png)

这个DApp中使用的证书数据库是法国Onisep证书库。认证机构
从数据库中选取一个证书：

![](tezos-dapp-tutorial/select-cert.png)

## 3、智能合约实现

智能合约采用Archetype语言开发，我们使用的版本是0.1.12。Archetype
可以转码为其他语言：

- 执行：ligo或scaml
- 验证：why3
- 文档：markdown

合约的存储建模为4种资产：学习者、机构、认证机构、证书。

```
archetype certification_token

asset learner {
  lid : role;
  ltokens : int = 0;
}

asset institution {
  iid : address;
  itokens : int = 0;
  ilearners : learner collection = [];
}

asset certifier {
  ccid : address;
}

asset certification {
  cid : string;
  cdate : date;
  ccer : string;
  clea : pkey of learner;
  cins : pkey of institution;
  ccertifier: pkey of certifier;
}

constant dtkl : int = 1
constant dtki : int = 1
```

合约包含5项：

- register_learner, register_institution 以及 register_certifier来注册角色
- register_learners 用于机构注册其相关的学习者，例如企业注册参与学习的员工
- certify用于认证机构注册证书

certify接收一个整数资产用于注册：

```
action certify(certified : certification collection) {
  require {
    r1: certifier.contains(caller);
  }
  effect {
   /* ... */
  }
}
```

在设计dapp以及相关的智能合约时，有一个基本原则：

>Dapp计算应当在链下完成，智能合约应当仅用于检查输入数据的一致性

智能合约只是检查关联是否与已知的一致：

```
action certify(certified : certification collection) {
  require {
    r1: certifier.contains(caller);
  }
  effect {
    for c in certified do

       if (not institution.get(c.cins).ilearners.contains(c.clea)) then
         fail ("inconsistent institution id");

       /* ... */

    done
  }
}
```

## 4、Tezbridge

目前我们使用Tezbridge进行Tezos账户选择和交易构造。你需要准备3个账户 ——
1个学习者、1个机构、1个认证机构 —— 来使用Dapp。

## 5、Taquito

Taquito是一个功能强大的开发包，可以用来轻松地读取合约存储、构造交易。
实际上它提供了一个关于智能合约的统一的视图，你可以通过单一对象接口
与合约交互。

例如，下面的示例代码提取学习者tezid收获的token数量：

```
import { Tezos } from '@taquito/taquito';

...

{ 
  ...
  Tezos.contract.at('KT19E1fZ5toRNLtSV1uZ4SSvZbJbswyWFuGx')
  .then(function (contract) {
    contract.storage()
    .then(function (storage) {
      const nbTokens = storage.institution_assets[tezid].itokens;
      setNbTokens(nbTokens);
  })
  ...
}

...
```


## 6、小结

本文介绍了一个基于Tezos区块链的教育Dapp的基本结构以及开发过程，
你可以在[这里](https://github.com/ezpod/tezos-cert-dapp)下载源代码，或者查看更多的区块链开发教程：
[以太坊](http://xc.hubwiz.com/course/5b36629bc02e6b6a59171de3?affid=blog7878) |
[比特币](http://xc.hubwiz.com/course/5b9e779ac02e6b6a59171def?affid=blog7878) |
[EOS](http://xc.hubwiz.com/course/5b52c0a2c02e6b6a59171ded?affid=blog7878) |
[Tendermint Core](http://xc.hubwiz.com/course/5bdec63ac02e6b6a59171df3?affid=blog7878) |
[Hyperledger Fabric](http://xc.hubwiz.com/course/5c9b89f54898e59b7b63430a?affid=blog7878) |
[Omni/USDT](http://sc.hubwiz.com/codebag/omni-php-lib/?affid=blog7878) |
[Ripple](http://sc.hubwiz.com/codebag/xrp-php-lib/?affid=blog7878) 


---
原文链接：[Developing a Dapp on Tezos](https://medium.com/@benoit.rognier/developing-a-dapp-on-tezos-35549a1b3ec4)

汇智网翻译整理，转载请标明出处