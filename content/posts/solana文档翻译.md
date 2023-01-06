---
title: "solana文档翻译"
date: 2022-08-11T13:26:57+08:00
draft: true
---

# Cluster

一个solana集群是一群验证器的集合,他们为客户提供交易并维护账本正确性的服务.许多集群可以共存.当两个集群共享相同的创世块的时候,它们会尝试收敛,否则他们会简单的忽略掉彼此,发送给错误节点的交易会被悄悄的拒绝掉.在这一节,我们会讨论,如何创建集群,如何让节点加入集群,它们是怎么共享账本、确保账本被复制,如何对抗有缺陷或者恶意的节点.

## 创建集群
在启动任何验证器钱,首先需要创建一个创世配置.这个配置包含两个公钥:一个铸币厂密钥和引导验证器迷药.持有引导验证其密钥的验证器会负责将第一个entry加入账本.它使用铸币厂账户初始化内部状态.这个账户会持有创世时定义的原生代币数量.第二个验证器连接引导验证器注册为一个验证器后,其余新加入验证器连接集群中的任何一个验证器注册加入.

验证者从领导者那里接收所有的条目并提交确认条目有效的投票.投票后,验证者应当存储这些条目.当验证者观察到足够数量的副本存在,就会删除自己的副本.

## 加入一个集群

验证者通过发送其控制层面的信息来加入集群.其控制层面是通过gossip(留言)协议实现的,这意味着一个节点可以和任意节点注册,并期待其注册信息传播到集群的每一个节点.同步所消耗的时间与参与集群治理的节点数量的平方成正比.从算法上,这被认为非常缓慢.但作为交换,一个节点可以确信拥有和其他节点相同的信息,且信息不会被任意一个节点审查.

## 将交易发送给一个集群

客户端将交易发送给任意验证器的TPU(Transaction Proccessing Unit 交易处理单元)端口.
如果这个节点是是验证器角色,它会将交易转发给制定的领导者.
如果这个节点是领导者角色,它会打包传入的交易,为这些交易添加时间戳创建条目,然后将其推送到集群的数据层面. 一旦数据落入数据层面,数据中的交易会被节点验证,并有效的加入到账本中.

## 确认交易

一个solana集群能在上千个节点的情况下实现亚秒级扩容,并计划扩展到十万个节点.确认时间只会随着验证节点的增加而对数增加,同时对数的基数非常高.举个例子🌰:如果这个基数是1000, 意味着对于参与确认的前1000个网络节点,确认交易的周期将是三个网络跃点时间加上绝对多数票中最慢的验证者. 对于下个百万数量的节点,验证时间的增加只是一个网络跃点时间.

Solana defines confirmation as the duration of time from when the leader timestamps a new entry to the moment when it recognizes a supermajority of ledger votes.

可以使用混合如下技术来实现对扩展的确认:
    1. 带有 VDF 样本的时间戳交易并签署时间戳.
    2. 将交易分成批次，将每个批次发送到单独的节点，并让每个节点与其对等节点共享其批次.
    3. 递归地重复上一步，直到所有节点都有所有批次.

solana定期的轮换领导者,这种行为被称作 slots. 每个领导者只在其分配的slots里产生条目(entries). 因此领导者在交易上标记时间戳让验证者查找领导者的公钥.领导者在时间戳伤签名以便验证着验证,以证明签名者是领导者的公钥所签.

接下来,交易被发送给几个批次以便节点发送交易给多方时不需要制作多个副本.举个例子🌰:如果领导者需要向 6 个节点发送 60 个事务，它会将 60 个事务的集合分成 10 个事务的批次，然后向每个节点发送一个。这允许领导者在网络上放置 60 个事务，而不是每个节点的 60 个事务。然后每个节点与其对等节点共享其批次。一旦节点收集了所有 6 个批次，它就会重建原始的 60 个交易集。

一批交易只有经历过足够的次拆分,才能使其笑到让其头信息成为网络带宽的主要消费者. 为了将节点扩展到数十万个验证者,每个节点都可以与主节点相同的技术方式扩展到另一组相同大小的节点.我们叫这个技术为 “涡轮块传播“.

# 同步

快速,可靠的同步是solana能实现如此高吞吐的主要原因.传统的区块链在一个叫做 blocks的大块交易上进行同步.通过块同步,知道经过被称为“块时间”的区块才能处理交易.在工作量证明共识下,这个块时间需要非常大(约10min)以尽量减少大量验证者在同一时刻声称新的有效块的几率.在质押证明共识下没有这样的限制,但是失去了可靠的时间戳,一个验证着将无法确定传入块的顺序.流行的解决方案是用挂钟时间戳标记每个块. 因为时钟漂流和网络延迟的原因,时间戳只能在一两个小时内准确.为了解决这个解决方案带来的麻烦,系统通常会延长区块时间,提供合理的准确性,以确保每个区块的时间戳在增长.

solana使用了截然不同的方法,这个被称为基于历史的证明(POH).领导节点的时间戳区块和其加密证明到最后一个证明经过了一段时间.所有的证明中数据的哈希肯定是在证明之前产生.该节点与能够验证这些证明的验证节点共享新块. 这些块可以以任何顺序到达验证者，甚至可以在数年后重播。 有了这种可靠的同步保证，Solana 能够将块分成更小的事务批次，称为条目(entries). 在任何区块共识概念之前，条目(entries)会实时流式传输到验证器。

Leader nodes "timestamp" blocks with cryptographic proofs that some duration of time has passed since the last proof. All data hashed into the proof most certainly have occurred before the proof was generated. The node then shares the new block with validator nodes, which are able to verify those proofs. The blocks can arrive at validators in any order or even could be replayed years later. With such reliable synchronization guarantees, Solana is able to break blocks into smaller batches of transactions called entries. Entries are streamed to validators in realtime, before any notion of block consensus.

Solana技术上的来说从未发送任何一个块， 但是可以通过这种方式来描述solana验证器之间投票确认的过程.In that way, Solana's confirmation times can be compared apples to apples to block-based systems. 当前的实现下块时间是800ms.

Solana technically never sends a block, but uses the term to describe the sequence of entries that validators vote on to achieve confirmation. In that way, Solana's confirmation times can be compared apples to apples to block-based systems. The current implementation sets block time to 800ms.

背后发生的故事时,只要领导节点将一组有效的交易批处理到一个条目中,条目就会被流式传输到验证其. 验证器在对他们的有效性进行投票很久之前就进行了处理. 通过乐观的处理这些交易, 最后一个收到交易的节点可以没有延迟的开始投票. 如果没有达成共识,那么节点会简单的回滚其状态. 这种乐观处理技术在1981年被提出,被称为乐观并发控制.它可以被应用到区块链中, 以集群的方式对代表了完整账本的哈希进行投票,直到抵达某个高度.在solana中,它使用最后一个条目的PoH哈希来简单的实现.

What's happening under the hood is that entries are streamed to validators as quickly as a leader node can batch a set of valid transactions into an entry. Validators process those entries long before it is time to vote on their validity. By processing the transactions optimistically, there is effectively no delay between the time the last entry is received and the time when the node can vote. In the event consensus is not achieved, a node simply rolls back its state. This optimisic processing technique was introduced in 1981 and called Optimistic Concurrency Control. It can be applied to blockchain architecture where a cluster votes on a hash that represents the full ledger up to some block height. In Solana, it is implemented trivially using the last entry's PoH hash.

## 与VDF之间的关系
## Relationship to VDFs#

基于历史的证明的技术为了使用在solana区块链上第一次提出是在2017年的11约.第二年的六月,斯坦福大学提出了一个叫做的VDF(可延迟证明函数)的相似技术.

The Proof of History technique was first described for use in blockchain by Solana in November of 2017. In June of the following year, a similar technique was described at Stanford and called a verifiable delay function or VDF.


VDF 的一个理想特性是验证时间非常快。 Solana 验证其延迟函数的方法与创建它所花费的时间成正比。通过拆分成 4000 核 GPU的任务，它的速度足以满足 Solana 的需求，但如果你问上面引用的论文的作者，他们可能会告诉你（并且有）Solana 的方法在算法上很慢，不应该被称为 VDF .我们认为 VDF 一词应该代表可验证延迟函数的类别，而不仅仅是具有某些性能特征的子集。在解决这个问题之前，Solana 可能会继续使用 PoH 一词来表示其特定于应用程序的 VDF。
A desirable property of a VDF is that verification time is very fast. Solana's approach to verifying its delay function is proportional to the time it took to create it. Split over a 4000 core GPU, it is sufficiently fast for Solana's needs, but if you asked the authors of the paper cited above, they might tell you (and have) that Solana's approach is algorithmically slow and it shouldn't be called a VDF. We argue the term VDF should represent the category of verifiable delay functions and not just the subset with certain performance characteristics. Until that's resolved, Solana will likely continue using the term PoH for its application-specific VDF.

PoH 和 VDF 之间的另一个区别是 VDF 仅用于跟踪持续时间。一方面，PoH 的哈希链包括应用程序观察到的任何数据的哈希值。这些数据是一把双刃剑。一方面，数据“证明了历史”——数据肯定存在于它之后的散列之前。另一方面，这意味着应用程序可以通过更改数据散列的时间来操纵散列链。因此，PoH 链不能作为随机性的良好来源，而没有该数据的 VDF 则可以。例如，Solana 的领导者旋转算法仅来自 VDF 高度，而不是其在该高度的哈希值。
Another difference between PoH and VDFs is that a VDF is used only for tracking duration. PoH's hash chain, on the other hand, includes hashes of any data the application observed. That data is a double-edged sword. On one side, the data "proves history" - that the data most certainly existed before hashes after it. On the other side, it means the application can manipulate the hash chain by changing when the data is hashed. The PoH chain therefore does not serve as a good source of randomness whereas a VDF without that data could. Solana's leader rotation algorithm, for example, is derived only from the VDF height and not its hash at that height.

## Relationship to Consensus Mechanisms#
## 与共识机制的关系
基于历史的证明不是一个共识机制,它只是用来提高solana 基于质押共识机制的性能和数据传输性能的一种手段.
Proof of History is not a consensus mechanism, but it is used to improve the performance of Solana's Proof of Stake consensus. It is also used to improve the performance of the data plane protocols.

## 领导者轮换
## Leader Rotation
在给定的任意时刻, 一个集群都希望只有一个验证者去处理账本条目. 通过一次只有一个领导者，所有验证者都能够重放相同的分类账副本.然而，一次只有一个领导者的缺点是，一个恶意的领导者能够审查投票和交易。由于审查无法与网络丢弃数据包区分开来，集群不能简单地选出一个节点无限期地担任领导者的角色.相反，集群通过轮换哪个节点来最大限度地减少恶意领导者的影响.
At any given moment, a cluster expects only one validator to produce ledger entries. By having only one leader at a time, all validators are able to replay identical copies of the ledger. The drawback of only one leader at a time, however, is that a malicious leader is capable of censoring votes and transactions. Since censoring cannot be distinguished from the network dropping packets, the cluster cannot simply elect a single node to hold the leader role indefinitely. Instead, the cluster minimizes the influence of a malicious leader by rotating which node takes the lead.

每个验证者使用相同的算法选择预期的领导者，如下所述。当验证者收到一个新的签名分类账条目时，它可以确定条目是由预期的领导者产生的。每个领导者被分配到的槽的顺序被称为领导者时间表。
Each validator selects the expected leader using the same algorithm, described below. When the validator receives a new signed ledger entry, it can be certain that an entry was produced by the expected leader. The order of slots which each leader is assigned a slot is called a leader schedule.

## 领导者轮换日程表
## Leader Schedule Rotation#

验证器会拒绝没有槽位领导者签名的块。所有槽位领导者的身份列表被称为领导者时间表。领导者日程表是在本地定期重新计算的。它为某一时间段分配槽位领导者，称为纪元。时间表必须在它所分配的槽位之前提前计算，这样它用来计算时间表的分类账状态就会最终确定。该持续时间被称为领导者时间表偏移。Solana将该偏移量设置为直到下一个纪元的槽的持续时间。也就是说，一个纪元的领导周期是根据上一个纪元开始时的分类账状态计算的。一个纪元的偏移量是相当随意的，并假定其足够长，以便所有验证者在生成下一个时间表之前完成其分类账状态。一个集群可以选择缩短偏移量，以减少股权变动和领导者时间表更新之间的时间。
A validator rejects blocks that are not signed by the slot leader. The list of identities of all slot leaders is called a leader schedule. The leader schedule is recomputed locally and periodically. It assigns slot leaders for a duration of time called an epoch. The schedule must be computed far in advance of the slots it assigns, such that the ledger state it uses to compute the schedule is finalized. That duration is called the leader schedule offset. Solana sets the offset to the duration of slots until the next epoch. That is, the leader schedule for an epoch is calculated from the ledger state at the start of the previous epoch. The offset of one epoch is fairly arbitrary and assumed to be sufficiently long such that all validators will have finalized their ledger state before the next schedule is generated. A cluster may choose to shorten the offset to reduce the time between stake changes and leader schedule updates.

在没有持续时间超过一个纪元的分区的情况下，只有当根分叉跨越纪元边界时才需要生成时间表。由于时间表是针对下一个纪元的，所以任何新的质押提交给root的fork在下一个纪元之前不会被激活。用于生成领导时间表的区块是第一个跨越纪元边界的区块。

While operating without partitions lasting longer than an epoch, the schedule only needs to be generated when the root fork crosses the epoch boundary. Since the schedule is for the next epoch, any new stakes committed to the root fork will not be active until the next epoch. The block used for generating the leader schedule is the first block to cross the epoch boundary.

如果没有一个持续时间超过一个纪元的分区，集群将按如下方式工作。
Without a partition lasting longer than an epoch, the cluster will work as follows:

1. 一个验证者在投票时持续更新自己的根叉。    
1. A validator continuously updates its own root fork as it votes.
2. 验证者在每次slot高越过纪元边界时，都会更新其领导日程表。
2. The validator updates its leader schedule each time the slot height crosses an epoch boundary.

举个例子:

让我们假设纪元周期为100个slot，在现实中这一数字要高得多。fork完成计算的slot高度在从99更新到102时会更新root fork.其中slot的100和101高度会因为失败二倍跳过.新的领导者周期是在slot高度为102的fork上进行计算.新的领导者会在slot高度200的地方开始活动,直到再次被更新.
Let's assume an epoch duration of 100 slots, which in reality is magnitudes higher. The root fork is updated from fork computed at slot height 99 to a fork computed at slot height 102. Forks with slots at height 100, 101 were skipped because of failures. The new leader schedule is computed using fork at slot height 102. It is active from slot 200 until it is updated again.

不可能存在不一致的情况，因为每个与集群一起投票的验证者在其根部通过102时都跳过了100和101。所有的验证者，无论其投票模式如何，都会承诺其根是102，或者是102的子孙。
No inconsistency can exist because every validator that is voting with the cluster has skipped 100 and 101 when its root passes 102. All validators, 
regardless of voting pattern, would be committing to a root that is either 102, or a descendant of 102.


## 领导者轮换日程表和纪元等级的分区
## Leader Schedule Rotation with Epoch Sized Partitions.#
领导者的日程偏移的持续时间和集群对正确的领导者日程表看法不一致有着直接的关系.
The duration of the leader schedule offset has a direct relationship to the likelihood of a cluster having an inconsistent view of the correct leader schedule.

考虑一下的情况
Consider the following scenario:

两个分区个字产生一半的区块.两者都没有达到定义的超级多数分叉.两者都能将跨越纪元100和200,但没有实际承诺到一个根上,因此也没有在整个集群中得出一个新的领导者日程表.
Two partitions that are generating half of the blocks each. Neither is coming to a definitive supermajority fork. Both will cross epoch 100 and 200 without actually committing to a root and therefore a cluster-wide commitment to a new leader schedule.

在这种不稳定的情况下,存在多个有效的领导者日程表.
In this unstable scenario, multiple valid leader schedules exist.

1. 每个分叉声称一个领导者日程表,其直接的父辈在上一个纪元.
2. 在下一个纪元开始后,领导者日程表对后续的分叉依然有效,直到它被更新.

1. A leader schedule is generated for every fork whose direct parent is in the previous epoch.
2. The leader schedule is valid after the start of the next epoch for descendant forks until it is updated.

在分区持续时间超过一个纪元后，每个分区的时间表将发生分歧。出于这个原因，应选择纪元持续时间，使其远远大于槽时间和一个分叉被提交给根的预期长度。
Each partition's schedule will diverge after the partition lasts more than an epoch. For this reason, the epoch duration should be selected to be much much larger then slot time and the expected length for a fork to be committed to root.

在对集群观察了足够长的时间后，可以根据分区持续时间的中位数及其标准偏差来选择领导者的日程表偏移。例如，一个比分区持续时间中位数长的偏移量加上六个标准差，将使集群中出现不一致的分账的可能性降低到100万分之一。
After observing the cluster for a sufficient amount of time, the leader schedule offset can be selected based on the median partition duration and its standard deviation. For example, an offset longer then the median partition duration plus six standard deviations would reduce the likelihood of an inconsistent ledger schedule in the cluster to 1 in 1 million.

## 在创世纪声称领导者日程表.
## Leader Schedule Generation at Genesis#

创世配置宣布了第一个纪元的第一个领导者.这个领导者最终会被安排在前两个纪元，因为领导者时间表也是在0槽生成的，用于下一个纪元。前两个纪元的长度也可以在genesis config中指定。第一个纪元的最小长度必须大于或等于塔tower BFT中定义的最大回滚深度。

The genesis config declares the first leader for the first epoch. This leader ends up scheduled for the first two epochs because the leader schedule is also generated at slot 0 for the next epoch. The length of the first two epochs can be specified in the genesis config as well. The minimum length of the first epochs must be greater than or equal to the maximum rollback depth as defined in Tower BFT.

## 领导日程表生成算法
## Leader Schedule Generation Algorithm#

领导者日程表是用预先设定的种子生成的,其流程如下:
Leader schedule is generated using a predefined seed. The process is as follows:

1. 定期使用PoH tick高度（一个单调增加的计数器）来作为稳定的伪随机算法的种子。
2. 在该高度，对银行中所有在集群配置的tick数内投过票的具有领导身份的被押账户进行抽样。这个样本被称为活动集.
3. 按照质押的比重对活跃集合进行排序
4. 使用随机种子来选择按质押加权的节点，以创建一个质押加权的排序。
5. 这个排序在集群配置tick数目后有效

英文:

1. Periodically use the PoH tick height (a monotonically increasing counter) to seed a stable pseudo-random algorithm.
2. At that height, sample the bank for all the staked accounts with leader identities that have voted within a cluster-configured number of ticks. The sample is called the active set.
3. Sort the active set by stake weight.
4. Use the random seed to select nodes weighted by stake to create a stake-weighted ordering.
5. This ordering becomes valid after a cluster-configured number of ticks.

## 日程表攻击的因素
## Schedule Attack Vectors#

### 种子
### Seed#
选择的种子是可预测的但不可偏向. 没有推敲攻击带来的影响.
The seed that is selected is predictable but unbiasable. There is no grinding attack to influence its outcome.

### 活动集
### Active Set#

领导者可以通过审查验证者的投票来偏袒活动集.领导者可能有两种方法来审查活动集.
A leader can bias the active set by censoring validator votes. Two possible ways exist for leaders to censor the active set:

1. 忽略验证者的投票
2. 拒绝为有验证者投票的区块进行投票

英文:
1. Ignore votes from validators
2. Refuse to vote for blocks with votes from validators

为了减少审查的可能性，活动集是在活动集抽样持续时间内，在领导者日程表偏移边界计算的。活动集抽样持续时间足够长，以便多个领导者都能收集到投票。

To reduce the likelihood of censorship, the active set is calculated at the leader schedule offset boundary over an active set sampling duration. The active set sampling duration is long enough such that votes will have been collected by multiple leaders.

## 质押
## Staking#
领导者可以审查新的质押交易或拒绝验证有新质押的区块。这种攻击类似于对验证者投票权的审查。
Leaders can censor new staking transactions or refuse to validate blocks with new stakes. This attack is similar to censorship of validator votes.

## 验证者操作密钥丢失
## Validator operational key loss#

领导者和验证者应该使用短暂的密钥进行操作，股权所有者通过委托授权验证者对其股权进行工作。

Leaders and validators are expected to use ephemeral keys for operation, and stake owners authorize the validators to do work with their stake via delegation.

集群应该能够从领导者和验证者使用的所有短暂密钥的丢失中恢复，这可能是通过所有节点共享的共同软件漏洞发生的。利益相关者应该能够通过共同签署验证人的投票来直接投票，即使该利益相关者目前被委托给验证者。
The cluster should be able to recover from the loss of all the ephemeral keys used by leaders and validators, which could occur through a common software vulnerability shared by all the nodes. Stake owners should be able to vote directly by co-signing a validator vote even though the stake is currently delegated to a validator.

## 追加条目
## Appending Entries

一个领导者时间表的寿命被称为一个纪元。纪元被分割成若干时段，每个时段的持续时间为T PoH ticks。
The lifetime of a leader schedule is called an epoch. The epoch is split into slots, where each slot has a duration of T PoH ticks.

一个领导者在其时段内传输条目。在T刻度之后，所有验证者都切换到下一个预定的领导者。验证者必须忽略在领导者分配的时段之外发送的条目。
A leader transmits entries during its slot. After T ticks, all the validators switch to the next scheduled leader. Validators must ignore entries sent outside a leader's assigned slot.

下一个领导者必须观察到所有的Ticks，以便它建立自己的条目。如果没有观察到条目（领头羊倒下了）或者条目无效（领头羊有问题或者是恶意的），下一个领头羊必须产生ticks来填补上一个领头羊的槽。请注意，下一个领导者应该平行地进行修复请求，并推迟发送ticks，直到它确信其他验证者也未能观察到前一个领导者的条目。如果一个领导者错误地建立在自己的小数点上，紧随其后的领导者必须替换其所有小数点。
All T ticks must be observed by the next leader for it to build its own entries on. If entries are not observed (leader is down) or entries are invalid (leader is buggy or malicious), the next leader must produce ticks to fill the previous leader's slot. Note that the next leader should do repair requests in parallel, and postpone sending ticks until it is confident other validators also failed to observe the previous leader's entries. If a leader incorrectly builds on its own ticks, the leader following it must replace all its ticks.

## 分叉生成
## Fork Generation
本节描述了作为领导者轮换的结果, 分叉是如何自然发生的。
This section describes how forks naturally occur as a consequence of leader rotation.

### 概述
### Overview

节点轮流担任领导者并生成编码状态变化的PoH。集群可以通过合成领导者在连接的情况下会产生的东西来容忍与任何领导者的失联，但不摄取任何状态变化。因此，分叉的可能数量被限制在一个 "有/无 "分叉的跳过列表中，这些分叉可能出现在领导者旋转槽的边界。在任何给定的时段，只有一个领导者的事务会被接受。
Nodes take turns being leader and generating the PoH that encodes state changes. The cluster can tolerate loss of connection to any leader by synthesizing what the leader would have generated had it been connected but not ingesting any state changes. The possible number of forks is thereby limited to a "there/not-there" skip list of forks that may arise on leader rotation slot boundaries. At any given slot, only a single leader's transactions will be accepted.

### 信息流
### Message Flow
1. 交易由当前领导者接收。
2. 领导者过滤有效的交易。
3. 领导者执行有效的交易，更新其状态。
4. 领导者根据其当前的PoH槽将交易打包成条目。
5. 领导者将条目传送给验证者节点（以签名的碎片形式）。
    1. PoH流包括ticks,空条目，表示领导者的有效性和集群上的时间流逝。
    2. 领导者的数据流从完成PoH所需的tick条目开始，回到领导者最近观察到的前一个领导者时段。
6. 验证者向其集合中的对等体和进一步的下游节点重新传送条目。
7. 验证者验证交易并在其状态上执行。
8. 验证者计算状态的哈希值。
9. 在特定的时间，即特定的PoH tick计数，验证者向领导者发送投票。
    1. 投票是该PoH tick计数时计算出的状态的哈希值的签名。
    2. 投票也是通过gossip协议传播的。
10. 领导者执行投票，与其他交易一样，并将其广播给集群。
11. 验证者观察他们的投票和集群中的所有投票。

英文:
1. Transactions are ingested by the current leader.
2. Leader filters valid transactions.
3. Leader executes valid transactions updating its state.
4. Leader packages transactions into entries based off its current PoH slot.
5. Leader transmits the entries to validator nodes (in signed shreds)
    1. The PoH stream includes ticks; empty entries that indicate liveness of the leader and the passage of time on the cluster.
    2. A leader's stream begins with the tick entries necessary to complete PoH back to the leader's most recently observed prior leader slot.
6. Validators retransmit entries to peers in their set and to further downstream nodes.
7. Validators validate the transactions and execute them on their state.
8. Validators compute the hash of the state.
9. At specific times, i.e. specific PoH tick counts, validators transmit votes to the leader.
    1. Votes are signatures of the hash of the computed state at that PoH tick count.
    2. Votes are also propagated via gossip.
10. Leader executes the votes, the same as any other transaction, and broadcasts them to the cluster.
11. Validators observe their votes and all the votes from the cluster.

### 分区，分叉
### Partitions, Forks#

在对应于投票的PoH tick计数时，会出现分叉。下一个领导者可能没有观察到最后一个投票时段，并可能以生成的虚拟PoH条目开始他们的时段。这些空tick是由集群中的所有节点以集群配置的hash/per/tick Z的速率生成的。
Forks can arise at PoH tick counts that correspond to a vote. The next leader may not have observed the last vote slot and may start their slot with generated virtual PoH entries. These empty ticks are generated by all nodes in the cluster at a cluster-configured rate for hashes/per/tick Z.

在一个投票时段，只有两种可能的PoH版本。含有T刻度和由当前领导者产生的条目的PoH，或者只有刻度的PoH。PoH的 "仅有刻度 "版本可以被认为是一个虚拟账本，集群中的所有节点都可以从上一个时段的最后一个刻度中得出。
There are only two possible versions of the PoH during a voting slot: PoH with T ticks and entries generated by the current leader, or PoH with just ticks. The "just ticks" version of the PoH can be thought of as a virtual ledger, one that all nodes in the cluster can derive from the last tick in the previous slot.

验证者可以忽略其他点的分叉（例如，来自错误的领导者），或者砍掉负责分叉的领导者。
Validators can ignore forks at other points (e.g. from the wrong leader), or slash the leader responsible for the fork.

验证者基于贪婪的选择进行投票，以最大化他们在Tower BFT中描述的奖励。
Validators vote based on a greedy choice to maximize their reward described in Tower BFT.

### 管理分叉
### Managing Forks
账本被允许在槽的边界处分叉。由此产生的数据结构形成一棵树，称为blockstore。当验证器解释区块存储时，它必须为链上的每个分叉维持状态。我们称每个实例为活动分叉。验证器的责任是权衡这些分叉，这样它就可以最终选择一个分叉。
The ledger is permitted to fork at slot boundaries. The resulting data structure forms a tree called a blockstore. When the validator interprets the blockstore, it must maintain state for each fork in the chain. We call each instance an active fork. It is the responsibility of a validator to weigh those forks, such that it may eventually select a fork.

验证者通过向该分叉的槽位领导者提交投票来选择一个分叉。该投票使验证者在一定时间内承诺，称为锁定期。在锁定期结束之前，验证人不允许在不同的分叉上投票。在同一分叉上的每一次后续投票都会使锁定期的长度增加一倍。在某个集群配置的投票数（目前是32）之后，锁定期的长度达到所谓的最大锁定。在达到最大锁定期之前，验证者可以选择等待，直到锁定期结束，然后在另一个分叉上投票。当它在另一个分叉上投票时，它执行一个叫做回滚的操作，即状态在时间上回滚到一个共享的检查点，然后向前跳到它刚刚投票的分叉的顶端。一个分叉可以回滚的最大距离被称为回滚深度。回滚深度是实现最大锁定所需的投票数量。每当验证者投票时，任何超出回滚深度的检查点都变得无法到达。也就是说，不存在验证器需要回滚超过回滚深度的情况。因此，它可以安全地修剪无法到达的分叉，并将超出回滚深度的所有检查点压入根检查点。
A validator selects a fork by submiting a vote to a slot leader on that fork. The vote commits the validator for a duration of time called a lockout period. The validator is not permitted to vote on a different fork until that lockout period expires. Each subsequent vote on the same fork doubles the length of the lockout period. After some cluster-configured number of votes (currently 32), the length of the lockout period reaches what's called max lockout. Until the max lockout is reached, the validator has the option to wait until the lockout period is over and then vote on another fork. When it votes on another fork, it performs an operation called rollback, whereby the state rolls back in time to a shared checkpoint and then jumps forward to the tip of the fork that it just voted on. The maximum distance that a fork may roll back is called the rollback depth. Rollback depth is the number of votes required to achieve max lockout. Whenever a validator votes, any checkpoints beyond the rollback depth become unreachable. That is, there is no scenario in which the validator will need to roll back beyond rollback depth. It therefore may safely prune unreachable forks and squash all checkpoints beyond rollback depth into the root checkpoint.

## 安全投票签名
## Secure Vote Signing

一个验证者从当前的领导者那里接收条目，并提交确认这些条目有效的投票。这种投票的提交带来了安全挑战，因为违反共识规则的伪造投票可以用来削减验证者的股份。
A validator receives entries from the current leader and submits votes confirming those entries are valid. This vote submission presents a security challenge, because forged votes that violate consensus rules could be used to slash the validator's stake.

验证者通过提交一个交易对其选择的分叉进行投票，该交易使用非对称密钥对其验证工作的结果进行签名。其他实体可以使用验证者的公钥验证这个签名。如果验证者的密钥被用来签署不正确的数据（例如，对分类帐的多个分叉的投票），节点的股权或其资源可能被破坏。
The validator votes on its chosen fork by submitting a transaction that uses an asymmetric key to sign the result of its validation work. Other entities can verify this signature using the validator's public key. If the validator's key is used to sign incorrect data (e.g. votes on multiple forks of the ledger), the node's stake or its resources could be compromised.

### 验证人、投票签名人和利益相关者
### Validators, Vote Signers, and Stakeholders#

当验证者收到同一时段的多个区块时，它会跟踪所有可能的分叉，直到确定一个 "最佳 "的分叉。验证者通过向其提交投票来选择最佳分叉。
When a validator receives multiple blocks for the same slot, it tracks all possible forks until it can determine a "best" one. A validator selects the best fork by submitting a vote to it.

利益相关者是一个对所投资本拥有控制权的身份。利益相关者可以将其股权委托给投票签名者。一旦股权被委托，投票签名者的投票就代表了所有被委托的股权的投票权重，并为所有被委托的股权产生奖励。
A stakeholder is an identity that has control of the staked capital. The stakeholder can delegate its stake to the vote signer. Once a stake is delegated, the vote signer's votes represent the voting weight of all the delegated stakes, and produce rewards for all the delegated stakes.

### 验证人投票
### Validator voting#
一个验证者节点，在启动时，创建一个新的投票账户，并通过流言蜚语在集群中注册。集群中的其他节点将新的验证者列入活动集。随后，验证人在每个投票事件中提交一个用验证人的投票私钥签名的 "新投票 "交易。
A validator node, at startup, creates a new vote account and registers it with the cluster via gossip. The other nodes on the cluster include the new validator in the active set. Subsequently, the validator submits a "new vote" transaction signed with the validator's voting private key on each voting event.

## Stake Delegation and Rewards