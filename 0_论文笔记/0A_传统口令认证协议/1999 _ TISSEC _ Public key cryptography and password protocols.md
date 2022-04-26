> 1999 - ACM Transactions on Information and System Security (TISSEC) /CCS - Public-Key Cryptography and Password Protocols - 抗离线字典攻击需公钥操作 identity protection 在移动远程认证中很重要，最初版本发表在CCS会议中，本版本的内容发行在TOPS期刊中，它的前身是TISSEC
>
> **作者：**Haveil and crawczyk
>
> **关键词：**Dictionary attacks, hand-held certificates, key exchange, passwords, public-key protocols, public passwords
>
> **摘要：**作者描述了在非对称的场景中进行鉴别和密钥交换的协议，其中鉴别服务器持有公私钥对，但客户端只包含一个弱口令作为鉴别密钥。作者分析了几种协议并发现这些协议可以基于标准密码猜想做验证。发现，选择了适合公钥加密算法的协议可以抵抗离线字典攻击。除了用户鉴别，作者提供了让协议提供双向鉴别、认证密钥交换、对服务器破坏的防护和用户匿名性，作者的发现表明公钥技术在对抗离线猜测攻击中是非常重要的。另外，作者提出了`public passwords`的概念，充当`hand-held certificates`的角色。

### 1. Introduction

当前使用用户口令的鉴别方法实际呈现了一种不对称性，即服务器可以储存比较强的secret（比如公私钥对），但是客户端用户只能使用低熵的口令（这里指用户没有存储strong secret的设备）。

但是用户选择的口令的低熵性本身带来了安全问题， 即可能被敌手捕获发动离线字典攻击，作者在Gong的基础上进一步做了基于公钥的方法，从几个方面做了扩展，特别是提出协议的分析和证明。

- 分析了几种协议的安全性，表示它们的安全性基于对公钥加密算法的选择，也表明一些看起来安全的协议实现是存在问题的。作者证明出，最优的对于离线口令猜测攻击的抵抗方式是：敌手最多只能在线猜测攻击来做。
- 描述了如何让这些协议可以提供一些更重要的特性，比如双向认证、认证密钥交换和服务器被破坏和用户匿名性等。
- 当客户端无法获取公钥时，引入了public password的概念
- 提供了通用理论结果，表明对抗离线猜测攻击需要公钥技术