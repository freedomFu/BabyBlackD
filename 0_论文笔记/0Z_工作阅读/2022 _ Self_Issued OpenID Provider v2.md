## 协议流程

```json
openid://?
    scope=openid%20profile
    &response_type=id_token
    &client_id=did%3Aexample%3AEiDrihTRe0GMdc3K16kgJB3Xbl9Hb8oqVHjzm6ufHcYDGA
    &redirect_uri=https%3A%2F%2Fclient.example.org%2Fcb
    &claims=...
    &registration=%7B%22subject_syntax_types_supported%22%3A
    %5B%22did%3Aexample%22%5D%2C%0A%20%20%20%20
    %22id_token_signing_alg_values_supported%22%3A%5B%22ES256%22%5D%7D
    &nonce=n-0S6_WzA2Mj
```

##  如何保证SIOP的安全性

### SIOP From Github

https://github.com/Sphereon-Opensource/did-auth-siop

#### 简介

文档提供了一种鉴别库，其中客户端符合SIOPv2和OIDC4VP的标准。SIOPv2是OIDC的扩展，允许终端用户充当OP，并且可以直接向RP鉴别声明并提供声明，而无需依赖第三方IdP。这实现了完全自主的方案，不依赖于第三方且以peer to peer方式进行，仍然使用OIDC协议。在此协议中，用户充当OpenID Provider，另外还使用Verifiable Presentations。此时，RP可以提出对客户端或OP提供的VC的要求，OP也需要检查自己是否具有符合要求的凭据，只有符合提交，OP才会在响应阶段提供相应的凭证作为VP，RP会检查VP的有效性以及是否与要求匹配，只有验证通过后，用户才能正常安全地访问RP。

其中SI的来源是终端用户（OP）发行了SI ID Token来证明标识符和声明的有效性。与传统的OIDC相比，签发ID Token的第三方实体是经过用户的同意直接向RP发放ID Token的，也就是终端用户自己掌控着数据，而不是通过第三方的OP。

#### SIOP Requirements draft 2020

-   SIOP request
    -   OP签发响应的能力是OIDC core的扩展
    -   SIOP可以用于登录或者传输实体特征
    -   SIOP应支持协议类型的最佳实践
-   SIOP response
    -   SIOP在响应中应该可以返回VC和VP
-   Key recovery and key rotation
    -   密钥信息可以从DID文档中解析出DID或者从URN中解析出sub_jwk。（Uniform Resource Name）
-   Trust model between RP and SIOP（考虑到RP和SIOP之间的信任模型）
    -   SIOP必须能够说明自己是支持SIOP的OP，支持Invocation过程
    -   SIOP必须能够发送配置信息给RP，支持Discovery过程
    -   RP必须能够在SIOP上注册，支持Registration过程
-   Issuance of the claims
    -   SIOP可以被claim供应商注册
-   Privacy protection
    -   RP应该了解SIOP的安全/隐私状况
    -   SIOP应该支持成对、全向和临时的标识符
    -   过去的证明应该仍然有效
    -   RP必须在终端用户离线时接收声明，而且不会与claim提供者勾结
-   Claims binding
-   Various OpenID providers deployment architectures
    -   支持基于PWA的SIOP实现
    -   SIOP应该支持浏览器路径，设备路径以及两者的结合
-   Use-case specific requirements
    -   SIOP可以支持RP共享富实体信息
    -   SIOP应允许对claim集的选择性披露
    -   SIOP应允许离线鉴别





