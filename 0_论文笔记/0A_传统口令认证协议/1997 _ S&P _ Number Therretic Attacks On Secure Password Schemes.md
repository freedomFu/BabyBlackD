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

#### 总结

本文从*数论角度*对1992 EKE和1993 confounder 中提到的协议及变形进行了安全性分析，对它们进行了口令猜测攻击并提供了一些防护方法，其中包括RSA-EKE，Diffie-Hellman-EKE，ElGamal-EKE以及GLNS协议的后两种的攻击并给出了一定的解决方法。最终，作者揭示了攻击可行的原因和相对应的建议

#### 前言

##### 离线字典攻击与EKE （用户选择的弱口令可能遭受到离线字典攻击）

针对这样的攻击，一些协议采用隐藏$R$和$P(R)$的关系来避免，如不直接加密$R$而是加密一个复杂的函数$f(R)$，或使用$f(P)$加密$R$，但由于$f()$可能是已知的，因此仍然可能去做字典攻击。

EKE是通过保证对一个口令的猜测的验证无法通过网络中交互的信息来进行，可以通过加密一个随机生成的公钥，这样解密结果也足够随机，这无法验证猜测。会话密钥由公钥和口令加密，除非攻破公钥系统，否则无法猜测

##### 攻击者的分类 （根据攻击者能力）

- *querying attacker* 最弱的攻击者，它只能假装Alice与Bob发起会话，攻击者利用Bob的响应来发现口令
	- 它不能欺骗网络地址也不能窃听会话，只有发起会话接收响应的能力
- *eavesdropping attacker* 在合法会话中窃听
- *active attacker* 可以拦截包和插入包，比如可以拦截Alice发给Bob的包，与Alice进行会话

> 文章中假设所有的攻击只需要攻击者的能力为`querying attacker`
>
> - 另外，根据相关研究表明，正确的口令一般会处于攻击者的字典当中，这在实际中占比很大。

##### 数论相关知识

**定理1** $p$是一个素数，且$d|(p-1)$，当$a^{(p-1)/d}≡1(mod(p))$时，同余等式$x^d≡a(mod(p))$有解，而且$a$是$d$次余数(dth power residue)，反之则不是

**推论1** 模p d次剩余个数为 $(p-1)/d$

定理1有更一般的表示：$p$为素数，$m$为一个整数，$d=gcd(m,p-1)$，则当$a^{(p-1)/d}≡1(mod(p))$时$x^m≡a(mod(p))$有解且$a$是m次剩余否则就不是，定理1是$d=m$

**定理2** $p$和$q$是素数，且$d|(p-1),d|(q-1)$，$n=pq$，当且仅当$a^{(p-1)/d}≡1(mod(p))$且$a^{(q-1)/d}≡1(mod(q))$成立时，$x^d≡a(mod(n))$有$d$次剩余

**推论2** 根据中国剩余定理，模n的d次剩余的数量为$((p-1)/d)((q-1)/d)$，其中p, q与n互素，在下文的例子中，$d=3$，因此立方剩余的个数为$φ(n)/(3^2)$，每一次有$1/9$可以作为`cubic residues`，那么只有$1/9$的口令可以作为候选者，其他的口令就会被丢弃了

#### RSA EKE 

 **RSA EKE如何防止离线字典攻击？** ： 验证口令猜测值的用户需要穷举$n$和$R$的空间来进行，这是不可行的。如果攻击者拥有了一部分明文和相应的密文，它可能能得到$e'$和$(R^e)'$，但如果不攻破$d$，就不会被攻破

**RSA EKE可能存在问题的部分：**

- 划分攻击
	- 公钥$e$一定是奇数，这导致当攻击者窃听了足够多会话后，可以在解密$P'(e)$之后得到偶数的情况剔除掉，这样可以以对数级别降低候选口令空间，最终下降到1个
	- EKE中给出结果是给$e$随机加1，接收者可以移除，但是攻击者不知道一个偶数是$e+1$还好是其他的
- 将最大大小为$n$的数字拟合到$2^m$的分组中也可能泄露信息，因为解密大于$n$是可以被拒绝

> 口令加密的信息有特征，这让离线字典攻击变得容易

- $n$为什么不加密，因为$n$由大素数组成，如果$P(n)$解密得到的数字$n'$有小因子（比p, q的size小很多），可以排除，进而验证猜测。
- 如果攻击者可以恶意替换$n$（对EKE的说法做了修正，EKE说明即使替换也不会有安全问题）
	- 一个`querying attacker`，*伪装成Alice*，自己选择了n和X代表$P(e)$，B可能未知，而得到了$e =p^{-1}(X)$，生成随机R，并使用口令加密发给攻击者$P(R^e) =P(R^{3k})$ （假设已知$e$有一个小因子3，$e=3k$）
	- 攻击者可以尝试猜测$P'$，检查解密得到的值是否为一个立方剩余 cubic residue，根据上述理论，$φ(n)$个数中大概$1/3^2φ(n)$可能产生立方剩余，经过多次尝试，可以将可能口令空间以对数级别下降到1
	- 如果$e$没有3作为小因子呢，可以选取不同的$X$得到$e$，测试这个$X$能不能让口令空间降低为1，如果没有3作为因子，则最后没有剩余的口令可以产生`cubic residue`，此时就知道这个$e$没有3这个因子
	- 接下来可以尝试新的$e$，直到找到一个$e$它可以伴随着口令空间的下降一直产生`cubic residue`，3个随机的$e$中大概有一个有素因子3，所以找到一个合理的e大概需要3次
- 根据定理2，如果想验证$(R^e)'$是模$n$的立方剩余，必须验证它是模$p$和$q$的立方剩余，这样必须保证$3|(p-1),3|(q-1)$，可以验证以下式子根据费马小定理来验证这个是不是立方剩余
	- $(R^e)'^{(p-1)/3} (mod(p)) ≡((R^{k3})^{(p-1)/3})(mod(p))≡R^{k(p-1)}(mod(p))$

> 事实上，每次可以减少$1/9$，如果口令空间100万，则大概6次就可以缩小到1，每次寻找$e$需要3次会话，所以破解口令平均需要18次
>
> - 一个缓解措施为：限制不成功尝试的次数，但可能让用户改口令不好，而且DoS攻击也可以进行

**尝试的解决方法：**

*更改1：合法用户不响应有小因子的e* 当e只能有大的因子时，攻击者需要尝试更多次数来找到合适的$e$，但是对于每个$e$也会有一个大的口令子集被排除，但如果$e$足够大，则攻击很难。另外，如果不接受带有小因子的$e$，攻击者发送$X$代替$P(e)$，如果会话被拒绝了说明$e$有小因子，那么攻击者$P'^{-1}(X)$可以求出$e$，如果有小因子则可以排除（这里是，如果e有小因子，不回应，如果回应，则说明可以，而解出来有小因子的P可以筛除）

*更改2：合法用户响应带有小因子的e,但是使用假数来替代$P(R^e)$，但合法用户使用的e不包含小因子* 窃听者可以窃听合法会话，P(e)，如果$P'$解密出来$e$有小因子，则可以排除

*更改3：从一个随机数X开始检查是否有小因子的e，一直累加并找到那个e，发送X给B，B也会找到e，窃听者无法拿到任何验证口令的信息，因为等概率产生的* 小因子仍然是不允许的，攻击者可以得到响应，去查找那些不会产生立方剩余的数字，如果没有则留下来，有就剔除。（与上述数论攻击刚好相反）

**对于RSA-EKE的攻击是成功的** 根据数论攻击和更改3的攻击，我们可以发现无论规定有没有小因子，都可能导致d 次剩余攻击，尽管下降速度不同，但都是对数级别下降，只有彻底修改协议才有可能避免攻击。当factor因子越大时，每次可以剔除的子集也就越多，但是寻找e的过程也会相应的变长，攻击者可以选择最恰当的因子来让攻击速度最快。

#### Diffie-Hellman EKE

对于一个足够大的素数来说，合理时间求出$R_A$和$R_B$是离散对数困难问题，EKE也可以带来身份认证，避免中间人攻击，攻击者即使能猜出来$g^{R_B}$也无法生成$g^{R_AR_B}$，而且不知道口令，攻击者也无法读和传递消息

**针对DH-EKE的数论攻击**

*有关生成元*

- 模素数$p$一个原根是$Z_p$中的整数$r$，使得$Z_p$中的每个非零元素都是$r$的一个幂次，比如2是模11的原根，而3不是

- 数论中一个重要事实是对于每个素数$p$都存在一个模$p$的原根。假设$p$是一个素数而$r$是一个模$p$的原根，如果整数$a$介于1到$p-1$之间，则存在唯一的指数$e$使$r^e=a$，即有$r^e(mod(p))=a$

- **离散对数问题**：输入是一个素数$p$,一个模$p$的原根$r$和一个正整数$a∈Z_p$，输出以$r$为底$a$模$p$的离散对数，这个问题实际上没有已知多项式算法可以求解

*数论攻击*

querying attacker 伪装成Alice，发送它构造的$g,p$，并用随机$X$替代$P(g^{R_A}(mod(p)))$

B收到后计算$P(g^{R_B}(mod(p)))$给attacker，这是一个足够随机的；**然而，g,p的选择可能会带来问题**

在第一条消息中，攻击者发送$g^d,p$，其中，$d$是一个小素数，而且$d|(p-1)$

B会计算$a≡(g^d)^{R_B}(mod(p))$，即$g^{R_Bd}(mod(p))$，显然，$a$是mod p 的 d次剩余，即有$a^{(p-1)/d}≡1(mod(p))$。更直观地，代入$a$可得$g^{R_Bd((p-1)/d)}(mod(p))$根据费马小定理，值为$1(mod(p))$

攻击者收到$P(g^{R_Bd}(mod(p)))$，通过猜测口令求解，并添加$((p-1)/d)(mod(p))$去计算，如果不为1，则可以排除猜测的口令

根据推论可知，p-1个数字中有 $(p-1)/d$个都可能产生1，也就是每次的会话可能可以减少$1/d$，也可以通过对数级别减少到$1$

*避免攻击*

如果允许攻击者自行选择$g^d$和$p$，这样的攻击无法避免，只能通过限制选择来做。但是实际上$g^d$并不是生成元，它却满足$g^{d(p-1)/d}≡1(mod(p))$，所以它不是生成元

> to find a generator $g$ it is necessary and sufficient to check that $g^{(p-1)/k}not≡1(mod(p))$ for all factors k of $p-1$

本文要求$g$为本原根。测试生成元的方法是使用$p-1$，其中$p-1= kp'$，$p'$是一个大素数，对于合法的k的范围，$p’$都可以找到，通过素数检测可以找到生成元，需要检查p或p’是素数，也需要检查g是p的生成元

**针对半加密的DH-EKE的攻击**

作者建议DH-EKE两条消息都用口令加密，除非挑战是特殊类型的

*第一条不加密*

> challenge是有类型的，一个bit位，0表示A是发送者，1表示B是发送者

querying attacker可以伪装为A，选择$R_A$并发送$g^{R_A}(mod(p))$给$B$，收到后生成$R_B$，求出$K=g^{R_AR_B}(mod(p))$，给A发送$P(g^{R_B}(mod(p)))$和$K(C_B)$

攻击者可以通过$P'$得到$(g^{R_B})'$，自己有$R^A$，可以生成$K'$，如果在$C_B$中有冗余信息，攻击者就可能猜测口令。

比如实际发送的挑战是$K(C_B,1)$，攻击者将会拒绝所有解析出$C_B,0$的口令，大概有一半可以被筛除，也就是以对数级别下降

*第二条不加密*

即只有第一条消息是加密的，active attacker拦截了会话，伪装成Bob

A 发送$P(g^{R_A}(mod(p)))$，攻击者在第二条消息中返回明文$g^{R_B}(mod(p))$，攻击者自己选择了$R_B$，可以通过猜测$P'$得到$g^{R_A}$，得到$K'$，但攻击者可能不知道在挑战响应信息中是否有多余的信息。

攻击者可以用间接的方式，它无法发送一个合法类型的挑战，因为它不知道$K$，所以它只返回一个随机数$X$，如果A接受了消息，就说明$K(X)=C_B',1$，如果拒绝了则为$C_B',0$。可以通过这样的方式拒绝一般的口令，最后达到只剩一个口令

**对策** 这样，A可以通过总是回应并且不要挂起会话等避免在验证$K$的过程中泄露有用的信息

下一步A需要发送$K(C_A,C_B)$给攻击者，A应当如何使用$C_B$，应该解密$X$得到$C_B$还是选择一个随机数。

- 如果$K^{-1}(X)$被使用，攻击者可以尝试所有的从猜测$P'$得到的候选$K'$，它们可以找到$K'^{-1}(X)$和$K'^{-1}(C_B)$是否匹配 接受了
- 如果采用随机数，攻击者可以尝试找到$K'^{-1}(X)$和$K'^{-1}(C_B)$是否匹配，如果都不匹配，攻击者就知道A发送了一个假的$C_B$，因为A发送的$X$解析后应该为$(C_B',0)$而不是$(C_B,1)$，这样又可以过滤一些password   拒绝了

以下是一种解决方法，如果A用特别复杂的类型而不是一个比特，比如A用64个0而B用64个1，这样A可以很快拒绝X，因为$K^{-1}(X)$可以产生$(C_B',111...1111)$是几乎不可能的

#### ElGamal EKE

这种协议已经足够成熟

**针对它的数论攻击**

与DH-EKE相似，也就是需要一个生成元来作g，否则攻击者可以伪装为Alice，并发送X，收到响应$P(g^{kd}(mod(p)),RY^k(mod(p)))$，敌手猜测$P'$，如果正确则前面有指数$(p-1)/d$，为1，如果不是1则排除，这样也可以减小空间

**半加密攻击**

第一条为什么必须加密（第二条可能遭受到类型攻击）

攻击者伪装为A，以明文发送$g^{R_A}(mod(p))$，B响应，攻击者可以构造R’

- 先通过候选口令解密B的消息，得到$g^{k'}$和$(Rg^{R_Ak})'$，攻击者选了$R_A$，可以计算$R'$
- 攻击者向B发送X，如果接受了，说明X用K解密出了$C_A',0$，否则是$C_A',1$，攻击者可以利用这个信息来验证猜测，只有用比较复杂的类型值才能抵抗

#### GLNS based on RSA

它使用了confounders实际为随机数来充当一次一密的手段。对于有可信S的情况，它们是安全的，可抵抗字典攻击，但是有两个版本不行。

*Direct Authentication Protocol*允许A,B在没有服务器的帮助下建立会话密钥

*Secret Public Key Protocol*允许A和B不记得服务器公钥的情况下建立会话密钥

文章使用RSA的变形作为例子

##### DAP A每次生成公钥$K_{ab1}$，AB共享口令$K{ab}$

$1.A→B: A,ra,K_{ab}(e),n$

$2.B→A:(B,A,...)^e(mod(n))$

一个攻击者可以伪装成Alice,以X替换$K_{ab}(e)$，已知n的因式分解，也就对于任何e都能知道d，B会解密这个X，得到e，攻击者可以猜测P’也解密得到e’，也可以得到d’，如果去$ed'$恰好解出来$(B',A')$为真实的(B,A)，说明这个口令正确

##### SPKP

服务器发送给两者公钥，$ka,kb$为与服务器共享的密钥

$1.A→S:A,B$

$2.S→A: A,B,K_a(e_a),n_a,K_b(e_b),n_b$

$3.A→B:(A,B,...)^{e_a}(mod(n_a)),K_b(e_b),n_b$

攻击者可以伪装为服务器，发送给A，$A,B,X_a,n_a,X_b,n_b$，攻击者使用自己的na，已知素数因子，已知p,q，可以得到对于e的d，如果A解密Xa得到ea，并且发送给B，攻击者可以窃听，并尝试使用自己猜测P’得到的ea对应的da来求，如果解出来A,B匹配则可以证明找到了$K_a$，相似也可以用于$K_b$

#### 攻击可行的原因

最根本的原因是，对于非EKE版本的协议的基本假设都被违背了（所以反过来去研究？）

比如生成元

公共的常量信息可能被密码分析者分析，

可以使用较大的类型的信息

#### 设计安全协议最重要的因素



> *费马小定理*
>
> - 如果p为素数，a是一个不能被p整除的整数，则$a^{p-1}≡1(mod(p))$，再者对于每个整数a都有$a^p≡a(mod(p))$
>
> *伪素数*
>
> - 令b是一个正整数，如果n是一个正合数且$b^{n-1}≡1(mod(n))$，则n称为以b为基数的伪素数

伪代码也比较好理解

![a5f870323ea1d264e4089b39431ecf2](https://user-images.githubusercontent.com/40269368/148508065-74ac08ed-b4c5-43ec-a853-cbace243fde8.png)、
