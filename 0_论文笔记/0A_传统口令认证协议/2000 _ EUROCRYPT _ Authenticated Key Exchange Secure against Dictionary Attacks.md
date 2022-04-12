> 2000 - EUROCRYPT - Bellare, Pointcheval, Rogaway
>
> Authenticated Key Exchange Secure against Dictionary Attacks 抵抗字典攻击的安全AKE协议
>
> **BPR ROM模型**
>
> **优点：**
>
> **缺点：**

遗留的问题：



### 翻译记录

#### 摘要

对于基于口令的AKE协议的底层理论处于滞后的阶段，作者提出了一种模型来处理口令猜测、前向安全性、服务器故障和会话密钥丢失的问题。该模型可以用于定义各种目标，以**隐式认证的AKE**作为基本目标，另外还有实体认证的目标。

作者以EKE作为基础来证明安全性：证明了two-flow的EKE协议在ideal-cipher model下的安全性

> 隐式认证：指协议的参与者（可能包括服务器）可以确保除了指定实体外，其他任何实体都不能得到会话密钥

#### 1 Introduction

##### 问题的提出 The Problem

文章研究基于口令的AKE协议，讨论以下场景：

一个客户端$A$和服务器$B$，A有一个口令$pw$，而$B$拥有的是一个与此相关的$key$，两方会参与会话并最终都会拥有一个会话密钥$sk$，除了它们两者别人都不知道。

有一个敌手$Ã$可以进行离线攻击，枚举在字典D中的口令（字典中大概率包括$pw$）

> 在一个定义为*好*的协议中，作者认为一个攻击者击败协议目标的机会取决于它与协议参与者交互的次数，而不是取决于它离线计算的时间。**※**

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
- 在本文的方法下，对于字典攻击的抵抗实际上是敌手优势和资源消耗的问题。本文以定理的形式给出，让协议的安全性定义为要执行多少次计算和要做多少次交互
- **看协议是否足够安全来抵抗字典攻击**可以看最大的敌手优势是否会随着交互次数和口令空间大小的比值增长而增长
- 作者证明了EKE2在Ideal-Cipher Model下是安全的，此处安全性需要实现前向安全性

##### 在做的工作

- 客户端持有$pw$，服务器持有$f(pw)$，一个单向函数
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

敌手$Ã$是一个有独特查询条带的概率算法，所有查询的响应都在图一中。

> The protocol is P , the LL-key generator is PW , and the session-key space SK . Probablity space depends on the model of computation.

在执行过程中，每一个主体可能运行许多实例`instances`，本文将主体$U$的实例$i$称为一个$oracle$，表示为$∏_U^i$。每个主体的实例可能体现为一个运行在机器上的由主体控制的一个过程。

通常来说，客户端实例会生成第一条消息`Flow 1`，服务器会响应第二条`Flow 2`，这个过程通常会继续2~5个flow，知道两个实例都被终止。此时，两个实例都被接受`accept`了，持有一个特殊的`session key(SK)`，`session id(SID)`，`partner id(PID)`。

任何时间点，`oracle`都可能被`accept`，而且会持有会话密钥`sk`，会话id `sid`和一个伙伴id`pid`

假设这些值都是写在一个只写的纸带上，`SK`会话密钥空间是实例想要得到的内容，`SID`可以用于唯一标识会话的ID，`PID`用于标识刚刚与一个实例交互过完成交换密钥（而且这个实例也认同）的实体，即伙伴ID，但是`SID`和`PID`都不是机密的，敌手是可以获得的，但是`SK`肯定是一个秘密，一个客户端实例和一个服务器实例最多只能`accept`一次

> **<u>Remark 1</u>**
>
> 本文使用session IDs来定义伙伴关系
>
> - 1995年采用的`matching conversations`，过多集中于不相关的句法当中
> - 1995年他们又采用了存在的保证伙伴关系的函数，可能存在一些反直觉的函数
>
> 一个显式的SID通常是比较好的，实际上一些协议也使用`SID`，有一些地方`SID`没有被显式声明，但是可以通过其他方式来定义，比如通过对于协议流的连接



> **<u>Remark 2</u>**
>
> 作者强调了`accept`和`terminate`是有区别的，当一个实例`terminate`时，它已经完成了，也就是拥有了它想拥有的东西，而且接下来也不会发送任何信息了。
>
> **一些协议可能是现在想`accept`，随后再`terminate`**
>
> - 这种情况一般出现在，实例认为自己正持有一个好的会话密钥，但是使用它之前，该示例想去确认它想的伙伴真实存在，而且也拥有过相同的`session key`，此时实例可以先`accept`，然后等待一个确认消息来`terminate`。
> - 这个区别看起来是人为的，但是这其实是一个非常好的方法，这是一种`asymmetry-breaking device`，这样可以解决一个问题，因为这样可以让最后一个发消息的人能确认这个消息被收到了✨

**敌手Ã**可以对任何一个实例做`query`，它可以拥有无穷尽的$∏_U^i$预言（$U∈ID,i∈N$）也就是协议的实例，一共有6种`query`，对于这6种的响应都在图一中，接下来解释对于不同`query`的响应所代表的`query`代表的能力

![image](https://user-images.githubusercontent.com/40269368/148863004-4fd6f427-b5b7-4819-a7b2-d25da199a1fb.png)

- $Send(U,i,M)$ 该预言$∏_U^i$，`oracle`会计算要返回的信息并且发送回响应。敌手能看到什么呢？
	- oracle是否`accept`了，以及`SID`和`PID`
	- `oracle`是否被`terminate`了，敌手也能看到
	- 为了进入一个客户端A和服务器B交换的协议当中，敌手应该发送$M=B$给未使用过的实例A。`Send-query`模型化了敌手Ã产生一个实例的可能性，例如这个协议接受Ã制造的通信，以及该协议按照协议规定的方式来响应。
- $Reveal(U,i)$  一个oracle $∏_U^i$已经被`accept`，持有会话密钥`sk`，查询给敌手返回`sk`。这个`query`模型化了丢失会话密钥不应该对其他会话产生影响的想法。其实会话密钥可能各种原因丢失，例如hacking, cryptanalysis以及当会话被破坏的时候按照要求毁坏
- $Corrupt(U,pw)$  敌手获取$pw_U$以及$U$的所有实例的状态，该`query`模型化了通过观察用户类型、安装特洛伊木马等方式来破坏一个主体的可能性。这是一种特定的攻击类型，而且让作者必须要去考虑前向安全性。指向客户端U的$Corrupt$询问可能被用来替换服务器$B$使用的$pw_B[U]$，这也是第二个参数的含义。拥有这样的能力可以让一个不诚实的客户端A尝试在服务器B上安装将转换为$pw_B$的字符串来击破协议
- $Execute(A,i,B,j)$  假设客户端oracle$∏_A^i$和服务器oracle$∏_B^i$从未被使用过，该调用会产生这两个协议执行的诚实结果
	- $Send$查询也提供了一样的功能
	- 本query本质上是为了恰当处理字典攻击，此时攻击者应该可以访问足够多的诚实执行，即许多被动窃听，此时敌手也就不能执行主动修改协议流的操作，假的流可能被审计，而且惩罚措施也很多。
- $Test(U,i)$  如果$∏_U^i$已经`accepted`持有一个session key $sk$，接下来的流程就发生了。一个coin 的面为$b$，如果$b=0$，$sk$就会发送给敌手，而如果是$b=1$，返回的就是随机的`session key`（但是来自的密钥空间仍然是一样的）。这个类型的`query`只是用于衡量敌手攻击的成功与否，但是不与攻击能力相对应。敌手只会执行此`query`一次。
- $Oracle(M)$ 敌手拥有对于函数$h$的`oracle access`，该函数时从概率空间$Ω$中随机选取的，而除了敌手，协议和`LL-key`生成器都可能取决于`h`，而$Ω$的选择决定了我们当前处于`standard model`,`ideal-hash model`和`ideal-cipher`

> <u>**Remark 3**</u>
>
> 针对$U$的`Corrupt query`得到了`LL-key`$pw_U$，以及主体$U$的所有实例的状态，这是一种强破坏模型。另一种弱类型的$Corrupt$只返回主体的$LL-key$，称为弱破坏模型，这种知识把口令剥离出来，而强的则是直接破坏用户的机器。



> **<u>Remark 4</u>**
>
> `Corrupt`的询问不会导致$U$的会话密钥的公开，敌手已经有了通过$Reveal$询问来获得会话密钥的能力，而如果`Corrupt`如果也可以公开，那么前向安全性就不可能了。



> <u>**Remark 5**</u>
>
> $Test$查询不一定是敌手的最后一个`query`

##### standard model, ideal-hash model, ideal-cipher model

概率空间$Ω$，可能性的不同，对应不同的模型

- standard model，$Ω$在标准模型中，所有的概率都集中在一个函数上:对于任何查询m，返回空字符串“的常数函数。因此，在标准模型中，所有提到h的都可以忽略。
- Ideal-hash model (random oracle model)固定了字符串的有限集合，从Ω选随机函数意味着选择一个函数$h$让$\{0,1\}^*$到字符串。其实就是密码学的哈希函数，从分析的角度，可以认为是一个公共的随机函数。
- Ideal-cipher model是定义一个优先级和的$G$，其中$|G|=|C|$，它非常强大，有一些比较好的方法来在实际协议中实例化一个理想密码。

> **<u>Remark 6</u>**
>
> `ideal-cipher`模型比$RO-model$更好，但是目前没有办法直接只用ROM实现ideal-cipher

#### 3 Definitions

除了伙伴关系与之前不同之外，还引入了**前向安全性**作为安全目标。

##### Partnering Using SIDs

对于如下的场景，如何定义两个实例是伙伴呢？ 这是一个严格的概念

> $协议P，敌手Ã，LL-key生成器PW，会话密钥空间SK$，如果两个实例$∏_U^i$和$∏_{U'}^{i'}$都`accept`了，分别持有$(sk,sid,pid)和(sk',sid',pid')$，而且满足以下条件：
>
> - $sid=sid',sk=sk',pid=U',pid'=U$
> - $U∈Client,U'∈Server 或 U∈Server，U'∈Client$
> - 除了这两者之外没有`oracle` accept了这个pid

##### Two Flavors of Freshness

下图中有两种新鲜度的定义

如果敌手已经知道协议中的$SK$，那$∏_U^i$就认为是`unfresh`，下图包含两种，一种是具有前向安全性的，一种不具有。

![image](https://user-images.githubusercontent.com/40269368/149057772-fb77d131-0159-457b-a3d5-c4558b2631f7.png)



#### 4 Secure AKE Protocol EKE2

##### Theorem 1

该定理假设口令空间$Password$是已知的，敌手可以高效地从口令空间中取样。







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