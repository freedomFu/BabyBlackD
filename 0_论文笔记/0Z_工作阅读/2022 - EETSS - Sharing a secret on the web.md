> **概述：**这篇文章基于著名的Shamir共享秘密的门限方案和Pedersen的方案提出了增强传统口令鉴别方案安全性的方法。作者通过利用one time的口令增强安全性，设计了两个协议来完成整个注册和登录的流程，分析了攻击模型并证明了安全性，并设计了原型表明方案设计的可行性，最后提出了基于云的设计方案，鉴别即服务。
>
> 文章的立意是比较好的，但还有一些问题需要解决。
>
> **Strength.**
>
> - 动机和思路是较为清楚的
> - 理论和实际的分析都给出
> - 证明和分析看起来是合理的
>
> **Weakness**
>
> - 一些描述不是很清晰
> 	- 在文章中最好区分好secret和share，它们的含义不同的话就不要混用
> 	- 
> - 整个逻辑感觉很乱
> 	- 2.2中，您提到…，这一点似乎与VSS关系不大。
> - 口令的内容的问题，
> - 第三章留了一个问题但是没有结论。
> - 4.2中，永久存储与one-time有关吗？
> - 4.3中，没看懂
> - Threat model 和check 保持格式一致。
> - 符号的问题。
>
> - 公式的问题。
> 	- 2.3节的公式(5)中，$a^{n^m}=a^{nm}$是错误的，正确的公式为$(a^n)^m=a^{nm}$，您在文中的计算有两步使用到了这个公式，但是最终得到了正确的结果
> 	- 您的公式没有引用，读者可能难以知道这个公式出现的位置，如公式
> - 标题似乎不能突出您做的工作的重点？

还有一点问题，文章是从会议论文修改来的，这里在评阅时是否需要增加与原文章的对比呢？

### 结构

- 1 Introduction
- 2 Preliminary Concepts 初步概念
	- 2.1 Shamir Secret Sharing  Shamir秘密共享
	- 2.2 Verifiable Secret Sharing Verifiable的秘密共享
	- 2.3 Pedersen’s VSS
- 3 SECRET SHARING SCHEMES FOR AUTHENTICATION PURPOSES 用于鉴别目的的秘密共享方案
- 4 FRAMEWORK
	- 4.1 Actors and roles 
	- 4.2 Preliminaries 预备知识
	- 4.3 Sharing phase 共享阶段
	- 4.4 Reconstruction phase 重构阶段
- 5 SECURITY ANALYSIS
	- 5.1 Introduction
	- 5.2 Attack Model
	- 5.3 Attack Model Check
		- Dealer analysis
		- Compromised Shareholders
		- A compromised Service
		- A fake Client
	- 5.4 Fault tolerance analysis
- 6 PROOF OF CONCEPT: A WORKING PROTOTYPE 概念性证明
	- 6.1 The implemented application
	- 6.2 The system infrastructure
	- 6.3 Network infrastructure
	- 6.4 Performance qualitative analysis
- 7 CONCLUSIONS

### 文章脉络

#### 摘要

提出了一种安全的传统口令认证协议。目标是解决口令认证协议的问题。基于共享secret的Shamir的(k,n)门限方案，本文方案中，secret是口令，(k,n)代表：n个基于口令派生的secret(share)，$k≤n$个share是重构口令的必要充分条件。该方案具有信息理论安全性，其中口令是`one-time`的。每个share存储在不同的主机(shareholder)中，敌手需要攻破k个不同的Shareholders才能获得足够的信息重构secret。为了防止Dealer被破坏，作者定义了传统Shamir的变体，即Shamir的横坐标是Dealer和Shareholders未知的，这样当Dealer和Shareholders被破坏，重构也是不可能的。除此之外，作者采用了Pedersen的方法来允许对share的验证。

作者设计了两个协议，允许应用之间的通信，相关的用户可以参与注册和认证阶段。并且分析了几个威胁场景，确认这些都不会破坏协议。另外分析了一个或多个拜占庭服务器的场景并分析了安全性。并且开发了一个原型表明本文的方法是可以正确、有效、有效率地工作。

提出的方案可以解决基于口令的鉴别方案，原型也验证了这个方案，本文提供了第一个可行性研究，可以提供基于云的鉴别即服务。

#### 引言

本文不考虑重放攻击。口令重用的问题导致安全性取决于最弱安全性的应用，敌手对其攻击更容易造成口令泄露。本文希望提供一种类似的口令认证机制，并提供了概念证明。

客户端和服务器都可能被攻破，但是服务器被攻破更困难，但是危害明显更大。主要的目标是避免使用最易受攻击的应用，而是让一堆主机持有口令。即，将口令分为n份，并与n个服务器共享，一个服务器一个部分。只有通过将这些部分组合起来才能重构原始信息。因此敌手必须拿到这些信息才能知道这个secret。作者提出了一种方法，secret通过threshold来管理，基于Shamir secret sharing方案。此时，如果用户名和口令不存储在一个位置，当用户注册自己的信息时，它们会被分散到服务器中。在鉴别时，这些服务器，通过足够的信息交换，就可以组合出足够的信息来验证用户。

#### 预备知识

##### Shamir Secret Sharing

Dealer首先选择$P(x)$，选择n个不同的元素并将($x_i$)分发给不同的Shareholder，此值可以公开，并通过计算$s_i=P(x_i)$将$s_i$分发给不同的Shareholder，最终每个人都有$(x_i,s_i)$。通过拉格朗日插值得到重构公式，将$x=0$可以得到最终的$P(x)$。

##### Verifiable Secret Sharing 

这种方法引入了一个概念，即每个用户可以验证他是否收到了一个valid的share。这种方案可以抵抗两种类型的主动攻击：

- 误导的Dealer将错误的$s_i$发送给一个Shareholder来蓄意破坏重构过程。
- 参与者在重构过程传递错误的share

这种方案需要能够抵抗恶意的Dealer或Shareholders

VSS方案基于Dealer和Shareholders可以发送互相发送`secret messages`，而且在Dealer与其他实体之间存在一个broadcast channel。VSS是非常重要，即使存在一定的错误，Dealer和Shareholder仍然可以工作。

##### Pedersen’s VSS

他的方案作为Shamir方案的上层，通过组合Shamir方案和承诺方案得到的，可以被视为分布式承诺方案。distributed commitment scheme

包含两个规范阶段和检查share的正确性。

而且秘密S是嵌入在$g^{a_0}h^{b_0}$，因此$S$不会被直接揭露，即使有一个计算能力无限的敌手也无法获取信息。

#### 用于认证目标的秘密共享方案

基本思想是将口令分成n份并分发到服务器中。关键点是只有至少$t$个部分才能恢复秘密，且该技术可以与口令哈希和盐值组合。只有敌手控制了t个服务器，它们才能恢复出密钥。Shamir用于分割秘密，Pedersen的工作用于检测share的正确性。本文作者提出了对认证的需求。

在Shamir文章中，Dealer是秘密的拥有者，将其分割成多条并分发给Shareholders。口令认证协议不可能一直隐藏secret，验证者必须拥有secret的先验知识，每种口令鉴别协议都存在这样的问题，这也就导致了许多secret的隐私问题。

口令不可以通过明文传输而是必须经过一定的保护机制的保护，这样即使Dealer被破坏了，敌手也无法获取任何有用的信息。

#### 系统架构

##### Actors and roles 

**Actors.** 

- Client.  登录用户，需要发送必要的信息来注册和登录，也需要进行一些检查保证其他的 *actor*不会欺骗。
- Dealer. 系统的管理者。Dealer从客户端收到了秘密 *S*，把 S 分成多个部分并将其分发给 Shareholders. 在登录阶段，它们会收到client的$S'$，并且通过 share信息构建原始的 $S$来比较。 如果相同，客户端就被认证了。 Dealer需要检查Shareholder Service和Client不会欺骗。
- Shareholders.  类似于containers. 存储着每个client的share，必要时需要把信息传输给Dealer。Shareholders需要检查共享的一致性，也就是可以间接验证Dealer的正确性。
- Service. 逻辑上Service是认证服务实现的主机，需要每种类型的在线功能都应该提供给用户的。

##### Preliminaries 预备知识

secret具有绝对的保密性，一旦被揭露就不再是一个secret。

##### Sharing phase

图2展示了共享阶段的消息交换。

##### Reconstruction phase

图3展示了重构阶段的消息交换。

#### Security Analysis

##### Introduction

展示了本系统可以处理的所有攻击类型，考虑了malicious Dealer, Shareholder, Service and Client. `honest`代表这个参与者遵循了协议且不会修改任何信息，`passive`代表参与者可以窃听一些数据，`dishonest(malicious)`表示参与者可以修改协议。

##### Attack Model

威胁模型，存在很多种类的攻击场景。

这一节就不能仿照下一节的格式写吗？？？

##### Attack Model Check

**Dealer analysis.**



**Compromised Shareholders.**





