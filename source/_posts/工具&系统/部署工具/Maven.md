---
title: Maven
date: 2022-10-31 21:32:25
tags: 
  - Maven
categories: 
  - [工具&系统, 部署工具, Maven]
description: Maven日常使用记录

---

#  不同版本Maven下载

`https://archive.apache.org/dist/maven/`

# Maven项目固定JDK版本

默认版本

Maven项目中，编译器和JRE的版本默认为1.5(所以Alt + F5刷新项目后，多个参数值会变成1.5)，参数如下(选中项目，Alt + Enter，查看项目属性)

```
Java Build Path下的Libraries下的JRE System Lirbrary的版本。
Java Compiler下的JDK Compiler的版本。
Maven下的Project Facts下的Java的版本。
```

永久设置maven项目的JDK版本，直接修改maven的setting.xml文件，在里面添加如下内容

```xml
<profile>
  <id>jdk-1.8</id>
  <activation>
    <activeByDefault>true</activeByDefault>
    <jdk>1.8</jdk>
  </activation>
  <properties>
    <maven.compiler.source>1.8</maven.compiler.source>
    <maven.compiler.target>1.8</maven.compiler.target>
    <maven.compiler.compilerVersion>1.8</maven.compiler.compilerVersion>
  </properties>
</profile>

```

配置properties节点下的maven编译器信息

```xml
<properties>
  <maven.compiler.source>1.8</maven.compiler.source>
  <maven.compiler.target>1.8</maven.compiler.target>
</properties>
```

配置build>plugins>plugins>configuration节点下的source和target节点值

```xml
<build>
  <plugins>
    <plugin>
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-compiler-plugin</artifactId>
      <version>2.3.2</version>
      <configuration>
        <source>1.8</source>
        <target>1.8</target>
      </configuration>
    </plugin>
  </plugins>
</build>
```



# 常用命令

| 参数                        | 说明                                                         |
| --------------------------- | ------------------------------------------------------------ |
| mvn artchetype:generate     | 使用交互式的方法建立工程，代替artchetype：create             |
| mvn compile                 | 编译工程                                                     |
| mvn clean                   | 清理生成的文件，一般与packageinstall等命令一起使用           |
| mvn test                    | 测试src/main/test下的文件                                    |
| mvn package                 | 生成target目录，编译、测试代码，生成测试报告，生成jar/war文件 |
| mvn install                 | 将工程打包并部署到本地库中                                   |
| mvn help:effective-pom      | 实际的pom文件，显示所有的默认配置                            |
| mvn help:effective-settings | 运行时使用setting文件                                        |
| mvn eclipse:eclipse         | 生成eclipse工程文件                                          |
| mvn help:describe           | 显示某个插件（目标）的功能                                   |
| mvn jetty:run               | 启动jetty容器，可以在测试时代替tomcat                        |
| mvn tomcat:run              | 启动tomcat容器                                               |
| mvn Debug                   | tomcat:run可以在eclipse中设置断点进行调试                    |
| mvn dependency:analyze      | 查看工程所依赖的插件，进行pom优化时可以用到                  |
| mvn dependency:sources      | 自动寻找并下载jar包的源码                                    |
| mvn dependency:resolved     | 查看已经解决的依赖                                           |
| mvn dependency:tree         | 查看依赖树，可以分析出间接依赖关系                           |
| mvn exec:java               | 运行指定的应用                                               |
| mvn assembly:assembly       | 生成一个单独的可运行的jar包                                  |

