### 如何快速搭建 HTTPS 网站
#### 什么是HTTPS?
HTTPS 简单来说在 HTTP 协议基础上加了一层TLS/ SSL 加密协议，由网景公司发明。使用HTTP明文传输相当于数据在互联网上裸奔，任何时候都有可能被软硬件厂商、路由器厂商、网络服务商篡改伪造。
主流电商金融支付网站都已实现全站 HTTPS 化，Google 一直是强力推行 HTTPS 化的排头兵，用过Chrome 浏览器的人会发现对非 HTTP 网站做出安全性提示。

![Alt text](https://github.com/nosqlcoco/public-images/blob/master/blog_nosqlcoco/https-1.png?raw=true)

#### HTTPS 和 HTTP 的区别
HTTP 是明文传输，数据容易被篡改，HTTPS 属于加密传输，即便得到密文，在不知道私钥的情况下，无法知道真实数据。

#### HTTPS 原理

![Alt text](https://github.com/nosqlcoco/public-images/blob/master/blog_nosqlcoco/https-2.png?raw=true)
1. 客户端发起一个https的请求，把自身支持的一系列Cipher Suite（密钥算法套件，简称Cipher）发送给服务端。
2. 服务端，接收到客户端所有的Cipher后与自身支持的对比，如果不支持则连接断开，反之以证书的形式返回给客户端 证书中还包含了 公钥 颁证机构 网址 失效日期。
3. 客户端收到服务端响应后会做以下几件事：
	- 向CA机构验证证书的合法性
	- 生成随机字符串，并用证书中的公钥加密。
	- 随机字符串Hash编码
4. 服务端拿到客户端传来的密文，用私钥解密握手消息取出随机数密码，再用HASH编码，并与传过来的HASH值做对比确认是否一致。
5. 客户端用随机数解密并计算握手消息的HASH，如果与服务端发来的HASH一致，此时握手过程结束，之后所有的通信数据将由之前浏览器生成的随机密码并利用对称加密算法进行加密 。

（HTTPS原理部分摘自https://www.cnblogs.com/zery/p/5164795.html）

#### 如何搭建
##### 1. 申请证书
证书由第三方 CA 认证机构颁发，网站所有者向 CA 机构申请证书，证书中包含了公钥、颁证机构、网址、失效日期。如果网站使用假冒证书，浏览器向 CA 认证机构发送证书是否合法的请求，如果检测非法证书，浏览器可以直接断开请求。

腾讯云提供免费1年的SSL证书，申请步骤如下：
- 注册登录 https://cloud.tencent.com
- 点击"产品"，搜索 "SSL"， 点击 “SSL证书”
![Alt text](https://github.com/nosqlcoco/public-images/blob/master/blog_nosqlcoco/https-3.png?raw=true)
- 选择“域名型免费版“，显示的价格为0元，
![Alt text](https://github.com/nosqlcoco/public-images/blob/master/blog_nosqlcoco/https-4.png?raw=true)
- 填写域名信息
![Alt text](https://github.com/nosqlcoco/public-images/blob/master/blog_nosqlcoco/https-5.png?raw=true)
- 域名验证，证明域名所有人是你本人，我使用“手动DNS验证“
![Alt text](https://github.com/nosqlcoco/public-images/blob/master/blog_nosqlcoco/https-6.png?raw=true)
确认申请后，跳转到证书详情信息页面
![Alt text](https://github.com/nosqlcoco/public-images/blob/master/blog_nosqlcoco/https-7.png?raw=true)
- 域名解析，因为我的域名在万网注册的，所以打开登录万网，新增一条TXT记录类型。
![Alt text](https://github.com/nosqlcoco/public-images/blob/master/blog_nosqlcoco/https-8.png?raw=true)
- 返回到腾讯云证书申请管理页面，等几分钟就能显示“已颁发”状态。
 ![Alt text](https://github.com/nosqlcoco/public-images/blob/master/blog_nosqlcoco/https-9.png?raw=true)
将证书下载到本地，完成证书申请流程。

##### 2. 证书部署到 Tomcat
解压刚下载的证书，可以看到已经帮我们生成好了不同WEB服务器的HTTPS证书。
![Alt text](https://github.com/nosqlcoco/public-images/blob/master/blog_nosqlcoco/https-10.png?raw=true)
jks就是我们需要的证书文件。

接下来将HTTPS安装到Tomcat中。

2.1 配置SSL连接器
将www.domain.com.jks文件存放到conf目录下，然后配置同目录下的server.xml文件：
```
<Connector port="443" protocol="HTTP/1.1" SSLEnabled="true"
    maxThreads="150" scheme="https" secure="true"
    keystoreFile="conf/www.domain.com.jks"
    keystorePass="changeit"
    clientAuth="false" sslProtocol="TLS" />
```

| 配置文件参数|     说明| 
| :-------- | :--------| 
|clientAuth	 |  如果设为true，表示Tomcat要求所有的SSL客户出示安全证书，对SSL客户进行身份验证 |
| keystoreFile	| 指定keystore文件的存放位置，可以指定绝对路径，也可以指定相对于 （Tomcat安装目录）环境变量的相对路径。如果此项没有设定，默认情况下，Tomcat将从当前操作系统用户的用户目录下读取名为 “.keystore”的文件。  |
| keystorePass	|  密钥库密码，指定keystore的密码。（如果申请证书时有填写私钥密码，密钥库密码即私钥密码，否则填写密钥库密码文件中的密码） |
|sslProtocol	 |  指定套接字（Socket）使用的加密/解密协议，默认值为TLS |

2.2 http自动跳转https的安全配置
到conf目录下的web.xml。在`</welcome-file-list>`后面，`</web-app>`，也就是倒数第二段里，加上这样一段
```
    <login-config>
    <!-- Authorization setting for SSL -->
    <auth-method>CLIENT-CERT</auth-method>
    <realm-name>Client Cert Users-only Area</realm-name>
    </login-config>
    <security-constraint>
    <!-- Authorization setting for SSL -->
    <web-resource-collection>
    <web-resource-name>SSL</web-resource-name>
    <url-pattern>/*</url-pattern>
    </web-resource-collection>
    <user-data-constraint>
    <transport-guarantee>CONFIDENTIAL</transport-guarantee>
    </user-data-constraint>
    </security-constraint>
```
这步目的是让非ssl的connector跳转到ssl的connector去。所以还需要前往server.xml进行配置：
```
<Connector port="8080" protocol="HTTP/1.1"
    connectionTimeout="20000"
    redirectPort="443" />
```
重启 Tomcat，输入 HTTPS 网址访问成功。

网站 HTTPS 化是目前的主流趋势，但 HTTPS 依然只能做到暂时安全，如果量子计算机成为可能，HTTPS 握手后数据对称加密依然不安全。
