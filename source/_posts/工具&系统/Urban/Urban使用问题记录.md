---
title:  Urban使用问题记录
date: 2022-10-30 15:39:23
tags: 
 -  系统配置
categories: 
 - [系统,  Urban , 系统配置]
---

# 基础工具安装

- apt-get -y install wget        (wget)
- apt-get -y install net-tools   (ifconfig)

- snap install curl



<!-- more -->

# Urban 首次登录切换root

su切换至root权限时报错`su: Authentication failure`

分析原因：**可能是初次使用此命令，需要更新root密码**

解决方法：执行`sudo passwd root`命令，完成后再次输入su即可切换权限

![image-20220505102534819](D:\0_Notes\Hexo\hmxyl\source\_images\image-20220505102534819.png) 



# 安装软件

```sh
teamviewer_linux.deb

sudo dpkg --install teamviewer_linux.deb
```



# 安装ssh服务器端

- 执行`apt-get install openssh-server` ，安装服务端

- 允许远程使用root账号ssh登入

  修改/etc/ssh/sshd_config文件，修改如下：

  ```sh
  #PermitRootLogin prohibit-password
  PermitRootLogin yes
  ```

  需要重启系统或者sshd服务

  - sudo /etc/init.d/ssh stop
  - sudo /etc/init.d/ssh start
  - sudo service ssh start

- 开机启动`sudo systemctl enable ssh`

- 重启之后，`/usr/bin/xauth: file /root/.Xauthority does not exist` 错误消失



# Ubuntu中防火墙的使用和开放、关闭端口

## 查看防火墙的状态

```sh
sudo ufw status
```

`inactive`表示防火墙没有开启，并不是没有安装防火墙。

安装防火墙（Ubuntu系统默认是安装了ufw防火墙的）：

```
sudo apt-get install ufw
```

## Ubuntu开启防火墙

```sh
sudo ufw enable
```

命令可能会中断现有的ssh连接。继续操作(y|n)?

因为是在远程的Xshell进行连接开启防火墙的，有的系统是没有将SSH的22端口设置为public的，所以会有这样的提示.

这里分为两种情况，如果开启防火墙时在防火墙之中检测到22端口已添加为防火墙的开放端口，那么输入y继续操作以后，当前Xshell会自动断开连接；相反，如果开启防火墙时在防火墙之中没有检测到22端口，那么输入y继续操作以后22端口将会不再支持其他连接，只支持当前已有的这个连接，保持当前连接的原因是可以通过该连接开放22端口。

这里之前没有设置过，直接输入y继续执行

## Ubuntu防火墙添加开放普通端口

开放22端口

```sh
sudo ufw allow 22
```

开启完成，需要重启防火墙生效：

```sh
sudo ufw reload
```

查看防火墙的状态

```sh
root@ubuntu:/opt/docker/elasticsearch# sudo ufw status
Status: active

To                         Action      From
--                         ------      ----
22                         ALLOW       Anywhere               
22 (v6)                    ALLOW       Anywhere (v6)               
```

查看22端口的监听状态

```sh
root@ubuntu:/opt/docker/elasticsearch# sudo netstat -tunlp | grep 22
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      839/sshd: /usr/sbin 
tcp6       0      0 :::22                   :::*                    LISTEN      839/sshd: /usr/sbin 
```

## Ubuntu防火墙关闭普通端口

```sh
sudo ufw delete allow 21
```

## Ubuntu防火墙开放规定协议的端口

```sh
sudo ufw allow 8001/tcp
```

## Ubuntu防火墙关闭指定协议端口

```sh
sudo ufw delete allow 8001/tcp
```

## Ubuntu防火墙开放限定ip地址端口

- 开放指定ip所有操作

  ```sh
  sudo ufw allow from 192.168.1.11
  ```

- 关闭指定ip所有操作

  ```sh
  sudo ufw delete allow from 192.168.1.11
  ```

- 开放指定ip对应端口操作

  ```sh
  sudo ufw allow from 192.168.1.12 to any port 3306
  ```

- 开放指定ip对应端口操作

  ```sh
  sudo ufw delete allow from 192.168.1.12 to any port 3306
  ```

# 安装Docker

## 1.卸载旧版本Docker

```sh
#卸载旧版本docker
sudo apt-get remove docker docker-engine docker-ce docker.io  

#清空旧版docker占用的内存
sudo apt-get remove --auto-remove docker

#更新系统源
sudo apt-get update
```

## 2.配置安装环境

```sh
sudo apt-get install apt-transport-https ca-certificates curl gnupg-agent software-properties-common
```

## 3. 添加阿里云的docker GPG密钥

```sh
curl -fsSL http://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo apt-key add -
```

## 4. 添加阿里镜像源

```sh
sudo add-apt-repository "deb [arch=amd64] http://mirrors.aliyun.com/docker-ce/linux/ubuntu $(lsb_release -cs) stable"

#更新
sudo apt-get update
```

## 5. 查看有哪些版本

```sh
apt-cache madison docker-ce
```

[![img](D:\0_Notes\Hexo\hmxyl\source\_images\20200613220949677.png)](https://img-blog.csdnimg.cn/20200613220949677.png)

## 6. 安装最新版/指定版本

```sh
#安装最新版
sudo apt-get install -y docker-ce

#安装5:19.03.6~3-0~ubuntu-bionic版
sudo apt-get install -y docker-ce=5:19.03.6~3-0~ubuntu-bionic
```

## 7. 重启Docker

```sh
sudo service docker restart
#或者
sudo systemctl restart docker
```

## 8. 查看Docke版本

```sh
sudo docker version
```

## 9. 配置阿里容器镜像加速器

![配置阿里容器镜像加速器](D:\0_Notes\Hexo\hmxyl\source\_images\3f08ca0f-65ec-427d-87f4-3f3d27fb6f52.png)

- 针对Docker客户端版本大于 1.10.0 的用户
- 您可以通过修改daemon配置文件/etc/docker/daemon.json来使用加速器

```sh
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://7ixh250y.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```

## 10. 运行hello-world验证docker-ce是否安装成功

```sh
sudo docker run hello-world
```

 安装成功显示

![img](D:\0_Notes\Hexo\hmxyl\source\_images\69c425d9-ff18-4d9d-ac66-2946ae3bd013.png) 



##  11. 安装docker-compose

- 安装pip

```sh
sudo apt install python3-pip
```

- 更新一下库

```sh
sudo apt-get update
```

- 更新一下pip

```sh
sudo pip3 install --upgrade pip
```

- 安装docker-compose

```sh
sudo pip3 install docker-compose
```

- 如果出错

  ![img](D:\0_Notes\Hexo\hmxyl\source\_images\2020061322045958.png) 

- 就更新一下 six

```sh
pip3 install six --user -U
```

- 查看docker-compose版本

```sh
docker-compose --version
```

![img](D:\0_Notes\Hexo\hmxyl\source\_images\20200613220759470.png) 