
* [DeFi Network 更新内容](#defi-network-更新内容)
* [DeFis Network 概述](#defis-network-概述)
* [DeFis Pool 概述](#defis-pool-概述)
* [DFS分发模型中的顶层不变规则](#dfs分发模型中的顶层不变规则)
* [矿池类型介绍](#矿池类型介绍)
    * [一、交易挖矿矿池设计概述](#一交易挖矿矿池设计概述)
        * [设计思路:](#设计思路)
        * [交易挖矿算法:](#交易挖矿算法)
        * [案例详解:](#案例详解)
   * [交易挖矿彩蛋: Lucky Hint 幸运暴击](#交易挖矿彩蛋-lucky-hint-幸运暴击)
       * [设计思路](#设计思路-1)
       * [规则和算法](#规则和算法)
       * [案例详解：](#案例详解)
    * [二、流动性挖矿矿池设计概述](#二流动性挖矿矿池设计概述)
        * [设计思路:](#设计思路-1)
        * [流动性挖矿算法：](#流动性挖矿算法)
        * [案例详解:](#案例详解-1)
    * [三、铸币挖矿矿池设计概述](#三铸币挖矿矿池设计概述)
        * [设计思路:](#设计思路-2)
        * [铸币挖矿算法:](#铸币挖矿算法)
        * [案例详解:](#案例详解-2)
    * [四、邀请挖矿设计概述 (optional)](#四邀请挖矿设计概述-optional)
        * [设计思路:](#设计思路-3)
        * [邀请挖矿设计的初衷](#邀请挖矿设计的初衷)
        * [邀请码注册](#邀请码注册)
        * [邀请人](#邀请人)
* [DSR: DFS Saving Pool 和 DFS Saving Rate 介绍](#dsr-dfs-saving-pool-和-dfs-saving-rate-介绍)
    * [简介:](#简介)
    * [DSR收益来源](#dsr收益来源)
* [总结](#总结)
* [关于币价对挖矿难度的影响](#关于币价对挖矿难度的影响)
* [DFS的使用场景](#dfs的使用场景)
    * [1.支付 DFS 创建交易对](#1支付-dfs-创建交易对)
    * [2. 支付 DFS 获得邀请码](#2-支付-dfs-获得邀请码)
    * [3. 存入DSR系统, 获得更多DFS](#3-存入dsr系统-获得更多dfs)
    * [4. 治理](#4-治理)
* [DFS的价值](#dfs的价值)
* [上线时间安排](#上线时间安排)
* [重要提示](#重要提示)

# DeFi Network 更新内容

**该文档为DeFis Pool的设计文档。在具体实现过程中，可能还会有参数调整。不代表最终正式执行方案。**

## DeFis Network 概述


DeFis Network使命是构建去中心化的，人人皆可参与、共建、共享的开放式金融基础设施。让普惠金融平等服务于全世界每个角落的人。

DeFis Network的当下目标是做聚合多种有价值的DeFi协议的去中心化金融网络。

现阶段已上线Bank、Swap两个项目。

DFS是DeFis Network的平台代币。全称 Decentralised Finance Share, 中文绰号: 大丰收。

DFS是EOS公链上，首个0预挖，无私募，无创始人奖励的DeFi项目代币。

DeFis Pool 作为DeFis Network的扩展模块。

主要承担了DFS代币的分发和系统业务增长激励。

本文档介绍 DeFis Pool 的设计思路。

## DeFis Pool 概述

得益于DeFis Network的设计良好的可持续扩展架构, 在Bank、Swap Core不停转、不修改的情况下，DeFis Network还将是个可持续进化的系统。

DeFis Pool，矿池模块，就是系统的external logic的一个子模块。

DeFis Pool上线部署后，将在完全不影响Bank、Swap的运行的同时。负责激励Swap、Bank的增长。

并基于**公平、公正、公开**的算法。将DFS代币线性的分发出去。

挖矿的过程，所有的结算逻辑都在链上自动进行，挖矿结果，完全取决于系统初始设定的不变参数和动态参数的变化。 类比比特币挖矿中的算力增减、难度调节、产量减半。

DFS的挖矿机制里，一切由算法决定。**无需项目方定时结算。**

DFS，让大家都有机会，在POS公链上，去参与一个与比特币模型相似的DeFi项目代币的挖矿。

## DFS分发模型中的顶层不变规则

系统的确定性参数，如同比特币的总量、减半的周期、自动难度调节算法一样。确定后不会再变。DFS分发系统中的不变规则。一共有三：

1. 最大发行量: max_supply = 1000万
2. 挖矿难度系数增加(产量衰减)周期： 每发行100万,  自动衰减一次
3. 自动调节算法: 

```
damping = pow(0.75, (int)(current_supply / 1000000)); 
```

DFS在挖矿总量达到最大发行量1000万的过程中，一共会经历9次衰减。

这个行为，类似于比特币挖矿系统在130年内将经历的32次减半。

```
> Math.pow(0.75, 0);
1
> Math.pow(0.75, 1);
0.75
> Math.pow(0.75, 2);
0.5625
> Math.pow(0.75, 3);
0.421875
> Math.pow(0.75, 4);
0.31640625
> Math.pow(0.75, 5);
0.2373046875
> Math.pow(0.75, 6);
0.177978515625
> Math.pow(0.75, 7);
0.13348388671875
> Math.pow(0.75, 8);
0.1001129150390625
> Math.pow(0.75, 9);
0.07508468627929688
```

## 矿池类型介绍

当前矿池模块里规划了3大矿池:

* pool1: 交易挖矿(交易激励)
* pool2: 流动性挖矿(做市商激励)
* pool3: 铸币挖矿(USDD铸币激励)

其中 pool1 交易挖矿矿池, 将会包含一个子矿池, 称为**邀请挖矿分池**

当一笔交易存在邀请人的时候，受邀请者，**会获得5%的额外算力加成。其中4%属于受邀请者。1%分配给邀请人**。

以下分别介绍三大矿池和"邀请机制"的设计思路和算法细节。

### 一、交易挖矿矿池设计概述
    
#### 设计思路:

1. 用户交易即为系统增长做了细微可量化的增量贡献, 即便是千万分之一的增量贡献，都会被系统算法捕获。进行激励。

2. 系统感知, 立刻发行DFS, 当场回馈用户,回馈比例将由算法自动计算。实时进入用户账户。

即使当下的价值只有手续费的十分之一。但由于DFS未来价格不可估算。因此这笔交易，对交易者来说，至少是绝对不亏的手续费打折。那至多呢？ DFS接着涨，那这笔交易挖矿产生的DFS，就是价值远大于这笔交易本身的馈赠。

通过交易挖矿的获得的DFS，你可理解为，是送出了一张加入DFS大家庭的体验卡。意义如同当年大佬送我的0.1个BTC体验卡。

#### 交易挖矿算法:

交易挖矿，不再是无脑的激励，不再是可以免费薅的羊毛，它是建立在真实交易需求之上的手续费补贴、是发DFS体验卡，是一套系统自动、动态回馈任何微量增长贡献的算法。

抓住了核心需求，交易挖矿算法随之而来了:

```
trading_mining_reward =  trading_fee / dfs_price * discount * damping * pool_weight;
```
    
#### 案例详解:

参数假设：

* DFS市场均价: 1 DFS = 0.15 EOS， 
* 交易额: 100EOS， 手续费0.3% ，所以这笔交易，产生 0.3 EOS 的手续费。 
* discount:  0.2
* damping:  0.75
    
**问题:**  100 EOS的交易，交了 0.3 EOS的手续费，那么交易挖矿会获得多少的 DFS ？

套用交易挖矿公式:

```
trading_mining_reward = 0.3 / 0.15 * 0.2 * 0.75 = 0.3 DFS 
```
最终交易挖矿获得 0.3 DFS。按当下1 DFS = 0.15 EOS的价值，如果选择当场卖出DFS 即可获得 0.045 EOS,  相当于这笔交易手续费打了8.5折，不卖出，相当于拿住了无限可能。

对于这笔交易，如果存在邀请人。那么算力自动加成5%。其中4%给交易者，1%给邀请人。
(关于邀请机制的细节，详情看本文后面的 “邀请挖矿”小节)

### 交易挖矿彩蛋: Lucky Hint 幸运暴击（限时活动）

#### 设计思路

在比特币挖矿系统中，存在幸运值的概念。平均每十分钟出块的基础上，通常有幸运的矿工，能够在十分钟内提前算出哈希值，提前获得矿工奖励。

在DFS的交易挖矿矿池中，我们同样保留这“幸运值”的概念。这一彩蛋，将改变平平无奇的“手续费补贴”挖矿模型。

#### 规则和算法

平均每十分钟，会出现一次Lucky Hint。 
满足以下两个条件的交易，可视为触发 Lucky Hint，获得Lucky Hint的额外奖励:

1. 交易额小于2500 EOS。
2. 交易的时间，在每个十分钟的第一分钟的60秒内。

代码示例： 

```
uint64_t lucky_key = current_time_point().sec_since_epoch() / 60;
bool is_lucky_hint = lucky_key % 10 == 0;
double discount = trading_mining_discount; //default 0.2
if (is_lucky_hint && tranding_eos.amount <= (2500 * 10000))
{
   discount = 3;
}
```

在交易挖矿算法中，有一个 discount 系数，表示交易手续费的打折力度。

触发Lucky Hint的 奖励，将获得 discount=3 的奖赏。这意味着这笔交易手续费不仅全免， 还有额外DFS赠送。

但交易挖矿最终所能获得的DFS奖励，依旧受到 damping 参数、pool_weight 、dfs_price参数的影响，

挖矿所得DFS，同样20%进入DSP saving pool 公共账户，等待DSR分配。 

#### 案例详解：

挖矿示例: 

* 交易额 1000 EOS -> EOS/DFS , 手续费 3 EOS
* pool_weight:  1.2
* damping : 0.75
* discount: 3 (默认0.2)
* dfs price: 0.5


假设这笔交易，触发了每十分钟一次的幸运暴击 :

```
trading_mining_reward = 3 / 0.5 * 3 * 0.75 * 1.2 * 0.8 = 12.96 DFS 
```

假设这笔交易，没有触发幸运暴击 :

```
trading_mining_reward = 3 / 0.5 * 0.2 * 0.75 * 1.2 * 0.8 = 0.8640 DFS 
```

15 倍交易挖矿收益！十分钟一次。

不仅科学家可以进行技术竞赛。

矿机生产商一样有肉吃。

做市商则更是无风险躺赚做市收益。

DFS吃货的大佬的DFS买入需求，也得到了满足。

### 二、流动性挖矿矿池设计概述

#### 设计思路:

对流动性提供者来说，团队的重心将会是建立一个安全的体验良好的平台，让交易双方安全及时的完成仓位配置与交易决策，并在此基础上，进行一部分激励。

对流动性提供者来说，第一考虑风险、第二考虑收益。

**第一点：风险**，流动性资金凭什么到你这提供流动性？ 风险一定是第一个考虑的要素。

DFS的swap核心模块，经受过33小时9500万美元交易量，安全上已经有了良好的基础。合约的安全审计对接了慢雾、派盾、后续还将对接海外知名审计机构。

在多方审计的加持下，对核心代码进行逐行逐字的轮番审核，swap core的代码，最终将变得无懈可击的稳固，如同的精简的诗一样，改其中一个单字都是画蛇添足。

**第二点： 收益**。Compound在开启挖矿的第一周，成功吸引到借贷公司大资金的加入，锁仓金额暴涨，甚至超越了Maker，这是为什么，一定是大资金提供方在权衡了安全和收益后，才作出的选择。
Compound的安全性，建立在已经运行很久不出问题。收益率则众所周知，Comp，一度暴涨至300多美金以上。 大资金的逐利性，也体现在USDT池挖矿收益率降低后的几天，大资金快速的撤离。大资金的撤离，是因为BAT池的风险已经过高，而USDT的收益又不足。

那么，DFS的流动性激励如何做？ 

1. 安全保障： DFS Swap的运行时间虽然还不长，但有过33小时9500万美元交易量的考验。加上进行中的多方审计。安全性，目前为止，是可接受的。小额的投入可以非常放心。
2. 收益率： 流动性激励的收益，需要在当下或未来，给流动性挖矿的人足额的回报。否则流动性资金，会理智的流到其他更好的地方。

流动性挖矿的设计原则，就是给流动性做市商，提供一份当下可估值的收益。不仅可以cover掉流动性资金的做市损失，还提供当下即可估值、但放在未来无法估值的收益。


#### 流动性挖矿算法：

抓住了核心需求，挖矿算法随之而来了:

```
liquidity_mining_reward =  (liquidity * pow(apr_s, time_elapsed) - liquidity) / dfs_price * damping * pool_weight;
```
其中的 apr_s , 是一个可治理参数，代表以秒单位计算的年化收益率。

#### 案例详解:

参数假设：

* DFS市场均价: 1 DFS = 0.15 EOS， 
* 流动性挖矿金额: 10000 EOS 。 
* apr_s:  1.0000000015471259828814254433382
* damping:  0.75
    
**问题:**  10000 EOS，做市一天，能获得多少 DFS ？

套用做市挖矿公式:

```
liquidity_mining_reward =  (10000 * pow(1.0000000015471259828814254433382, 86400) - 10000 )/  0.15 * 0.75  * 1 = 6.6840 DFS
```

最终交易挖矿获得 6.6840 DFS = 1.0026 EOS。如果选择当场卖出DFS, 相当于获得了年化5%左右的日收益，不卖出，放到以后，则可能是回报率不可估计的收益。

系统中的每一个DFS，都将是稀缺且珍贵。

### 三、铸币挖矿矿池设计概述

#### 设计思路:

1. USDD需要具备一定规模的发行量，以及越来越多的应用场景，才能建立起网络效应。成为有效、有用的稳定币。
因此铸币挖矿的激励措施，正是为了推动USDD的发展。
USDD铸币挖矿算法非常简单，是一种铸币手续费补贴政策，和交易挖矿类似。放当下看是手续费打折，放未来看，是回报率不可估的系统增长贡献奖。

**ps: 团队正在研究，如何针对抵押EOS生成USDD的这部分EOS再进行一次流动性挖矿。加大激励抵押EOS生成USDD。**

#### 铸币挖矿算法: 

```
mint_mininng_reward = mint_fee / dfs_price * discount * damping
```

#### 案例详解:

参数假设,

* DFS市场均价: 1 DFS = 0.15 EOS， 
* 铸币10000.0000 USDD，手续费0.3% = 30 USDD = 10.7 EOS  
* discount: 0.2， 
* damping 0.75

问题，铸币10000.0000 USDD，手续费0.3% = 30 USDD = 10.7 EOS， 那么铸币挖矿会获得多少的 DFS ？

套用铸币挖矿公式 : 

```
mint_mininng_reward = 10.7 / 0.15 * 0.2 * 0.75 = 10.7 DFS 
```

最终铸币挖矿获得 10.7 DFS = 1.6 EOS。直接卖出DFS相当于铸币手续费打了8.5折，不卖出，相当于拿住了无限可能。


### 四、邀请挖矿设计概述 (optional)

#### 设计思路:

回顾互联网风起云涌的20年，任何顶级app或爆款应用的启动初期，一般都会有一个**“人传人”的病毒式设计**。

包括圈内的各大中心化交易所、火爆过的bc dapp、甚至著名的zjp，都存在“人传人”的现象。

其中 “邀请“ 这个动作，基本可以覆盖各种玩法。区别无非是如何邀请、邀请奖励如何结算和分发罢了。
    
#### 邀请挖矿设计的初衷

DeFi的协议层，理应更加的开放，不仅可以公正、平等的接入一切需求方。还应该提供适当的回馈机制。如果Swap成为一个深度良好的去中心化交易平台，那么除了人人都可以提供资金做市的模式，对于促进了交易达成的角色，也应该给予奖励。

邀请人，是类似分销商或平台流量提供者的角色。
邀请机制，可以借助网络的力量加速传播。

更开放性的协议层设计，不仅有助于协议被第三方应用集成。也给潜在的合作方，比如钱包、中心化交易所、代购商，提供了系统性永久收益的手段。 

保持最简单、最开放的协议、最纯粹透明的链上去中心化，最可能成为EOS公链上被大规模使用和接入的基础流动性协议，DeFis Swap 已经做好了准备。

#### 邀请码注册

如果看好这个可能会“人传人”的去中心化交易所。

那么强力建议在 DFS还低价的时候，速速抢注邀请码。因为DFS会涨价。珍贵的邀请码也会升值。

拥有了邀请码，相当于拥有了DeFis Swap的分销牌照。只要有自己的流量入口，就能获得长久的邀请挖矿收益。

当一个普通用户，是通过邀请码的链接进行交易的时候，挖到的DFS量，会获得5%的加成，其中1%分给邀请人。另外4%直接分给用户。没有一个用户，会主动拒绝额外的4%收益。

往 dfsdfsfamily 转账 100 DFS，即可获得和转账账号绑定的唯一邀请码 (后续邀请码还可转让出售)。

#### 邀请人

首先分析，想要成为邀请人的需求方。(可以理解为交易所的代理商、加盟商等概念)。潜在的邀请人，有以下几种角色:

   1. 钱包方： 有用户、有流量。申请成为邀请人，具备长期有效收益分成
   2. 交易所对接：交易深度是可以共享的，因为协议是开放的
   3. 第三方代购
   4. dex协议聚合器
   5. 长期有交易需求的用户，如高频量化做市商。

首先，申请成为邀请人的一方，是享受长期系统收益的。

因此邀请码不可能免费发放。需要用DFS买。暂定100 DFS一个邀请码。顺便也新增了一个DFS的使用场景。

## DSR: DFS Saving Pool 和 DFS Saving Rate 介绍

### 简介: 

如果你熟悉以太坊MakerDao。那么你对DSR的设计理念一定不陌生: 

它非常的 Simple. Free and Powerful.  

DSR 是区块链技术如何为每个人提供平等金融服务的方式的一个示例。

DSR 设计原则上，是一份具备保障性收入的底层协议。不仅可以非常方便的被个人使用，还能轻松的被中心化交易所、钱包理财等第三方app接入。

### DSR收益来源

在DFS1.0的挖矿版本中，我们在每发行一个DFS时，收取了其中20%的DFS作为创始人奖励。

为了致敬以太坊上YFI、YFII的真DeFi精神，在新版的挖矿模型中，新挖矿产出的DFS，团队决定**一个也不拿了**！

**让应该去中心化的部分，彻底的去中心化。**

我们团队之后获取DFS的途径，和所有人保持一致：参与挖矿或者二级市场买入。


新的挖矿模型中，每发行出一个DFS，其中的20% 会被存入公共的 DFS saving pool中，剩余的80%则直接发给矿工。

而累计在 DFS saving pool 中的 DFS，将再一次以保障性收入算法，线性的发放给所有存入DFS的人。

* 持有DFS
* 向DSR系统存入DFS
* 每秒计息、随存随取

举例：存入 1万 DFS， 按5%年化收益率计算，一年后取出得到 10500 DFS。一天后取出得到10001.3698个，每秒计息，哪怕只存入了1分钟，也能得到额外的 0.0009个 DFS。

DSR之于DFS的意义，如同REX对EOS的意义。

开发团队在度过发展期后，还将定期用25%利润回购DFS，存入DFS SAVING POOL，进一步赋能DFS和DSR。

**DFS Saving Rate is a Game-Changer for the EOS DeFi Ecosystem**. 😆


## 总结

可以看到，所有矿池的设计原则，都保持着目标和结果的一致性。 我们的设计原则，就是增涨。

涨，是交易量的涨、是流动性资金池的涨、是USDD铸币量的涨、是整个系统资金体量的涨。

当系统的每一个设计细节，都围绕“涨”来设计。长期来看，DFS本身内在价值的涨，是一种必然。

我们希望通过纯粹和透明的去中心化的思想、设计和机制，和大家共有共建共享一个去中心化资产配置和交易的金融活动新方式。也让我们的产品和服务，成为大家日常最常用的金融工具。

而被忽略的治理权才是DFS代币最不可估的价值，远不是手续费分红这种可量化的价值可以比较的。

我们的项目做的越好，治理权所体现的价值越高。

## 关于币价对挖矿难度的影响

在所有的挖矿算法中，币价因子是个动态参数，当币价越高，算力越大，挖矿难度就会越高，单位产出的DFS就会越少。

随着挖矿难度的提升，1000万DFS很可能100年也挖不完。

## DFS的使用场景

### 1.支付 DFS 创建交易对

为了维护良好的交易环境，避免因为无成本上币，而导致的垃圾币横生或科学家恶意创建大量无效交易对。

我们改进了自主、免费创建交易对的初始设定。

创建新的交易对的时候，需要消耗 DFS。

目前暂定50DFS创建一个交易对。

 
### 2. 支付 DFS 获得邀请码

邀请码的功能和价值。已经在前面的"邀请机制"小节里描述。

邀请码获取途径有2

1. 消耗 100 个 DFS 获得邀请码。 
2. 质押 5000 个 DFS 获得邀请码。(质押可赎回，赎回后，邀请码立即失效。）


### 3. 存入DSR系统, 获得更多DFS

将闲置的DFS存入DSR系统，即可享受5% DFS币本位年化收益。随存随取、每秒计息。

详情见 DRS: DFS saving rate 协议的设计文档。

### 4. 治理

在治理框架出来后，可以参与治理投票。以及发起DIP改进提案等。

DFS新的代币模型设计，充分考虑到了种种情况。

整个架构上预留了非常大的治理空间。为今后的去中心化治理框架，做好了准备。


**更多的使用场景，随着项目进展而添加。**

## DFS的价值 

DeFis Network is

Permissionless

Trustless

Serverless

DFS is priceless

公平的分发每一个代币。代表了我们的使命、愿景、价值观和做去中心化项目的决心。

长期来看，DFS的价值是随着愿景的实现来逐步体现的。

在上一版本的发行方案中，有20%的代币属于创使人奖励。

在新的发行方案中。我们做出了取舍。所有的代币，将全部公平的分发给矿工和持币人。

纯算法，真DeFi。

仅以此，致敬久远的中本聪精神。

## 上线时间安排 

由于每个模块都需要单独的安全审计，所以DeFis Pool的功能，是逐步开放的。

1. 新版UI上线 ------ 8月5号
2. 邀请码获取入口开放 ------ 8月5号
3. 交易挖矿矿池开放 ------ 8月14号
4. 铸币挖矿矿池开放 ------ 8月14号
5. 做市挖矿矿池开放 ------ 8月14号
6. DSR系统上线 ------ 8月25号

攒了一手牌，周周有利好。

以上为预估时间，实际可能会不可意料的因素，推迟或提前上线。

## 重要提示

在去中心化投票治理框架实现之前。

**项目方保留对项目中关键的可治理参数的修改权限。**

修改的原则和目的，是为了使项目更好的良性运转。

比如，对矿池收益权重的上调或下调，一定是基于项目上线后，收集到的市场反馈而作出的修正。

创新的项目，初期的启动和运转，不可能一步到位，一定是渐渐的变得更好的。

请谅解我们保留项目初期修改权限的理由，这个跟爬行期的婴儿需要有人照顾一个道理。

**PS： 如果在挖矿过程中出现了未知的、不可预料的科学家攻击等突发事件，我们的风控系统将自动启动矿池保护程序，暂停矿池出矿，等待风险排除后再次开启。避免矿工被埋。**



>
DeFis Network Founding Team
                                                                                        感谢社区高手们的支持和贡献
                                                                                         2020/08/04

