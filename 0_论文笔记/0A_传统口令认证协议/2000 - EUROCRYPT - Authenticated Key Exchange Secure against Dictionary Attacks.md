> 2000 - EUROCRYPT - Bellare, Pointcheval, Rogaway
>
> Authenticated Key Exchange Secure against Dictionary Attacks 抵抗字典攻击的安全AKE协议
>
> **BPR ROM模型**

### 翻译记录

#### 摘要

对于基于口令的AKE协议的底层理论处于滞后的阶段，作者提出了一种模型来处理口令猜测、前向安全性、服务器故障和会话密钥丢失的问题。该模型可以用于定义各种目标，以隐式认证的AKE作为基本目标，另外还有实体认证的目标。

作者以EKE作为基础来证明安全性：证明了两个flow的协议在理想密码模型下的安全性

> 隐式认证：指协议的参与者（可能包括服务器）可以确保除了指定实体外，其他任何实体都不能得到会话密钥

#### 1 Introduction

##### 问题的提出 The Problem

文章研究基于口令的AKE协议，讨论以下场景：

一个客户端$A$和服务器$B$，A有一个口令$pw$，而$B$拥有的是一个与此相关的$key$，两方会参与会话并最终都会拥有一个会话密钥$sk$，除了它们两者别人都不知道。

有一个敌手$Ã$可以进行离线攻击，枚举在字典D中的口令（字典中大概率包括$pw$）

> 在一个定义为*好*的协议中，作者认为一个攻击者击败协议目标的机会取决于它与协议参与者交互的次数，而不是取决于它离线计算的时间。

这个问题首先由EKE协议的提出者提供了解决思路，他们在1992年提出了EKE并给出了不正式的安全分析，引起了研究人员广泛地研究。 因为：**口令猜测攻击时侵入系统常见方式，这个领域可以通过好的密码协议来做**

##### 贡献 Contributions

- 找到一个帮助管理该领域*定义和证明复杂性*的方法
	- 模型基于BR模型，可以在不同的设定中定义认证和密钥交换协议的执行
	- 使用伪码来解释，用来解释合适的合作、会话密钥的新鲜度、AKE安全性的衡量，单方鉴别和双向鉴别
		- session IDs代表的合作
		- 接受和终止会话密钥的含义
		- 对`Test`询问的技术更正
		- 让敌手拥有获取协议执行的能力
		- 提供给敌手破坏能力，引出对于前向安全性的讨论
- 专注于AKE，它比双向鉴别更基础，从实用角度，它更简单而且flow更少。但是当引入口令猜测安全的时候，方法将会更加复杂。
- 在本文的方法下，对于字典攻击的抵抗实际上是敌手优势和资源消耗的问题。本文以定理的形式给出，让协议的安全性定义为要执行多少次计算和要做多少次交互。所以看协议是否足够安全来抵抗字典攻击可以看最大的敌手优势是否会随着交互次数和口令空间大小的比值增长而增长
- 作者证明了EKE2在理想密码模型下是安全的，而且这里需要前向安全性

##### 在做的工作

- 客户端持有$pw$服务器持有$f(pw)$，一个单向函数
- $H(pw)$是一个随机预言，算术是基于群来做的

#### 2 模型

通过敌手具有的oracle来将主题模型化；通过对于oracle的询问来模型化各种攻击；对于伙伴有定义；通过`Test`询问来要求会话密钥的语义安全性

##### 协议参与者

固定了一个主体的非空集合$ID$。每一个主体要么是客户端要么是服务器，$ID$是一个有限的、不相交的、非空的$Client$和$Server$的集合，其中每个主体$U$通过固定长度的$string$命名，当$U$出现在协议流或者作为函数的参数的时候，以这个$string$来命名

##### Long-Lived Keys

$A∈Client$持有口令$pw_A$，$B∈Server$持有向量$pw_B=<pw_B[A]>_{A∈Client}$，每个客户端都有一个记录，$pw_B[A]$称为`transformed-password`，在一个对称模型的协议中$pw_A=pw_B[A]$，即服务器和客户端拥有相同的$password$。对于非对称的模型，$pw_B[A]$结合$A,B$很难反向计算出$pw_A$。这两者都可能是不好的口令，在本文中，称$pw_A$和$pw_B$为`long-lived keys(LL-keys)`

下图描述了协议是如何运行的，在`Initialization`阶段，$pw_A$和$pw_B$被生成，每个人的$LL-key$是通过运行`LL-key generator, PW`来得到的，即有客户端$A$的口令由$pw_A←PW_A$。

$h$是从$Ω$空间选择 ，即$PW$会基于一个理想的哈希函数来做，不同生成器可以用于不同的设定。

##### Executing the Protocol

协议是一个将字符串变成另一个字符串的概率算法，算法决定了主体的实例将会怎么对环境中的**信号**（敌手发送的信号）做出响应。，由于有`LL-key generator`，$P$可能也取决于$h$。

敌手$Ã$是一个有独特查询能力的概率算法，所有查询的响应都在图一中。

> The protocol is P , the LL-key generator is PW , and the session-key space SK . Probablity space depends on the model of computation.

![image](https://user-images.githubusercontent.com/40269368/148863004-4fd6f427-b5b7-4819-a7b2-d25da199a1fb.png)





### 小标题

- 1 Introduction
	- The Problem
	- Contributions
	- Related Work
	- Ongoing Work
- 2 Model
	- Protocol Participants
	- Long-Lived Keys
	- Executing the Protocol
	- Standard Model
- 3 Definitions
	- Partnering Using SIDs
	- Two Flavors of Freshness
	- Explanation
	- AKE Security (With and Without Forward Secrecy)
	- Authentication
	- Measuring Adversarial Resources
	- Diffie-Hellman Assumption
- 4 Secure AKE: Protocol EKE2
	- Description of EKE2
	- Security Theorem
- 5 Adding Authentication
	- The Transformations
	- Properties
	- Simplifications