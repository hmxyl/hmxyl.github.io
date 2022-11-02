---
title:  Logback日常使用记录
date: 2022-10-30 15:39:23
tags: 
  -  Logback
categories: 
  - [框架&中间件,  Logback ]
description: ' Logback日常使用记录'
---



# Logback 配置Demo

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration> 
  <appender name="consoleLog" class="ch.qos.logback.core.ConsoleAppender"> 
    <layout class="ch.qos.logback.classic.PatternLayout"> 
      <pattern>%d - %msg%n</pattern> 
    </layout> 
  </appender>  
  <appender name="fileInfoLog" class="ch.qos.logback.core.rolling.RollingFileAppender"> 
    <filter class="ch.qos.logback.classic.filter.LevelFilter"> 
      <level>ERROR</level>  
      <onMatch>DENY</onMatch>  
      <onMismatch>ACCEPT</onMismatch> 
    </filter>  
    <encoder> 
      <pattern>%d - %msg%n</pattern> 
    </encoder>  
    <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy"> 
      <fileNamePattern>C:\Users\hots_\Downloads\info.%d.log</fileNamePattern> 
    </rollingPolicy> 
  </appender>  
  <appender name="fileErrorLog" class="ch.qos.logback.core.rolling.RollingFileAppender"> 
    <filter class="ch.qos.logback.classic.filter.ThresholdFilter"> 
      <level>ERROR</level> 
    </filter>  
    <encoder> 
      <pattern>%d - %msg%n</pattern> 
    </encoder>  
    <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy"> 
      <fileNamePattern>C:\Users\hots_\Downloads\error.%d.log</fileNamePattern> 
    </rollingPolicy> 
  </appender>  
  <root level="info"> 
    <appender-ref ref="consoleLog"/>  
    <appender-ref ref="fileInfoLog"/>  
    <appender-ref ref="fileErrorLog"/> 
  </root> 
</configuration>
```



 