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



