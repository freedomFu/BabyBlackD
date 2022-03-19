> 2003 - CCS - Security Proofs for an Efficient Password-Based Key Exchange
>
> - **摘要：**对于基于口令的密钥交换，如何定义真正的安全性仍然是一个开放性的问题，作者通过分析AuthA的方案并基于Diffie-Hellman的难解假设在random-oracle模型和ideal-cipher模型中进行了分析并证明了AuthA及其变体的安全性。
> - **老师评注：**基于BRP2000和Ideal Cipher-CDH概率讲的很好

## 1. Introduction

为了鉴别用户的身份，口令是最常用的攻击。PAKE来协商会话密钥的过程是用户经常使用的，特别是因为口令低熵性。PAKE协议的根本安全目标仍然是抵抗字典攻击，希望敌手只能做一件事情，通过n次与其他实体的主动交互至多能排除n个口令，而被动窃听对敌手无用，即离线猜测攻击不会让敌手泄露任何信息。轻量级的鉴别过程也是被需求的。

本文对AuthA协议通过BPR模型进行了安全性验证。

- 将协议实体通过oracle模型化
- 不同类型的攻击通过对oracle的query模型化

AuthA抵抗字典攻击的安全性取决于敌手可以做多少次交互，而不是敌手的算力

**相关工作。**作者提供了一种安全模型，在random-oracle和ideal-cipher模型下证明。Ideal models即random-oracle和ideal-cipher提供了可选的安全结果，这样方案一般基于更弱的计算假设，比如说计算型的Diffie-Hellman问题。

One-Encryption Key-Exchange (OEKE)：只有一个flow是加密的，这种方案是更容易被集成的。

## 2.Model

敌手的能力通过 *queries*来模型化，在此处， *players*不会不遵守协议的规定执行，而 *adversary*不是 *player*，但是敌手可以控制整个网络通信。

**Players.** Server → $S$，client→$U$，参与协议$P$。其中每一个player都可能有几个称为 *oracle* 的 *instance*，可能在不同但同时发生的协议$P$中，*instance*分别为$U^i$和$S^j$。双方共享低熵的秘密$pw$，从大小为 *N* 的 *Password*中获取。

**密钥交换算法。**$KEYEXCH(U^i,S^j)$是双方通过会话密钥$sk$交互的协议。

**Queries.**敌手Ã通过做不同的 *queries*与实体通信

- $Execute(U^i,S^j)$：被动攻击，敌手可以通过窃听访问到协议$P$中$U^i$和$S^j$的通信信息。
- $Reveal(I)$：*instance*$I$会话密钥的误用，只有当 *instance*持有一个会话密钥并将$sk$泄露给Ã才可用。
- $Send(I,m)$：敌手发送消息$m$给$I$，敌手可以得到响应。其中$Send(U^i,Start)$表示初始化密钥交换算法，敌手可以得到客户端应发送给服务器的 *flow*。$q_s$代表敌手自己建立的 *flow*，也代表它尝试的口令数。

**Security Notions.**

- **Freshness.**表示敌手不知道$sk$。如果一个 *instance*是 *Fresh*，则说明 *instance*已经被接受，但它与伙伴的会话没有被 $Reveal$过。
	- *the intuitive fact that a session key is not obviously known to the adversary*
- **The Test-query.** $Test(I)$模型化语义安全性。这个最多被敌手执行一次而且只有当$I$是 *Fresh*时才可用。流程为：翻(private)硬币，如果$b=1$则得到$sk$（从Reveal(I)得到），如果$b=0$，则输出随机值。
- **AKE Security.**$Game^{ake}(Ã,P)$通过从$Password$中找$pw$开始，并给敌手提供硬币、所有的 *oracle*，接下来敌手可以做多项式次 *queries*，游戏最后，敌手会针对$Test(I)$得到的值输出猜测的$b'$。其中 *AKE advantage*表示敌手可以正确猜测$b$的概率，即$Adv^{ake}_{P}(Ã)=2Pr[b=b']-1$，概率空间为所有可能的coin。如果敌手的优势可以忽略不计则认为协议$P$是 *AKE-secure*的。
- **Authentication.**敌手的另一个目标是模仿客户端或服务器。本文主要关注对客户端的单向鉴别，用$Succ^{c-auth}_P(Ã)$代表敌手可以模仿客户端的概率，意味着服务器会接受key但后续不与任何客户端共享。如果该概率是可以忽略不计的则认为协议$P$是 *C-Auth-secure*的。

**Computational Diffie-Hellman Assumption.**$G$是一个有效循环群，阶为$l$bit的素数$q$，$Succ^{cdh}_G(Δ)=Pr_{x,y}[Δ(g^x,g^y)=g^{xy}]≥ε$，如果不存在$(t,ε)$则该问题是难解的，*intractable.*

## 3. One-Encryption Key Exchange

整个协议的运算在一个有限循环群中操作，完成了单向的身份鉴别并得到了最终的会话密钥$sk$。

**语义安全性。**证明了语义安全性，但是在下面的证明中没有考虑前向安全性。所有的安全结果都视为并发执行。





