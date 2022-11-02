---
title: MySQL安装、版本差异、JDBC连接配置
date: 2022-10-25 19:52:44
tags: 
  - MySQL
category: 
  - [数据库, MySQL]
description: 'MySQL安装、版本差异、JDBC连接配置'
---

# Windows安装（5.6）

1. . 准备

   拷贝文件夹：mysql-5.6.10-winx64到D:\ chaoxing \ software \目录下

2. 环境变量

   MYSQL_HOME: D:\ chaoxing \ software \ mysql-5.6.10-winx64（新建）

   path追加：;%MYSQL_HOME%\bin

3. 安装服务，并启动

   启动cmd：进入目录D:\ chaoxing \ software \ mysql-5.6.10-winx64\bin

   运行：mysqld install MySQL

   启动：net start MySQL

4. 修改数据库密码

   启动cmd：进入目录D:\ chaoxing \ software \ mysql-5.6.10-winx64\bin

   运行：mysql –u root

　　　mysql>show databases;

　　　mysql>use mysql;

　　　mysql>UPDATE user SET password=PASSWORD("**自定义密码**") WHERE user='root';

　　　mysql>FLUSH PRIVILEGES;

　　　mysql>QUIT

# LINUX安装

## 准备工作

1. 下载MySQL安装包

   下载路径：`https://dev.mysql.com/downloads/mirrors/`，点左侧，Other Downloads，选择需要的镜像下载。

   选择版本：mysql-5.7.23-linux-glibc2.12-x86_64.tar.gz

   或者：` wget https://dev.mysql.com/get/Downloads/MySQL-5.7/mysql-5.7.23-linux-glibc2.12-x86_64.tar.gz `直接下载

2. 上传& 解压  （rz -y 上传）压缩包存放路径：/opt

   ```sh
   tar -zxvf mysql-5.7.23-linux-glibc2.12-x86_64.tar.gz
   mv mysql-5.7.23-linux-glibc2.12-x86_64 mysql-5.7.23
   ```

## 安装MySQL

1. 安装依赖 

   ```sh
   yum install -y cmake make gcc gcc-c++ libaio ncurses ncurses-devel
   ```

2. 添加系统mysql组和mysql用户 

​		添加系统mysql组 `groupadd mysql`

​		添加mysql用户` useradd -r -g mysql mysql `（添加完成后可用id mysql查看）

3. 安装数据库

​		切到mysql目录： `cd /opt/mysql-5.7.23`

​		修改当前目录拥有者为mysql用户：` chown -R mysql:mysql ./`

​		安装数据库`bin/mysqld --initialize --user=mysql --basedir=/opt/mysql-5.7.23 --datadir=/opt/mysql-5.7.23/data`，  保存临时密码：*123123123123*

```sh
可能报这个错

bin/mysqld: error while loading shared libraries: *libaio.so.1:* cannot open shared object file: No such file or directory

解决方法
yum install -y libaio //安装后在初始化
```

​		执行以下命令创建RSA private key 

```sh
bin/mysql_ssl_rsa_setup --datadir=/opt/mysql-5.7.23/data
```

​		修改当前目录拥有者为mysql用户 `chown -R mysql:mysql ./`

​		修改当前data目录拥有者为mysql用户 `chown -R mysql:mysql data`

4. 配置my.cnf

   ```sh
   vi /etc/my.cnf
   ```

   ```properties
   [mysqld]
   character_set_server=utf8
   init_connect='SET NAMES utf8'
   basedir=/opt/mysql-5.7.23
   datadir=/opt/mysql-5.7.23/data
   socket=/tmp/mysql.sock
   
   #不区分大小写
   lower_case_table_names = 1
   
   #不开启sql严格模式
   sql_mode="STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION"
   
   log-error=/var/log/mysqld.log
   pid-file=/opt/mysql-5.7.23/data/mysqld.pid
   ```

   添加开机启动项

   ```sh
   cp /opt/mysql-5.7.23/support-files/mysql.server /etc/init.d/mysqld
   ```

   修改 ：  `vi /etc/init.d/mysqld `

   添加路径 

   ```pr
   basedir=/opt/mysql-5.7.23
   datadir=/opt/mysql-5.7.23/data
   ```

5. 启动mysql  ：`service mysqld start `

   ```sh
   加入开机起动 ： ` chkconfig --add mysqld  `
   ```

6. 登录修改密码 ：`mysql -uroot -p 上面初始化时的密码`（123123123123）

​		如果出现错误 需要添加软连接： ` ln -s /opt/mysql-5.7.23/bin/mysql /usr/bin`

​		或者，修改环境变量。

​         ![img](D:\0_Notes\Hexo\hmxyl\source\_images\clip_image001-16382968264995.png)    

​       第一件事先修改密码：

```sh
mysql>alter user 'root'@'localhost' identified by '修改后的密码';  

mysql>flush privileges;  #刷新权限
```

7. 重启后执行，如果看到有监听说明服务启动了：`netstat -na | grep 3306`

8. 防火墙

   ```sh
   # 设置
   firewall-cmd --zone=public --add-port=3306/tcp --permanent
   
   # 重新载入
   firewall-cmd --reload
   
   # 查看
   firewall-cmd --zone= public --query-port=3306/tcp 
   或 
   firewall-cmd --zone=public --list-ports
   ```

   

9. 设置mysql的远程登录

   ```sql
   # grant all privileges on 库.表 to 用户@'%' identified by '修改后的密码';
   
   mysql> grant all privileges on *.* to root@'%' identified by 'root密码';
   
   mysql> flush privileges;
   ```

   

# 版本差异

## 修改密码

MySQL5.6，

```sql
UPDATE user SET password=PASSWORD("gese45ew&20") WHERE user='root'; 
```



MySQL5.7 

```sql
update user set authentication_string = password('gese45ew&20') , password_expired = 'N', password_last_changed = now() where user = 'root';
```







## 创建索引

因为，MySQL5.5和MySQL5.0 之间，建索引的语句不一样 所以，直接拷贝5.5的sql语句，不能在5.0上运行 

方法： 

1. 删除索引之后，再拷贝数据 

   ```sql
   ALTER TABLE consult_userinfo DROP INDEX idx_user_info_userid ; 
   ```

2. 在5.0的数据库建索引 

   例如：

   ```sql
   CREATE INDEX idx_user_info_userid on test_table (`userid`); 
   CREATE INDEX index_author on test_table(`author`(255)); 
   CREATE INDEX index_orderid on test_table(`id`,`orderid`); 
   ```

3. 5.5建索引 

   ```sql
   ALTER TABLE `test_table` ADD INDEX index_name ( `column1`, `column2`, `column3`)
   ```



# JDBC连接配置

## JDBC连接配置参数（5.7）

| 参数                  | 说明                                                         | 默认值    | 常用值                    |
| --------------------- | ------------------------------------------------------------ | --------- | ------------------------- |
| autoReconnect         | 自动连接                                                     | false     | true                      |
| autoReconnectForPools | 自动连接连接池                                               | false     | true                      |
| characterEncoding     | 当useUnicode=true时，指定字符集                              |           | UTF-8                     |
| allowMultiQueries     | 在一条语句中，允许使用“;”来分隔多条查询                      | false     | true                      |
| failOverReadOnly      | 在autoReconnect模式下出现故障切换时，是否应将连接设置为“只读” | true      | false                     |
| useSSL                | 与服务器进行通信时使用SSL                                    | true      | false                     |
| useUnicode            | 是否使用Unicode                                              | false     | true                      |
| socketTimeout         | 数据库无返回时，应用等待时间（ms）。要大于等于数据库配置的Socket TimeOut的值 | 0         | 60000                     |
| serverTimezone        | 配置时区                                                     | 系统时区  | Asia/Shanghai <br>GMT%2B8 |
| zeroDateTimeBehavior  | 配置空值存入DataTime<br><br/>1.	exception：默认值，即抛出SQL state [S1009]. Cannot convert value....的异常<br/>2.	convertToNull：将日期转换成NULL值<br/><br/>3.	round：替换成最近的日期即0001-01-01 | exception | convertToNull             |

时区异常处理

错误信息

```
…  is unrecognized or represents more than one time zone
```

配置时区

```
jdbc:mysql://localhost:3306/db?useUnicode=true&characterEncoding=UTF-8&zeroDateTimeBehavior=convertToNull&useSSL=false&serverTimezone=Asia/Shanghai
```

## 数据库客户端超时设置

数据库客户端的超时主要可以分为JDBC超时/连接池超时/Statement超时/事务超时等。

### 超时配置的关系和层级示意图

![img](D:\0_Notes\Hexo\hmxyl\source\_images\be1022af-1837-42c2-ae32-9d7e0ec4aa2c.png)    

![img](D:\0_Notes\Hexo\hmxyl\source\_images\43530e52-774a-401b-8e8c-0e10e2cc0fdc.png) 

上图中，更上层的超时依赖于下层的超时，只有当较低层的超时机制正常工作，上层的超时才会正常。如果 JDBC 驱动程序的socket超时工作不正常，那么更上层的超时比如 Statement 超时和事务超时都不会正常工作。



### Transaction Timeout(事务超时)

​		transaction timeout一般存在于框架（Spring, EJB）或应用级。transaction timeout或许是个相对陌生的概念，简单地说，transaction timeout就是“statement Timeout * N（需要执行的statement数量） + @（垃圾回收等其他时间）”。

​		transaction timeout用来限制执行statement的总时长。

​		例如：假设执行一个statement需要0.1秒，那么执行少量statement不会有什么问题，但若是要执行100,000个statement则需要10,000秒（约7个小时）。这时，transaction timeout就派上用场了。EJB CMT (Container Managed Transaction)就是一种典型的实现，它提供了多种方法供开发者选择。但我们并不使用EJB，Spring的transaction timeout设置会更常用一些。在Spring中，你可以使用下面展示的XML或是在源码中使用@Transactional注解来进行设置。 

```xml
<tx:attributes>  
        <tx:method name="…" timeout="3"/>  
</tx:attributes>  
```

​		Spring提供的transaction timeout配置非常简单，它会记录每个事务的开始时间和消耗时间，当特定的事件发生时就会对消耗时间做校验，当超出timeout值时将抛出异常。 

　　Spring中，被保存在ThreadLocal里，这被称为事务同步（Transaction Synchronization），与此同时，事务的开始时间和消耗时间也被保存下来。当使用这种代理连接创建statement时，就会校验事务的消耗时间。EJB CMT的实现方式与之类似，其结构本身也十分简单。 

　　当你选用的容器或框架并不支持transaction timeout这一特性，你可以考虑自己来实现。transaction timeout并没有标准的API。Lucy框架的1.5和1.6版本都不支持transaction timeout，但是你可以通过使用Spring的Transaction Manager来达到与之同样的效果。 

　　假设某个事务中包含5个statement，每个statement的执行时间是200ms，其他业务逻辑的执行时间是100ms，那么transaction timeout至少应该设置为1,100ms（200 * 5 + 100）。



### Statement Timeout

​		statement timeout用来限制statement的执行时长，timeout的值通过调用JDBC的.sql.Statement.setQueryTimeout(int timeout) API进行设置。不过现在开发者已经很少直接在代码中设置，而多是通过框架来进行设置。 

　　以iBatis为例，statement timeout的默认值可以通过**map-config.xml**中的**defaultStatementTimeout** 属性进行设置。同时，你还可以设置sqlmap中select，insert，update标签的timeout属性，从而对不同sql语句的超时时间进行独立的配置。 

　　如果你使用的是Lucy1.5或1.6版本，通过设置**queryTimeout**属性可以在datasource层面对statement timeout进行设置。 

　　statement timeout的具体值需要依据应用本身的特性而定，并没有可供推荐的配置



#### 	QueryTimeout处理过程

1. 通过调用Connection的createStatement()方法创建statement 

2. 调用**statement**的executeQuery()方法 

3. **statement**通过自身connection将query发送给MySQL数据库 

4. **statement**创建一个新的timeout-execution线程用于超时处理

5. 5.1版本后改为每个connection分配一个timeout-execution线程 

6. 向timeout-execution线程进行注册 

7. 达到超时时间 

8. TimerThread调用JtdsStatement实例中的TsdCore.cancel()方法 

9. timeout-execution线程创建一个和statement配置相同的connection 

10. 使用新创建的connection向超时query发送cancel query（KILL QUERY “connectionId”） 

![image-20211201004107227](D:\0_Notes\Hexo\hmxyl\source\_images\image-20211201004107227.png) 

### JDBC的socket timeout

​		JDBC的 timeout在被突然停掉或是发生网络错误（由于设备故障等原因）时十分重要。由于TCP/IP的结构原因，socket没有办法探测到网络错误，因此应用也无法主动发现断开。如果没有设置socket timeout的话，应用在数据库返回结果前会无期限地等下去，这种连接被称为dead connection。 

　　为了避免dead connections，socket必须要有超时配置。socket timeout可以通过JDBC设置，socket timeout能够避免应用在发生网络错误时产生无休止等待的情况，缩短服务失效的时间。 

　　不推荐使用socket timeout来限制statement的执行时长，因此socket timeout的值必须要高于statement timeout，否则，socket timeout将会先生效，这样statement timeout就变得毫无意义，也无法生效。 

​		下面展示了socket timeout的两个设置项，不同的JDBC驱动其配置方式会有所不同。 

- socket连接时的timeout：通过**Socket.connect(SocketAddress endpoint, int timeout)**设置

- socket读写时的timeout：通过**Socket.setSoTimeout(int timeout)**设置

  

通过查看CUBRID，MySQL，MS SQL Server (JTDS)和Oracle的JDBC驱动源码，我们发现所有的驱动内部都是使用上面的2个API来设置socket timeout的。 

- connectTimeout和socketTimeout的默认值为0时，timeout不生效。



### 操作系统的socket timeout配置

​		如果不设置 timeout或connect timeout，应用多数情况下是无法发现网络错误的。因此，当网络错误发生后，在连接重新连接成功或成功接收到数据之前，应用会无限制地等下去。但是，通过本文开篇处的实际案例我们发现，30分钟后应用的连接问题奇迹般的解决了，这是因为操作系统同样能够对socket timeout进行配置。公司的Linux服务器将socket timeout设置为了30分钟，从而会在操作系统的层面对网络连接做校验，因此即使JDBC的socket timeout设置为0，由网络错误造成的问题的持续时间也不会超过30分钟。 

　　通常，应用会在调用Socket.read()时由于网络问题被阻塞住，而很少在调用Socket.write()时进入waiting状态，这取决于网络构成和错误类型。当Socket.write()被调用时，数据被写入到操作系统内核的缓冲区，控制权立即回到应用手上。因此，一旦数据被写入内核缓冲区，Socket.write() 调用就必然会成功。但是，如果系统内核缓冲区由于某种网络错误而满了的话，Socket.write()也会进入waiting状态。这种情况下，操作系统会尝试重新发包，当达到重试的时间限制时，将产生系统错误。在我们公司，重新发包的超时时间被设置为15分钟。 

### FAQ

　　Q1. 我已经使用Statement.setQueryTimeout()方法设置了查询超时，但在网络出错时并没有产生作用。 

　　➔ 查询超时仅在socket timeout生效的前提下才有效，它并不能用来解决外部的网络错误，要解决这种问题，必须设置JDBC的socket timeout。 

　　Q2. transaction timeout，statement timeout和 timeout和DBCP的配置有什么关系？ 

　　➔ 当通过DBCP获取时，除了DBCP获取连接时的waitTimeout配置以外，其他配置对JDBC没有什么影响。 

　　Q3. 如果设置了JDBC的socket timeout，那DBCP连接池中处于IDLE状态的连接是否也会在达到超时时间后被关闭？ 

　　➔ 不会。socket的设置只会在产生数据读写时生效，而不会对DBCP中的IDLE连接产生影响。当DBCP中发生新连接创建，老的IDLE连接被移除，或是连接有效性校验的时候，socket设置会对其产生一定的影响，但除非发生网络问题，否则影响很小。 



## JDBC安全链接警告



用JDBC连接Mysql 5.6的时候，log里面一直有如下的warning, 虽然并不是error，但是log里面在每次连接数据库的时候会一直打印这个warning.

```sql
WARN: Establishing SSL connection without server's identity verification is not recommended. According to MySQL 5.5.45+, 5.6.26+ and 5.7.6+ requirements SSL connection must be established by default if explicit option isn't set. For compliance with existing applications not using SSL the verifyServerCertificate property is set to 'false'. You need either to explicitly disable SSL by setting useSSL=false, or set useSSL=true and provide truststore for server certificate verification.
```

可以在JDBC的配置里面添加useSSL=false配置使用非SSL连接即可：

```properties
jdbc.url=jdbc:mysql://localhost:3306/test?useUnicode=true&characterEncoding=UTF-8&useSSL=false
```

备注：启用SSL加密连接后，性能必然会有下降

由于SSL开销较大的环节在建立连接，所以短链接的开销可能会更大，因此推荐使用长连接或者连接池的方式来减小SSL所带来的额外开销，不过好在MySQL的应用习惯大部分也是长连接的方式。

总结 

1.MySQL 5.7配置SSL要比5.6来的简单的多 

2.MySQL 5.7客户端默认开启SSL加密连接 

3.通常来说，开启SSL加密连接后，性能最大的开销在25%左右
