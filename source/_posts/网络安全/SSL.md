## 一：漏洞扫描结果



>
>
>1. SSL/TLS 存在Bar Mitzvah Attack漏洞
>
>   | 详细描述  | 该漏洞是由功能较弱而且已经过时的RC4加密算法中一个问题所导致的。它能够在某些情况下泄露SSL/TLS加密流量中的密文，从而将账户用户名密码、信用卡数据和其他敏感信息泄露给黑客。 |
>   | :-------- | ------------------------------------------------------------ |
>   | 解决办法  | 1、服务器端禁止使用RC4加密算法。 2、客户端应在浏览器TLS配置中禁止RC4。 |
>   | 威胁分值  | 5                                                            |
>   | 危险插件  | 否                                                           |
>   | 发现日期  | 2015-03-29                                                   |
>   | CVE编号   | [CVE-2015-2808](http://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2015-2808 ) |
>   | BUGTRAQ   | [73684](http://www.securityfocus.com/bid/73684)              |
>   | NSFOCUS   | [30491](http://www.nsfocus.net/vulndb/30491)                 |
>   | CNNVD编号 | [CNNVD-201503-654](http://www.cnnvd.org.cn/web/xxk/ldxqById.tag?CNNVD=CNNVD-201503-654) |
>   | CNCVE编号 | CNCVE-20152808                                               |
>   | CNVD编号  | [CNVD-2015-02171](http://www.cnvd.org.cn/flaw/show/CNVD-2015-02171) |
>   | CVSS评分  | 5.3(CVSS:3.0/AV:N/AC:L/PR:N/UI:N/S:U/C:L/I:N/A:N)            |
>
>2. SLv3 存在严重设计缺陷漏洞 (CVE-2014-3566)
>
>   | 详细描述 | SSLv3漏洞（CVE-2014-3566），该漏洞贯穿于所有的SSLv3版本中，利用该漏洞，黑客可以通过中间人攻击等类似的方式(只要劫持到的数据加密两端均使用SSL3.0)，便可以成功获取到传输数据(例如cookies)。 针对此漏洞，需要服务器端和客户端均停用SSLv3协议。 |
>   | :------- | ------------------------------------------------------------ |

## 二：说明

------

首先说明一下，SSL 2 和 SSL 3 协议是两种过时的协议，原因是它们存在很严重的漏洞，所以我们要在服务端禁用 SSL 2 和 SSL 3 协议，以避免一些安全问题。

- SSL 2 协议：漏洞名为 DROWN（溺水攻击 / 溺亡攻击）。DROWN 漏洞可以利用过时的 SSL 2 协议来解密与之共享相同 RSA 私钥的 TLS 协议所保护的流量。
- SSL 3 协议：漏洞名为 POODLE（卷毛狗攻击）。POODLE 漏洞只对 CBC 模式的明文进行了身份验证，但是没有对填充字节进行完整性验证，攻击者窃取采用 SSL 3 版加密通信过程中的内容，对填充字节修改并且利用预置填充来恢复加密内容，以达到攻击目的。

关于更多基于 SSL/TLS 协议的漏洞，请查看这篇文章《[常见的几种 SSL/TLS 漏洞及攻击方式](https://www.123si.org/os/article/several-common-ssl-tls-vulnerabilities-and-attacks/)》。

在 Windows 系统中，服务器如果使用 Windows Server 2016 版本系统，那么恭喜你了，你可以省去禁用 SSL 2 和 SSL 3 协议的工作了，因为在此版本系统以后，微软已经默认禁用这两种协议了。如果你还不放心，下面有 SSL 服务器测试能帮你检测。其它版本系统，可参照下文介绍的 2 种设置方法来禁用协议。



## 三：SSL 服务器测试

------

使用下面两个测试网站，可以查看你的网站的安全状态。

- SSL Labs 网址：https://www.ssllabs.com/ssltest/index.html
- My SSL 网址：https://myssl.com/



##  四：处理

### 方法一： 禁用 SSL 2 和 SSL 3 协议

------

#### 1、通过修改注册表禁用协议

1. `Win + R` 键，打开运行，输入 regedit ，打开“注册表编辑器”。

2. 在注册表编辑器，找到以下注册表项/文件夹：`HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols`

3. 在“SSL 2.0”文件夹，右键单击并选择“新建”，然后单击“项(K)”。然后重命名文件夹为“Server”。

4. 右键点击“Sever”文件夹，选择“新建”，然后单击“DWORD（32-bit）值”。

5. 将新建的 DWORD 重命名为“Enabled”，并按下回车键或者双击查看。

6. 请确保它显示 0x00000000（0）。如果没有，请右键单击并选择修改，输入 0 作为数值数据。

7. 现在，禁用 SSL 3，对“SSL 3.0”文件夹，右键单击并选择“新建”，然后单击“项(K)”。命名新的文件夹为“Server”。

8. 右键点击“Sever”文件夹，选择“新建”，然后单击 DWORD（32-bit）值。

9. 将新建的 DWORD 重命名为“Enabled”，并按下回车键或者双击查看。

10. 请确保它显示 0x00000000（0）的数据列下。如果没有，请右键单击并选择修改，输入 0 作为数值数据。

    ![通过修改注册表禁用协议](D:\0_Notes\Hexo\hmxyl\source\_images\123si-org-img-1550666306359-16498366977241.jpg)

11. 重新启动计算机。

#### 2、使用 IIS Crypto 工具

IIS Crypto 是一个免费工具，使管理员能够在 Windows Server 2008，2012，2016 和 2019 上启用或禁用协议，密码，哈希和密钥交换算法。它还允许您重新排序 IIS 提供的 SSL / TLS 密码套件，更改高级设置，只需单击即可实施最佳实践，创建自定义模板并测试您的网站。

![IIS Crypto 工具](D:\0_Notes\Hexo\hmxyl\source\_images\123si-org-img-1550668068846-16498366977253.jpg)

使用这个工具，设置简单方便，功能也多，懒人必备啊。这个工具需要安装到服务器，并且需要管理员权限。

*IIS Crypto 工具网址：https://www.nartac.com/Products/IISCrypto/*

*参考文献：微软文档：[Protocols in TLS/SSL (Schannel SSP) - Windows applications](https://docs.microsoft.com/zh-cn/windows/desktop/SecAuthN/protocols-in-tls-ssl--schannel-ssp-)*

### 方法二： 开启Tomcat7的HTTPS访问

在方法一策略实施之后，扫描结果 SSL3.0 依然支持访问，也就意味着漏洞扫描结果依然存在，决定开启Tomcat7 的 https 访问方式。

   *参考文献： [CA证书服务配置Tomcat](https://blog.csdn.net/pang_ping/article/details/80604585)、[Tomcat7下对HTTPS的部署配置](https://www.cnblogs.com/bojuetech/p/6209657.html)*

#### 一、什么是CA证书，可以用来做什么，为啥大家都爱用？

​	云盾证书服务（Alibaba Cloud Certificates Service）是阿里云联合多家国内外知名 CA 证书厂商，在阿里云平台上直接提供服务器数字证书，阿里云用户可以在云平台上直接购买、甚至免费获取所需类型的数字证书，并一键部署在阿里云产品中，以最小的成本实现将所持服务从 HTTP 转换成 HTTPS。

​	其实按照个人理解简化说的话，CA可以帮助我们从HTTP转化为HTTPS，保证了中间传输数据的安全性。至于大家为啥都爱用，主要有两点：安全性和强制性。

​	安全性我们都知道，相比起HTTPS协议来说，HTTP协议是以明文方式发送内容，不提供任何方式的数据加密，如果攻击者拦截了Web浏览器和服务器之间的传输报文，便能直接知道里面的信息，因此http不适合传输一些含有敏感信息，例如：卡号、密码等。为了解决HTTP协议的这个缺陷，所以另一个协议就诞生了：安全套接字层超文本传输协议HTTPS，为了数据传输的安全，HTTPS在HTTP基础上面增加了SSL协议，SSL协议依靠证书来验证服务器的身份，并且为浏览器和服务器之间的通信加密，所以比起HTTP更多人用HTTPS的其中之一原因就是这样来的。



#### 二、如何申请CA证书服务（用户提供，跳过）

1. 进入控制台选择CA证书服务

   ![img](D:\0_Notes\Hexo\hmxyl\source\_images\70.png) 

2. 点击购买证书

   ![img](D:\0_Notes\Hexo\hmxyl\source\_images\70-16498375346979.png) 

3. 选择，我这里是选择免费的，所以我会点击一个域名，品牌使用Symantec，然后就会有一个免费型的DV出来，如果自己测试想要免费的话跟着我来就可以用了。

   ![img](D:\0_Notes\Hexo\hmxyl\source\_images\70-164983754971512.png) 





4. 点击立即购买

   ![img](D:\0_Notes\Hexo\hmxyl\source\_images\70-164983759067215.png) 

5. 选择支付

   ![img](D:\0_Notes\Hexo\hmxyl\source\_images\70-164983760383818.png) 

6. 进入证书控制台

   ![img](D:\0_Notes\Hexo\hmxyl\source\_images\70-164983761975221.png) 



7. 购买成功后会有一条记录，我们选择补全

   ![img](D:\0_Notes\Hexo\hmxyl\source\_images\70-164983763141024.png) 

8. 输入你的域名

   ![img](D:\0_Notes\Hexo\hmxyl\source\_images\70-164983764712027.png) 



9. 填入你的相关信息（建议：个人建议域名验证类型使用DNS不要选择文件，选择系统生成的CSR，由于小编自己的域名是买的腾讯云的，服务器是阿里的所以我就不点击证书绑定的域名了，如果服务器和域名都在阿里的话可以点击）然后下一步。

   ![img](D:\0_Notes\Hexo\hmxyl\source\_images\70-164983766359730.png) 

10. 提交审核，审核时间一天内就可以了 

    ![img](D:\0_Notes\Hexo\hmxyl\source\_images\70-164983767884933.png) 

#### 三、CA证书服务配置Tomcat



##### 准备证书文件

###### 自己拥有的证书（未参考，跳过）

- 选择审核通过的证书进行下载

  ![img](D:\0_Notes\Hexo\hmxyl\source\_images\70-164983854427836.png) 

- 我这里选择的是Tomcat，你们可以自行选择，下载压缩包

  ![img](D:\0_Notes\Hexo\hmxyl\source\_images\70-164983855342039.png) 

  ![img](D:\0_Notes\Hexo\hmxyl\source\_images\70-164983857428142.png) 

- 选择PFX格式的到Tomcat进行相关配置，跟着我上面操作的可以直接跳到第二步，完整信息配置需要记下，后面配置需要

  ![img](D:\0_Notes\Hexo\hmxyl\source\_images\70-164983859275545.png) 



###### 用户提供证书，转PFX格式

- 用户提供证书文件列表
  - XXXXX.pem
  - XXXXX.key



- 注册网站：https://app.certbase.com，利用证书工具将证书`.PEM`格式转换为`.pfx`格式

  ![image-20220413162330755](D:\0_Notes\Hexo\hmxyl\source\_images\image-20220413162330755.png)



- 转换并下载，获得文件列表
  - XXXXX.pfx
  - password.txt （保存的是页面设置PFX密码）



##### Tomcat配置证书

1. 将下载好的压缩包解压之后将所有的文件都放到`tomcat`目录下创建的`cert`目录中（目录名称自定义）

2. 点开server.xml文件, 添加配置

   ```xml
   <Connector port="443"
   		protocol="org.apache.coyote.http11.Http11Protocol"
   		SSLEnabled="true"
   		scheme="https"
   		secure="true"
   		keystoreType="PKCS12"
           keystoreFile="PFX所在地的完整路径"
           keystorePass="password.txt中的文本内容"
   		clientAuth="false"
   		SSLProtocol="TLSv1+TLSv1.1+TLSv1.2"
              ciphers="TLS_RSA_WITH_AES_128_CBC_SHA,TLS_RSA_WITH_AES_256_CBC_SHA,TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA,TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256,TLS_RSA_WITH_AES_128_CBC_SHA256,TLS_RSA_WITH_AES_256_CBC_SHA256"/>
   ```

   说明：

   | 配置文件参数 | 说明                                                         |
   | ------------ | ------------------------------------------------------------ |
   | clientAuth   | 如果设为true，表示Tomcat要求所有的SSL客户出示安全证书，对SSL客户进行身份验证 |
   | keystoreFile | 指定keystore文件的存放位置，可以指定绝对路径，也可以指定相对于<catalina_home> （Tomcat安装目录）环境变量的相对路径。<br/>如果此项没有设定，默认情况下，Tomcat将从当前操作系统用户的用户目录下读取名为 “.keystore”的文件。 |
   | keystorePass | 密钥库密码，指定keystore的密码。（如果申请证书时有填写私钥密码，密钥库密码即私钥密码） |
   | SSLProtocol  | 指定套接字（Socket）使用的加密/解密协议，默认值为TLS         |



3. http自动跳转https的安全配置（）

   到conf目录下的web.xml。在`</welcome-file-list>`后面，`</web-app>`，也就是倒数第二段里，加上这样一段

   ```xml
   <security-constraint><web-resource-collection >
       <web-resource-name >SSL</web-resource-name>
       <url-pattern>/*</url-pattern>
   </web-resource-collection>
   <user-data-constraint>
       <transport-guarantee>CONFIDENTIAL</transport-guarantee>
   </user-data-constraint></security-constraint>
   ```

   这步目的是让非ssl的connector跳转到ssl的connector去。所以还需要前往server.xml进行配置`80`端口跳转`443`：

   ```xml
   <Connector port="8080" protocol="HTTP/1.1"
       connectionTimeout="20000"
       redirectPort="443" />
   ```

4. 由于tomcat对ssl的实现由两种方式，tomcat7默认实现是APR方式，所以这里我们要对server.xml再进行相关修改

   ![img](D:\0_Notes\Hexo\hmxyl\source\_images\789495-20161222005744761-894166255.png) 

5. 重启Tomcat

6. 本地进行`https:// 域名`测试，是否可正常访问

7. 防火墙开放`443`端口出口访问

8. 需要网络管理员进行内网端口映射

   参考图：

   ![img](D:\0_Notes\Hexo\hmxyl\source\_images\2015010319370191.jpg) 



9. 外网访问`https://域名` 测试，是否可正常访问
10. 漏洞再次扫描：此时SSL 3 状态变更为关闭状态。
