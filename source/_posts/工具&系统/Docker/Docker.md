---
title: Docker日常使用记录
date: 2022-10-31 21:32:25
tags: 
  - Docker
categories: 
  - [工具&系统, Docker]
description: Docker日常使用记录
---





# Docker 升级到最新版本

## 升级步骤

### 1、查看系统要求

Docker 要求 CentOS 系统的内核版本高于 3.10 ,查看CentOS的内核版本。

```
uname -a
```

### 2、删除旧版本

```sh
yum remove docker  docker-common docker-selinux docker-engine
```

### 3、安装需要的软件包

yum-util 提供yum-config-manager功能，另外两个是devicemapper驱动依赖的

```sh
sudo yum install -y yum-utils device-mapper-persistent-data lvm2
```

### 4、设置Docker yum源

```sh
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```

### 5、查看所有仓库中所有docker版本

可以查看所有仓库中所有docker版本,并选择特定的版本安装。

```sh
sudo yum install docker-ce-18.06.1.ce  
```

### 7、启动

设置为开机启动

```sh
systemctl enable docker
```

启动

```sh
systemctl start docker
```

查看启动状态

```sh
systemctl status docker
```

查看版本

```sh
docker version
```



## 升级过程中的问题

1. 容器报错Unknown runtime specified docker-run

```html
[root@nginx discourse]# grep -rl 'docker-runc' /var/lib/docker/containers/ | xargs sed -i 's/docker-runc/runc/g'
[root@wxb-h5-weixin discourse]# systemctl restart docker
```

# 迁移Docker默认存储目录

Docker默认路径存储空间不足，迁移Docker默认存储目录

## 关掉所有正在运行的容器

```sh
# 关闭docker服务
docker stop $(docker ps -q -f status=running)
systemctl stop docker

# 将Docker现目录挪到一个新目录下，这两个目录依照具体情况而定，我的分别是/var/lib/docker和/home/docker
mv /var/lib/docker /home/docker

将原来的数据备份一份，备份大法好，万一不行还不至于损坏数据

cd /home
tar zcf docker_file_bak.tar.gz /home/docker
```

## 修改启动文件

```sh
# 修改服务启动命令，服务的service文件为/lib/systemd/system/docker.service，将里面的内容ExecStart=/usr/bin/dockerd修改为如下：

ExecStart=/usr/bin/dockerd -g /home/docker

# 重新加载修改后的service文件
systemctl daemon-reload
```

## 重启

```sh
# 启动Docker服务
systemctl start docker

验证修改成功
docker info | grep "Docker Root Dir"
```

# [IDEA远程一键部署Springboot到Docker](https://juejin.cn/post/6844903865192562696)

## 一、开发前准备

#### 1. Docker的安装可以参考https://docs.docker.com/install/

#### 2. 配置docker远程连接端口

```sh
  vi /usr/lib/systemd/system/docker.service
```

找到 **ExecStart**，在最后面添加 **-H tcp://0.0.0.0:2375**，如下图所示

![img](D:\0_Notes\Hexo\hmxyl\source\_images\16b5152965b0277atplv-t2oaga2asx-zoom-in-crop-mark1304000.webp) 

#### 3. 重启docker

```sh
systemctl stop docker
systemctl start docker
```

#### 4. 开放端口

```sh
firewall-cmd --zone=public --add-port=2375/tcp --permanent
```

#### 5. Idea安装插件,重启

![img](D:\0_Notes\Hexo\hmxyl\source\_images\16b5155b235261e1tplv-t2oaga2asx-zoom-in-crop-mark1304000.webp) 

#### 6. 连接远程docker

####    (1) 编辑配置



![img](D:\0_Notes\Hexo\hmxyl\source\_images\16b5156207039d0etplv-t2oaga2asx-zoom-in-crop-mark1304000.webp)    

![image-20220315094629072](D:\0_Notes\Hexo\hmxyl\source\_images\image-20220315094629072.png) 

####    (3) 连接成功，会列出远程docker容器和镜像

![img](D:\0_Notes\Hexo\hmxyl\source\_images\16b5156db1f07008tplv-t2oaga2asx-zoom-in-crop-mark1304000.webp) 

## 二、新建项目

#### 1. 创建springboot项目

     项目结构图

![img](D:\0_Notes\Hexo\hmxyl\source\_images\16b51572f7be11e0tplv-t2oaga2asx-zoom-in-crop-mark1304000.webp) 



####   (1) 配置pom文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>docker-demo</groupId>
    <artifactId>com.demo</artifactId>
    <version>1.0-SNAPSHOT</version>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.0.2.RELEASE</version>
        <relativePath />
    </parent>

    <properties>
         <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
         <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
         <docker.image.prefix>com.demo</docker.image.prefix>
         <java.version>1.8</java.version>
    </properties>
    <build>
        <plugins>
          <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
          </plugin>
        <plugin>
           <groupId>com.spotify</groupId>
           <artifactId>docker-maven-plugin</artifactId>
           <version>1.0.0</version>
           <configuration>
              <dockerDirectory>src/main/docker</dockerDirectory>
              <resources>
                <resource>
                    <targetPath>/</targetPath>
                    <directory>${project.build.directory}</directory>
                    <include>${project.build.finalName}.jar</include>
                </resource>
              </resources>
           </configuration>
        </plugin>
        <plugin>
            <artifactId>maven-antrun-plugin</artifactId>
            <executions>
                 <execution>
                     <phase>package</phase>
                    <configuration>
                        <target>
                            <copy todir="src/main/docker" file="target/${project.artifactId}-${project.version}.${project.packaging}"></copy>
                        </target>
                     </configuration>
                    <goals>
                        <goal>run</goal>
                    </goals>
                    </execution>
            </executions>
        </plugin>
       </plugins>
    </build>
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
  <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>log4j</groupId>
        <artifactId>log4j</artifactId>
        <version>1.2.17</version>
    </dependency>
</dependencies>
</project>
```

####   (2) 在src/main目录下创建docker目录，并创建Dockerfile文件

```sh
FROM openjdk:8-jdk-alpine
ADD *.jar app.jar
ENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"]
```

####   (3) 在resource目录下创建application.properties文件

```properties
logging.config=classpath:logback.xml
logging.path=/home/developer/app/logs/
server.port=8990
```

####   (4) 创建DockerApplication文件

```java
@SpringBootApplication
public class DockerApplication {
    public static void main(String[] args) {
        SpringApplication.run(DockerApplication.class, args);
    }
}
```

####   (5) 创建DockerController文件

```java
@RestController
public class DockerController {
    static Log log = LogFactory.getLog(DockerController.class);

    @RequestMapping("/")
    public String index() {
        log.info("Hello Docker!");
        return "Hello Docker!";
    }
}
```

####   (6) 增加配置

![img](D:\0_Notes\Hexo\hmxyl\source\_images\16b5161faed2393atplv-t2oaga2asx-zoom-in-crop-mark1304000.webp) 



![image-20220315095235903](D:\0_Notes\Hexo\hmxyl\source\_images\image-20220315095235903.png) 

- **Image tag :** 指定镜像名称和**tag**，镜像名称为 **docker-demo**，**tag**
- **Bind ports :** 绑定宿主机端口到容器内部端口。格式为[宿主机端口]:[容器内部端口]
- **Bind mounts :** 将宿主机目录挂到到容器内部目录中。格式为[宿主机目录]:[容器内部目录]。这个springboot项目会将日志打印在容器 **/home/developer/app/logs/** 目录下，将宿主机目录挂载到容器内部目录后，那么日志就会持久化容器外部的宿主机目录中。

#### (7) Maven打包

![img](D:\0_Notes\Hexo\hmxyl\source\_images\16b5167788e14ee1tplv-t2oaga2asx-zoom-in-crop-mark1304000.webp) 





####   (8) 运行

![img](D:\0_Notes\Hexo\hmxyl\source\_images\16b51679f663afe8tplv-t2oaga2asx-zoom-in-crop-mark1304000.webp) 

![img](D:\0_Notes\Hexo\hmxyl\source\_images\16b5167bec448fe4tplv-t2oaga2asx-zoom-in-crop-mark1304000.webp) 

先pull基础镜像，然后再打包镜像，并将镜像部署到远程docker运行 

![img](D:\0_Notes\Hexo\hmxyl\source\_images\16b5168992f0d1f4tplv-t2oaga2asx-zoom-in-crop-mark1304000.webp) 

这里我们可以看到镜像名称为docker-demo:1.1，docker容器为docker-server

####   (9) 运行成功

![img](D:\0_Notes\Hexo\hmxyl\source\_images\16b5168d1c05f997tplv-t2oaga2asx-zoom-in-crop-mark1304000.webp) 



####   (10) 浏览器访问

![img](D:\0_Notes\Hexo\hmxyl\source\_images\16b5168ed469a871tplv-t2oaga2asx-zoom-in-crop-mark1304000.webp) 

####   (11) 日志查看

![img](D:\0_Notes\Hexo\hmxyl\source\_images\16b516908d72a340tplv-t2oaga2asx-zoom-in-crop-mark1304000.webp) 





# 进入容器命令行

```sh
docker container ls
docker exec -it 3052dd731d05  bash
```



