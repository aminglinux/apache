####apache SSL配置
```
模块： ssl_module
```

#####配置文件
```
<VirtualHost *:443>
    DocumentRoot /tmp/
    ServerName 1.com
    <IfModule mod_ssl.c>
    SSLEngine on
    SSLCertificateFile conf/server.crt
    SSLCertificateKeyFile conf/server.key
    </IfModule>
</VirtualHost>

```

#####ssl优化
```
1）openssl升级到最新版本

2）ssl相关优化配置

#禁用 SSLV3 SSLV2不安全的协议
SSLProtocol             all -SSLv3 -SSLv2 -TLSv1

#优化加密算法套件 (参考官方文档 https://httpd.apache.org/docs/trunk/ssl/ssl_howto.html#onlystrong)
SSLCipherSuite ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA256

#强制使用服务器端的加密算法套件优先级顺序
SSLHonorCipherOrder     on

#禁止压缩
SSLCompression          off

#ssl的session ticket功能（并不是所有的浏览器都支持）
##server将session信息加密成ticket发送给浏览器，浏览器后续握手请求时会发送ticket，
##server 端如果能成功解密和处理 ticket，就能完成简化握手，但增加了安全风险。
SSLSessionTickets       off

# OCSP Stapling, httpd 2.3.3之后的版本支持
SSLUseStapling          on
SSLStaplingResponderTimeout 5
SSLStaplingReturnResponderErrors off
SSLStaplingCache "shmcb:logs/stapling-cache(150000)"  //这行配置需要放到httpd.conf里，并且需要socache_shmcb_module支持

```

#####扩展 - SSL/TSL版本
```
1）历史
1994年，NetScape公司设计了SSL协议（Secure Sockets Layer）的1.0版，但是未发布。
1995年，NetScape公司发布SSL 2.0版，很快发现有严重漏洞。
1996年，SSL 3.0版问世，得到大规模应用。
1999年，互联网标准化组织ISOC接替NetScape公司，发布了SSL的升级版TLS 1.0版。
2006年和2008年，TLS进行了两次升级，分别为TLS 1.1版和TLS 1.2版。
2011年TLS1.2重构，不再向后兼容。
2016年TLS1.3出了第一个草案版本
2018年TLS1.3正式批准问世

TLS1.3版本，在安全性和访问速度上都有明显提升。目前使用主流版本依然是TLS1.2。

小常识：TLS 1.0通常被标示为SSL 3.1，TLS 1.1为SSL 3.2，TLS 1.2为SSL 3.3。

2）查看版本
linux查看系统支持版本
openssl ciphers -v | awk '{print $2}' | sort | uniq

chrome浏览器查看网站使用的版本
使用Chrome浏览器，打开你的https网站任意页面，在空白处右键，检查，在弹窗顶端找到'Security'选项卡，
打开，找到右边'Secure Connection'的介绍，寻找例如'(TLS 1.2)'等字眼，即可。有时是TLS 1.2，TLS 1.1，SSL 3等等。

```

#####扩展 - SSL密码套件（CipherSuite）
```
参考文档：  https://httpd.apache.org/docs/2.4/mod/mod_ssl.html#sslciphersuite

1）首先看openssl所支持的加密套件
openssl ciphers -v 

2）再来看加密套件名称组成：（例如ECDHE-RSA-AES256-GCM-SHA384）定义一个密钥交换算法、一个认证算法、一个加密算法，以及一个MAC摘要算法


3）算法示例：
密钥交换/协商算法 ：RSA、Diffie–Hellman（DH）、ECDH
认证算法：RSA、DH、DSS、ECDSA
密码/加密算法：AES、IDEA、DES、RC4、RC2、IDEA
MAC摘要算法：对于TLS来说，密钥散列消息认证码使用MD5或一种SHA算法。对于SSL，则SHA、MD5、MD4及MD2都可使用。

4）别名
SSv3 所有ssl 3.0版本
TLSv1 所有TLS 1.0版本
EXp  所有export
LOW  所有低强度
MEDIMU  所有128位加密
HIGH 所有使用Triple-DES
等等
```
![image](https://coding.net/u/aminglinux/p/apache/git/blob/master/%E5%AF%86%E7%A0%81%E5%88%AB%E5%90%8D.png)
```
5）连接符
+ 将匹配的密码移动到列表中的当前位置
- 从列表中移除密码（可稍后再添加）
! 完全从列表中去除密码（以后也不能再添加）

```

#####扩展-OCSP Stapling
```
出于某些原因，证书颁发者有时候需要作废某些证书。那么证书使用者（例如浏览器）如何知道一个证书是否已被作废呢？
通常有两种方式：CRL（Certificate Revocation List，证书撤销名单）和 OCSP（Online Certificate Status Protocol，在线证书状态协议）。

CRL 是由证书颁发机构定期更新的一个列表，包含了所有已被作废的证书，浏览器可以定期下载这个列表用于验证证书合法性。
不难想象，CRL 会随着时间推移变得越来越大，而且实时性很难得到保证。
OCSP 是一个在线查询接口，浏览器可以实时查询单个证书的合法性。在每个证书的详细信息中，都可以找到对应颁发机构的 CRL 和 OCSP 地址。

OCSP 的问题在于，某些客户端会在 TLS 握手阶段进一步协商时，实时查询 OCSP 接口，
并在获得结果前阻塞后续流程，这对性能影响很大。而 OCSP Stapling（OCSP 封套），
是指服务端在证书链中包含颁发机构对证书的 OCSP 查询结果，从而让浏览器跳过自己去验证的过程。
服务端有更快的网络，获取 OCSP 响应更容易，也可以将 OCSP 响应缓存起来。

OCSP 响应本身经过了数字签名，无法伪造，所以 OCSP Stapling 技术既提高了握手效率，也不会影响安全性。
```