---
title: Mutual SSL in Spring With Self-Signed Certificate
date: 2023-05-23 21:44:51
tags: SSL,计算机网络
---

# SSL 知识

## 能够建立彼此信任通讯的数学基石

非对称算法中的密钥对具有这样的特性

（1） 若内容被这对密钥中的密钥A所加密，则只能使用密钥B解密，密钥A自己都不能解密得到原来的内容。反之，密钥B加密的内容也只有密钥A能解密。

（2）不可能使用密钥A推出密钥B。

（3）非对称密钥对有无穷多个且不难生成。

由此，试图通讯的甲乙双方可以自己生产自己的非对称算法密钥对，并毫无顾忌地分享其中一个（这个被分享出去的称为**公钥**）同时谨慎地保管另一个（称为**私钥**）。此时，甲若用乙分享的公钥加密了一段自己刚随便生成没有公开的内容，然后将这个加密公开出去，不考虑无限次尝试，碰巧以及乙私钥泄露的情况，那么只有乙能用自己的私钥解密得到这个内容。进而，谁能告诉乙他之前公开的加密内容所对应的原内容是啥，那么那个人一定是甲。

由此，通讯双方可以确认自己在和谁通信，并且在使用公钥加密的基础上，通信内容也只有对方可以解密。

## 诡计多端的中间人

由上一节我们知道，只要甲和乙已经获知了彼此的公钥，那么甲和乙就可以彼此验明正身，交换只有彼此可知的信息。

但『只要甲和乙已经获知了彼此的公钥』这个条件并不是那么容易确保的。如果有一个中间人丙，它对甲说自己的公钥是乙的公钥，对乙说自己的公钥是甲的公钥。 那么甲就会把丙当做乙，乙也会把丙当做甲。此时甲和乙之间的通讯内容也对丙完全透明了，因为他们加密用的是丙的公钥，自然可以被丙的私钥解密。

怎么避免这样的**中间人攻击**呢？

（1）操作系统都内置一些可信的权威机构的公钥。

（2）不想被中间人顶替自己蒙骗自己的通讯者的那一方，向这些被操作系统默认内置的权威机构中的一个注册自己的公钥和自己的信息，并拿到由那个机构基于它的私钥和信息后加密的结果。

（3）所有通讯的发起者在验明正身的时候，都必须让另一方说明自己是注册在了哪个权威机构。然后用系统内置的那个权威机构的公钥对内容解密，获得其中的公钥。

中间人此时无法拿自己的公钥来伪造这里的公钥，因为：

（1）中间人是不可信的违法犯罪者，没有权威机构的私钥，它使用另外一个私钥的加密结果，用正确的公钥解密，解密的结果里的公钥也不可能是他自己的公钥。而且解密出来的其它信息，比如IP信息（或者域名信息）也往往是不可对或者与通讯发起者的目标不一致的，这都会导致通讯发起者认为没有与对的目标建立其绘画

（2）由于非对称加密密钥对的性质，无法基于公钥解密的结果，去反推用公钥加密前的内容应该长什么样。

这里的权威机构就是我们说的 CA (Certificate Authority), 也即**数字证书认证机构**。

## 证书与相关概念

<table>
<tr/>
<td>CA</td> <td>Certificate Authority</td> <td>数字证书认证机构</td> <td></td>
<tr/>
<tr/>
<td>CSR</td> <td>* Certificate Signing Request</td> <td>凭证签发请求</td> <td>站点在向全文机构申请证书前准备的自己的资料（包含公钥但不应当包含私钥）。CA 基于 CSR 签发 Cert。</td>
<tr/>
<tr>
<td>Cert</td> <td>Certificate</td> <td>证书</td> <td>一般公钥以及站点的信息已经封装在其中。</td>
</tr>
<tr>
<td>X.509</td> <td>X.509</td> <td></td> <td>X.509是密码学里公钥证书的格式标准。符合X.509标准的证书里含有公钥、身份信息（比如网络主机名，组织的名称或个体名称等）和签名信息（可以是证书签发机构CA的签名，也可以是自签名）。这是最流行的证书标准。</td>
</tr>
<tr>
<td>Key</td> <td>Private Key</td> <td>私钥</td> <td>因为公钥一般都封装在证书中，因此在和 Cert 同时出现的语境中，一般就简单地指私钥。比如保存私钥的文件的后缀名便是key。</td>
</tr>
<tr>
<td>KeyStore</td> <td>KeyStore</td> <td>无公认的中文说法</td> <td>一个 KeyStore 中可以存放多组密钥对，或证书私钥对，亦或者不成对的。KeyStore 是对 SSL 资料的打包，将多个 SSL 资料文件合并为一个方便使用。</td>
</tr>
<td>Self-signed CA/Cert</td> <td>Self-signed CA/Cert</td> <td>自签名CA/证书</td> <td>在局域网，比如公司的内网，只要这个局域网的参与者认可的 CA，以及由这个 CA 签发的证书。</td>
</tr>
</tr>
<td>Mutual SSL/2-way SSL</td> <td>Mutual SSL/2-way SSL</td> <td>SSL双向认证</td> <td>常见的浏览器访问 HTTPS 的站点，其实只是单向验证。我们验证了站点的身份，但站点并不验证我们的身份。而通讯双方都需要彼此验明正身的，就是 SSL 双向认证。</td>
</tr>
</tr>
<td>PKCS</td> <td>Public Key Cryptography Standards</td> <td>公钥加密标准</td> <td>PKCS #10	为 证书申请标准（Certification Request Standard）, 规范了向证书中心申请证书之CSR（certificate signing request）的格式。PKCS 下共有15个子项，每个子项对应一个公钥加密标准的所涉及的技术的标准规范。</td>
</tr>
</table>

** 亦可用 Certificate Request 指 CSR，在 openssl 的 Linux 手册中，使用 Certificate Request 而非 Certificate Signing Request，但说的是同一个东西。

# Self-signed CA

我在自己的 windows PC 上开了一个 Unbutun 的虚拟机，在更新了 openssh，安装 Java 环境（确保 JDK 中的 keytool 可用）后，使用这个虚拟机作为自签名 CA 来签发 Cert。

参考资料：
- [X.509 Authentication in Spring Security](https://www.baeldung.com/x-509-authentication-in-spring-security) 
- [man openssl](https://linux.die.net/man/1/openssl)
- [man openssl req](https://linux.die.net/man/1/req)
- [man oepnssl x509](https://linux.die.net/man/1/x509)

作为CA，我们也需要有自己的证书，使用如下命令创建：

```shell
openssl req -x509 -sha256 -days 3650 -newkey rsa:4096 -keyout rootCA.key -out rootCA.crt
```

- openssl: OpenSSL is a cryptography toolkit implementing the Secure Sockets Layer (SSL v2/v3) and Transport Layer Security (TLS v1) network protocols and related cryptography standards required by them. The openssl program is a command line tool for using the various cryptography functions of OpenSSL's crypto library from the shell.
- req: PKCS#10 X.509 Certificate Signing Request ( CSR ) Management.
- -x509: **this option outputs a self signed certificate instead of a certificate request**. This is typically used to generate a test certificate or a self signed root CA . 
- -sha256: -[digest] this specifies the message digest to sign the request with (such as -md5, -sha1). Here, the digest is sha256. 指定生成的证书中，证书内容的摘要所用的算法。此处为 SHA256。
- -days 3650: 该证书将在 3650 天后过期。
- -newkey rsa:4096: this option creates a new certificate request and a new private key. The argument takes one of several forms. rsa:nbits, where nbits is the number of bits, generates an RSA key nbits in size. If nbits is omitted, i.e. -newkey rsa specified, the default key size, specified in the configuration file is used.
- -keyout rootCA.key: 将生成的私钥保存到当前工作路径下的 rootCA.key 文件。
- -out rootCA.crt: 将生成的证书保存为工作路径下的 rootCA.crt 文件。

这个命令执行后，会要求你输入一个密码以保护私钥。这个密码会加密私钥，但这不意味着 rootCA.key 文件用记事本打开会是大片乱码。

# 为服务端签发证书

在 Windows PC 上写 Spring 程序，写好之后 jar 包扔到虚拟机上跑。虚拟机的 IP 是 `192.168.19.128`。要让 `https://192.168.19.128/` 能够被成功访问，要为这个程序签发证书并让程序加载它，在收到 SSL 请求时使用它。

## 生成 CSR

```shell
openssl req -new -newkey rsa:4096 -keyout localhost.key -out localhost.csr
```

- -new: this option generates a new certificate request.
- -out localhost.csr: 生成的 CSR 保存为工作路径下的 localhost.scr 文件。

## 获得 Cert 所需的其它信息

在当前工作路径下创建 localhost.ext 文件，内容如下：

```
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
subjectAltName = @alt_names
[alt_names]
IP.1 = 192.168.19.128
```

如果使用的是域名而非 IP，则 IP.1 那一行可以改为 `DNS.1 = localhost`。

## 使用 Self-signed CA 签发 localhost 证书

```shell
openssl x509 -req -CA rootCA.crt -CAkey rootCA.key -in localhost.csr -out localhost.crt -days 365 -CAcreateserial -extfile localhost.ext
```

- -CA rootCA.crt: 指定签发证书的 CA 的证书是在当前工作路径下的 rootCA.crt 文件。
- -CAkey rootCA.key：指定签发证书的 CA 的私钥保存在 rootCA.key 文件中。
- -in localhost.scr: 请求证书者的简历，其中最重要的是包含公钥。
- -out localhost.crt: 为请求者签发的证书输出为当前工作目录下的 locahost.crt 文件
- -days 365: 签发的这个证书有效期为 365 天，即一年。
- -CAcreateserial: with this option the CA serial number file is created if it does not exist: it will contain the serial number "02" and the certificate being signed will have the 1 as its serial number. Normally if the -CA option is specified and the serial number file does not exist it is an error.
- -extfile localhost.ext: 在签发证书的时，参考 localhost.ext 进行签发并在证书中写入对应信息。
