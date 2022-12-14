# The Merge


初次发布: 2022.11.25 / 最新修改2022.11.25

关键词: **ERC721** **pak**

pak是一个加密艺术家，

[The Merge](https://www.niftygateway.com/collections/pakmerge) 是pak的加密艺术的作品，

根据2021年12月的 [这篇新闻](https://www.barrons.com/articles/paks-nft-artwork-the-merge-sells-for-91-8-million-01638918205) 的报道数据显示，售价为 $10.8 Million。

The Merge 是一组动态的、全链上的NFT，设计的规则也很巧妙, 

这篇文章主要记录一下自己阅读merge源码的思考。

## Merge结构梳理

https://etherscan.io/address/0xc3f8a0f5841abff777d3eefa5047e8d413a1c9ab#code

上面是The Merge的代码地址，一共由6个智能合约文件组成。

Strings.sol 包含的是如果把uint256的值转化为string类型值的toString函数。

Roots.sol 包含的是用来求a的n次方根的函数，返回值是uint类型，可想而知是近似的结果。

Base64.sol 包含处理base64编码解码的函数。

ABDKMath64x64.sol 利用int128来表示 64.64-bit fixed point numbers 的库，包含处理一些小数计算的函数，同时也会涉及到一些数值类型的转换。

整体The Merge的代码重点还是会放在 Merge.sol 和 MergeMetadata.sol 上，接下来会重点分析这两个代码文件的代码。

为了涉及到白名单的管理，还会涉及到一个NifyRegistry.sol 的文件。

## NifyRegistry.sol

这个代码文件本身并不是特别长，主要涉及到的数据结构如下

```solidity
  mapping(address => bool) validNiftyKeys;
  mapping (address => bool) public isOwner;
```

isOwner主要用来存储owner信息，如果是owner就可以进行addNiftyKey 和 removeNiftyKey的相关操作。

 如果是owner的也可以进行addOwner 和 removeOwner的操作，

代码当中还设置了MAX_OWNER_COUNT=50， 但是似乎这个常数并没有体现到后续的代码当中。

如果想大批量的实现owner和key的初始化，肯定还是需要接触constructor函数，传入 _owners 和 signing_keys的address类型的数组，就可以批量的进行owner和key（白名单）的初始化了。

## MergeMetadata.sol

这个代码文件主要是为了生成Metadata的实现，为了实现全链上NFT的目标，和loot类似，the Merge使用了svg格式来存储图片的信息，实际上就是在一个方形的平面中心根据参数绘制不同颜色和大小的圆形，这里就会涉及到一些处理的逻辑。

```solidity
interface IMergeMetadata {    
    function tokenMetadata(
        uint256 tokenId, 
        uint256 rarity, 
        uint256 tokenMass, 
        uint256 alphaMass, 
        bool isAlpha, 
        uint256 mergeCount) external view returns (string memory);
}
```

这个contract实现了上述的接口，可见最关键的就是tokenMetadata函数， 

在输入上述展示的参数的情况下，能够输出一个string类型的metadata信息(以json格式)。

一般NFT的实现会存储去中心化存储服务的地址，

比较标新立异的NFT项目（例如loot）就会存储svg格式的信息，

在阅读The Merge代码的时候，我惊喜地发现，这两种metadata的实现方式实际上都存在于代码当中。



## Merge.sol

merge主要的逻辑其实是实现了 ERC721, 以及最为重要的Merge函数。

初次看的时候感觉比较惊艳，感觉工程上有可以优化的空间。



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


