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

### 





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