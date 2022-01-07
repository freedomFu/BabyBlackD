> **基本信息**
>
> 1997 - S&P - Sarvar Patel
>
> Number Theoretic Attacks On Secure Password Schemes
>
> **摘要**：从数论角度对EKE文章中提到的协议进行口令猜测攻击，并提供了解决方法。但对于RSA版本的EKE，简单的修改可能是不足够的，对于其他提出的EKE相关的协议也做了类似的测试，作者展示了为什么这些协议可能会存在问题以及设计一个好的协议应该具备的条件。
>
> **结论**：对EKE的所有版本提出了数论攻击，对于RSA版本这种攻击是不可避免的。
>
> **老师批注**：针对EKE的几个变形，及离线字典攻击1997

### 文章脉络

#### 









### 翻译记录

#### 1 Introduction

对于离线字典攻击，可以通过隐藏$R$和$P(R)$的关系来避免，如不直接加密$R$而是加密一个复杂的函数$f(R)$，或使用函数$f(P)$和$P$一起加密$R$，但它们仍然可能遭受字典攻击，因为$f$可能是已知的，所以仍然可以去做字典攻击。

#### 2 Encrypted Key Exchange (EKE)

EKE保证一个口令的猜测是无法通过网络中交互的信息验证的，它使用口令加密一个随机生成的公钥，所以尝试解密出来的内容也是随机的，这对于验证口令是无效的。

#### 3 RSA Version

> 当已知解密密钥d即e模(p-1)(q-1)的逆时，可以很快地从密文消息恢复出明文消息。 $ed≡1(mod φ(n))$

文章在这里没有描述用于防止重放攻击的挑战响应机制，而是去假设用户的口令就在攻击者所拥有的字典当中，虽然这不一定是真的，但是根据研究这其实占据了很大一部分。EKE怎么防止字典攻击呢？

- 验证口令猜测值$P'$的方法要么是攻破公钥系统要么攻破会话密钥系统，但是$n$和$R$都是来自足够大的密钥空间，理论上可以防止这样的攻击（可证明安全性理论）
- 如果敌手拥有了一些明文和相应的密文，虽然可能可以得到$e$和$R^e$，但是如果不破解公钥系统，永远无法拿到$R$

**RSA-EKE存在的问题**

- $e$为奇数，所以攻击者可以通过这个特性以近似对数级别来排除一些口令
	- 通过让e随机+1来解决，接收者可以知道，但是攻击者无法分别这是一个偶数还是e
- 将最大值为n的数字拟合到$2^m$的分组中也可能造成信息泄露，因为解密大于$n$也可以被拒绝

**攻击者的类别：根据能力**

- *querying attacker* 最弱的攻击者，它只能假装Alice与Bob发起会话，而Bob的响应会被攻击者用来发现口令，它不能欺骗网络地址也不能窃听会话
- *eavesdropping attacker* 在合法会话中窃听
- *active attacker* 可以拦截包和插入包，比如可以拦截Alice发给Bob的包，与Alice进行会话

> 文章中假设所有的攻击只需要攻击者的能力为`querying attacker`

**对于RSA-EKE的数论攻击**

### 小标题

- 1 Introduction
- 2 EKE
- 3 RSA Version
	- 3.1 Brief Description of RSA
	- 3.2 Description of RSA-EKE
	- 3.3 Subtleties of RSA-EKE
	- 3.4 Classifications of Attackers 攻击者分类
	- 3.5 Number Theoretic Attack On RSA-EKE
	- 3.6 Neutralizing the Attack 中和攻击
	- 3.7 Facts from Number Theory  数论知识
	- 3.8 The Attack on RSA-EKE is Successful
- 4 The Diffie Hellman Version
	- 4.1 Brief Description Hellman Version
	- 4.2 Description of the Diffie Hellman Encrypted Key Exchange(DH-EKE)
	- 4.3 Number Theoretic Attack on DH-EKE
	- 4.4 Attack Avoidance
	- 4.5 Attack on Half-Encrypted DH-EKE
		- 4.5.1 Attack with the First Message Unencrypted
		- 4.5.2 Attack with the Second Message Unencrypted
- 5 The ElGamal Version
	- 5.1 Brief Description of the ElGamal Cryptosystem
	- 5.2 Description of ElGamal-EKE
	- 5.3 Number Theoretic Attack on the ElGamal EKE
	- 5.4 Attack on the Half Encrypted ElGamal-EKE
- 6 Gong-Lomas-Needham-Saltzer Protocols
	- 6.1 Direct Authentication Protocol (RSA Variant)
	- 6.2 Secret Public Key Protocol (RSA Variant)
- 7 Why the Attacks are Possible
- 8 Conclusion