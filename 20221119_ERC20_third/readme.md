# ERC20 进阶篇-2


初次发布: 2022.11.25 / 最新修改2022.11.25

关键词:  **openzepplin**  **ERC20  Vote**   **ERC3156**

## 前置阅读

 ERC20初探篇 / ERC20进阶篇

## 本段航行的主要旅途

继续阅读@openzeppelin/contracts/token/ERC20/文件夹内的源码

本节主要关注：

@openzeppelin/contracts/token/ERC20/extensions/ 当中的

ERC20Votes.sol

ERC20VotesComp.sol

ERC20FlashMint.sol



## vote 相关

主要涉及到的代码文件: ERC20Votes.sol  ERC20VotesComp.sol

代码注释当中也提到: 

token的余额并不代表投票的权利，这样实现可以让转账函数的代价比较小，

但是需要用户来委托给自己从而激活检查点、同时追踪到他们的投票权。

基本的数据结构如下:

```solidity
struct Checkpoint {
        uint32 fromBlock;
        uint224 votes;
    }

mapping(address => address) private _delegates;
mapping(address => Checkpoint[]) private _checkpoints;
Checkpoint[] private _totalSupplyCheckpoints;
```

用户可以委托给其他人来进行投票。

delegate / delegateBySig 两种委托的方式，一种直接链上进行，一种是通过签名的方式进行。

用户的投票委托情况和投票权的大小通过检查点的方式进行记录和维护。

不是特别明白为什么需要进行变量类型的转换, 可能是为了节约gas。

 

## 闪电贷 ERC3156

[闪电贷技术详解-Part 1](https://www.aqniu.com/vendor/87319.html)

openzepplin的代码主要实现了Lender端的逻辑。

接下来要看的协议也非常明确了: dYdX, Compound, Curve, Uniswap。



## 关于之前 ERC20 篇的总结

ERC20初探篇  ERC20进阶篇-1 ERC20进阶篇-2 

留下了很多可以探索的内容，这里统一进行记录。

**关于openzeeplin当中可能和ERC相关的内容**

govenance https://docs.openzeppelin.com/contracts/4.x/governance#token

Upgradeablity : transparent && UUPS   https://docs.alchemy.com/docs/how-to-create-a-dao-governance-token

**更多的标准**

ERC223 

ERC777

**更多的项目**

usdt源码分析

usdc源码分析

shibtoken源码分析

DAI源码分析

yearn finance

Uniswap

dYdX

Compound

Curve

ERC4626 可能的重入攻击的分析 

ERC4626 滑点问题的分析 [fei protocol的 ERC4626Router](https://github.com/fei-protocol/ERC4626#erc4626router-and-base)

**安全相关**

钓鱼ERC20合约

## 结束语

一天夜里，准程序员[ddy](https://twitter.com/ddy_mainland)从睡梦中醒来，窗外闪烁着炫目的灯光，0x807船长驾驶着神秘的飞船接他去web3的宇宙航行。在旅途中，ddy会记录下自己的见闻和思考，打开他的日记本，一笔一划地写下**ddy的web3漫游日记**。

<div align="center">
	<img width="500" src="https://user-images.githubusercontent.com/25214732/196032084-6c1d6531-2a80-4672-b620-04b9e4ae3baa.PNG" alt="DDY WEB# TOUR">
</div>
<p align="center">
	<b>DDY WEB3 TOUR</b>
</p>

<p align="center">
  Contributor: <a href="https://twitter.com/ddy_mainland">@ddy_mainland</a>
</p>


