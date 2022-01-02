### 文章内容介绍

> 1992 - IEEE Symposium on Research in Security and Privacy - Encrypted Key Exchange: Password-Based Protocols Secure Against Dictionary Attacks
>
> **摘要**：传统的基于用户密钥的协议让攻击者可以发起口令猜测攻击，本文引入一种公钥`public-key`和对称密码算法`secret-key`的结合体。两个实体可以共享口令进而在不安全的网络中交换机密且认证过的信息。该协议可以对抗主动攻击，而且它的`password`可以对抗离线字典攻击。另外，该协议还可以用于其他用途
>
> **老师批注**：讲述一类抗字典攻击的口令协议 EKE

### 总结、评价与应用

1. 该文献研究了什么？

该文章引入了一种基于口令的公钥密码算法和对称密码算法结合的密钥交换算法，EKE，还可以保护用户选择的口令不会遭受字典攻击，即希望可以通过用户选择的弱口令来保护用户，它可以用于很多场景……

2. 创新点在哪？

本文使用一种反常态的方法介绍了一种基于弱的口令的密钥交换的协议，这一点就是很创新的。另外，作者基于该协议，给出了RSA和ECC的实现，以及基于指数运算的实现，并对于安全性进行了分析，也给出了一些使用建议等，



3. 研究方法是什么？

可以流程图化



4. 得出的结论是什么？

EKE的出现很重要，而且很必要

### 文章背景与主要贡献

#### 研究背景

人们通常会选择不够好的口令，而本文提出的协议是可以在资源 *只受到了弱口令的保护* 时也能提供合理安全性的，它使用一种对称与非对称密码体制结合的方法，其中对称密码体制的`secret-key`用于加密一个 *随机生成的*公钥。两个实体共享一个`secret`如`password`来交换认证和机密的信息，比如`session key`和`ticket`。该协议是EKE (Encrypted Key Exchange)协议，可以保护口令防止离线字典攻击。

EKE可以与多种非对称密码体制结合使用，特别是指数运算的密钥交换（exponential key exchange）

- 第二节 描述RSA和ECC的非对称密码体制，它们都存在不同类型的问题
- 第三节 概括EKE，描述了公钥分发系统的使用方法
- 第四节 描述了选择公钥和对称密码体制的一般性问题
- 第五节 描述EKE的应用
- 第六节 讨论相关工作

> - symmetric 表示常规的密码系统，密钥使用secret key
>- asymmetric 表示公钥密码系统，密钥区分为public key 和 private key

#### 传统密钥协商以及它可能带来的问题

$A$和$B$共享一个`secret` $P$，目的是建立一个安全的`session key`

-   $A$生成一个随机密钥$R$，使用$P$进行加密并发送$P(R)$给B
	-   $P(R)$是$P$加密了$R$得到的结果
-   $A$和$B$共享$R$，可以把它作为`session key`，$B$可能给$A$回复`R(Terminal type)`
-   窃听者可能会记录这些信息，针对$P$使用字典攻击，并且记录成功解密$P(R)$的$P'$，并进而计算出候选的`session key`，即$R' = P'^{-1}(P(R))$并解密得到`R(Terminal type)`，可以继续测试结果（存在冗余性，但可以一直查找直到可以找到有意义的结果）

上述的协议具有一些其他的缺陷，比如重放攻击。但是，对于这样的密钥交换协议，实际上有一个共同的缺陷：

-   一个**持久化的加密密钥**是容易遭受到*离线的、暴力的*攻击的，如果这个`secret`是一个长的随机串可能还好。但是很多时候，如果`secret`是用户选择的弱口令的话，就很容易出问题

### 文章最终结论

文章作者提出了一种反直觉的概念（使用`secret key`加密一个`public key`），这种思想可以用于许多应用。作者的出发点（目标）是使用弱口令保护用户。当前的智能卡设备、便携鉴别器和零知识证明普遍应用的环境中，担心弱口令仿佛是没有必要的，然而`weak passwords are fact of life.` 通过句法的方式来增强用户口令不是很成功，对于口令生态的审计发现仍然有很多弱口令，一些UNIX系统会使用单向函数来保存，并对口令文件添加写保护。但这样的方法无法保护网络系统，只有EKE这样的协议才可以。

### 思想脉络

#### 文章结构

- 1 Introduction
	- 1.1 Notation
	- 1.2 Classical key negotiation
- 2 EKE using public keys
	- 2.1 A complete protocol
	- 2.2 When to encrypt with the password
	- 2.3 Implementing EKE with RSA
	- 2.4 Using the ElGamla asymmetric cryptosystem
	- 2.5 Security considerations
		- 2.5.1 **Partition attacks**
		- 2.5.2 Tacit assumptions
		- 2.5.3 Strengthening EKE against cryptanalytic attacks
- 3 EKE using exponential key exchange
	- 3.1 A complete protocol
	- 3.2 Choosing α and β
- 4 The encryption layers
	- 4.1 Selecting symmetric cryptosystems
	- 4.2 Selecting an asymmetric cryptosystem
- 5 Applications
- 6 Related work
- 7 Conclusions

#### 思想脉络

##### 2 EKE using public keys

![image](https://user-images.githubusercontent.com/40269368/147842348-60ef1452-9d4e-4a7b-9ac9-8d313ed2a956.png)

- 对于最后一点的理解，即使我拿到了一个猜想的$P'$，算出来了一个$E_A'$，必须对$E_A'$做进一步破解才能知道$P'$对不对，因为$E_A'$是非对称加密，我们也只能通过用$E_A'$加密来观察是否匹配，这个大的密钥空间让工作很难

上图为文中的协议，这里我们应该关注的除了协议本身之外，还有$E_A$和$D_A$的生成过程，以及$P$扮演的角色

###### 2.1 A complete protocol

实际的协议比上述协议要复杂，比如重放攻击，控制了通信通道的攻击者可以插入旧的消息，让用户一直使用旧的会话密钥，一般情况下协议需要增加一些防护，比如增加随机数或者时间戳

![image](https://user-images.githubusercontent.com/40269368/147842472-ed2a28ef-1169-43a3-8075-e03529d916a1.png)

> **challenge-response** 是常见的验证密钥的方式
>
> - 一个实体发送了$R(c)$，其中$c$之前从来没用过，然后收到了另一个包含$c$的密文，则可以认为这个消息的来源具有使用$R$加密消息的能力
>
> 随机数机制可以采用时间戳来代替，但是需要时间是无变化的且是同步的

###### 2.2 When to encrypt with the password

上述协议中，`password`使用了两次，*RPK1, 2*。通常，两个加密可以省略一个，但是省略哪个通常与选择的公钥密码体制有关

- *RPK1* $A, P(E_A)$
- *RPK2* $P(E_A(R))$

> Which one  can be skipped will vary, depending on the particular asymmetric cryptosystem chosen.

口令`password`的使用有一个比较大的约束：它加密的消息必须尽可能得随机 `The message to be encrypted by the password must be indistinguishable from a random number`  比如：

- 一些密码体制需要素数作为公钥，则 *RPK1*不能加密，攻击者会发现对于$P$的猜测验证变得简单，因为它可以通过验证解密结果是否为素数来看
- 如果加密（这里应该指公钥加密后的）的消息有一些特征，则 *RPK2*也不能加密

> 还有一些其它应该考虑的事情

另外，对于消息的加密也与协议的设计有关，比如以明文传输的实体不能生成第一挑战 `Specifically, the party that transmits in the clear cannot be allowed to generate the first challenge.`

这里的理解是，上述的图中的协议实际上有问题，也就是这里我们不应该让A去先生成 `challenge`，这里是说挑战不应该直接由`R`加密就第一个传输了？？？

- 这里没看懂？后续再看。**？** 
	- 这里的理解应该是，对于这两个消息而言，分别是两个实体产生的，A和B，如果它们中的一个是明文传输，即不用$P$加密，那么这一方不应该首先生成挑战


另外，每一个实体都应该证明自己知道`P`，可以通过让对方读取自己加密的消息，或者通过挑战响应机制

###### 2.3 Implementing EKE using RSA

![image](https://user-images.githubusercontent.com/40269368/147844767-38a17487-90d4-4ad2-867d-9f6c80fd2401.png)

在图中写，另外还有如下问题：

- 计算的细节应该更努力去明确一些

- 倒数第二句是 **来攻击**

- > 签名部分：this is an example of how a slight change in a cryptographic protocol may have profound and unforeseen implications for the security of protocol

###### 2.4 Using the ElGamal asymmetric cryptosystem

ElGamal算法也可以用于EKE，与RSA很不相同，在特殊的场景中，用户只能加密第二条消息而不是第一条。

![image](https://user-images.githubusercontent.com/40269368/147845110-544f49b4-94e9-4fe5-a1b4-98deb373157c.png)

###### 2.5 Security considerations

**Partition attacks** 划分攻击

文中提到了使用P进行的加密一定不可以泄露任何信息，但是密码算法通常使用了数学性质，所以这一点很困难，但这些也会导致很严重的问题

- RSA公钥总是奇数
- 如果不做任何预防措施，攻击者可以排除掉一半的候选的$P'$（偶数）

顺着RSA的思维去思考，实际上，每一次一个不合法的$e'$都会排除不同的$P'$，也就是每一次会话都会把剩余的候选的密钥空间分区成两个近似相同的等份

这是一种对数量级的下降，相对来说几次拦截会话就足够拒绝所有对于$P$的不合法的猜测，这种攻击称为`partition attack`

- 利用了密码算法的数学性质，知道一些属性，这样就可以排除很多不符合这个属性的值

对于一些密码系统，人们可能选择接受一个最小化的分组，比如一个场景下，必须要用$P$加密模一个素数$p$的整数，如果这个数字需要`n`bit，猜测的值就有了一个范围$[p, 2^n - 1]$，这样可以排除一部分，但是如果$p$与$2^n-1$很相近，可能排除不了几个值，但这样就很糟糕。对于任何一个`p`来说，计算出多少次拦截才能分析出口令空间大小是非常有可能的

*第二种* 一个加密时需要很大 分组大小的密码系统，一般这种就需要添加高位的0来补充，这就引入了风险，一般这种会通过添加一些随机数字来做

通常可以通过一种方法解决这些问题。加密一个模`p`的整数，此时理想输入的加密分组大小是`m`bit，其中$2^m > p$，x是$2^m / p$的下界，它也是一个添加到加密分组的理想的分区大小所需要的次数，可选择一个随机的$j∈[0, x-1]$，添加$jp$到输入值中，若输入值$<2^m - xp$，就使用区间$[0,x]$代替

而接收者知道模数，则可以把解密值解密到合适的区间  （**疑问很大**）

**Tacit assumptions** 

EKE的安全性基于几个假设

- 对称和非对称密码系统必须不能泄露任何有用的信息
	- 比如使用$E_A$加密的随机密钥$R$不能泄露 任何$E_A$或$R$的信息



![image](https://user-images.githubusercontent.com/40269368/147846064-3d038711-5bdc-4afe-91a7-0351dfb4191d.png)



**Strengthening EKE against cryptanalytic attacks**

如果密码分析攻击者恢复了部分会话密钥，就为攻击$P$提供了条件，可以直接攻击$E_A$，而且也可以用于验证对$P$的猜测。很少有协议可以阻止这样的攻击，即不具备前向安全性

**解决方法**

在挑战响应中，让A和B生成随机的子密钥$S_A$和$S_B$，它们通过$R$加密传输

![image](https://user-images.githubusercontent.com/40269368/147846107-7fa33fe1-1fcc-46e0-9942-1954c5ed1b39.png)

##### 3 EKE using exponential key exchange

以上的使用被Diffie-Hellman称为`public key distribution system`，在那篇文章中，他们也描述了使用`exponential key exchange`作为公钥分发系统，但这样的方式比公钥系统要简单一些，因为它不提供身份鉴别，所以就容易受到主动的窃听攻击和中间人攻击，但是使用一个`secret key` $P$，这些问题都可以解决

对于传统的DH协议来说，即使$P$被破解了，攻击者仍然无法获取到会话密钥（由于DH协议本身的性质）

###### 3.1 A complete protocol

![image](https://user-images.githubusercontent.com/40269368/147846589-93c062bc-cd6a-4b27-b464-def0639f494a.png)

###### 3.2 Choosing α and β

对于α和β的选择也是对安全和性能的权衡

相对来说比较大的β值作为模是更安全的，而且理想状态下，α最好是$GF(β)$的本原根。

一般β可以为$β=2p + 1$其中$p$为素数，这样的β很多。

**A和B怎么知道使用的α和β呢？如何尽可能少地避免泄露信息**

- 肯定不可以传输$P(β)$
- 一个好的选择是让α和β固定且公开，这样不会有信息泄露的风险也不会受到分区攻击，但这样协议实现的灵活度就降低了，因为所有的实体都使用一样的值
	- 此时，为了保证安全，β应该要尽可能大
	- 这样让幂运算代价很高

对于模的长度的攻击也是可能的，有学者建议1000bit，而且它们假设攻击者能获得指数。但是使用EKE，口令`P`可以用来加密这些值，如果不对$P$进行猜测是无法得到所有可能的值的。

文章的目标是，选择一个足够让猜测攻击昂贵的β即可，比如200bit，这样离散对数问题要求几分钟，已经足够了

**另一个倾向大模数的考虑是：**如果口令被破解了，所有的指数都被攻击者获得了，如果这样的话，过去的会话也会被读取，而如果使用大模数，这些会话还是安全的

对于β大小的需求实际上源于希望组织在$GF(β)$上的离散对数运算，这样的计算通常都需要很大的预算，如果每次使用不同的β，攻击者也没办法建立一个表来存储，所以一个小一些的相对廉价一些的β在使用当中。

- 所以本文提出，α和β在首次会话中都以明文传输，攻击者知道这些也没事
- 最多是攻击者会做`cut-and-paste attack`，但只要B对容易解决的选择做了防护就降低了风险：β真的是素数，而且足够大，而且$β-1$至少有一个大的素数因数，而且α是$GF(β)$的本原根
	- 后两个是有关联的，必须得知$β-1$的因式分解来验证α
	- 如果β是$kp+1$，其中$p$是素数而$k$是一个很小的整数就可以了  **这里需要学习一下本原根之类的知识**

**最近的研究**

建议选择一个包含隐藏陷门的β

确定β之后找本原根就比较简单了

##### 4 The encryption layers

###### 4.1 Selecting symmetric cryptosystems

对称加密算法在EKE中的三个部分使用：（不同需求，但可以使用相同的密码系统）

- 加密最初的非对称密钥交换
	- 信息必须不可以使用ASN.1或其他有标记的数据格式，否则很容易被发动猜测攻击
	- 初始的明文信息中不应该包含任何不随机的填充（为了满足分组大小等），也不应该有错误检测的校验和
		- 对于通信错误的检测应该交给低层级的协议
		- 对于多个分组连接到一起的操作，这个机制没有很重要，因为bit都是随机传输的，攻击者也不能很容易操作，挑战响应机制也能提供必要防护
	- 加密算法可能比较弱，比如使用XOR操作，但也足够了，因为对于公钥的加密实际上提供了身份鉴别，而且公钥是随机生成的，对于加密它的`password`也有保护
		- 如果公钥被泄露，攻击者可能立刻会知道口令，所以仍然推荐强的加密算法
- 交换挑战和响应
	- 不需要很强的密码系统
		- 这里假设攻击者不会发动有用的`cut and paste`操作
			- 比如从`R(challaenge_a, challenge_b)`中剪切出`R(challenge_a)`
			- 在特定的系统中使用标准技术如密码分组串也是可以的
			- 也可以选择使用$R$来生成子密钥$R_A,R_B$，它们只用于单向传输
			- 采用消息类型鉴别和消息鉴别码
				- 但，这可能会带来一定的冗余，面对密码分析攻击是不好的
					- 可以使用单向函数
- 保护随后的应用会话
	- $R$不应泄露任何自己的信息
	- 如果$R$被攻破了，攻击者可能发动口令猜测攻击，
	- 通信保护实体之间任意的会话，所以需要检测对称密码系统是否可以抵抗选择密文攻击，如果怀疑，则可以使用`seperate key exchange`

###### 4.2 Selecting an asymmetric cryptosystem

- 原则上，EKE可以与所有的公钥算法一起使用。但实际上，很多就必须要被排除了。比如大素数作为公钥

- 公钥是否可以被编码成一个随机串，如果不是的话则可能是一个大问题
	- 使用随机种子产生公钥？
		- 昂贵
		- 双方需要消耗时间来生成公私钥
		- 但，攻击者也可以通过检索会话密钥来验证候选密码
	- 对于指数运算也需要大素数生成，也很耗费时间，而且是必须做的
- 作者推荐仔细分析选择什么非对称加密系统，而且作者强加了一个约束，即加密的随机数不能泄露任何信息
	- 对于公开加密系统的讨论很少
	- 如果数字理论攻击真得攻破了EKE的特定的公钥系统呢，即使他们对于其他应用是安全的
- 最后一个警告，有关联的对称和非对称算法是不应该同时使用的，即满足 $P(E_A(R))≠E_{P(E_A)}(R)$
	- 这样会有害，攻击者可以拿第一步的值去加密$R$，虽然这个属性在上述算法中不存在，但它是有可能的

##### 5 Applications

EKE的主要创立动机是鉴别用户，但也有其他用途，比如安全公共电话。

- Public telephones
- Cellular phone
	- `since the PIN is not stored within the phone, it is not possible to retrieve one from a stolen unit`
- Interlock Protocol的替代品
	- 检测主动窃听
- privacy amplifier 隐私放大器
	- 让一些相对较弱对称或非对称密码体制变得安全一些
- 防止口令猜测攻击
	- 如果需要解决一个`exponential key exchange`，时间就会比较长，代价过高

##### 6 相关工作

[21]提出`verifiable plaintext`，即本文中的`random plaintext constraint`，也介绍了为什么能防止口令猜测攻击的协议是需要的

[22]提炼了这个概念

- 文章都提出了如果一个明文分组包含了用户的口令以及随机数，那么使用公钥算法加密的话这个密钥是不会受到口令猜测攻击的，但是仍然需要一些额外的防护手段，如重放攻击，已知明文攻击等