# ddy的web3漫游日记 ERC20初探篇

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
初次发布: 2022.10.16 / 最新修改2022.10.17

关键词: **ERC20** **WETH** **USDT** **openzepplin**

## 上手建议

学习ERC20可以先阅读[wtf.cademy的ERC20章节](https://wtf.academy/solidity-advanced/ERC20/) ，建立一个基础的理解。

然后阅读[openzepplin的文档](https://docs.openzeppelin.com/contracts/4.x/)当中的 Tokens/ERC20，Tokens/ERC20/Creating Supply， 目前openzepplin最新版是4.0，支持的solidity ^0.8.0。

ERC20现在模块化相当之高, 在文档的[Wizard](https://docs.openzeppelin.com/contracts/4.x/wizard) 中甚至可以通过点击几下按钮就完成一个基础ERC20的合约。

但是FEATURES 当中的 Mintable, Burnable, Pausable,Permit, Votes，Flash Minting, Snapshots,

以及ACCESS CONTROL 当中的Ownable, Roles，

以及UPGRADEABILTY当中的Transparent, UUPS 分别都是什么含义呢?

想整明白就需要结合文档下方的API当中的 ERC20 当中的接口文档来理解，强烈推荐直接阅读源码，注释很详细。

openzeppelin注释当中推荐的论坛上的[入门向tutorials](https://forum.openzeppelin.com/t/how-to-implement-erc20-supply-mechanisms/226), 大家也可以先从这个看起。

## openzepplin部分源码阅读

对@openzeppelin/contracts/token/ERC20/文件夹内的源码阅读，注释很详细，

我会记录一些我所关注的细节。

**ERC20.sol**

unchecked {} 语句主要是为了不进行safeMath检查，从而节省gas费, [stackexhange上的回答](https://ethereum.stackexchange.com/questions/113221/what-is-the-purpose-of-unchecked-in-solidity) 。

取到某个变量最大值的方法  type(*uint256*).max 。

**extensions/burnable.sol** 支持burn的功能。

**extensions/capped.sol** 是对token总供应量有一个上限。

**extensions/Pausable.sol** 如果暂停时不能进行转账。

**extensions/ERC20Snapshot.sol 顾名思义，是进行一个快照的**

防止投票时一个人投票后转账给别人，别人继续投票，“double spend attack"

```solidity
 using Arrays for uint256[];
 using Counters for Counters.Counter;
```

使用了openzeplin的utils库中Counter ，使用 .current()  .increment()函数访问单调递增的计数器。

使用了openzepplin的utils库中的Arrays ，使用Arrays.findUpperBound在单调递增的数组内二分查找。

注释中陈述实现的灵感来源是[minime token当中的实现](https://github.com/Giveth/minime/blob/ea04d950eea153a04c51fa510b068b9dded390cb/contracts/MiniMeToken.sol) ，minime token 是ERC20 compatiable clonable token，看起来也很有趣，之后可以研究一下。

注释中说明使用ERCSnapshot在每个块进行快照会产生大量的gas费, 替代方案是ERC20Votes。

**extensions/ERC20Wapper.sol** 是把其他的ERC20 token打包起来。

abstract 抽象合约，会涉及到一系列继承相关的内容，可以参考 [Solidity极简入门: 13. 继承 父与子](https://mirror.xyz/daoassange.eth/HTCOqhsxTXs42NNv3wfzNRQMN6qGHGYY9iaTJhhKBb4) 。

使用到了 SafeERC20.safeTransfer函数，SafeERC20的实现在 utils/SafeERC20.sol当中，会在后续展开。

如果有 underlying (被打包的ERC20 token address) 的erc20 token不小心打进来 还可以_recover，当然这个函数需要被一些有管理员权限的账号继承。

**utils/SafeERC20.sol**

这是一个库函数，用于处理和ERC20合约相关的交互，主要是为了处理非标准的transfer，因为没有规定 ERC-20 合约必须在发生失败时回退交易, 有的项目会交易失败时返回false之类的。

```
即使有SafeERC20的各种函数，但是和合约交互的时候还是不能掉以轻心，
要阅读ERC20合约的具体实现，然后针对性的编写调用合约。
```

感兴趣的可以深入阅读[登链社区的翻译文章安全的处理 ERC20 转账（解决非标准 ERC20 问题）](https://learnblockchain.cn/article/3074)

```
果然还是要好好学英语, 阅读第一手的英文资料。
```

查看SafeERC20的代码，各种操作当中设计到了一个很重要的函数`_callOptionalReturn` （注释当中的解释更加丰富），

```solidity
function _callOptionalReturn(IERC20 token, bytes memory data) private {
        bytes memory returndata = address(token).functionCall(data, "SafeERC20: low-level call failed");
        if (returndata.length > 0) {
            // Return data is optional
            require(abi.decode(returndata, (bool)), "SafeERC20: ERC20 operation did not succeed");
        }
    }
```

IERC20是 IERC20.sol 当中制定的标准, data 是调用函数时候的 the call data（ 通常使用 abi.encode或者它的变种来包装）。

这里的address在开头使用了openzepplin的utils库中的Address, 使用的函数functionCall的输入输出参数如下

```solidity
function functionCall(address target, bytes memory data) internal returns (bytes memory) 
```

 If `target` reverts with a revert reason, it is bubbled up by this  function (like regular Solidity function calls).

相当于对底层接口进行了一个封装，更加的安全，封装过程比较复杂，还利用了assembly，

具体实现见 @openzepplin/utils/Address.sol。

原文注释: We need to perform a low level call here, to bypass Solidity's return data size checking mechanism, since we're implementing it ourselves. We use {Address.functionCall} to perform this call, which verifies that the target address contains contract code and also asserts for success in the low-level call.

如果调用有返回值的话, 利用[ABI](https://learnblockchain.cn/docs/solidity/abi-spec.html#abi)-decode 对提供的数据进行解码，类型在括号中作为第二个参数给出，返回值为false时不满足require的要求会进行回退。

整体函数的注释:

Imitates a Solidity high-level call (i.e. a regular function call to a contract), relaxing the requirement on the return value: the return value is optional (but if data is returned, it must not be false).

```
openzepplin写的注释是相当的好，赞赞赞。
```

## 真实项目

完成上述过程之后，就可以对真实的web3世界当中的各种项目进行源代码的查看了。

不过很多知名项目是在solidity ^0.8.0之前出现的，我们也可以通过对比来进行一个更加深入的理解。

#### weth

这是eth和weth相互兑换的一个合约。

介绍网站: https://weth.io/ 

网站当中提到 ERC223 是一个ERC20的替代方案。

weth address: 0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2

weth的代码版本是^0.4.18更新到0.8.0的时候有很多语法不适配了。

例如: 

```solidity
function() public payable {
     deposit();
}
```

Expected a state variable declaration. If you intended this as a fallback function or a function to handle plain ether transactions, use the "fallback" keyword or the "receive" keyword instead.solidity(2915)。

关于receive 和 fallback 可以查看 [Solidity极简入门: 19. 接收ETH receive和fallback](https://wtf.academy/solidity-advanced/Fallback/)   

#### usdt

关于 usdt 的一个坑 https://learnblockchain.cn/2018/11/23/65a75ab1341e

使用SafeERC20可以防范这种问题。

```
即使是和非常知名的项目进行交互时，也要阅读智能合约了解具体的逻辑，要不然可能踩坑。
```



### What's Next...

欢迎大家持续关注，之后和ERC20相关的内容包括但不限于:

ERC223 

usdt源码分析

DAI源码分析

usdc源码分析

shibtoken源码分析



剩下没看的ERC20文件夹下的openzepplin源码，主要涉及到govenance, ownership, 

ERC4626

ERC20FlashMint.sol

ERC20Votes.sol

ERC20VotesComp.sol

govenance https://docs.openzeppelin.com/contracts/4.x/governance#token

TokenTimelock.sol



Upgradeablity : transparent && UUPS   https://docs.alchemy.com/docs/how-to-create-a-dao-governance-token

Minimi Token. ERC20 compatible clonable token  https://github.com/Giveth/minime commit-id ea04d95

钓鱼ERC20合约



### 懂得都懂

既然都看到结尾了，0x807船长诚挚地邀请您 :-)

关注ddy的推特 [@ddy_mainland](https://twitter.com/ddy_mainland)

为github仓库star [ddyWeb3Tour](https://github.com/fromddy/ddyWeb3Tour) 
