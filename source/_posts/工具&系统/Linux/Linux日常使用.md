---
title: Linux日常使用
date: 2022-10-30 15:39:23
tags: 
  -  系统命令
categories: 
  - [系统,  Linux, 系统命令]
description: 'Linux系统命令记录'
---

# 不同Linux操作系统的安装命令

| **系统**             | **命令** |
| -------------------- | -------- |
| CentOS               | yum      |
| Debian或Ubuntu Linux | apt-get  |

# Centos防火墙开放端口/关闭端口

- 查看

  ```
  firewall-cmd --zone= public --query-port=3306/tcp
  或
  firewall-cmd --zone=public --list-ports
  ```

- 重新载入

  ```
  firewall-cmd --reload
  ```

- 添加

  ```
  firewall-cmd --zone=public --add-port=3306/tcp --permanent
  ```

- 关闭

  ```
  systemctl stop firewalld.service 
  或者
  systemctl disable firewalld.service
  ```



# Linux系统增加zysong字体

- 问题发生 ：在开发项目中，使用到了Jfreechart，在本机环境测试正常，部署到服务器上Linux，发现Jfreechart里面的中文不能正常显示。 

- 问题解决：

1. 首先确认你的服务器上的javahome ，执行命令 echo $JAVA_HOME ,显示出java的目录 
2. 将zysong.ttf文件拷贝到%JavaHome%/jre/lib/fonts目录下 zysong.ttf 需要网上下载 
3. 在%JavaHome%/jre/lib/fonts目录下执行"ttmkfdir -o fonts.dir"命令,重新生成fonts.dir文件 
4. 确认/usr/share/zh_CN/TrueType目录存在,如果不存在则mkdir创建 ，，一般开始是没有的，所有这样执行：到/usr/share/fonts下，"mkdir zh_CN" 命令创建 zh_CN文件夹，到zh_CN目录下 "mkdir TrueType"命令创建TrueType文件夹 
5. 把zysong.ttf文件拷贝到TrueType下 
6. 在%JavaHome%/jre/lib目录下,执行 cp fontconfig.RedHat.3.properties.src fontconfig.properties 7、重新启动tomcat（resin等web容器）,现在再看看，中文显示正常了



# 提高打开文件限制量（ulimit命令）

1. 解除 Linux 系统的最大进程数和最大文件打开数限制

   ```
   vi /etc/security/limits.conf
   
   # 添加如下的行
        * soft noproc 11000
   
        * hard noproc 11000
        * soft nofile 4100
        * hard nofile 4100
        
    说明：* 代表针对所有用户
        noproc 是代表最大进程数
        nofile 是代表最大文件打开数
   ```

2. 让 SSH 接受 Login 程式的登入，方便在 ssh 客户端查看 ulimit -a 资源限制：

   ```
   vi /etc/ssh/sshd_config 
    
   把 UserLogin 的值改为 yes，并把 # 注释去掉
   重启 sshd 服务： /etc/init.d/sshd restart
   ```

3. 修改所有 linux 用户的环境变量文件：

   ```
   vi /etc/profile
   
   ulimit -u 10000
   ulimit -n 4096
   ulimit -d unlimited
   ulimit -m unlimited
   ulimit -s unlimited
   ulimit -t unlimited
   ulimit -v unlimited
   ```

# 虚拟机之间，SSH免密连接

```
ssh-keygen -t rsa
cd /root/.ssh
mv id_rsa.pub authorized_keys_master.pub


scp  authorized_keys_node1.pub root@master:/root/.ssh
scp  authorized_keys_node2.pub root@master:/root/.ssh

cat authorized_keys_master.pub>> authorized_keys
cat authorized_keys_node1.pub>> authorized_keys
cat authorized_keys_node2.pub>> authorized_keys

scp authorized_keys  root@node1:/root/.ssh
scp authorized_keys  root@node2:/root/.ssh
```



# 非root权限启用80端口

服务器一开始重启的时候报错误：

```
 严重: Failed to initialize end point associated with ProtocolHandler ["http-bio-80"]
 java.net.BindException: Permission denied <null>:80
```

- 因为服务器非root权限用户只能使用1024以下端口。(重启服务时，使用的用户，是非root权限的cxdev用户)：参考，http://blog.csdn.net/mchdba/article/details/46335861

- 解决：重新使用root用户重启服务

  

# 清空catalina.out文件

- 使用 truncate 命令清空文件

  ```
  truncate -s 0 catalina.out  
  
  -s参数是设置文件的大小，清空文件的话，就设定为0
  ```

- 重定向方法清空文件

  ```
  [root@localhost logs]# du -h catalina.out 查看文件大小17M catalina.out
  [root@localhost logs]# > catalina.out  重定向清空文件
  [root@localhost logs]# du -h catalina.out  查看文件大小0 catalina.out
  ```

- 使用 true 命令重定向清空文件

  ```
  [root@localhost logs]# du -h catalina.out448K catalina.out
  [root@localhost logs]# true > catalina.out
  [root@localhost logs]# du -h catalina.out0 catalina.out
  ```

- 使用 echo 命令清空文件 

  ```
  echo -n " " > catalina.out  
  ```



# Centos系统安装

## 安装系统

1. 准备安装VMware和下载Centos

   ```
   安装VMware Workstation 15 Pro （永久激活密钥）
    YG5H2-ANZ0H-M8ERY-TXZZZ-YKRV8
    UG5J2-0ME12-M89WY-NPWXX-WQH88
    UA5DR-2ZD4H-089FY-6YQ5T-YPRX6
    GA590-86Y05-4806Y-X4PEE-ZV8E0
    ZF582-0NW5N-H8D2P-0XZEE-Z22VA
    YA18K-0WY8P-H85DY-L4NZG-X7RAD
   
    安装VMware Workstation 16 （永久激活密钥）
    ZF3R0-FHED2-M80TY-8QYGC-NPKYFYF390-0HF8P-M81RQ-2DXQE-M2UT6ZF71R-DMX85-08DQY-8YMNC-PPHV8
   
    下载虚拟机Centos：https://www.centos.org/download/
   ```

2. 虚拟网络说明

- VMNet1

  ```
  使用的是host-only的链接模式，即虚拟机只能与主机构成内部通信，无法对外网进行访问。
  ```

- VMNet0

  ```
  模式：使用桥接模式，安装VM后，在VM里建立虚拟机 默认 就是该模式。
  场景：如果你只是需要一台虚拟机可以和宿主互通，并可以访问外网，此模式即可。
  描述：安装虚拟机系统后不需要调整网络，物理网络中的 “路由” 所包含的DHCP服务器会自动识别该虚拟机并为其分配IP地址；
  如果没有路由，可以自己手动在系统分配，原则是和宿主机在同一网段并指向相同的网关即可通信。
  ```

- VMNet8

  ```
  模式：NAT网络模式
  场景：在宿主机安装多台虚拟机，和宿主组成一个小局域网，宿主机，虚拟机之间都可以互相通信，虚拟机也可访问外网，例如 搭建 hadoop 集群，分布式服务
  ```

## 安装图形界面

- 命令安装

  ```
  yum groupinstall "X Window System"
  
  yum groupinstall "GNOME Desktop"
  ```

- 进入图形界面 

  ```
  startx 或者 init 5 
  ```

- 修改图形界面为默认启动方式 

  ```
  命令行输入命令后重启系统 
  
  systemctl set-default graphical.target 
  ```

- 安装中文支持

  ```
  yum groupinstall "Chinese Support" -y 
  ```

- 修改系统默认语言为中文 

  ```
  命令行输入命令后重启系统
  
  localectl set-locale LANG=zh_CN.UTF-8 
  ```

- 图形界面想要卸载

  ```
  yum groupremove "GNOME Desktop Environment"
  yum groupremove "X Window System"
  ```

## 系统基础配置

###  修改 hotsname

```
hostnamectl --static set-hostname 名称 
vi /etc/hostname(缓存？)，要先把这个改好了，下面的配置文件才会生效。 
vi  /etc/sysconfig/network（重启，永久） 
echo hostname > /proc/sys/kernel/hostname（即时生效，临时）
```

### 基础工具安装

- yum -y install wget        (wget)
- yum -y install net-tools   (ifconfig)
- yum -y install lrzsz   (sz和rz)

### 默认ROOT用户登录

使用root账户进入系统后，打开’/etc/gdm/custom.conf’文件，在[daemon]下添加两行：

```
AutomaticLoginEnable=True
AutomaticLogin=root
```



### 虚拟机手动配置静态IP

1. `cat` `/etc/sysconfig/network-scripts/ifcfg-ens32

   ```properties
   TYPE=Ethernet
   PROXY_METHOD=none
   BROWSER_ONLY=no
   BOOTPROTO=static
   DEFROUTE=yes
   IPV4_FAILURE_FATAL=no
   IPV6INIT=yes
   IPV6_AUTOCONF=yes
   IPV6_DEFROUTE=yes
   IPV6_FAILURE_FATAL=no
   IPV6_ADDR_GEN_MODE=stable-privacy
   NAME=ens32
   UUID=066b4926-b40c-4c28-a5b4-2310d2b96613
   DEVICE=ens32
   ONBOOT=yes
   IPADDR=192.168.1.200
   NETMASK=255.255.255.0
   GATEWAY=192.168.1.254
   DNS1=223.5.5.5
   PREFIX=24
   ```

2. 使用nmcli重新回载网络配置：`nmcli c reload`

3. 查看IP：`nmcli`

###  Centos8 时钟同步

1. cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime

2. vim /etc/chrony.conf

```
注释掉 
  pool 2.centos.pool.ntp.org iburst  
加入新的的时间服务器
  server 210.72.145.44 iburst
  server ntp.aliyun.com iburst
```

![image-20211125155904615](D:\0_Notes\Hexo\hmxyl\source\_images\image-20211125155904615.png) 

​                               

3. 重启服务，此时时间已经与网络时间同步

   ```
   systemctl restart chronyd.service
   ```

4. 设置开机自启

   ```
   systemctl enable chronyd.service
   ```



### CentOS 8修改yum源为国内源

1. 备份

   目录`/etc/yum.repos.d`下新建文件夹`repo_back_all`，内容备份新建的文件夹中

2. 修改为阿里云

   `wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-8.repo`

3. 清理&拉取

   `yum clean all`

   `yum makecache`

   > 异常：提示镜像地址404
   >
   > 排查：repo文件中`$releasever`映射为8，对应的镜像不存在
   >
   > 说明：yum中的变量`$releasever`是由/etc/yum.conf中的distroverpkg定义的。centos-release为一个rpm包，所谓“distroverpkg=centos-release”的意思，其实是将 $releasever设置为centos-release 这个RPM包的版本号。查看版本号命令：`rpm -q centos-release`
   >
   > 解决：修改yum源文件，把$releasever全部替换为8-strea

## 安装过程中的问题以及解决措施

### 虚拟机ping不通主机，但是主机可以ping通虚拟机

1. Windows10的防火墙没有打开ICMPv4-in这个规则
   - 打开WIN10防火墙->选择高级设置->入站规则
   - 找到配置文件类型为`公用`的`文件和打印共享（回显请求 – ICMPv4-In）`规则，设置为允许
2. 确认本地主机IP
   - Vmware->编辑->虚拟网络编辑器
   - 查看VMnet8->NAT模式->NAT设置中网段信息
   - 本地windows主机打开：网络和Internet设置->以太网->更改适配器选项
   - 打开VMnet8，设置IPV4地址

