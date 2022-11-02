---
title: Log4j日常使用记录
date: 2022-10-30 15:39:23
tags: 
  -  Log4j
categories: 
  - [框架&中间件, Log4j]
description: 'Log4j日常使用记录'
---



# log4j.properties 配置说明

```properties
log4J 日志信息log4j.properties配置说明
##logger是进行记录的主要类，appender是记录的方式,layout是记录的格式
#Logger 日志写出器，供程序员输出日志信息
#Appender 日志目的地，把格式化好的日志信息输出到指定的地方去
#ConsoleAppender 目的地为控制台的Appender
#FileAppender 目的地为文件的Appender
#RollingFileAppender 目的地为大小受限的文件的Appender
#Layout 日志格式化器，用来把程序员的logging request格式化成字符串
#PatternLayout 用指定的pattern格式化logging request的Layou
#Log4j提供的appender有以下几种：
#　　org.apache.log4j.ConsoleAppender（控制台），
#　　org.apache.log4j.FileAppender（文件），
#　　org.apache.log4j.DailyRollingFileAppender（每天产生一个日志文件），
#　　org.apache.log4j.RollingFileAppender（文件大小到达指定尺寸的时候产生一个新的文件），
#　　org.apache.log4j.WriterAppender（将日志信息以流格式发送到任意指定的地方）
#Log4j提供的layout有以下几种：
#　　org.apache.log4j.HTMLLayout（以HTML表格形式布局），
#　　org.apache.log4j.PatternLayout（可以灵活地指定布局模式），
#　　org.apache.log4j.SimpleLayout（包含日志信息的级别和信息字符串），
#　　org.apache.log4j.TTCCLayout（包含日志产生的时间、线程、类别等等信息）
#Log4J采用类似C语言中的printf函数的打印格式格式化日志信息，打印参数如下
# %m 输出代码中指定的消息
# %M 输出日志发生的方法名
#　　%p 输出优先级，即DEBUG，INFO，WARN，ERROR，FATAL
#　　%r 输出自应用启动到输出该log信息耗费的毫秒数
#　　%c 输出所属的类目，通常就是所在类的全名
#　　%t 输出产生该日志事件的线程名
#　　%n 输出一个回车换行符，Windows平台为“rn”，Unix平台为“n”
#　　%d 输出日志时间点的日期或时间，默认格式为ISO8601，也可以在其后指定格式，比如：%d{yyyy MMM dd HH:mm:ss,SSS}，输出类似：2002年10月18日 22：10：28，921
#　　%l 输出日志事件的发生位置，包括类目名、发生的线程，以及在代码中的行数。举例：Testlog4.main(TestLog4.java:10)
# %L 输出日志发生的位置
# %F 输出类名
#####################################################################
#设置级别和目的地 -- 把日志等级为debug的日志信息输出到stdout和SYS,QUERY这三个目的地
log4j.rootLogger=debug,STDOUT
# stdout:目的地 -- 打印到屏幕
## org.apache.log4j.ConsoleAppender:控制台
log4j.appender.STDOUT=org.apache.log4j.ConsoleAppender
## org.apache.log4j.PatternLayout:灵活地指定布局模式
log4j.appender.STDOUT.layout=org.apache.log4j.PatternLayout
## 上一句设置了PatternLayout灵活指定格式，则要指定打印参数 [%-5p][%d{HH:mm:ss}][%c-%M] %m%n [%-5p][%d{HH:mm:ss}][%l] %m%n
log4j.appender.STDOUT.layout.ConversionPattern=[%-5p][%d{yyyy-MM-dd HH:mm:ss sss}][%t][%c-%M][%L](%F:%L) - %m%n
# QUERY:目的地 -- 输出到文件(限定每个文件大小)
## 凡是 info、warn、error、fatal 级别的数据都会在这里执行输出到 query.log 日志文件中
##log4j.logger.QUERY=INFO,QUERY
log4j.logger.QUERY=INFO
##输出到文件(这里默认为追加方式)，使用org.apache.log4j.FileAppender：日志会在一个文件中追加
log4j.appender.QUERY=org.apache.log4j.RollingFileAppender
##设置文件输出路径;html:log/query.html
log4j.appender.QUERY.File=log/query.log
##设置文件输出样式;html格式： org.apache.log4j.HTMLLayout
log4j.appender.QUERY.layout=org.apache.log4j.PatternLayout
## 上一句设置了PatternLayout灵活指定格式，则要指定打印参数 [%-5p][%d{HH:mm:ss}][%l] %m%n
log4j.appender.QUERY.layout.ConversionPattern=[%-5p][%d{yyyy-MM-dd HH:mm:ss}][%c-%M] %m%n
## 指定文件的最大 大小
log4j.appender.QUERY.MaxFileSize=2048KB
## 可被备份的日志数
log4j.appender.QUERY.MaxBackupIndex=100
# SYS:目的地 -- 输出到文件(每天产生一个文件)
## 凡是 error、fatal 级别的数据都会在这里执行输出到 sys.log 日志文件中
#log4j.logger.SYS=error,SYS
log4j.logger.SYS=error
## org.apache.log4j.RollingFileAppender:每天产生一个日志文件
#使用org.apache.log4j.FileAppender：日志会在一个文件中追加
log4j.appender.SYS=org.apache.log4j.DailyRollingFileAppender
##设置文件输出路径 ${user.home}/log/sys.log
log4j.appender.SYS.File=log/sys.log
## org.apache.log4j.PatternLayout:灵活地指定布局模式
log4j.appender.SYS.layout=org.apache.log4j.PatternLayout
## 上一句设置了PatternLayout灵活指定格式，则要指定打印参数 [%-5p][%d{HH:mm:ss}][%l] %m%n
log4j.appender.SYS.layout.ConversionPattern=[%-5p][%d{HH:mm:ss}][%C-%M] %m%n
#设置特定包的级别
## 把com.swh.weixin包下的日志内容显示级别为debug,和目的地
## 把com.swh.weixin.util包下日志等级为debug的信息输出到pack 目的地
#log4j.logger.com.swh.weixin.util=debug,pack
##输出到文件(这里默认为追加方式)，使用org.apache.log4j.FileAppender：日志会在一个文件中追加
log4j.appender.pack=org.apache.log4j.RollingFileAppender
##设置文件输出路径 或者 ${user.home}/log/pack.log
log4j.appender.pack.File=log/pack.log
##设置文件输出样式
log4j.appender.pack.layout=org.apache.log4j.PatternLayout
## 上一句设置了PatternLayout灵活指定格式，则要指定打印参数 [%-5p][%d{HH:mm:ss}][%l] %m%n
log4j.appender.pack.layout.ConversionPattern=[%-5p][%d{yyyy MM dd HH:mm:ss}][%c-%M] %m%n
## 指定文件的最大 大小
log4j.appender.pack.MaxFileSize=1024KB
#日志最大备份数目
log4j.appender.pack.MaxBackupIndex=100
########################################################################
##设置级别和目的地
#log4j.rootLogger=debug,appender1,appender2
##只设置特定包的级别和目的地
#log4j.logger.com.coderdream=debug,appender1
#log4j.logger.com.coderdream.Dao=info,appender1,appender2
##输出到控制台
#log4j.appender.appender1=org.apache.log4j.ConsoleAppender
##设置输出样式
#log4j.appender.appender1.layout=org.apache.log4j.PatternLayout
##自定义样式
## %r 时间 0
## %t 方法名 main
## %p 优先级 DEBUG/INFO/ERROR
## %c 所属类的全名(包括包名)
## %l 发生的位置，在某个类的某行
## %m 输出代码中指定的讯息，如log(message)中的message
## %n 输出一个换行符号
#log4j.appender.appender1.layout.ConversionPattern=[%d{yy/MM/dd HH:mm:ss:SSS}][%C-%M] %m%n
##输出到文件(这里默认为追加方式)
#log4j.appender.appender2=org.apache.log4j.FileAppender
##设置文件输出路径
##【1】文本文件
#log4j.appender.appender2.File=c:/Log4JCRM_Dao.log
##设置文件输出样式
#log4j.appender.appender2.layout=org.apache.log4j.PatternLayout
#log4j.appender.appender2.layout.ConversionPattern=[%d{HH:mm:ss:SSS}][%C-%M] -%m%n
##把日志文件写入数据库
##########################日志输出到远程数据库########################################
##把日志文件写入数据库
##记录的日志级别
log4j.logger.db=info
##日志输出到数据库
log4j.appender.db = org.apache.log4j.jdbc.JDBCAppender
##缓存
log4j.appender.db.BufferSize = 0
##数据库驱动
log4j.appender.db.Driver = com.mysql.jdbc.Driver
##数据url地址 ，本地可简写：jdbc:mysql:///test
log4j.appender.db.URL = jdbc:mysql://localhost:3306/swh_hibernate4?useUnicode=true&characterEncoding=utf8
##数据库用户名
log4j.appender.db.User = root
##数据库密码
log4j.appender.db.Password = root
##日志布局模式
log4j.appender.db.layout = org.apache.log4j.PatternLayout
##日志插入数据库中，t_logs 表字段可自定义
log4j.appender.db.layout.ConversionPattern = INSERT INTO t_logs(createDate, thread, priority, category,<br /> methodName, message) values('%d', '%t', '%-5p', '%c','%M', '[%l]-%m')
```

# 打印SQL语句

```properties
log4j.logger.java.sql.Connection=DEBUG
log4j.logger.java.sql.Statement=DEBUG
log4j.logger.java.sql.PreparedStatement=DEBUG
log4j.logger.java.sql.ResultSet=DEBUG
```

