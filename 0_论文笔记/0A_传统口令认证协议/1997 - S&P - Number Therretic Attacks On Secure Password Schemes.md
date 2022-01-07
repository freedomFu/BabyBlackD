> **基本信息**
>
> 1997 - S&P - Sarvar Patel
>
> Number Theoretic Attacks On Secure Password Schemes
>
> **摘要**：从**数论角度**对EKE文章中提到的协议进行口令猜测攻击，并提供了解决方法。但对于RSA版本的EKE，简单的修改可能是不足够的，对于其他提出的EKE相关的协议也做了类似的测试，作者展示了为什么这些协议可能会存在问题以及*设计一个好的协议应该具备的条件*。
>
> *<u>EKE promises security against active attacks and dictionary attacks.</u>*
>
> **结论**：对EKE的所有版本提出了数论攻击，对于RSA版本这种攻击是不可避免的，除非彻底修改协议，对于RSA的变体同样可能造成攻击。在Diffie-Hellman和ElGamal版本中，$g$必须是$p$的生成元。作者说明了为什么这样的攻击可以成功，也发现了一些口令协议中没有被验证的假设，以及对于只加密一个消息的EKE的安全性进行了测评，建议都加密，除非是在类型攻击面前。
>
> **老师批注**：针对EKE的几个变形，及离线字典攻击1997





### 文章脉络

#### 

### 翻译记录

#### 1 Introduction

用户选择的弱口令可能遭受到离线字典攻击。对于离线字典攻击，可以通过隐藏$R$和$P(R)$的关系来避免，如不直接加密$R$而是加密一个复杂的函数$f(R)$，或使用函数$f(P)$和$P$一起加密$R$，但它们仍然可能遭受字典攻击，因为$f$可能是已知的，所以仍然可以去做字典攻击。

#### 2 Encrypted Key Exchange (EKE)

EKE保证一个口令的猜测是无法通过网络中交互的信息验证的，它使用口令加密一个随机生成的公钥，所以尝试解密出来的内容也是随机的，这对于验证口令是无效的。

#### 3 RSA Version

##### 数论相关知识

**定理1** $p$是一个素数，且$p-1$有一个因子$d$，当$a^{(p-1)/d}≡1(mod(p))$时，同余等式$x^d≡a(mod(p))$有解，而且$a$是$d$次余数(dth power residue)，反之则不是

**推论1** 模p d次剩余个数为 $(p-1)/d$

定理1有更一般的表示：$p$为素数，$m$为一个整数，$d=gcd(m,p-1)$，则当$a^{(p-1)/d}≡1(mod(p))$时$x^m≡a(mod(p))$有解且$a$是m次剩余否则就不是，定理1是$d=m$

**定理2** $p$和$q$是素数，且$p-1$和$q-1$有因子$d$，$n=pq$，当且仅当$a^{(p-1)/d}≡1(mod(p))$且$a^{(q-1)/d}≡1(mod(q))$成立时，$x^d≡a(mod(n))$有$d$次剩余

**推论2** 根据中国剩余定理，模n的d次剩余的数量为$((p-1)/d)((q-1)/d)$，其中p, q与n互素，在下文的例子中，$d=3$，因此立方剩余的个数为$φ(n)/(3^2)$，每一次有$1/9$可以作为`cubic residues`，那么只有$1/9$的口令可以作为候选者，其他的口令就会被丢弃了

> 当已知解密密钥d即e模(p-1)(q-1)的逆时，可以很快地从密文消息恢复出明文消息。 $ed≡1(mod φ(n))$

文章在这里没有描述用于防止重放攻击的挑战响应机制，而是去假设用户的口令就在攻击者所拥有的字典当中，虽然这不一定是真的，但是根据研究这其实占据了很大一部分。EKE怎么防止字典攻击呢？

- 验证口令猜测值$P'$的方法要么是攻破公钥系统要么攻破会话密钥系统，但是$n$和$R$都是来自足够大的密钥空间，理论上可以防止这样的攻击（可证明安全性理论）
- 如果敌手拥有了一些明文和相应的密文，虽然可能可以得到$e'$和$(R^e)'$，但是如果不破解公钥系统，永远无法拿到$R$的额外信息，也就是仍然需要破坏$R$的空间

**RSA-EKE**

- $e$为奇数，所以攻击者可以通过这个特性以近似*对数级别*来排除一些口令
	- 通过让e随机+1来解决，接收者可以知道，但是攻击者无法分别这是一个偶数还是e
- 将最大值为n的数字拟合到$2^m$的分组中也可能造成信息泄露，因为解密大于$n$也可以被拒绝

> 口令加密的信息有特征，这让离线字典攻击变得容易

**攻击者的类别：根据能力**

- *querying attacker* 最弱的攻击者，它只能假装Alice与Bob发起会话，而Bob的响应会被攻击者用来发现口令
	- 它不能欺骗网络地址也不能窃听会话，只有发起会话接收响应的能力
- *eavesdropping attacker* 在合法会话中窃听
- *active attacker* 可以拦截包和插入包，比如可以拦截Alice发给Bob的包，与Alice进行会话

> 文章中假设所有的攻击只需要攻击者的能力为`querying attacker`

**对于RSA-EKE的数论攻击**

*n以明文传输的问题*

n由大素数p q组成，如果加密n $P(n)$，如果解密的$n'$有小素数因子（比p, q的size小很多），则不可能为n，可以排除$P'$

*攻击者用自己的n替换RSA模数后仍不能发起字典攻击？*

假设一个`querying attacker`，伪装为Alice，自己选择了n和X代表$P(e)$，B可能未知，而得到了$e =p^{-1}(X)$，生成随机R，并使用口令加密发给攻击者，$P(R^e)=P(R^{3k})$，攻击者可以尝试猜测$P'$，查看此值是否为立方余数`cubic residue`，**如果不是则$P'$不正确**，根据文章后面的说明，大概$1/9$的口令解密会产生$mod(n)$的`cubic residue`，剩余部分的$9/10$可以被拒绝，此时攻击者可以以对数级别的速度减小可能口令的空间，最终缩小到一个比较小的范围内

*没有任何方法可以事先知道一个给定的$e$是$e = 3k$的形式*

如果攻击者可以重复上述的场景通过选择不同的随机X得到e，这就是可行的攻击。若$e$没有3作为因式，可以发现最终筛选之后，一直能产生`cubic  residue`的口令可能没有剩余，所以可以知道$e$没有3作为因式（此时应认为口令处于攻击者的字典中）

接下来，攻击者可以使用新的随机的$e$来尝试，直到产生一个口令总是可以产生`cubic residue`（这里我认为是考虑到R可能也是立方的形式，所以需要看是否一直都能产生`cubic residue`）

- 由于3个e中可能有一个e以3作为因子，所以平均查找3次就可以得到一个理想的e，即选择了一个好的$X$

*验证解密出来的$(R^e)'$是否为模n的立方余数，需要做的是什么？*

验证它是否是模p和q的立方余数，假设选择了p和q都有 $(p-1)$和$(q-1)$有3这个因子，则接下来可以看给出$(p-1)/3 mod (n)$是否为1

$(R^e)'^{(p-1)/3} (mod(p)) ≡((R^{k3})^{(p-1)/3})(mod(p))≡R^{k(p-1)}(mod(p))$，根据费曼小定理，我们可以知道最后这个值为1，如果满足这样的形式就证明存在一个立方余数

> *费马小定理*
>
> - 如果p为素数，a是一个不能被p整除的整数，则$a^{p-1}≡1(mod(p))$，再者对于每个整数a都有$a^p≡a(mod(p))$
>
> *伪素数*
>
> - 令b是一个正整数，如果n是一个正合数且$b^{n-1}≡1(mod(n))$，则n称为以b为基数的伪素数

**规避数论攻击**

协议总是可以通过添加一些特殊条件来规避攻击，比如RSA会要求p, q是强的素数

*更改1：合法用户对使用了小因式的e不响应*

**攻击者的对策** 当e只能有大的因式时，攻击者就需要尝试更多的e，但是对于每一个e来说，也会有一个大的口令子集被排除，但是合法用户总是可以自行检查让e足够大以让攻击者不实际，而不能让攻击变得无效的e具有小因式

但是另外一个角度是，用户通过不响应告知攻击者这个$e$具有小的因子，这样攻击者就可以通过猜测P得到e，进而排除一大部分的$P'$，如果$e'$有小因子，则对应的$P'$可以排除

*更改2：合法用户依然会响应带有小因子的$e$，但是使用一个假数替代$P(R^e)$*

**攻击者的对策** 这个更改可能可以阻止query attacker发动攻击，但是窃听者可以窃听到合法的会话来排除口令。

上述的更改规定了合法用户会避免小的因子，窃听者可以收集合法会话中的$P(e)$并尝试解密，如果$P'$解密出的$e$含有小因子，则可以排除，这样也可以得到可能的口令

*更改3：为了避免更改2的窃听者的攻击，在发送e时不泄露任何信息。为了选择e，可以从一个随机数X并检查它是否对应没有小因子的e，尝试直到找到一个没有小因子的e，将X发送给B，B会用相同的方式找到e，这样窃听者无法拿到任何可以验证口令的信息，因为这些数字都是等概率产生的*

**攻击者的对策** 前面的更改的问题可以规避，但是有一个更基础的问题因为有小因子的e是不允许的，攻击者可以得到一个响应，然后去找那些不会产生立方余数的猜测，排除会产生立方余数的项，比如攻击者发送X给B，它会从X中解密出e，e中没有小因子，Bob可以给攻击者发送$P(R^e)$，攻击者可以测试`cubic non-residue`来验证口令（即和上面的方式正好相反）

**对于RSA-EKE的攻击是成功的**

对于更改3，如果得知该值不是d次剩余，即不能有d作为因子，则大约$9/10$的口令仍可能是候选者，这也可能是会导致把口令选择减小到1。无论EKE允不允许小因子，都可能被发动上述的攻击，而只有对协议的彻底更改才能抵抗这样的攻击

伪代码也比较好理解

![a5f870323ea1d264e4089b39431ecf2](https://user-images.githubusercontent.com/40269368/148508065-74ac08ed-b4c5-43ec-a853-cbace243fde8.png)、

注意，当factor因子越大时，每次可以剔除的子集也就越多，但是寻找e的过程也会相应的变长，攻击者可以选择最恰当的因子来让攻击速度最快。

#### 4 The Diffie Hellman Version

素数$p$，本原元素或生成元$g$，$g^{R_A}(mod(p))$，$g^{R_B}(mod(p))$都是公开可知的，对于一个足够大的合适的素数来说，在合理的时间内求出$R_A$和$R_B$属于离散对数的困难问题，DH的问题是缺少身份鉴别。建立会话后通过口令鉴别也不行，因为中间人攻击是仍然可以执行的。EKE可以避免这个问题，因为中间人不知道口令，所以它不能读取和重放信息。

> An eavesdropper cannot validate password guesses because $R_A$ and $R_B$ are random hence $g^{R_A}(mod(p))$ and $g^{R_B}(mod(p))$ will also appear random.
>
> 即使是能猜出来，攻击者也算不出来 $K=g^{R_AR_B}(mod(p))$

**针对DH-EKE的数论攻击**







### 小标题

- 2 EKE
- 3 RSA Version
	- 3.3 Subtleties of RSA-EKE
	- 3.4 Classifications of Attackers 攻击者分类
	- 3.5 Number Theoretic Attack On RSA-EKE
	- 3.6 Neutralizing the Attack 中和攻击
	- 3.7 Facts from Number Theory  数论知识
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