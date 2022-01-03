> 1993 IEEE Journal on Selected Areas in Communications Li Gong
>
> Protecting Poorly Chosen Secrets from Guessing Attacks 保护弱口令以防止猜测攻击
>
> **老师批注**：关于离线字典攻击的原始攻击EKE

### 总结归纳和应用



### 研究内容与解决问题

#### 研究内容

文章提出了一种同时保证用户方便及高安全性的弱口令解决方法，基本思想为：保证攻击者能获取到的数据是不可预测的，这样可以防止攻击者能够离线验证猜测是否正确。

作者测试了猜测攻击的通用形式，举了免疫这种攻击的密码协议的例子并与之对称地提出了检测协议是否有这种漏洞的方法。

#### 提出问题

- 实践中，弱口令如何能保证安全呢？
- 检测猜测攻击的方法是什么？
- 在设计中是如何使用公钥密码体制的？它为什么重要？

### 研究方法（创新点）



### 文章脉络

#### 小标题结构

- 1 Introduction
- 2 Guessing Attacks
	- 2.1 The UNIX Password System
	- 2.2 SunOS Secure NFS
	- 2.3 The Kerberos Authentication System
- 3 Known Plaintext and Verifiable Text
- 4 Protection Techniques
- 5 Authentication Protocols
	- 5.1 A Mutual Authentication Protocol
	- 5.2 Reducing the Number of Messages
	- 5.3 Using Nonces
	- 5.4 Identification
	- 5.5 Using Secret Public Keys
	- 5.6 Direct Authentication
- 6 Detecting Vulnerability
- 7 Related Work
- 8 Summary and Suggestions for Further Work

#### 1 Introduction

##### 研究背景

`poorly choosen password`容易遭受到攻击：复制信息并离线实验（比如保留口令的哈希值或使用口令作为加密密钥的信息等）。

- 密码猜测攻击通常与存储口令的系统有关，而对于网络中信息受到这种攻击的可能性则经常被忽视。
- 用户选择的弱口令则使这种攻击更容易成功 `poorly chosen`，攻击者猜测成功的概率相对较高
- 从`large key space`随机选取的加密密钥则是`well chosen`，攻击者猜测成功的概率相对较低

###### 实际案例

Kerberos鉴别系统容易受到猜测攻击，它的密钥分发服务器使用用户的口令加密了一个初始的响应包，攻击者可以记录这个包并通过离线猜测口令。攻击者可以轻而易举地验证猜测是否正确，因为这个包的解密结果会产生一个有辨识度的数据：一天的时间和网络服务的名称

`guessing attacks`在可以自动生成猜测并可以被验证时是最有效的：因为这样攻击者可以验证猜测是否正确，而这样的猜测不会产生警报

在Kerberos中，这样的猜测可以离线进行，所以这样的攻击在计算上是可行的，而且错误的猜测尝试也不会被检测到

###### 对策

鼓励用户选择费解且难以猜测的口令，但是这是不好记的，不方便，长口令是类似的。（比如ATM的PIN码只有4~6位，再长的PIN码可能就需要写下来才能记住）

作者之前文章提到了一种在用户使用弱口令的情况下改善安全的方法，`verifiable plaintext`，可证实的明文，它是一种这样的消息：已知明文组成了一个子集，其它的写者无法得知两种明文的差异。

###### 本文工作

在上述的想法中进一步扩展，提出了`verifiable text`的普遍概念，并提供了让猜测攻击失败的技术

- 目标是让攻击者得到的数据足够不可预测，这样可以防止攻击者使用离线验证来看猜测是否正确。
	- 信息包含足够的冗余让目的接收者接受，但对于攻击者做离线字典攻击是不够的
		- 猜测验证的唯一方法就是与系统做交互，比如ATM记会记录猜测，当到达一定次数后会发出警告
- 作者构造了`Needham-Schroeder`风格的鉴别协议，**即所有猜测攻击都必须与鉴别服务器交互也因此可以被服务器检测。**
	- 作者给出了之前的协议样例和几种变形，考虑了实际应用因素，比如消息流量的最小值，使用随机数代替时间戳，使用简单的认证如单向函数而不是密钥分发，在没有可信第三方作为鉴别服务器的时候直接鉴别
- 提出了一种检测协议是否可能遭受到猜测攻击的方法

以上的研究内容可以用于很多用户可能选择弱口令的场景



#### 2 Guessing Attacks

不考虑一些专业的攻击，如切片攻击（密文的重排或处理可能对明文产生可预见的结果）。这里假设，密文的每个bit都取决于明文的所有bit

${m}_k$代表k加密m，而${m}_k^-$代表解密









### 

### 