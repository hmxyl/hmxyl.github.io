---
title:  Tomcat日常使用记录
date: 2022-10-30 15:39:23
tags: 
  -  Tomcat
categories: 
  - [工具&系统,  Tomcat]
description: ' Tomcat日常使用记录'
---



# 使用问题记录

## 在请求目标中找到无效字符。有效字符在rfc 7230和rfc 3986中定义

错误信息：在请求目标中找到无效字符。有效字符在rfc 7230和rfc 3986中定义

原因：请求地址中存在不支持的特殊符号

解决： `relaxedPathChars="|{}[],%" relaxedQueryChars="|{}[],%"`

```xml
    <Connector port="80" protocol="HTTP/1.1"
               connectionTimeout="20000"relaxedPathChars="|{}[],%" relaxedQueryChars="|{}[],%"
               redirectPort="8443"  />
```



## 给tomcat配置外部资源路径（应用场景：web项目访问图片视频等资源）

​		对于一个web项目来说，除了文字之外，图片，视频等媒体元素也是其重要的组成部分。我们知道，web项目中如果用到大量的图片、视屏的资源，我们
通常的做法是只在数据库中存储图片、视频等资源的路径，web项目直接通过路径来引用到对应的资源，而不是把整张图片以流的形式存储在数据库中，
​		当然对于系统中没用用到大量图片，或是对图片质量要求不是很高的一些小图标，我们也可以直接采用留的形式或者用base64编码以longtext的形式存储到数据库中。
可以不必费时费力去配置这些资源的路径。但是他的弊端在于增加了数据库的压力，只适用于那种格局比较小，对数据库服务器性能没有太大要求的小项目中。
​		给tomcat配置外部资源路径的好处在于他大大减小了服务器以及数据库的压力，数据库中只需要存储资源的路径，把图片上传到tomcat外的指定文件夹中，提供给
tomcat中wabapps下的web项目引用。

找到tomcat安装目录下conf/server.xml文件，在<Host>......</Host>标签内添加配置：

```xml
<!-- video image resources fload-->
<Context docBase="C:\resources" reloadable="true" debug="0" path="/resources"/>
```

其中，docBase是文件夹的物理路径，path是该文件夹的访问路径。然后重启tomcat即可。

假设在resources文件夹下存在一张名为test.jpg的图片，此时只需要`http://localhost:8080/resources/test.jsp`即可访问到该图片资源。

- 安装tomcat 为服务

  ```bash
  cd /tomcat/bin,
  service.bat install
  ```

  $CATALINA_HOME：为系统环境变量。查看方式

  ```bash
  windows： echo %CATALINA_HOME% 
  linux ：  echo $CATALINA_HOME
  ```

  输入localhost:8080默认访问的是`$CATALINA_HOME/webapps/ROOT`



## Linux查看Tomcat服务

- 查看Tomcat的PID：` ps aux|grep tomcat`
- 查看线程 ：` ps -T -p 需要查看的PID`



##  500 Internal Server Error

原因1：是服务本身的问题（后台日志能查到原因）
原因2：是Nginx服务器的原因。

可参考原因： （未验证）
	1.上传的文件权限设置错误
	2.htaccess文件写入错误的代码



## java.lang.NoSuchMethodException: org.apache.catalina.deploy.WebXml addFilter

```
java.lang.NoSuchMethodException: org.apache.catalina.deploy.WebXml addFilter
  at org.apache.tomcat.util.IntrospectionUtils.callMethod1(IntrospectionUtils.java:855)
  at org.apache.tomcat.util.digester.SetNextRule.end(SetNextRule.java:201)
  at org.apache.tomcat.util.digester.Digester.endElement(Digester.java:1051)
  at com.sun.org.apache.xerces.internal.parsers.AbstractSAXParser.endElement(AbstractSAXParser.java:601)
  at com.sun.org.apache.xerces.internal.impl.XMLDocumentFragmentScannerImpl.scanEndElement(XMLDocumentFragmentScannerImpl.java:1774)
  at com.sun.org.apache.xerces.internal.impl.XMLDocumentFragmentScannerImpl$FragmentContentDriver.next(XMLDocumentFragmentScannerImpl.java:2930)
  at com.sun.org.apache.xerces.internal.impl.XMLDocumentScannerImpl.next(XMLDocumentScannerImpl.java:648)
  at com.sun.org.apache.xerces.internal.impl.XMLDocumentFragmentScannerImpl.scanDocument(XMLDocumentFragmentScannerImpl.java:510)
  at com.sun.org.apache.xerces.internal.parsers.XML11Configuration.parse(XML11Configuration.java:807)
  at com.sun.org.apache.xerces.internal.parsers.XML11Configuration.parse(XML11Configuration.java:737)
  at com.sun.org.apache.xerces.internal.parsers.XMLParser.parse(XMLParser.java:107)
  at com.sun.org.apache.xerces.internal.parsers.AbstractSAXParser.parse(AbstractSAXParser.java:1205)
  at com.sun.org.apache.xerces.internal.jaxp.SAXParserImpl$JAXPSAXParser.parse(SAXParserImpl.java:522)
  at org.apache.tomcat.util.digester.Digester.parse(Digester.java:1537)
  at org.apache.catalina.startup.ContextConfig.parseWebXml(ContextConfig.java:1825)
  at org.apache.catalina.startup.ContextConfig.webConfig(ContextConfig.java:1201)
  at org.apache.catalina.startup.ContextConfig.configureStart(ContextConfig.java:855)
  at org.apache.catalina.startup.ContextConfig.lifecycleEvent(ContextConfig.java:345)
  at org.apache.catalina.util.LifecycleSupport.fireLifecycleEvent(LifecycleSupport.java:119)
  at org.apache.catalina.util.LifecycleBase.fireLifecycleEvent(LifecycleBase.java:90) 
...
```

解决方法为：在Tomacat7的`context.xml`文件里的`<Context>`中加上`<Loader delegate="true" />`



## 验证码图片无法显示

服务器段错误信息：`java.lang.NoClassDefFoundError: Could not initialize class sun.awt.X11.XToolkit`

解决：
在`Tomcat/bin/catalina.sh` 中增加 `-Djava.awt.headless=true`
如下：`JAVA_OPTS="$JAVA_OPTS -Djava.awt.headless=true"`

![image-20211214011929661](D:\0_Notes\Hexo\hmxyl\source\_images\image-20211214011929661.png)      

原因：对于一个Java服务器来说经常要处理一些图形元素，例如地图的创建或者图形和图表等。

这些API基本上总是需要运行一个`X-server`以便能使用AWT（Abstract Window Toolkit，抽象窗口工具集）。



# 基础

## Tomcat的四种基于HTTP协议的Connector性能比较

今天在osc上看到对Tomcat的四种基于HTTP协议的Connector性能比较

具体内容如下：

```xml
<Connector port="8081" protocol="org.apache.coyote.http11.Http11NioProtocol" connectionTimeout="20000" redirectPort="8443"/>

<Connector port="8081" protocol="HTTP/1.1" connectionTimeout="20000" redirectPort="8443"/>


<Connector port="8081" protocol="HTTP/1.1" executor="tomcatThreadPool" connectionTimeout="20000" redirectPort="8443" />

<Connector port="8081" protocol="org.apache.coyote.http11.Http11NioProtocol" executor="tomcatThreadPool" connectionTimeout="20000"  redirectPort="8443" />
```

我们姑且把上面四种Connector按照顺序命名为 NIO, HTTP, POOL, NIOP

为了不让其他因素影响测试结果，我们只对一个很简单的jsp页面进行测试，这个页面仅仅是输出一个Hello World。假设地址是 http://tomcat1/test.jsp

我们依次对四种Connector进行测试，测试的客户端在另外一台机器上用ab命令来完成，

测试命令为： ab -c 900 -n 2000 http://tomcat1/test.jsp ，

最终的测试结果如下表所示(单位:平均每秒处理的请求数)：

 

| NIO                               | HTTP                                               | POOL                                                      | NIOP                                                         |
| --------------------------------- | -------------------------------------------------- | --------------------------------------------------------- | ------------------------------------------------------------ |
| 281                               | 65                                                 | 208                                                       | 365                                                          |
| 666                               | 66                                                 | 110                                                       | 398                                                          |
| 692                               | 65                                                 | 66                                                        | 263                                                          |
| 256                               | 63                                                 | 94                                                        | 459                                                          |
| 440                               | 67                                                 | 145                                                       | 363                                                          |
| NIO方式波动很大，但没有低于280 的 | Tomcat的默认配置HTTP的性能是很稳定，但是也是最差的 | 而POOL方式则波动很大，测试期间和HTTP方式一 样，不时有停滞 | NIOP是在NIO的基础上加入线程池，可能是程序处理更复杂了，因此性能不见得比NIO强 |

 

由于linux的内核默认限制了最大打开文件数目是1024，因此此次并发数控制在900。

 

尽管这一个结果在实际的网站中因为各方面因素导致，可能差别没这么大，例如受限于数据库的性能等等的问题。

但对我们在部署网站应用时还是具有参考价值的。

```xml
<Connector executor="tomcatThreadPool" port="8090" redirectPort="8443" protocol="org.apache.coyote.http11.Http11NioProtocol" compression="on" compressionMinSize="2048" enableLookups="false" acceptCount="1000" URIEncoding="UTF-8" connectionTimeout="40000" />
```

| 说明                     | 参数                                                  |
| ------------------------ | ----------------------------------------------------- |
| 连接器使用的线程池的名子 | executor="tomcatThreadPool"                           |
| 连接器端口               | port="8090"                                           |
| 连接器使用的传输方式     | protocol="org.apache.coyote.http11.Http11NioProtocol" |
| 传输时是否支持压缩       | compression="on"                                      |
| 压缩的大小               | compressionMinSize="2048"                             |

```xml
<Executor name="tomcatThreadPool" namePrefix="catalina-exec-" maxThreads="800" minSpareThreads="400" maxSpareThreads="700" />
```

| 线程池名       | name="tomcatThreadPool"     |
| -------------- | --------------------------- |
| 线程前缀       | namePrefix="catalina-exec-" |
| 最大产生线程数 | maxThreads="800"            |
| 最小初始线程数 | minSpareThreads="400"       |
| 最大初始线程数 | maxSpareThreads="700"       |



## Tomcat开启JMX监控

背景：Tomcat系统运行过程出现错误，需要打开JMX，添加对JVM的监控。Tomcat运行在CentOS中。

前提：监控端windows系统，安装JDK。



1. 服务器关闭Tomcat

   ```sh
    cd /opt/apache-tomcat-7.0.54/bin
   
   ./shutdown.sh
   ```

   

2. 进入Tomcat/bin目录，修改catalina.sh，找到如下内容“#—–Execute The Requested Command”，在其上添加以下配置，此配置不需要用户名、密码

   ```sh
   CATALINA_OPTS=”$CATALINA_OPTS
   
   -Dcom.sun.management.jmxremote
   
   -Djava.rmi.server.hostname=192.168.23.1
   
   -Dcom.sun.management.jmxremote.port=9999
   
   -Dcom.sun.management.jmxremote.ssl=false
   
   -Dcom.sun.management.jmxremote.authenticate=false”
   ```

   | 参数         | 说明                                     |
   | ------------ | ---------------------------------------- |
   | ip           | 是你要监控的tomcat所在服务器的ip地址。   |
   | 端口号       | 是你要开启的监控端口号。                 |
   | ssl          | false表示不使用ssl链接。                 |
   | authenticate | false表示不使用监控,即不需要用户名和密码 |

   以下方式需要配置用户名、密码

   ```sh
   CATALINA_OPTS=”$CATALINA_OPTS
   
   -Dcom.sun.management.jmxremote
   
   -Djava.rmi.server.hostname=192.168.23.1
   
   -Dcom.sun.management.jmxremote.port=9999
   
   -Dcom.sun.management.jmxremote.ssl=false
   
   -Dcom.sun.management.jmxremote.authenticate=true
   
   -Dcom.sun.management.jmxremote.password.file=../conf/jmxremote.password
   
   -Dcom.sun.management.jmxremote.access.file=../conf/jmxremote.access”
   ```

   | 参数          | 说明             |
   | ------------- | ---------------- |
   | authenticate  | true开启鉴权功能 |
   | access.file   | 权限文件路径     |
   | password.file | 密码文件路径     |



 

3. 当没有配置密码时，无需此操作。当启用密码后，根据上述配置，将 `JAVA_HOME/jre/lib/management`下面的`jmxremote.access`和`jmxremote.password.template`拷贝到Tomcat的conf目录下，并对两个文件做以下修改：

   jmxremote.password.template文件名修改为jmxremote.password

   修改两个文件的权限

```sh
chmod 600 jmxremote.access
chmod 600 jmxremote.password
```

修改jmxremote.access文件，将文件最后两行显示【monitorRole和controlRole】的注释取消，

其中monitorRole为只拥有只读权限的角色，

controlRole有更高权限：读写等。编辑完成后，保存。

![img](D:\0_Notes\Hexo\hmxyl\source\_images\clip_image001-16383503107176.png)  

 

修改jmxremote.password文件。同样将文件最后两行显示【monitorRole和controlRole】的注释取消，两个用户名后面的字符即密码，然后保存。

![img](D:\0_Notes\Hexo\hmxyl\source\_images\clip_image002-16383503107187.png)  

4. 服务器启动Tomcat

   ```sh
    cd /opt/apache-tomcat-7.0.54/bin
   
   ./startup.sh
   ```

5. 做完以上操作后，使用jdk自带工具jvisualvm.exe连接，工具目录如下：JAVA_HOME/bin，连接方式如下：

   右击“远程”，“添加远程主机”

​	   ![img](D:\0_Notes\Hexo\hmxyl\source\_images\clip_image003-16383503107188.png) ![img](D:\0_Notes\Hexo\hmxyl\source\_images\clip_image004-16383503107189.png) 

 	右击添加好的主机，“添加JMX连接”，根据配置信息，填写相应的端口、用户名、密码等信息

  ![img](D:\0_Notes\Hexo\hmxyl\source\_images\clip_image005-163835031071810.png)  ![img](D:\0_Notes\Hexo\hmxyl\source\_images\clip_image006-163835031071811.png)

6. 添加完成后，效果如下：

​     ![img](D:\0_Notes\Hexo\hmxyl\source\_images\clip_image008.jpg)  

 

7. 如有其他需求，可下载其他附件

  ![img](D:\0_Notes\Hexo\hmxyl\source\_images\clip_image009-163835031071812.png) 

 

如果需要独立的监控软件可下载：VisualVM

下载地址：http://visualvm.github.io/download.html

入门指南：https://visualvm.github.io/gettingstarted.html?VisualVM_1.3.9

 

## Tomcat的连接数与线程池

 参考： https://www.cnblogs.com/kismetv/p/7806063.html

## Tomcat 配置文件server.xml

参考： https://www.cnblogs.com/kismetv/p/7228274.html

 