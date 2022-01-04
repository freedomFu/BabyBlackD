> **基本信息**
>
> 2004 - ASIACRYPT - Muxiang Zhang
>
> New Approaches to Password Authenticated Key Exchange based on RSA 基于RSA的口令鉴别密钥交换的新方法
>
> **关键词**：password authentication口令鉴别, off-line dictionary attack 离线字典攻击, public-key cryptography公钥密码学
>
> **老师批注**：ROM证明的描述很好

### 总结归纳应用

#### 研究背景及意义



#### 研究内容



#### 







### 原文摘抄

#### Abstract

> - In fact, many of the proposed protocols for password-authenticated key exchange based on RSA have been shown to be insecure; the only one that remains secure is the SNAPI protocol. **the SNAPI protoocl has to use a prime public exponent *e* larger than the RSA modulus n.** 这个的意义是什么？

#### Introduction

> - The EKE-like protocols do not rely on the existence of a public key infrastructure(PKI). This is desirable in many environments where the deployment of a public key infrastructure is either not possible or would be overly complex.

### 文章脉络

#### 小标题

- 1 Introduction
	- 1.1 Overview of Our Results
	- 1.2 Related Work
- 2 Preliminaries 预备知识
- 3 Security Model
- 4 Password Enabled Key Exchange Protocol ✨
- 5 Computationally-Efficient Key Exchange Protocol✨
- 6 Formal Security Analysis✨
- 7 Conclusion
- A **Proof of Theorem 3**✨

#### 翻译记录

##### Abstract

作者调研了基于RSA的PAKE，目前大部分PAKE协议都是基于Diffie-Hellman协议，而使用RSA或其他公钥算法设计PAKE协议比较困难，而且当前基于RSA的PAKE协议大部分都不够安全，只有SNAPI还是安全的，但是它需要$e$比$n$大很多，这不太实际。本文提出了一种新的PAKE协议，即PEKEP，对$e$的大小没有闲置，而且从数论方面，它可以抵抗离线字典攻击中的`e-residue`攻击（e次剩余攻击）。作者还基于RSA猜想和ROM对PEKEP进行了安全分析，另外还基于此提升了安全效率。

##### 1 Introduciton

一般密钥交换协议都需要实体之间共享一个从较大密钥空间获得的密钥数据，这样攻击者无论离线还是在线都很难枚举。然而在真实世界中，这个密钥却被一个容易记住的口令代替了，这样攻击者就可以采用猜测或字典攻击来获取口令。使用口令来实现安全鉴别似乎是自相矛盾的，92年的EKE证明这是可行的，使用公钥和对称密码体制结合的方法，使敌手无法获取成功验证猜测的方法。

EKE不依赖于PKI，保证了自己的简单性和便利性，它与`Diffie-Hellman key exchange`工作最好，而基于RSA协议却很多都被证明不安全，只有SNAPI仍然安全，但是它需要使用一个大素数$e$，比模数$n$还要大， 这让SNAPI协议变得实践性较差，公钥加密算法和大的指数的素数测试会引入大量运算。

###### 1.1 Overview of Our Results

作者提出了新的基于RSA的PAKE，`PEKEP`

- 包含两个实体Alice和Bob，共享一个短口令
- Alice有一对RSA密钥对，$n,e,d$ 其中$ed≡1(mod φ(n))$
	- Alice可以选择大或小的素数作为$e$，Bob无需测试A选择e是否与$φ(n)$互素的正确性
- 从数论角度，PEKEP可以抵抗e次剩余攻击`e-residue attack`
- 基于RSA假设和ROM对PEKEP做了形式化安全分析

**→** 为了减小实体的计算负载，作者提出了计算上高效的KE协议，即`CEKEP`

- 以PEKEP为基础增加了两个额外的流程，让攻击者可以成功进行`e-residue`攻击的概率下降到ε（$0<ε≤2^{-80}$），ε是Bob选择的
- 在这种情况下，每个模幂运算的复杂度为$O(log_2ε^{-1})$，大大减小了EKE的幂运算(160bit)，如果引入cache可以降低更多。

`PEKEP`和`CEKEP`只需要预先共享一个口令即可，这是非常实用的。

###### 1.2 相关工作

- BM提出的EKE是第一个PAKE协议，不需要实体知道另一个实体的公钥，EKE之后，许多学者提出了一些改进和扩展。
	- 在初始版本中，他们提出了EKE与公钥算法的可行版本，RSA、ElGamal和Diffie-Hellman
	- RSA的问题是B未知A公钥$(n,e)$，无法验证$e$是否与$φ(n)$互素，可能导致`e-residue`攻击。学者针对基于RSA的EKE做了新的研究，OKE、SNAPI等
- 为了避免大的指数$e$，学者提出了使用交互协议，但是交互可能会引入一个大的会话开销

##### 2 预备知识

![image](https://user-images.githubusercontent.com/40269368/148057452-cad51ca2-ba88-4621-a2cc-ef4db0c5ab63.png)

##### 3 Security Model 安全模型

**应用场景：**两个实体使用`password`进行`PAKE`，两个实体A和B共享一个来自small password space $D$的`secret password`，基于口令，A和B可以鉴别对方并建立一个安全的会话密钥（`session key`）

**敌手能力：**敌手attacker可以进行主动攻击，目标是击破该协议的安全目标

- 对A和B的通信有全部控制权，可以乱序发送信息并给其他人发送信息，编造自己选择的信息，也可以离线枚举所有在口令空间的口令；它甚至可以得到`session key`
- 文献2 Authenticated key exchange secure against dictionary attack

**敌手操作与安全性定义**

![image](https://user-images.githubusercontent.com/40269368/148065595-748a8c6d-fefd-4c0e-976b-e6f241476c6b.png)





