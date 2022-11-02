---
title: MySQL基础
date: 2022-10-25 19:52:44
tags: MySQL
category: 
  - [数据库, MySQL]
description: 'MySQL基础'
---

# 字段加密/解密

1. PASSWORD(明文)：

   创建一个经过加密的密码字符串，适合于插入到MySQL的安全系统。该加密过程不可逆，和unix密码加密过程使用不同的算法。主要用于MySQL的认证系统。

2. AES_ENCRYPT(明文，加密串)  AES_DECRYPT( 密文, 加密串 )

   使用UNIX crypt()系统加密字符串，AES_ENCRYPT()函数接收要加密的字符串和（可选的）用于加密过程的salt（一个可以唯一确定口令的字符串，就像钥匙一样）。加密程度比ENCODE较强。

   ```sql
    举例：
    
    加密
      SELECT HEX(AES_ENCRYPT('测试', '29a70b6c')) FROM DUAL; 
      输出：AFA3016D4EE259FE76D0D625F9BDF889
      
    解密
      SELECT CONVERT(AES_DECRYPT(UNHEX('AFA3016D4EE259FE76D0D625F9BDF889'), '29a70b6c') USING utf8) FROM DUAL;
      输出：测试 （CONVERT，字符集转换）
   ```

3. ENCODE(明文, 加密串)  DECODE(密文, 加密串)
   加密解密字符串。该函数有两个参数：被加密或解密的字符串和作为加密或解密基础的密钥。Encode结果是一个二进制字符串，以BLOB类型存储。加密成度相对比较弱。

4. MD5()：计算字符串的MD5校验和（128位），SHA5()：计算字符串的SHA5校验和（160位）

   这两个函数返回的校验和是16进制的，适合与认证系统中使用的口令。

   

# CONCAT函数

```sql
注意：和NULL连接的结果为NULL

SELECT CONCAT('test', '-', '1') FROM DUAL; 返回：test-1
SELECT CONCAT('test', '-', NULL) FROM DUAL; 返回：NULL
```



# IP地址/Long数据

```sql
 SELECT INET_ATON('112.253.20.48');->1895633968
 SELECT INET_NTOA('1895633968')->112.253.20.48
```



# 日期函数

## date_format参数

| 参数（年） |          | 参数（月） |                    | 参数（日） |                        |
| ---------- | -------- | ---------- | ------------------ | ---------- | ---------------------- |
| %Y         | 年，4 位 | %b         | 缩写月名           | %D         | 带有英文前缀的月中的天 |
| %y         | 年，2 位 | %c         | 月份的数值（1-12） | %d         | 月的天，数值(00-31)    |
|            |          | %M         | 月名               | %e         | 月的天，数值(0-31)     |
|            |          | %m         | 月，数值(01-12)    | %j         | 年的天 (001-366)       |

| 参数（时间） |                                    | 参数（时） |              | 参数（分） |                   | 参数（秒） |                   | 参数（微秒） |      |
| ------------ | ---------------------------------- | ---------- | ------------ | ---------- | ----------------- | ---------- | ----------------- | ------------ | ---- |
| %T           | 时间，24-小时 (hh:mm:ss)           | %H         | 小时 (00-23) | %i         | 分钟，数值(00-59) | %S         | 或者 %s 秒(00-59) | %f           | 微秒 |
| %r           | 时间，12-小时（hh:mm:ss AM 或 PM） | %h         | 小时 (01-12) |            |                   |            |                   |              |      |
| %p           | AM 或 PM                           | %I         | 小时 (01-12) |            |                   |            |                   |              |      |
|              |                                    | %k         | 小时 (0-23)  |            |                   |            |                   |              |      |

| 参数（年天） |                                                | 参数（周天） |                                               | 参数（星期） |                               |
| ------------ | ---------------------------------------------- | ------------ | --------------------------------------------- | ------------ | ----------------------------- |
| %X           | 年，其中的星期日是周的第一天，4 位，与 %V 使用 | %U           | 年周 (00-53) 星期日是一周的第一天             | %a           | 缩写星期名                    |
| %x           | 年，其中的星期一是周的第一天，4 位，与 %v 使用 | %u           | 年周 (00-53) 星期一是一周的第一天             | %W           | 星期名                        |
|              |                                                | %V           | 年周 (01-53) 星期日是一周的第一天，与 %X 使用 | %w           | 周的天 （0=星期日, 6=星期六） |
|              |                                                | %v           | 年周 (01-53) 星期一是一周的第一天，与 %x 使用 |              |                               |



| 函数                                | 举例                                                         | 说明                                                         |
| ----------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| CURDATE()  <br>CURRENT_DATE()       | SELECT CURDATE()<br/> 2021-11-30                             | 返回当前的日期（%Y-%m-%d）                                   |
| CURTIME()  <br/>CURRENT_TIME()      | SELECT CURTIME();<br>18:34:12                                | 返回当前的时间（%T或者%H:%i:%s）                             |
| DATE_ADD(date,INTERVAL expr unit)   | SELECT DATE_ADD(CURRENT_DATE,INTERVAL 6 MONTH);<br>2022-05-30 | 返回日期date加上间隔时间int的结果<br>(int必须按照关键字进行格式化) |
| DATE_SUB(date,INTERVAL int keyword) | SELECT DATE_SUB( CURRENT_DATE, INTERVAL 6 MONTH );<br>2021-05-30 | 返回日期date加上间隔时间int的结果<br/>(int必须按照关键字进行格式化) |
| PERIOD_DIFF(P1,P2)                  | # 月份差值<br>SELECT PERIOD_DIFF(date_format('2021-09-30', '%Y%m'), date_format('2021-06-20', '%Y%m'))<br><br/>#天数差值<br>SELECT PERIOD_DIFF(date_format('2021-09-30', '%Y%m%d'), date_format('2021-06-20', '%Y%m%d'))<br><br/># 上一个月<br/> SELECT * FROM 表名 WHERE PERIOD_DIFF(date_format(now(),'%Y%m'),date_format(时间字段名,'%Y%m') =1 | 计算两个日期之间的差值                                       |
| TO_DAYS(date)                       | \# 今天<br/>SELECT * FROM 表名 WHERE TO_DAYS(时间字段名) =TO_DAYS(NOW());<br/><br/> # 昨天<br/>SELECT * FROM 表名 WHERE TO_DAYS(NOW()) - TO_DAYS( 时间字段名) <= 1 | 日期转天数                                                   |
| YEARWEEK                            | SELECT YEARWEEK(now()) <br/>返回：202148<br/><br/>\# 本周<br/>SELECT * FROM 表名 WHERE YEARWEEK( date_format( 时间字段名,'%Y-%m-%d' ) ) = YEARWEEK( now() ) ; |                                                              |
| WEEK                                |                                                              | 返回日期date为一年中第几周(0~53)                             |
| DAYOFWEEK                           |                                                              | 返回date所代表的一星期中的第几天(1~7)                        |
| DAYOFMONTH                          |                                                              | 返回date是一个月的第几天(1~31)                               |
| DAYOFYEAR(date)                     |                                                              | 返回date是一年的第几天(1~366)                                |
| DAYNAME                             | SELECT DAYNAME(CURRENT_DATE);<br>返回：Tuesday               | 返回date的星期名                                             |
| FROM_UNIXTIME(ts,fmt)               |                                                              | 根据指定的fmt格式，格式化UNIX时间戳ts                        |
| YEAR                                | \# 本年<br/> SELECT * FROM 表名 WHERE YEAR( 时间字段名 ) = YEAR( NOW( ) ) |                                                              |
| MONTH                               |                                                              | 返回date的月份(1~12)                                         |
| MONTHNAME                           |                                                              | 返回date的月份名                                             |
| HOUR                                |                                                              | 返回time的小时值(0~23)                                       |
| MINUTE                              |                                                              | 返回time的分钟值(0~59)                                       |
| QUARTER                             | SELECT QUARTER(CURRENT_DATE);<br>返回：4                     | 返回date在一年中的季度(1~4)                                  |



# MySQL外键设置

MySQL外键设置: Cascade、NO ACTION、Restrict、SET NULL

| 参数        | 说明                                                         |
| ----------- | ------------------------------------------------------------ |
| cascade     | 在父表上update/delete记录时，同步update/delete掉子表的匹配记录 |
| set null    | 在父表上update/delete记录时，将子表上匹配记录的列设为null，要注意子表的外键列不能为not null |
| No action   | 如果子表中有匹配的记录,则不允许对父表对应候选键进行update/delete操作 |
| Restrict    | 同no action, 都是立即检查外键约束                            |
| Set default | 父表有变更时,子表将外键列设置成一个默认的值 但Innodb不能识别 |

# 查看/修改系统参数

##  版本号

```sql
select version();// 查看的是innodb_version
```

## 查看数据库运行中的进程

```sql
show full processlist  或
selec * from processlist
```



## 查看系统运行状态

性能优化的时候可参考

```sql
show status
```

## 系统参数配置

```sql
show variables
```

## 最大连接数

查看

```sql
select @@global.max_connections
```

修改

```
set global max_connections=1024;
```

## 配置 sql_mode

查看

```sql
# 全局配置
select @@global.sql_mode; 
# 已经存在的数据库sql_mode
select @@sql_mode;
```

修改（建议先查后改）

```sql
# 改变已经存在的数据库sql_mode 
set sql_mode=' ' 

# 改变全局配置sql_mode
set @@global.sql_mode=' ' 
```

## 系统参数总表参考

查询

```sql
SHOW VARIABLES;
SHOW VARIABLES LIKE 'autocommit';
```



## 配置执行语句长度限制

配置不够，执行错误提示

```sql
ERROR:The size of BLOB/TEXT data inserted in one transaction is greater than  10% of redo log size. Increase the redo log size using innodb_log_file_size.
```

查看配置

```sql
select @@global.max_allowed_packet
```

修改配置（临时）

```sql
set global max_allowed_packet  = 1024*1024*1024;
```

修改配置（永久）

```
在mysqld下面添加配置max_allowed_packed=1024M  
```





# 批量执行insert语句，进入了阻塞状态

原因一：磁盤空間已滿

原因二：`innodb_flush_log_at_trx_commit`是配置MySql日志何时写入硬盘的参数：

- 参数值说明

| 参数 | 说明                                                         |
| ---- | ------------------------------------------------------------ |
| 0    | log buffer将每秒一次地写入log file中，并且log  file的flush(刷到磁盘)操作同时进行。该模式下在事务提交的时候，不会主动触发写入磁盘的操作。<br>当设置为0，该模式速度最快，但不太安全，mysqld进程的崩溃会导致上一秒钟所有事务数据的丢失 |
| 1    | 每次事务提交时MySQL都会把log buffer的数据写入log  file，并且flush(刷到磁盘)中去，该模式为系统默认。<br/><br/>当设置为1，该模式是最安全的，但也是最慢的一种方式。在mysqld 服务崩溃或者服务器主机crash的情况下，binary log 只有可能丢失最多一个语句或者一个事务 |
| 2    | 每次事务提交时mysql都会把log buffer的数据写入log  file，但是flush(刷到磁盘)操作并不会同时进行。<br/>该模式下，MySQL会每秒执行一次 flush(刷到磁盘)操作<br/><br/>当设置为2，该模式速度较快，也比0安全，只有在操作系统崩溃或者系统断电的情况下，上一秒钟所有事务数据才可能丢失 |

解决

```sql
innodb_flush_log_at_trx_commit=2，sync_binlog=500 或1000 
```

​		查找资料时候看到其他文章说innodb_flush_log_at_trx_commit和sync_binlog 两个参数是控制MySQL 磁盘写入策略以及数据安全性的关键参数，当两个参数都设置为1的时候写入性能最差。

​		推荐做法是innodb_flush_log_at_trx_commit=2，sync_binlog=500 或1000 





