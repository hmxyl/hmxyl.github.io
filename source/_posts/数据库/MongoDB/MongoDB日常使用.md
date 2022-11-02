---
title: MongoDB日常使用
date: 2022-10-25 19:52:44
tags: MongoDB
category: 
  - [数据库, MongoDB]
description: 'MongoDB日常使用'
---

# Spring Boot集成Mongodb在控制台输出nosql的日志

大家只需要在`application.properties`的配置文件下增加以下的配置就可以了

```properties
logging.level.org.springframework.data.mongodb.core = DEBUG
```



```properties
###############log4j 4 SQL Output start################# 
log4j.logger.com.dayainfo.ssp=DEBUG 
log4j.logger.com.springframework=DEBUG 
log4j.logger.com.ibatis=DEBUG 
log4j.logger.com.ibatis.common.jdbc.SimpleDataSource=DEBUG 
log4j.logger.com.ibatis.common.jdbc.ScriptRunner=DEBUG 
log4j.logger.com.ibatis.sqlmap.engine.impl.SqlMapClientDelegate=DEBUG 
log4j.logger.java.sql.Connection=DEBUG 
log4j.logger.java.sql.Statement=DEBUG 
log4j.logger.java.sql.PreparedStatement=DEBUG 
log4j.logger.java.sql.ResultSet=DEBUG 
log4j.logger.org.apache.ibatis.logging.commons.JakartaCommonsLoggingImpl=DEBUG 
log4j.logger.java.sql=DEBUG,CONSOLE 
log4j.appender.CONSOLE = org.apache.
log4j.ConsoleAppender 
log4j.appender.CONSOLE.Target = System.out 
log4j.appender.CONSOLE.layout = org.apache.
log4j.PatternLayout 
log4j.appender.CONSOLE.layout.ConversionPattern =%d{ABSOLUTE} %5p %c{1}\:%L - %m%n 
###############log4j 4 SQL Output end###################
```

