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

当前提出的基于RSA的PAKE协议都不够安全，其中SNAPI虽然安全，但是它需要$e>n$，实用性较差。作者提出PEKEP，$e$为大素数和小素数都可，可抵抗`e-residue`攻击（针对RSA的离线字典攻击），基于RSA猜想和随机预言模型（ROM）对PEKEP进行了形式化安全分析，且提出了一种计算性能更高的协议CEKEP。这两种都无需共享公共参数，只需要提前共享口令即可，在应用上更具有吸引力。

### 原文摘抄

#### Abstract

> - In fact, many of the proposed protocols for password-authenticated key exchange based on RSA have been shown to be insecure; the only one that remains secure is the SNAPI protocol. **the SNAPI protoocl has to use a prime public exponent *e* larger than the RSA modulus n.**  实践应用性较差

#### Introduction

> - The EKE-like protocols do not rely on the existence of a public key infrastructure(PKI). This is desirable in many environments where the deployment of a public key infrastructure is either not possible or would be overly complex.

#### 翻译记录

##### Abstract

作者调研了基于RSA的PAKE，目前大部分PAKE协议都是基于Diffie-Hellman协议，而使用RSA或其他公钥算法设计PAKE协议比较困难，而且当前基于RSA的PAKE协议大部分都不够安全，只有SNAPI还是安全的，但是它需要$e$比$n$大很多，不适合实际应用。本文提出了一种新的PAKE协议，即PEKEP，对$e$的大小没有限制，而且从数论方面，它可以抵抗离线字典攻击中的`e-residue`攻击（e次剩余攻击）。作者还基于RSA猜想和ROM对PEKEP进行了安全分析，另外还基于此提升了安全效率。

##### 1 Introduciton

一般密钥交换协议都需要实体之间共享一个从较大密钥空间获得的密钥数据，这样攻击者无论离线还是在线都很难枚举。然而在真实世界中，这个密钥却被一个容易记住的口令代替了，这样攻击者就可以采用猜测或字典攻击来获取口令。使用口令来实现安全鉴别似乎是自相矛盾的，但92年的EKE证明这是可行的，使用公钥和对称密码体制结合使敌手无法获取成功验证猜测的方法。

EKE不依赖于PKI，保证了自己的简单性和便利性，它与`Diffie-Hellman key exchange`工作最好，而基于RSA协议却很多都被证明不安全，只有SNAPI仍然安全，但是它需要使用一个大素数$e$，比模数$n$还要大， 这让SNAPI协议变得实践性较差，公钥加密算法和*大的指数的素数测试*会引入大量运算。

###### 1.1 Overview of Our Results

作者提出了新的基于RSA的PAKE，`PEKEP`

- 包含两个实体Alice和Bob，共享一个短口令
- Alice有一对RSA密钥对，$n,e,d$ 其中$ed≡1(mod φ(n))$
	- Alice可以选择大或小的素数作为$e$，Bob无需测试A选择e是否与$φ(n)$互素的正确性，无需对$e$进行素数测试
- 从数论角度，PEKEP可以抵抗e次剩余攻击`e-residue attack`
- 基于RSA假设和ROM对PEKEP做了形式化安全分析

**→** 为了减小实体的计算负载，作者提出了计算上高效的KE协议，即`CEKEP`

- 以PEKEP为基础增加了两个额外的流程，让攻击者可以成功进行`e-residue`攻击的概率下降到ε（$0<ε≤2^{-80}$），ε是Bob选择的
- 在这种情况下，每个模幂运算的复杂度为$O(log_2ε^{-1})$，大大减小了EKE的幂运算(160bit)，如果引入cache可以降低更多。

`PEKEP`和`CEKEP`只需要预先共享一个口令即可，不需要引入PKI体系也不需要提前共享其他的公共参数，这是非常实用的。

###### 1.2 相关工作

- BM提出的EKE是第一个PAKE协议，不需要实体知道另一个实体的公钥，EKE之后，许多学者提出了一些改进和扩展。
	- 在初始版本中，他们提出了EKE与公钥算法的可行版本，RSA、ElGamal和Diffie-Hellman
	- RSA的问题是B未知A公钥$(n,e)$，无法验证$e$是否与$φ(n)$互素，可能导致`e-residue`攻击。学者针对基于RSA的EKE做了新的研究，OKE、SNAPI等
		- SNAPI强制让e是一个比n大的素数，可以保证e与$φ(n)$互素
- 为了避免大的指数$e$，学者提出了使用交互协议，但是交互可能会引入一个大的会话开销

##### 2 预备知识

![image](https://user-images.githubusercontent.com/40269368/148057452-cad51ca2-ba88-4621-a2cc-ef4db0c5ab63.png)

最后一条⑦是说的e次剩余的事情

##### 3 Security Model 安全模型

**应用场景：**两个实体使用`password`进行`PAKE`，两个实体A和B共享一个来自small password space $D$的`secret password`，基于口令，A和B可以鉴别对方并建立一个安全的会话密钥（`session key`）

**敌手能力：**敌手attacker可以进行主动攻击，目标是击破该协议的安全目标

- 对A和B的通信有全部控制权，可以乱序发送信息并给其他人发送信息，编造自己选择的信息，也可以离线枚举所有在口令空间的口令；它甚至可以得到`session key`
- 文献2 Authenticated key exchange secure against dictionary attack

**敌手操作与安全性定义**

![image](https://user-images.githubusercontent.com/40269368/148065595-748a8c6d-fefd-4c0e-976b-e6f241476c6b.png)

![image](https://user-images.githubusercontent.com/40269368/148076013-6b3a5c58-487c-4cde-b6aa-afc81c4e992e.png)

##### 4 Password Enabled Key Exchange Protocol

- 介绍基本的PEKEP
	- 等式有解和无解意味着什么？
- PEKEP是如何防御`e-residue attack`的？

![image](https://user-images.githubusercontent.com/40269368/148143358-c0e068e9-a51e-4c72-99f2-93637bab3547.png)

![image-20220105102858112](/Users/mac/Library/Application Support/typora-user-images/image-20220105102858112.png)

###### 定理1



先假设已经证明完毕

##### 5 Computationally-Efficient Key Exchange Protocol 提高计算效率





##### 附录A 定理3 的证明

作者定义了一系列混合实验，在每个实验中，更改了会话密钥的选取方式

###### 混合实验$P_0$







---

> 2004 - ASIACRYPT - Muxiang Zhang
>
> New Approaches to Password Authenticated Key Exchange based on RSA

---

老师好，我是付彦铎。

我认为这篇文章的主要问题包括：

在内容上：

​	本文设计的PEKEP协议和CEKEP协议都利用了数论技术来使协议不会遭受到e次剩余攻击。PEKEP中，Bob实体需要对信息进行$m+1$次加密，这让Bob的运算负担过大，在一些资源受限的场合中，可能是不适用的；在CEKEP中，作者在协议中增加了两个Flow，使整体协议达到了6个Flow，并不适用。另外，作者在背景中提到，SNAPI的问题包括素数测试和公钥加密两个部分，本文实际上只解决了素数测试运算大的问题，而另外一点并没有解决。

在格式上，一方面，作者在摘要和引言中先提到了对于协议的形式化安全性证明，后面才提到更高运算效率的CEKEP，而在正文中，这两点颠倒了；另一方面，在数学论证中有一些数学验证的说明并不完整。

老师好，我是付彦铎。

我认为这篇文章（2004 New Approaches to Password Authenticated Key Exchange based on RSA）的主要问题包括：

在内容上：

​         本文设计的PEKEP和CEKEP都利用了数论技术来保证协议不会遭受到e次剩余攻击，尽管能够抵抗e次剩余攻击，但是：

​        在PEKEP中，实体Bob需要进行m+1次加密操作，Bob的运算负担比较大，在一些资源受限的场合中，该协议可能是不适用的；另外，作者在背景中提到，SNAPI协议的性能问题包括素数测试和公钥加密两个部分，PEKEP实际上只解决了素数测试运算负担大的问题，而另外一点没有解决。

​        在CEKEP中，作者为了提高运算效率，在协议中增加了两个Flow，使整体协议达到了6个Flow才可以完成会话密钥的交换，交互的轮数同样会影响协议的性能。

在格式上：

​        作者在相关证明的部分，对于一些结论的说明不够完整，可能会产生歧义，另外也存在一些笔误。

​        作者在摘要和引言中均先提到了对于协议的形式化安全性证明，后面才提到具有更高运算效率的CEKEP，而在正文部分，这两点的顺序刚好颠倒过来。



老师，我还有一点问题：在阅读论文时，论文的问题会包括方案设计上的问题（如方案设计的假设性过强、方案没有完成初期设定的全部目标等）和逻辑上的问题（文章写作逻辑、实验设计思路的逻辑）等。从论文的背景和Motivation出发来看作者最终的结论以及在论证过程中的逻辑性是否是我在阅读或做笔记时最应该关注的呢？



谢谢老师！



付彦铎

---

**总结：**在基于RSA的PAKE协议安全性普遍较弱，安全的SNAPI协议由于要求$e>n$而实用性较差的背景下，作者提出了PEKEP，该协议允许$e$为小素数，而且可以抵抗$e$次剩余攻击。作者基于RSA假设和ROM模型对协议做出了形式化安全性分析，并进一步提出了具有更高计算效率的CEKEP。这两种协议无需共享公共参数，只需要共享口令即可。

---

**安全模型：**

- 定义实体$A,B∈I$，共享口令$p$，在协议$∏$中进行双向鉴别并生成会话密钥。$∏_A^i$表示实体$A$的第$i$个协议实例（也称为一个oracle），每个已接受的实例持有会话密钥$sk$，会话标识$sid$和伙伴标识$pid$。

- 敌手可以拦截、重组消息，也可以创建实例与其他实体的实例交互，也可以字典攻击，也可以从已接受的实体实例中获取会话密钥。敌手可以发送`oracle query`，类型包括：
	- $Send(A,i,M)$ : 代表中间人攻击，敌手向$∏_A^i$发送$M$，并接受响应，得到实例是否接受以及相对应的$sid$和$pid$
	- $Execute(A,i,B,j)$ : 代表窃听攻击，可以得到$A,B$交互过程中所有的交互信息，但无法获得关于会话密钥和口令的信息。
	- $Reveal(A,i)$ : $∏_A^i$的会话密钥$sk_A^i$发送给敌手
	- $Test(A,i)$ : 不是实际的攻击能力，而是度量会话密钥的语义安全性，会随机生成一个比特位，若$b=1$则返回给敌手真实的会话密钥，$b=0$返回随机会话密钥。该查询只能做一次
	- $Oracle(M)$ : 允许敌手访问函数$h$，即可以访问哈希函数并得到相应的值
- 协议实例的新鲜度
	- 实例已经被接受
	- $Reveal$查询没有发送给这个实例或它的伙伴
- 敌手的攻击目标
	- $Succ$代表$Ã$向一个新鲜的实例做$Test$查询，输出$b'$，而且$b'=b$的事件，敌手$Ã$的优势为$Adv_Ã^{ake}=2Pr(Succ)-1$
	- 如果不做限制，它就可以一直尝试，观察另一个协议的反应来验证猜测，也可以离线进行验证
- 对于一个安全协议，希望敌手一次只能测试一个口令，以$Send$作为在线猜测的次数，而且对于每个实例只记录一次$Send$，所以**一个安全的协议**，是对于多项式时间的敌手$Ã$对不同的实例至多做$Q_{send}$次$Send$查询，而且满足以下两个条件
	- 每个oracle call $Execute(A,i,B,j)$都会产生一对实例$∏_A^i$和$∏_B^j$，即这个查询会对已经接受的实例查询，它们一定有相同的会话密钥并互为伙伴
	- $Adv_Ã^{ake}≤Q_{send}/|D|+ε$，其中$|D|$为口令空间，这个表示敌手的攻击能力只与它能在线交互的次数相关。

---

**PEKEP** 基于RSA的PAKE协议，允许使用大的$e$和小的$e$，$A,B∈I，口令ω∈D，A生成n,e,d，n为大正数，以两个相同size的素数产生，e为正数与φ(n)互素，ed≡1(mod φ(n))$，加解密有$E(x)=x^e(mod(n)),D(x)=x^d(mod(n)),E^m(x)=x^{e^m}(mod(n)),D^m(x)=x^{d^m}(mod(n))$

有哈希函数 $H_1,H_2,H_3:\{0,1\}^*→\{0,1\}^k,H:\{0,1\}^*→Z_n,k为安全参数，如160$，哈希函数产生的数尽可能随机均匀  定义(验证e为奇素数，n为奇数为条件$C_1$)

*协议流程*

1. $A$发送公钥$(n,e)$和随机数$r_A∈_R\{0,1\}^k$给$B$
2. $B$验证条件$C_1$，不成立则拒绝；否则计算$m=└log_en┘$计算$a∈_R Z_n^*,r_B∈\{0,1\}^k$，计算$α=H(ω,r_A,r_B,A,B,n,e)$。检查$gcd(α,n)=1$，不成立，则$λ∈_R Z_n^*$,否则$λ=α$。计算$z=E^m(λE(a))$，发送$z,r_B$给A
3. A计算$α$，并判断$gcd(α,n)=1$，若不成立，则$b∈_R Z_n$，如果$gcd(α,n)=1$，则$z$是$n$的$e^m$次剩余，$b=D(α^{-1}D^m(z))$    $μ=H_1(b,r_A,r_B,A,B,n,e)$  发送$μ$给B
4. B计算$μ=H_1(a,r_A,r_B,A,B,n,e)$是否成立，如果不成立就拒绝，否则计算$η=H_2(a,r_A,r_B,A,B,n,e)$，$sk=H_3(a,r_A,r_B,A,B,n,e)$  发送$η$给A
5. A计算 $η=H_2(b,r_A,r_B,A,B,n,e)$看是否成立，不成立就拒绝，计算$sk=H_3(a,r_A,r_B,A,B,n,e)$，即完成相互认证并生成了会话密钥

> 上述协议流程，当$gcd(α,n)=1$不成立时会拒绝，但是为了不泄露信息，PEKEP会使用随机数进行后续流程（当n=pq，如果这两个值足够大且size相近时，这种情况几乎不存在，可以建立会话密钥）

但是上述流程中，B没有检查$e$与$φ(n)$是否互素，可能导致$e$次剩余攻击，敌手可以进行逐步猜测并从口令空间中一次排除多个口令：

- 从$D$中选择口令$π$,
- 计算$α=H(π,r_E,r_B,A,B,n,e)$
- 如果$gcd(α,n)=1$，可以验证$(αx^e)^{e^m}≡z(mod(n))$在$Z_n^*$上有解，如果有，则无法排除不符合的口令，如果无解则可以排除这个口令，如果一次可以排除一个以上口令，则敌手是可能成功的。

*定理1证明了PEKEP是可以防止e次剩余攻击的*，在$n>1,n=p_1^{a_1}p_2^{a_2}...p_r^{a_r},m为非负整数,e为奇素数，且e^{m+1}不整除φ(p_i^{a^i})$，如果$z是n的e^m次剩余，则对于任意λ∈Z_n^*$，等式$(λx^e)^{e^m}≡z(mod(n))$在$Z_n^*$有解

$n_i=p_i^{a_i}$只需要证明$(λx^e)^{e^m}≡z(mod(n))$







