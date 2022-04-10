之前做实验的时候，曾经写过一篇博客，如何使用OpenSSL创建证书。这里再做一个整理，增加一些内容，精简一部分内容。

参考链接

> [OpenSSL Certificate Authoirty](https://jamielinux.com/docs/openssl-certificate-authority/#)
>
> [信安实践——自建CA证书搭建https服务器](https://www.cnblogs.com/libaoquan/p/7965873.html)
>
> [您的连接不是私密连接：NET::ERR_CERT_COMMON_NAME_INVALID](https://blog.csdn.net/osfipin/article/details/106866275/)
>
> [使用OpenSSL制作自签名ECC证书](https://asakawa.org/creating-a-self-signed-ecc-certificate-with-openssl/)

运行环境

> Apache + Ubuntu 16.04

使用工具

> OpenSSL 、Namesilo、驰迅网络

## 前言

服务器在`驰迅网络`中买的香港的服务器（某宝上），域名在`Namesilo`上购买的，因此无需备案就可以使用，如果只是做实验还是挺好用的。本次实验使用Apache服务器，证书的申请使用`OpenSSL`。主要参考的来源是上方的几个链接，特别感谢师姐的帮助，这篇博客也参考了师姐整理的笔记。

## 安装Apache 和 SSL 扩展

首先在Ubuntu 16.04的环境下安装Apache：

由于这里我已经安装过了，所以重新敲一遍的过程可能有一些怪异。这里服务器自动给我的`root`，我还没有创建新用户，这里推荐重新建立一个新用户在自己的目录下进行实验，所以如果出现了`Permission denied`，记得`sudo`

```powershell
root@cloud:~# apt install apache2
```

如果你在这个过程中遇到 `xxx not upgraded`这种问题，可以通过执行下面这条命令进行更改。（这个一般是因为安装过程中手残按了终止，我也不是很清楚原因）

```powershell
root@cloud:~# apt-get dist-upgrade
```

接下来，检查Apache的状态：

```powershell
root@cloud:~# systemctl status apache2
# 启动
root@cloud:~# /etc/init.d/apache2 start
# 停止
root@cloud:~# /etc/init.d/apache2 stop
# 重启
root@cloud:~# /etc/init.d/apache2 restart
```

### 配置SSL模块

Apache开启SSL服务模块

```powershell
root@cloud:~# a2enmod ssl
```

接下来的步骤需要生成证书后完成

修改配置文件并将证书路径添加到`/etc/apache2/sites-available/default-ssl.conf`中，启用配置文件并重启服务器。

```powershell
# 启用配置文件
root@cloud:~# a2ensite default-ssl.conf
# 重启Apache
root@cloud:~# /etc/init.d/apache2 restart
```

> 如果遇到`site default-ssl not properly enabled, default-ssl.conf is a real file`这样的错误，可以启用软连接，本人不太清楚这样是否为最好的解决方案，可以看一下之前的[博客](https://blog.csdn.net/a1219532602/article/details/110246247)

## 使用OpenSSL生成SSL证书（RSA）

> OpenSSL在Ubuntu 16.04中是默认安装的

这里主要的内容参考[OpenSSL Certificate Authoirty](https://jamielinux.com/docs/openssl-certificate-authority/#)，大家可以自行前往原网站查看原作者的思路。

### 以下是本人操作的步骤，以及对于配置文件的修改：

#### 最终的文件列表

```powershell
root@cloud:~# tree
.
└── ca # 根CA文件夹
    ├── certs # 根CA证书文件夹
    │   └── ca.cert.pem  # 根CA证书
    ├── crl # 证书撤销列表
    ├── index.txt # CA文本数据库文件，索引文件
    ├── index.txt.attr # CA文本数据库属性文件，包含一个配置行
    ├── index.txt.old # CA文本数据库备份文件
    ├── intermediate # 中间CA证书文件夹
    │   ├── certs # 中间CA证书
    │   │   ├── ca-chain.cert.pem # 证书链文件
    │   │   ├── intermediate.cert.pem # 中间CA证书
    │   │   └── www.felixlab.shop.cert.pem # Web服务证书
    │   ├── crl # 证书撤销列表
    │   ├── crlnumber # CRL中证书吊销数目数目
    │   ├── csr # 证书签名请求
    │   │   ├── intermediate.csr.pem 
    │   │   └── www.felixlab.shop.csr.pem
    │   ├── index.txt # 中间CA文本数据库文件
    │   ├── index.txt.attr # 中间CA文本数据库属性文件
    │   ├── index.txt.old # 中间CA文本数据库备份文件
    │   ├── newcerts # 新生成证书
    │   │   └── 1000.pem 
    │   ├── openssl.cnf # 中间CA配置文件
    │   ├── private # 中间CA私钥文件夹
    │   │   ├── intermediate.key.pem # 中间CA私钥
    │   │   └── www.felixlab.shop.key.pem # 网站私钥
    │   ├── serial # 生成证书序列号文件
    │   └── serial.old # 备份文件
    ├── newcerts # 新生成证书文件夹
    │   └── 1000.pem
    ├── openssl.cnf # 主配置文件
    ├── private # 根CA私钥文件夹
    │   └── ca.key.pem # 根CA私钥
    ├── serial # 根CA序列号文件
    └── serial.old # 根CA序列号备份文件

11 directories, 24 files
```

#### 创建根CA证书

> 创建根CA证书的私钥和公钥

##### 创建文件夹

```powershell
root@cloud:/# mkdir /fydlab/ca
root@cloud:/# cd /fydlab/ca
root@cloud:/fydlab/ca# pwd
/fydlab/ca
root@cloud:/fydlab/ca# mkdir certs crl newcerts private
root@cloud:/fydlab/ca# chmod 700 private
root@cloud:/fydlab/ca# touch index.txt
root@cloud:/fydlab/ca# echo 1000 > serial
root@cloud:/fydlab/ca# touch openssl.cnf
root@cloud:/fydlab/ca# tree
.
├── certs
├── crl
├── index.txt
├── newcerts
├── openssl.cnf
├── private
└── serial

4 directories, 3 files
```

这里按照[blog](https://jamielinux.com/docs/openssl-certificate-authority/index.html#)的附录的内容进行一点点修改：

```powershell
# OpenSSL root CA configuration file.

[ ca ]
# `man ca`
default_ca = CA_default

[ CA_default ]
# Directory and file locations.
# 把 dir 修改为自己的目录
dir               = /fydlab/ca
certs             = $dir/certs
crl_dir           = $dir/crl
new_certs_dir     = $dir/newcerts
database          = $dir/index.txt
serial            = $dir/serial
RANDFILE          = $dir/private/.rand

# The root key and root certificate.
private_key       = $dir/private/ca.key.pem
certificate       = $dir/certs/ca.cert.pem

# For certificate revocation lists.
crlnumber         = $dir/crlnumber
crl               = $dir/crl/ca.crl.pem
crl_extensions    = crl_ext
default_crl_days  = 30

# SHA-1 is deprecated, so use SHA-2 instead.
default_md        = sha256

name_opt          = ca_default
cert_opt          = ca_default
default_days      = 375
preserve          = no
policy            = policy_strict

[ policy_strict ]
# The root CA should only sign intermediate certificates that match.
# See the POLICY FORMAT section of `man ca`.
countryName             = match
stateOrProvinceName     = match
organizationName        = match
organizationalUnitName  = optional
commonName              = supplied
emailAddress            = optional

[ policy_loose ]
# Allow the intermediate CA to sign a more diverse range of certificates.
# See the POLICY FORMAT section of the `ca` man page.
countryName             = optional
stateOrProvinceName     = optional
localityName            = optional
organizationName        = optional
organizationalUnitName  = optional
commonName              = supplied
emailAddress            = optional

[ req ]
# Options for the `req` tool (`man req`).
default_bits        = 2048
distinguished_name  = req_distinguished_name
string_mask         = utf8only

# SHA-1 is deprecated, so use SHA-2 instead.
default_md          = sha256

# Extension to add when the -x509 option is used.
x509_extensions     = v3_ca

[ req_distinguished_name ]
# See <https://en.wikipedia.org/wiki/Certificate_signing_request>.
countryName                     = Country Name (2 letter code)
stateOrProvinceName             = State or Province Name
localityName                    = Locality Name
0.organizationName              = Organization Name
organizationalUnitName          = Organizational Unit Name
commonName                      = Common Name
emailAddress                    = Email Address

# Optionally, specify some defaults.
# 这里可以配置自己的默认项 Common Name需要不同，自己制定即可
countryName_default             = CN
stateOrProvinceName_default     = Beijing
localityName_default            = Haidian
0.organizationName_default      = Felix Ltd
organizationalUnitName_default  = Felix Ltd Authority Certificate
emailAddress_default            = 1122334@qq.com

[ v3_ca ]
# Extensions for a typical CA (`man x509v3_config`).
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid:always,issuer
basicConstraints = critical, CA:true
keyUsage = critical, digitalSignature, cRLSign, keyCertSign

[ v3_intermediate_ca ]
# Extensions for a typical intermediate CA (`man x509v3_config`).
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid:always,issuer
basicConstraints = critical, CA:true, pathlen:0
keyUsage = critical, digitalSignature, cRLSign, keyCertSign

[ usr_cert ]
# Extensions for client certificates (`man x509v3_config`).
basicConstraints = CA:FALSE
nsCertType = client, email
nsComment = "OpenSSL Generated Client Certificate"
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid,issuer
keyUsage = critical, nonRepudiation, digitalSignature, keyEncipherment
extendedKeyUsage = clientAuth, emailProtection

[ server_cert ]
# Extensions for server certificates (`man x509v3_config`).
basicConstraints = CA:FALSE
nsCertType = server
nsComment = "OpenSSL Generated Server Certificate"
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid,issuer:always
keyUsage = critical, digitalSignature, keyEncipherment
extendedKeyUsage = serverAuth

[ crl_ext ]
# Extension for CRLs (`man x509v3_config`).
authorityKeyIdentifier=keyid:always

[ ocsp ]
# Extension for OCSP signing certificates (`man ocsp`).
basicConstraints = CA:FALSE
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid,issuer
keyUsage = critical, digitalSignature
extendedKeyUsage = critical, OCSPSigning
```

> 需要修改的内容在上述文字中用注释标明，其余内容也很关键，鉴于作者水平就不多解释了。

##### 创建Root Key

```powershell
root@cloud:/fydlab/ca# pwd /fydlab/ca 
# 这一步需要输入口令
root@cloud:/fydlab/ca# openssl genrsa -aes256 -out private/ca.key.pem 4096
root@cloud:/fydlab/ca# chmod 400 private/ca.key.pem 
```

##### 创建根证书

首先确保自己已经配置了`openssl.cnf`

```powershell
root@cloud:/fydlab/ca# openssl req -config openssl.cnf -key private/ca.key.pem -new -x509 -days 7300 -sha256 -extensions v3_ca -out certs/ca.cert.pem 
Enter pass phrase for private/ca.key.pem: 
----
Country Name (2 letter code) [CN]:
State or Province Name [Beijing]:
Locality Name [Beijing]:
Organization Name [Felix Ltd]:
Organizational Unit Name [Felix Ltd Authority Certificate]:
Common Name []:Felix Ltd Root CA 
Email Address [1122334@qq.com]: 
root@cloud:/fydlab/ca# chmod 444 certs/ca.cert.pem 
```

##### 验证根证书格式

```powershell
root@cloud:/fydlab/ca# openssl x509 -noout -text -in certs/ca.cert.pem
```

#### 创建中间证书

```powershell
# 创建文件夹并进入目录
root@cloud:/fydlab/ca# mkdir /fydlab/ca/intermediate
root@cloud:/fydlab/ca# cd /fydlab/ca/intermediate
root@cloud:/fydlab/ca/intermediate# mkdir certs crl csr newcerts private
root@cloud:/fydlab/ca/intermediate# chmod 700 private
root@cloud:/fydlab/ca/intermediate# touch index.txt
root@cloud:/fydlab/ca/intermediate# echo 1000 > serial
root@cloud:/fydlab/ca/intermediate# echo 1000 > crlnumber
root@cloud:/fydlab/ca/intermediate# touch openssl.cnf
```

以下为`openssl.cnf`的内容

```powershell
# OpenSSL intermediate CA configuration file.

[ ca ]
# `man ca`
default_ca = CA_default

[ CA_default ]
# Directory and file locations.
# 这里的路径要换成自己的
dir               = /fydlab/ca/intermediate
certs             = $dir/certs
crl_dir           = $dir/crl
new_certs_dir     = $dir/newcerts
database          = $dir/index.txt
serial            = $dir/serial
RANDFILE          = $dir/private/.rand

# The root key and root certificate.
private_key       = $dir/private/intermediate.key.pem
certificate       = $dir/certs/intermediate.cert.pem

# For certificate revocation lists.
crlnumber         = $dir/crlnumber
crl               = $dir/crl/intermediate.crl.pem
crl_extensions    = crl_ext
default_crl_days  = 30

# SHA-1 is deprecated, so use SHA-2 instead.
default_md        = sha256

name_opt          = ca_default
cert_opt          = ca_default
default_days      = 375
preserve          = no
policy            = policy_loose

[ policy_strict ]
# The root CA should only sign intermediate certificates that match.
# See the POLICY FORMAT section of `man ca`.
countryName             = match
stateOrProvinceName     = match
organizationName        = match
organizationalUnitName  = optional
commonName              = supplied
emailAddress            = optional

[ policy_loose ]
# Allow the intermediate CA to sign a more diverse range of certificates.
# See the POLICY FORMAT section of the `ca` man page.
countryName             = optional
stateOrProvinceName     = optional
localityName            = optional
organizationName        = optional
organizationalUnitName  = optional
commonName              = supplied
emailAddress            = optional

[ req ]
# Options for the `req` tool (`man req`).
default_bits        = 2048
distinguished_name  = req_distinguished_name
string_mask         = utf8only

# SHA-1 is deprecated, so use SHA-2 instead.
default_md          = sha256

# Extension to add when the -x509 option is used.
x509_extensions     = v3_ca

[ req_distinguished_name ]
# See <https://en.wikipedia.org/wiki/Certificate_signing_request>.
countryName                     = Country Name (2 letter code)
stateOrProvinceName             = State or Province Name
localityName                    = Locality Name
0.organizationName              = Organization Name
organizationalUnitName          = Organizational Unit Name
commonName                      = Common Name
emailAddress                    = Email Address

# Optionally, specify some defaults.
countryName_default             = CN
stateOrProvinceName_default     = Beijing
localityName_default            = Haidian
0.organizationName_default      = Felix Ltd
organizationalUnitName_default  = Felix Ltd Authority Certificate
emailAddress_default            = 1122334@qq.com

[ v3_ca ]
# Extensions for a typical CA (`man x509v3_config`).
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid:always,issuer
basicConstraints = critical, CA:true
keyUsage = critical, digitalSignature, cRLSign, keyCertSign

[ v3_intermediate_ca ]
# Extensions for a typical intermediate CA (`man x509v3_config`).
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid:always,issuer
basicConstraints = critical, CA:true, pathlen:0
keyUsage = critical, digitalSignature, cRLSign, keyCertSign

[ usr_cert ]
# Extensions for client certificates (`man x509v3_config`).
basicConstraints = CA:FALSE
nsCertType = client, email
nsComment = "OpenSSL Generated Client Certificate"
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid,issuer
keyUsage = critical, nonRepudiation, digitalSignature, keyEncipherment
extendedKeyUsage = clientAuth, emailProtection

[ server_cert ]
# 这里需要配置一下 SAN扩展，否则谷歌浏览器会报COMMON NAME错误 
# Extensions for server certificates (`man x509v3_config`).
basicConstraints = CA:FALSE
nsCertType = server
nsComment = "OpenSSL Generated Server Certificate"
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid,issuer:always
keyUsage = critical, digitalSignature, keyEncipherment
extendedKeyUsage = serverAuth
############添加的内容############
subjectAltName = @alt_names

[ alt_names ]
DNS.1 = *.fydlab.shop
DNS.2 = *fydlab.shop
DNS.3 = fydlab.shop
############添加的内容############

[ crl_ext ]
# Extension for CRLs (`man x509v3_config`).
authorityKeyIdentifier=keyid:always

[ ocsp ]
# Extension for OCSP signing certificates (`man ocsp`).
basicConstraints = CA:FALSE
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid,issuer
keyUsage = critical, digitalSignature
extendedKeyUsage = critical, OCSPSigning
```

```powershell
# 创建中间CA私钥
root@cloud:/fydlab/ca/intermediate# cd /fydlab/ca
root@cloud:/fydlab/ca# openssl genrsa -aes256 -out intermediate/private/intermediate.key.pem 4096
root@cloud:/fydlab/ca# chmod 400 intermediate/private/intermediate.key.pem

# 创建中间CA csr
root@cloud:/fydlab/ca# openssl req -config intermediate/openssl.cnf -new -sha256 -key intermediate/private/intermediate.key.pem -out intermediate/csr/intermediate.csr.pem
## 这里中间CA 的Common Name 可以定义为:Felix Ltd Intermediate CA

# 签署中间CA证书
root@cloud:/fydlab/ca# openssl ca -config openssl.cnf -extensions v3_intermediate_ca -days 3650 -notext -md sha256 -in intermediate/csr/intermediate.csr.pem -out intermediate/certs/intermediate.cert.pem
root@cloud:/fydlab/ca# chmod 444 intermediate/certs/intermediate.cert.pem

# 查看 `index.txt` 文件内容
root@cloud:/fydlab/ca# cat index.txt

# 查看并验证生成的证书
root@cloud:/fydlab/ca# openssl x509 -noout -text -in intermediate/certs/intermediate.cert.pem
root@cloud:/fydlab/ca# openssl verify -CAfile certs/ca.cert.pem intermediate/certs/intermediate.cert.pem

# 创建证书链文件
root@cloud:/fydlab/ca# cat intermediate/certs/intermediate.cert.pem certs/ca.cert.pem > intermediate/certs/ca-chain.cert.pem
root@cloud:/fydlab/ca# chmod 444 intermediate/certs/ca-chain.cert.pem
```

#### 创建网站证书

```powershell
# 创建私钥
root@cloud:/fydlab/ca# openssl genrsa -aes256 -out intermediate/private/www.fydlab.shop.key.pem

root@cloud:/fydlab/ca# chmod 400 intermediate/private/www.fydlab.shop.key.pem
# 创建CSR
root@cloud:/fydlab/ca# openssl req -config intermediate/openssl.cnf -key intermediate/private/www.fydlab.shop.key.pem -new -sha256 -out intermediate/csr/www.fydlab.shop.csr.pem
## 这里的 common name 可以写 网站域名
# 创建证书 需要用到 server_cert扩展
root@cloud:/fydlab/ca# openssl ca -config intermediate/openssl.cnf -extensions server_cert -days 375 -notext -md sha256 -in intermediate/csr/www.fydlab.shop.csr.pem -out intermediate/certs/www.fydlab.shop.cert.pem

# 查看 index.txt 中的内容:
root@cloud:/fydlab/ca# cat intermediate/index.txt

# 验证证书
root@cloud:/fydlab/ca# openssl x509 -noout -text -in intermediate/certs/www.fydlab.shop.cert.pem

root@cloud:/fydlab/ca# openssl verify -CAfile intermediate/certs/ca-chain.cert.pem intermediate/certs/www.fydlab.shop.cert.pem
```

#### 部署到服务器上

修改`/etc/apache/sites-enabled/default-ssl.conf`文件

```powershell
<IfModule mod_ssl.c>
        <VirtualHost _default_:443>
                ServerAdmin webmaster@localhost

                DocumentRoot /var/www/html

                # Available loglevels: trace8, ..., trace1, debug, info, notice, warn,
                # error, crit, alert, emerg.
                # It is also possible to configure the loglevel for particular
                # modules, e.g.
                #LogLevel info ssl:warn

                ErrorLog ${APACHE_LOG_DIR}/error.log
                CustomLog ${APACHE_LOG_DIR}/access.log combined

                # For most configuration files from conf-available/, which are
                # enabled or disabled at a global level, it is possible to
                # include a line for only one particular virtual host. For example the
                # following line enables the CGI configuration for this host only
                # after it has been globally disabled with "a2disconf".
                #Include conf-available/serve-cgi-bin.conf

                #   SSL Engine Switch:
                #   Enable/Disable SSL for this virtual host.
                SSLEngine on

                #   A self-signed (snakeoil) certificate can be created by installing
                #   the ssl-cert package. See
                #   /usr/share/doc/apache2/README.Debian.gz for more info.
                #   If both key and certificate are stored in the same file, only the
                #   SSLCertificateFile directive is needed.
                ### 需要修改的部分 ###
                SSLCertificateFile      /root/ca/intermediate/certs/www.fydlab.shop.cert.pem
                ### 需要修改的部分 ###
                SSLCertificateKeyFile /root/ca/intermediate/private/www.fydlab.shop.key.pem

                #   Server Certificate Chain:
                #   Point SSLCertificateChainFile at a file containing the
                #   concatenation of PEM encoded CA certificates which form the
                #   certificate chain for the server certificate. Alternatively
                #   the referenced file can be the same as SSLCertificateFile
                #   when the CA certificates are directly appended to the server
                #   certificate for convinience.
                ### 需要修改的部分 ###
                SSLCertificateChainFile /root/ca/intermediate/certs/ca-chain.cert.pem

                #   Certificate Authority (CA):
                #   Set the CA certificate verification path where to find CA
                #   certificates for client authentication or alternatively one
                #   huge file containing all of them (file must be PEM encoded)
                #   Note: Inside SSLCACertificatePath you need hash symlinks
                #                to point to the certificate files. Use the provided
                #                Makefile to update the hash symlinks after changes.
                #SSLCACertificatePath /etc/ssl/certs/
                #SSLCACertificateFile /etc/apache2/ssl.crt/ca-bundle.crt

                #   Certificate Revocation Lists (CRL):
                #   Set the CA revocation path where to find CA CRLs for client
                #   authentication or alternatively one huge file containing all
                #   of them (file must be PEM encoded)
                #   Note: Inside SSLCARevocationPath you need hash symlinks
                #                to point to the certificate files. Use the provided
                #                Makefile to update the hash symlinks after changes.
                #SSLCARevocationPath /etc/apache2/ssl.crl/
                #SSLCARevocationFile /etc/apache2/ssl.crl/ca-bundle.crl

                #   Client Authentication (Type):
                #   Client certificate verification type and depth.  Types are
                #   none, optional, require and optional_no_ca.  Depth is a
                #   number which specifies how deeply to verify the certificate
                #   issuer chain before deciding the certificate is not valid.
                #SSLVerifyClient require
                #SSLVerifyDepth  10

                #   SSL Engine Options:
                #   Set various options for the SSL engine.
                #   o FakeBasicAuth:
                #        Translate the client X.509 into a Basic Authorisation.  This means that
                #        the standard Auth/DBMAuth methods can be used for access control.  The
                #        user name is the `one line' version of the client's X.509 certificate.
                #        Note that no password is obtained from the user. Every entry in the user
                #        file needs this password: `xxj31ZMTZzkVA'.
                #   o ExportCertData:
                #        This exports two additional environment variables: SSL_CLIENT_CERT and
                #        SSL_SERVER_CERT. These contain the PEM-encoded certificates of the
                #        server (always existing) and the client (only existing when client
                #        authentication is used). This can be used to import the certificates
                #        into CGI scripts.
                #   o StdEnvVars:
                #        This exports the standard SSL/TLS related `SSL_*' environment variables.
                #        Per default this exportation is switched off for performance reasons,
                #        because the extraction step is an expensive operation and is usually
                #        useless for serving static content. So one usually enables the
                #        exportation for CGI and SSI requests only.
                #   o OptRenegotiate:
                #        This enables optimized SSL connection renegotiation handling when SSL
                #        directives are used in per-directory context.
                #SSLOptions +FakeBasicAuth +ExportCertData +StrictRequire
                <FilesMatch "\.(cgi|shtml|phtml|php)$">
                                SSLOptions +StdEnvVars
                </FilesMatch>
                <Directory /usr/lib/cgi-bin>
                                SSLOptions +StdEnvVars
                </Directory>

                #   SSL Protocol Adjustments:
                #   The safe and default but still SSL/TLS standard compliant shutdown
                #   approach is that mod_ssl sends the close notify alert but doesn't wait for
                #   the close notify alert from client. When you need a different shutdown
                #   approach you can use one of the following variables:
                #   o ssl-unclean-shutdown:
                #        This forces an unclean shutdown when the connection is closed, i.e. no
                #        SSL close notify alert is send or allowed to received.  This violates
                #        the SSL/TLS standard but is needed for some brain-dead browsers. Use
                #        this when you receive I/O errors because of the standard approach where
                #        mod_ssl sends the close notify alert.
                #   o ssl-accurate-shutdown:
                #        This forces an accurate shutdown when the connection is closed, i.e. a
                #        SSL close notify alert is send and mod_ssl waits for the close notify
                #        alert of the client. This is 100% SSL/TLS standard compliant, but in
                #        practice often causes hanging connections with brain-dead browsers. Use
                #        this only for browsers where you know that their SSL implementation
                #        works correctly.
                #   Notice: Most problems of broken clients are also related to the HTTP
                #   keep-alive facility, so you usually additionally want to disable
                #   keep-alive for those clients, too. Use variable "nokeepalive" for this.
                #   Similarly, one has to force some clients to use HTTP/1.0 to workaround
                #   their broken HTTP/1.1 implementation. Use variables "downgrade-1.0" and
                #   "force-response-1.0" for this.
                # BrowserMatch "MSIE [2-6]" \
                #               nokeepalive ssl-unclean-shutdown \
                #               downgrade-1.0 force-response-1.0

        </VirtualHost>
</IfModule>

# vim: syntax=apache ts=4 sw=4 sts=4 sr noet

```

接下来指定配置文件，然后重启Apache即可

```powershell
# 启用配置文件
root@cloud:~# a2ensite default-ssl.conf
# 重启Apache
root@cloud:~# /etc/init.d/apache2 restart
```

在浏览器中访问一下自己的网站，应该是没问题的。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201129134311807.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ExMjE5NTMyNjAy,size_16,color_FFFFFF,t_70#pic_center)

## 使用OpenSSL生成SSL证书（ECC）

以下只展示命令行输入

```powershell
mkdir ca
cd ca/
mkdir certs crl newcerts private
chmod 700 private/
touch index.txt
echo 1000 > serial
touch openssl.cnf
vim openssl.cnf
# 这里是重点！
openssl ecparam -genkey -out private/ca.key.pem -name prime256v1
openssl req -config openssl.cnf -key private/ca.key.pem -new -x509 -days 730 -sha256 -extensions v3_ca -out certs/ca.cert.pem
chmod 444 certs/ca.cert.pem
openssl x509 -noout -text -in certs/ca.cert.pem

# 中间证书
mkdir intermediate
cd intermediate/
mkdir certs crl csr newcerts private
chmod 700 private/
touch index.txt
echo 1000 > serial
echo 1000 > crlnumber
touch openssl.cnf
vim openssl.cnf
cd ../
openssl ecparam -genkey -name prime256v1 -out intermediate/private/intermediate.key.pem
chmod 400 intermediate/private/intermediate.key.pem\n
openssl req -config intermediate/openssl.cnf -new -sha256 -key intermediate/private/intermediate.key.pem -out intermediate/csr/intermediate.csr.pem
openssl ca -config openssl.cnf -extensions v3_intermediate_ca -days 720 -notext -md sha256 -in intermediate/csr/intermediate.csr.pem -out intermediate/certs/intermediate.cert.pem
chmod 444 intermediate/certs/intermediate.cert.pem
cat index.txt
openssl x509 -noout -text -in intermediate/certs/intermediate.cert.pem
openssl verify -CAfile certs/ca.cert.pem intermediate/certs/intermediate.cert.pem
cat intermediate/certs/intermediate.cert.pem certs/ca.cert.pem > intermediate/certs/ca-chain.cert.pem

# 叶子证书
chmod 444 intermediate/certs/ca-chain.cert.pem
openssl ecparam -genkey -name prime256v1 -out intermediate/private/cert.key.pem
chmod 400 intermediate/private/cert.key.pem
openssl req -config intermediate/openssl.cnf -key intermediate/private/cert.key.pem  -new -sha256 -out intermediate/csr/cert.shop.csr.pem
openssl ca -config intermediate/openssl.cnf -extensions server_cert -days 375 -notext -md sha256 -in intermediate/csr/cert.shop.csr.pem   -out intermediate/certs/cert.cert.pem
1cat intermediate/index.txt
openssl x509 -noout -text -in intermediate/certs/cert.cert.pem
openssl verify -CAfile intermediate/certs/ca-chain.cert.pem intermediate/certs/cert.cert.pem
```

## 服务器和域名怎么弄的呢？

上面已经说了，可以去某宝上搜索`驰迅网络`，购买之后商家会要求去网站注册一个账号，然后他会开一个服务器并把账号密码发送给你。然后使用`xshell`或者`MobaXTerm`甚至`Jetbrains`的工具都可以，这里推荐使用前两者，`MobaXterm`超级好用，如果你和我一样在Mac上找不到特别好用的工具（或许是我比较懒），可以使用`Jetbrains→Tools→Deployment→Configuration`，然后选择`SFTP`，配置自己的服务器就可以了。这样就可以在自己的笔记本上配置自己的服务器了，具体怎么做参考前两节就可以了。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201129134257892.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ExMjE5NTMyNjAy,size_16,color_FFFFFF,t_70#pic_center)


然后是域名，可以去[NameSilo](namesilo.com)这个网站申请域名，具体的配置可以参考这篇[Blog](https://www.zhudc.com/website/2413/)，我选择的`A记录`解析，配置完成后可以去[whatsmydns](https://www.whatsmydns.net/)这个网站进行查阅，看什么时候可以在自己的笔记本上访问自己的域名了，就可以继续完成数字证书的实验了。