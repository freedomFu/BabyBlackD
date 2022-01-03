> **基本信息**
>
> 1993 - IEEE Journal on Selected Areas in Communications - Li Gong
>
> Protecting Poorly Chosen Secrets from Guessing Attacks 保护弱口令以防止猜测攻击
>
> **老师批注**：关于**离线字典攻击**的原始攻击EKE

### 总结归纳和应用

这篇文章结构严谨，介绍事情的思路非常值得学习和思考。

#### 研究背景

在安全系统中，用户选择弱口令是难以避免的，如何在使用弱口令（便利性）的情况下保证高安全性是一个热点问题

- 基本的思想是保证攻击者获取的数据是足够不可预测的，攻击者无法进行离线验证它的猜测是否正确

现实世界中，基于用户选择的弱口令的猜测攻击是存在的，包括存储口令的系统和基于口令加密通信网络，三个典型的例子：

- 





### 研究内容与解决问题

#### 研究内容

文章提出了一种同时保证用户方便及高安全性的弱口令解决方法，**基本思想**为：保证攻击者能获取到的数据是不可预测的，这样可以防止攻击者**能够离线验证猜测**是否正确。

- 这里的重点是：获得的数据是不可预测的，这样攻击者将无法迅速判断猜测的口令是不是正确的。

作者测试了猜测攻击的通用形式，举了免疫这种攻击的密码协议的例子并与之对称地提出了检测协议是否有这种漏洞的方法。

#### 提出问题

- 实践中，弱口令如何能保证安全呢？
- 检测猜测攻击的方法是什么？
- 在设计中是如何使用公钥密码体制的？它为什么重要？

### 研究方法（创新点）

以…文章中介绍的协议为基础，提出了几种保护技术及能对抗口令猜测攻击的协议，并且提出了一种检测协议是否有这种猜测攻击漏洞的方法。



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

`poorly choosen password`容易遭受到离线猜测攻击：复制信息并离线实验（比如保留口令的哈希值或使用口令作为加密密钥的加密信息等）。

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

这个部分对于猜测攻击的可能的实际攻击场景进行了描述，包括

- UNIX口令系统
	- 存储对口令和盐值的哈希，以及盐值在`/etc/passwd`中，构造$h(p) = g(p, s)$对$p$进行猜测攻击
	- 通过限制对`/etc/passwd`的访问避免
- SunOS Secure NFS
	- `/etc/publickey`存储着口令加密的私钥，这可能被字典攻击
	- 通过限制访问`/etc/publickey`文件来防止
	- 另外一个用户在不同系统中的口令加密内容如果有关联是可能导致一连串的危害的
- Kerberos鉴别系统
	- 服务器给用户请求的初始响应一般会带有时间戳信息或者服务器名称，$\{t,S,...\}_p$，这可能被猜测攻击$m'$被限制在一个合理的范围内并进行判断

#### 3 Known Plaintext and Verifiable

##### Known Plaintext

如果一个密码分析师可以在拿到一部分密文的情况下预测出部分或所有明文，那么这个消息就保护`known plaintext`。任何可以预测的信息都可能构成`known plaintext`，比如这篇文章即使加密了，也可以知道这篇文章中包含`key`、`password`等

- 攻击者可能拿到了$\{m,n\}_k$，并且已知$m$，它可以猜测$k'$并计算$\{\{m,n\}_k\}_{k'}^-$来看结果中是否包含$m$，如果包含则证明可能猜测是正确的
- 如果是公钥系统，攻击者不能去猜测私钥，但是如果它知道$\{m,n\}_k$，$m$和$k$，这样它可以猜测$n'$，使它匹配$\{m,n\}_k$,如果匹配则证明猜测正确了，因为它解密不能解密出两种结果

##### Verifiable text

它可能是明文、密文或者是一系列计算之后的结果

攻击者可能会重新进行口令猜测，当它重新发现一些有意义的值时，则可能猜测是正确的

$P\{y|x\}$表示当$x$为真时，$y$为真的概率，$t$为门限，对于一个`s`，`v`满足以下条件则可认为是`verifiable text`

- 攻击者可以分辨出`v`
- 存在一个函数$f()$有$v =f(s)$，而且攻击者可以猜测$s'$计算$f(s')$
- $P\{s=s'|f(s)=f(s')\}>=t$

此时可以去思考，攻击者希望这样的攻击时可以以离线方式进行的，$t$是系统的容忍程度，如果$t=1$，攻击者是可以做离线猜测的，而如果$t=1/n$，其中`n`是所有可能是真的的口令的总数，这样猜测攻击就可能不成立， 如果$t$处于两者之间，攻击者仍有可能通过离线攻击来缩小范围，只有当$t=1/n$时，系统才能抵抗这种攻击。



#### 4 Protection Techniques

> - These structural properties generally increase the vulnerability to guessing attacks. When considering the pair as a single message with sufficient redundancy, redundancy is diiicult to access objectively. A message that can be recognized could appear to contain unrecognizable random to another.

本节介绍了一些基础的保护握手会话免受口令猜测攻击的方法，以提出问题、解决问题或提出方案、改进方案的方式进行介绍。（后续整理）



#### 5 Authentication Protocols

> - The protection techniques could be applied to develop practical protocols, such as  authentication protocols, that are resistant to guessing attacks.
> - If the owner of a key is a person rather than a computer, the key would be derived algorithmically from the users’s password.

本节介绍了上述保护技术的应用，比如可以用于一些鉴别协议当中

- 普通的双向鉴别协议
- 减少上述协议的消息数目
- 使用随机数的形式替代时间戳（时间戳不够方便）
- 



### 

### 