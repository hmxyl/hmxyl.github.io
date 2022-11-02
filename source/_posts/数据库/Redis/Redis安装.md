---
title: Redis安装
date: 2022-10-25 19:52:44
tags: 
  - Redis
category: 
  - [数据库, Redis]
description: 'Redis安装'
---



# 单机安装：Windows

Redis 是一个开源（BSD许可）的，内存中的数据结构存储系统，它可以用作数据库、缓存和消息中间件。

## 下载地址：

```
windows版本： https://github.com/MSOpenTech/redis/releases
Linux版本：官网下载： http://www.redis.cn/
git下载：https://github.com/antirez/redis/releases
```

 我们现在讨论的是windows下的安装部署，目前windows下最新版本是：3.2.100。下载地址，提供多种下载内容，

 ```
Redis-x64-3.2.100.msi是在windows下，最简单的安装文件，方便，直接会将Redis写入windows服务。
Redis-x64-3.2.100.zip是需要解压安装的，接下来讨论的是这种。
Source code (zip) 源码的zip压缩版
Source code (tar.gz) 源码的tar.gz压缩版 
 ```

![8cc0c90bc2b723d9f6cdc3a1356559b9.png](D:\0_Notes\Hexo\hmxyl\source\_images\clip_image002-16383227748541.png)   



## 安装

解压安装将下载的Redis-x64-3.2.100.zip 解压到某个地址。

![5bfe10debe216c18d70d3d0e555be8ae.png](D:\0_Notes\Hexo\hmxyl\source\_images\clip_image004-16383227748552.png) 

启动命令通过cmd指定到该redis目录。
 使用命令：`redis-server.exe `启动服务

 ![263f236f685ad552e15c64422eda72d6.png](D:\0_Notes\Hexo\hmxyl\source\_images\clip_image005-16383227748554.png) 

 出现这种效果，表明启动服务成功。启动另一个cmd，在该redis目录下，使用命令：redis-cli.exe 启动客户端,连接服务器

![ff00ad0320f2dbe6e8d3a50aa8e8b1a8.png](D:\0_Notes\Hexo\hmxyl\source\_images\clip_image006-16383227748553.png)  

出现这种效果，表明启动客户度成功。

## 部署

 由于上面虽然启动了redis服务，但是，只要一关闭cmd窗口，redis服务就关闭了。所以，把redis设置为一个windows服务。安装之前，windows服务是不包含redis服务的 
 ![95da60801d83c5caf9c8286ecab1574f.png](D:\0_Notes\Hexo\hmxyl\source\_images\clip_image008-16383227748555.png) 
 安装为windows服务安装命令: 

```
redis-server.exe --service-install redis.windows.conf
```

使用命令，安装成功，如图所示


 ![cf536a1024ae8ece56703496ca29dbff.png](D:\0_Notes\Hexo\hmxyl\source\_images\clip_image009.png) 



最后的参数` --loglevel verbose`表示记录日志等级

安装之后，windows目前的服务列表


 ![ad9747b797fe689579ca4642902afe49.png](D:\0_Notes\Hexo\hmxyl\source\_images\clip_image011.png) 



常用的redis服务命令。

```sh
卸载服务：redis-server --service-uninstall
开启服务：redis-server --service-start
停止服务：redis-server --service-stop
重命名服务：redis-server --service-name name
```


 重命名服务，需要写在前三个参数之后

例如： The following would install and start three separate instances of Redis as a service:

以下将会安装并启动三个不同的Redis实例作服务：

```
redis-server --service-install --service-name redisService1 --port 10001
redis-server --service-start --service-name redisService1

redis-server --service-install --service-name redisService2 --port 10002
redis-server --service-start --service-name redisService2

redis-server --service-install --service-name redisService3 --port 10003
redis-server --service-start --service-name redisService3
```

四：测试启动服务

客户端命令

```sh
redis-server --service-start
```

精简模式

```sh
redis-cli.exe
```

指定模式

```sh
redis-cli.exe -h 127.0.0.1 -p 6379 -a requirepass
```

| 参数 | 说明                                                         |
| ---- | ------------------------------------------------------------ |
| -h   | 服务器地址                                                   |
| -p   | 指定端口号                                                   |
| -a   | 连接数据库的密码[可以在redis.windows.conf中配置]，默认无密码 |

![05c2f1e1f504cb3480c6a96e746c0554.png](D:\0_Notes\Hexo\hmxyl\source\_images\clip_image012.png) 
 安装测试成功。

# 单机安装：Centos

## 准备

1. 下载redis安装包

 ```sh
wget http://download.redis.io/releases/redis-4.0.6.tar.gz
 ```

![4500b2d7d6f71a55db9aef7a8050394e.png](D:\0_Notes\Hexo\hmxyl\source\_images\clip_image013.png)  

2. 解压压缩包

```sh
tar zxvf redis-4.0.6.tar.gz
```

 ![632ce9c44a953178bb01cedd24d4d8ab.png](D:\0_Notes\Hexo\hmxyl\source\_images\clip_image014.png)  

3. yum安装gcc依赖

```sh
yum -y install gcc
```

## 安装

1. 跳转到redis解压目录下

```sh
cd /usr/software/redis-4.0.6
```

2. 编译安装 

```sh
make
```

![9760df7be3b6d1534296047905b13e61.png](D:\0_Notes\Hexo\hmxyl\source\_images\clip_image015.png)  


 ```sh
将/usr/software/redis-4.0.6/src目录下的文件加到/usr/local/bin目录

cd src && make install
 ```

![4aaeefe75f7c20cb5039b355f7062e7f.png](D:\0_Notes\Hexo\hmxyl\source\_images\clip_image016.png)  

## 启动redis的三种方式

### 直接启动

```sh
先切换到redis src目录下

cd /usr/software/redis-4.0.6/src
```

直接启动redis

 ```
./redis-server
 ```

![277d11179a4ec94b22e978a0f10f1a36.png](D:\0_Notes\Hexo\hmxyl\source\_images\clip_image017.png)  
 如上图：redis启动成功，但是这种启动方式需要一直打开窗口，不能进行其他操作，不太方便。
 按 ctrl + c可以关闭窗口。

![19606b9997c671ce2722f2315ddff2dd.png](D:\0_Notes\Hexo\hmxyl\source\_images\clip_image018.png)  



### 以后台进程方式启动redis

1. 修改redis.conf文件

   ```
   cd /usr/software/redis-4.0.6
   vi redis.conf
   ```

2. 将 `daemonize no` 修改 `daemonize yes`

    ![4690346a97fc0b147c66c8220cfb6e70.png](D:\0_Notes\Hexo\hmxyl\source\_images\clip_image019.png) 

      ![e5b2377dbc25cbf8dbd5a97341b744db.png](D:\0_Notes\Hexo\hmxyl\source\_images\clip_image020.png) 

3. 指定redis.conf文件启动

   ```
   cd /usr/software/redis-4.0.6/src
   ./redis-server /usr/software/redis-4.0.6/redis.conf
   ```

4. 关闭redis进程

   ```
   ps -aux | grep redis
   ```

   ![4b9b6364ca42da21b0e27f1e1dc90824.png](D:\0_Notes\Hexo\hmxyl\source\_images\clip_image021.png) 

   ```
   kill -9 11753
   ```

   

### 设置redis为系统服务

   1. 在/etc目录下新建redis目录

      ```
      cd /etc
      mkdir redis
      ```

   2. 将 `/usr/software/redis-4.0.6/redis.conf `文件复制一份到/etc/redis目录下，并命名为6379.conf　　

      ```sh
      cp /usr/software/redis-4.0.6/redis.conf /etc/redis/6379.conf
      ```

   3. 将redis的启动脚本复制一份放到/etc/init.d目录下

      ```sh
      cp /usr/software/redis-4.0.6/utils/redis_init_script /etc/init.d/redisd
      ```

   4. 设置redis开机自启动

      ```sh
      cd /etc/init.d
      chkconfig redisd on
      ```

      

      出现问题1：

      ```
      service redisd does not support chkconfig　
      ```

       看结果是redisd不支持chkconfig
       解决方法：`vi /etc/init.d/redisd`，加入如下两行注释，保存退出

       ```
      # chkconfig: 2345 90 10
      # description: Redis is a persistent key-value database
       ```

      ![ea9dc06f591188ad0d2c8ff4c0e8c4be.png](D:\0_Notes\Hexo\hmxyl\source\_images\clip_image022.png)   

      注释的意思是：redis服务必须在运行级2，3，4，5下被启动或关闭，启动的优先级是90，关闭的优先级是10。

      ![48608ef900b2bd7ed0f5a08598d11463.png](D:\0_Notes\Hexo\hmxyl\source\_images\clip_image023.png)  

       再次执行开机自启命令，成功

      ```sh
      chkconfig redisd on
      ```

      现在可以直接以服务的形式启动和关闭redis了

      

      再次启动：

       ```
      service redisd start
       ```

       

      出现问题2：

       ![95d24c1fc0745a3f0e55b6140ee87462.png](D:\0_Notes\Hexo\hmxyl\source\_images\clip_image024.png)

      删除：`redis_6379.pid`后，再次执行：

       ![b401291681b760b35c2643e97dbddb25.png](D:\0_Notes\Hexo\hmxyl\source\_images\clip_image025.png)

   5. 关闭

      ```sh
      service redisd stop
      ```

### 配置密码登录

1. 修改配置文件，使用密码登录

   ```sh
   vi /usr/software/redis-4.0.6/redis.conf
   ```

    ![89a9f187a67da6a0bdc92f8f05ff54b1.png](D:\0_Notes\Hexo\hmxyl\source\_images\clip_image026.png)

   

   此时，访问redis客户端查询

    ![7742f0112fb76da057cc87316510da4d.png](D:\0_Notes\Hexo\hmxyl\source\_images\clip_image027.png)

   

   使用密码后，停止服务会报错：

   ```sh
   service redisd stop
   ```

   

    ![9d88c06e13e7f4116629ae69fdaecd03.png](D:\0_Notes\Hexo\hmxyl\source\_images\clip_image028.png)

   

2. 修改启动配置文件

   ```
   vi /etc/init.d/redisd
   
   
   将其中的
   CLIEXEC -p REDISPORT shutdown
   改为
   CLIEXEC -a "3448395502" -p REDISPORT shutdown
   ```

## 防火墙开放端口6179

1. 开启6179

   ```sh
   /sbin/iptables -I INPUT -p tcp --dport 6179 -j ACCEPT
   ```

2. 保存

   ```sh
   /etc/rc.d/init.d/iptables save
   ```

3. centos 7下执行

   ```sh
   service iptables save
   ```

   

```
 参考：
 https://blog.csdn.net/zc474235918/article/details/50974483
 https://www.cnblogs.com/aqicheng/p/11512153.html
```



# Redis集群安装

> 集群至少6个节点
>
> 1. 集群至少3个主节点
> 2. 每个主节点至少一个从节点（若一个主节点设置2个从节点，则需要9个节点）

## 1. redis.conf模板文件redis-cluster.tmpl

```
# redis端口
port ${PORT}
# 关闭保护模式
protected-mode no
# 开启集群
cluster-enabled yes
# 集群节点配置
cluster-config-file nodes.conf
# 超时
cluster-node-timeout 5000
# 集群节点IP host模式为宿主机IP
cluster-announce-ip 192.168.100.62
# 集群节点端口 6379 - 6384
cluster-announce-port ${PORT}
cluster-announce-bus-port 1${PORT}
# 开启 appendonly 备份模式
appendonly yes
# 每秒钟备份
appendfsync everysec
# 对aof文件进行压缩时，是否执行同步操作
no-appendfsync-on-rewrite no
# 当目前aof文件大小超过上一次重写时的aof文件大小的100%时会再次进行重写
auto-aof-rewrite-percentage 100
# 重写前AOF文件的大小最小值 默认 64mb
auto-aof-rewrite-min-size 64mb
# redis最多能用多少内存，如果不设置的话，redis会一直消耗完系统所有的内存
maxmemory 100m
# redis达到maxmemory后的内存回收策略，lfu比lru性能更好
maxmemory-policy volatile-lfu

requirepass 123456
```

## 2. 批量生成节点文件的批处理程序：redis-cluster-config.sh

```sh
#!/bin/bash  
for port in $(seq 6379 6384)
do 
 mkdir -p ./redis-cluster/${port}/conf; 
 mkdir -p ./redis-cluster/${port}/data; 
 PORT=${port} envsubst < ./redis-cluster.tmpl > ./redis-cluster/${port}/conf/redis.conf;
done
```

## 3. 创建集群文件夹：redis-cluster

## 4. 执行redis-cluster-config.sh，准备文件环境

## 5. 准备docker-compose文件

```yml
version: '3.7'

services:
  redis_1:
    image: 'daocloud.io/library/redis:6.0.4'
    container_name: redis_1
    privileged: true
    command:
     - /bin/bash
     - -c
     - |
      echo 551 > /proc/sys/net/core/somaxconn
      echo never > /sys/kernel/mm/transparent_hugepage/enabled
      touch /etc/sysctl.conf
      echo vm.overcommit_memory=1 > /etc/sysctl.conf
      echo net.core.somaxconn=551 > /etc/sysctl.conf
      redis-server /usr/local/etc/redis/redis.conf --loglevel debug
    volumes:
      - ./redis-cluster/6379/conf/redis.conf:/usr/local/etc/redis/redis.conf
      - ./redis-cluster/6379/data:/data
      - ./redis-cluster/6379/log:/log
    ports:
      - "6379:6379"
      - "16379:16379"
    environment:
      - TZ=Asia/Shanghai
      
  redis_2:
    image: 'daocloud.io/library/redis:6.0.4'
    container_name: redis_2
    privileged: true
    command:
     - /bin/bash
     - -c
     - |
      echo 551 > /proc/sys/net/core/somaxconn
      echo never > /sys/kernel/mm/transparent_hugepage/enabled
      touch /etc/sysctl.conf
      echo vm.overcommit_memory=1 > /etc/sysctl.conf
      echo net.core.somaxconn=551 > /etc/sysctl.conf
      redis-server /usr/local/etc/redis/redis.conf --loglevel debug
    volumes:
      - ./redis-cluster/6380/conf/redis.conf:/usr/local/etc/redis/redis.conf
      - ./redis-cluster/6380/data:/data
      - ./redis-cluster/6380/log:/log
    ports:
      - "6380:6380"
      - "16380:16380"
    environment:
      - TZ=Asia/Shanghai
      
  redis_3:
    image: 'daocloud.io/library/redis:6.0.4'
    container_name: redis_3
    privileged: true
    command:
     - /bin/bash
     - -c
     - |
      echo 551 > /proc/sys/net/core/somaxconn
      echo never > /sys/kernel/mm/transparent_hugepage/enabled
      touch /etc/sysctl.conf
      echo vm.overcommit_memory=1 > /etc/sysctl.conf
      echo net.core.somaxconn=551 > /etc/sysctl.conf
      redis-server /usr/local/etc/redis/redis.conf --loglevel debug
    volumes:
      - ./redis-cluster/6381/conf/redis.conf:/usr/local/etc/redis/redis.conf
      - ./redis-cluster/6381/data:/data
      - ./redis-cluster/6381/log:/log
    ports:
      - "6381:6381"
      - "16381:16381"
    environment:
      - TZ=Asia/Shanghai
      
      
  redis_4:
    image: 'daocloud.io/library/redis:6.0.4'
    container_name: redis_4
    privileged: true
    command:
     - /bin/bash
     - -c
     - |
      echo 551 > /proc/sys/net/core/somaxconn
      echo never > /sys/kernel/mm/transparent_hugepage/enabled
      touch /etc/sysctl.conf
      echo vm.overcommit_memory=1 > /etc/sysctl.conf
      echo net.core.somaxconn=551 > /etc/sysctl.conf
      redis-server /usr/local/etc/redis/redis.conf --loglevel debug
    volumes:
      - ./redis-cluster/6382/conf/redis.conf:/usr/local/etc/redis/redis.conf
      - ./redis-cluster/6382/data:/data
      - ./redis-cluster/6382/log:/log
    ports:
      - "6382:6382"
      - "16382:16382"
    environment:
      - TZ=Asia/Shanghai
      
      
  redis_5:
    image: 'daocloud.io/library/redis:6.0.4'
    container_name: redis_5
    privileged: true
    command:
     - /bin/bash
     - -c
     - |
      echo 551 > /proc/sys/net/core/somaxconn
      echo never > /sys/kernel/mm/transparent_hugepage/enabled
      touch /etc/sysctl.conf
      echo vm.overcommit_memory=1 > /etc/sysctl.conf
      echo net.core.somaxconn=551 > /etc/sysctl.conf
      redis-server /usr/local/etc/redis/redis.conf --loglevel debug
    volumes:
      - ./redis-cluster/6383/conf/redis.conf:/usr/local/etc/redis/redis.conf
      - ./redis-cluster/6383/data:/data
      - ./redis-cluster/6383/log:/log
    ports:
      - "6383:6383"
      - "16383:16383"
    environment:
      - TZ=Asia/Shanghai
      
  redis_6:
    image: 'daocloud.io/library/redis:6.0.4'
    container_name: redis_6
    privileged: true
    command:
     - /bin/bash
     - -c
     - |
      echo 551 > /proc/sys/net/core/somaxconn
      echo never > /sys/kernel/mm/transparent_hugepage/enabled
      touch /etc/sysctl.conf
      echo vm.overcommit_memory=1 > /etc/sysctl.conf
      echo net.core.somaxconn=551 > /etc/sysctl.conf
      redis-server /usr/local/etc/redis/redis.conf --loglevel debug
    volumes:
      - ./redis-cluster/6384/conf/redis.conf:/usr/local/etc/redis/redis.conf
      - ./redis-cluster/6384/data:/data
      - ./redis-cluster/6384/log:/log
    ports:
      - "6384:6384"
      - "16384:16384"
    environment:
      - TZ=Asia/Shanghai
```

## 6. 此时的文件结构

![image-20211223160142926](D:\0_Notes\Hexo\hmxyl\source\_images\image-20211223160142926.png)  

## 7. 生成容器

```sh
docker-compose up -d 
```

![image-20211223160512682](D:\0_Notes\Hexo\hmxyl\source\_images\image-20211223160512682.png) 

## 8. 集群配置

```sh
docker exec -it redis_1 redis-cli -p 6379 -a 123456 --cluster create 192.168.100.62:6379 192.168.100.62:6380 192.168.100.62:6381 192.168.100.62:6382 192.168.100.62:6383 192.168.100.62:6384  --cluster-replicas 1
```

![image-20211223160705022](D:\0_Notes\Hexo\hmxyl\source\_images\image-20211223160705022.png) 

## 9. 测试

```sh
docker exec -it redis_1 redis-cli -h 192.168.100.62 -p 6379 -a 123456 -c
```

1. 执行cluster info

![image-20211223160817584](D:\0_Notes\Hexo\hmxyl\source\_images\image-20211223160817584.png)  

2. 执行cluster nodes

![image-20211223160839571](D:\0_Notes\Hexo\hmxyl\source\_images\image-20211223160839571.png) 