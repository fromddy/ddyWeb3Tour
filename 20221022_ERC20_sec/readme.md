# ddy的web3漫游日记 ERC20进阶篇1

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

初次发布: 2022.10.22 / 最新修改2022.10.22

关键词: **ERC20**  **openzepplin** **EIP712** **EIP2612** **ERC4626** **DeFi**

## 前置阅读

 ERC20初探篇

## 本段航行的主要旅途

继续阅读@openzeppelin/contracts/token/ERC20/文件夹内的源码

本节主要关注：

@openzeppelin/contracts/token/ERC20/文件夹内的源码阅读，在ERC20初探篇之后

还有extensions/ 当中的

ERC4626.sol，

~~ERC20FlashMint.sol~~

~~ERC20Votes.sol~~

~~ERC20VotesComp.sol~~

draft-ERC20Permit.sol

draft-IERC20Permit.sol

presets/当中的 ERC20PresetFixedSupply.sol，ERC20PresetMinterPauser.sol

utils/当中的TokenTimelock.sol



## openzepplin源码阅读

### utils/TokenTimelock.sol

注意: 这里的utills/TokenTimelock.sol 是@openzeppelin/contracts/token/ERC20/ 当中的，

​         还有一个@openzeppelin/contracts/utills/的文件夹（文章中称为openzepplin的全局utils库)。



实现了一个锁仓的逻辑，是需要有一个IERC20的智能合约持有token, 然后这个TokenTimelock的智能合约再调用IERC20的智能合约，需要IERC20的合约里面有足够的token，以及TokenTimelock的智能合约有操作token的权限才能够实现成功的release。

可能更符合去中心化思路的是直接一个合约，ERC20的钱直接打到这个合约账户，实现锁仓的相关逻辑。

### presets/

里面的两个智能合约都为了支持[wizard](https://wizard.openzeppelin.com/)的功能已经**放弃使用**了。

ERC20PresetFixedSupply.sol 继承了ERC20Burnable, 初始化代币的总量。

ERC20PresetMinterPauser.sol 利用了 openzepplin的access库实现了不用的角色和权限，

miner 可以 mint出更多的token,

pauser 可以控制是否暂停token的transfer。

### EIP2612

涉及到  extensions/draft-ERC20Permit.sol，extensions/draft-IERC20Permit.sol

用户可以通过sign事务的方式来修改一个账户当中ERC20的allowance，

不必发送一笔事物上链，从而节约了gas费。

使用到了**EIP712**，用于结构化hash与签名的，

在metamask里面能让用户更清楚的看到自己到底签名的是什么结构，

在一些dex当中也会用于简化操作步骤。

推荐 [xyyme.eth](https://linktr.ee/xyymeeth) 写的 [EIP-712 使用详解](https://mirror.xyz/xyyme.eth/cJX3zqiiUg2dxB1nmbXbDcQ1DSdajHP5iNgBc6wEZz4) 

```
这里的permit简化操作步骤的内容可以在之后看uniswap源码的时候回顾。
```

### ERC4626

源码位置: extensions/ERC4626.sol

这里有一篇介绍的文章 [ERC-4626‌ 的一小步，DeFi 的一大步](https://www.defidaonews.com/article/6741850)

alchemy的介绍文章 [What is the ERC-4626 token standard?](https://www.alchemy.com/overviews/erc-4626)

这里有另外一版的实现  https://github.com/fubuloubu/ERC4626

一个中文的分析文章, 大家可以看看这一篇 [ERC4626-Tokenized Vault Standard](https://mirror.xyz/0xaaE7a1AD2764626d09a233a9bC06C38b413637cf/MXVXpQgPjQcH4fsMarOobpgE-1q_doxnG7XyoyiJPBk)

info 网站: https://erc4626.info/

Vault: A valut is a multi-sig solution or smart contract that can store and manage assets such as crypto.

保险箱的名称就非常的形象生动。

Popular DeFi protocols that use vaults include Sushiswap, Aave, Balancer and Compound ...

但是不同协议实现不同，用户和开发者都需要熟悉不同的接口，为黑客留下可乘之机。

所以ERC4626就出现啦，

it standardize tokenized vaults to make protocol integration easier and less prone to error.

**直接看代码**

注释开头就直接强调: 代码开头就希望开发者注意: Deposits 和 withdrawals 可能会引发滑点，**用户需要验证自己收到的份额和资产是否符合预期**。

EOA 需要通过 一些检查的wrapper来进行操作, 比如 [fei protocol的 ERC4626Router](https://github.com/fei-protocol/ERC4626#erc4626router-and-base)

实现的了IERC4626 的接口，接口代码文件位置: @openzepplin/interfaces/IERC4626.sol

#### 用到的金融的词汇

yield 收益率， compounding 复利  collateralized 被抵押

deposit 存款， redeem赎回



#### 函数阅读

asset()  返回 underlying token 的地址

totalAssets() 返回underlying token被金库合约所管理的数量

**converToShares()** 在满足 所有条件的情况下（条件在注释当中给出)  能换到shareToken的数量

使用到了_confvertToAssets()函数

assets 底层token数量 

supply 表示现在token的总供应量 

注: 这里可能出现supply ==0 的情况



**_convertToShares() **

assets uint256 类型，表示底层token的数量，supply 表示保险箱token的供应量。

如果assets ==0 || supply ==0 时，可想而知底层token还没有放入到合约当中，

所以这里的计算是一种预先的估计；

其他情况，就是 数量为assets的底层token存放到了合约中，同时发行了supply数量的保险箱token。

_convertToAssets() 其实也是同理



下面是四组函数，分别对应 deposit（存款)， withdraw（提款）， mint 和 redeem 

maxDeposit/previewDeposit/deposit

maxMint/previewMint/mint

maxWithdraw/previewWithdraw/withdraw

maxRedeem/previewRedeem/redeem



**_deposit() && _withdraw() **

注释当中对于针对ERC777的重入攻击进行了一定的分析

```solidity
首先改变调用者的状态，然后改变合约本身的状态，从而可以保证合约的安全。
```



#### 关于ERC4626 要更进一步分析的地方

erc777

可能的重入攻击的分析 

滑点问题的分析 [fei protocol的 ERC4626Router](https://github.com/fei-protocol/ERC4626#erc4626router-and-base)



### What's Next...

之后和ERC20相关的内容包括但不限于:

ERC223 

ERC777

yearn finance

usdt源码分析

DAI源码分析

usdc源码分析

shibtoken源码分析





剩下没看的ERC20文件夹下的openzepplin源码，

主要涉及到govenance, 还有闪电贷，以及可升级的代理合约的部分：

ERC20FlashMint.sol

ERC20Votes.sol

ERC20VotesComp.sol

govenance https://docs.openzeppelin.com/contracts/4.x/governance#token

Upgradeablity : transparent && UUPS   https://docs.alchemy.com/docs/how-to-create-a-dao-governance-token

Minimi Token. ERC20 compatible clonable token  https://github.com/Giveth/minime commit-id ea04d95

钓鱼ERC20合约







