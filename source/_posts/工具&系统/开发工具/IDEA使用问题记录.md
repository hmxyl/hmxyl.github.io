---
title: IDEA使用记录
date: 2022-10-30 15:36:23
tags: 
  -  IDEA
categories: 
  - [工具, 开发工具, IDEA]
description: 'IDEA日常使用遇到的问题'
---

#  报错:找不到包或者找不到符号

##  利用Maven-Reimport

![image-20211125093632598](D:\0_Notes\Hexo\hmxyl\source\_images\image-20211125093632598-1667118386299-59.png) 

## Invalidate and Restart

 ![image-20211125093653072](D:\0_Notes\Hexo\hmxyl\source\_images\image-20211125093653072-1667118386300-60.png)



## 重新编译

### 点开Project Structure 找到项目编译输出目录

### 将target目录下文件清空

### 右键项目重新build

 ![image-20211125093733929](D:\0_Notes\Hexo\hmxyl\source\_images\image-20211125093733929-1667118386300-61.png)

![image-20211125093748957](D:\0_Notes\Hexo\hmxyl\source\_images\image-20211125093748957-1667118386300-62.png) 

## 利用Maven-Install

![image-20211125093759368](D:\0_Notes\Hexo\hmxyl\source\_images\image-20211125093759368-1667118386300-63.png)   

# 清除安装数据

删除安装包（也可以重新安装时删除版本文件）
C:\Program Files\JetBrains  下面的目录
删除安装数据
C:\Users\Administrator\AppData\Local\JetBrains
C:\Users\Administrator\AppData\Roaming\JetBrains
C:\Users\Administrator\已安装版本

# JUnit单元测试%MODULE_WORKING_DIR%' does not exist

解决办法：idea > Run -> Edit Configurations>JUnit（或者application，根据自己报错的类型选择）
单独设置
选中有单元测试的类 把working directory 原先为:%MODULE_WORKING_DIR% 设置为 . （表示当前目录）或者直接填写项目绝对路径（eg:P:\ideatestspace\zztPro）
全局设置
也可以点击defaults(全局配置) ，找到testng 同样把working directory 设置为.或者项目的绝对路径（eg:P:\ideatestspace\zztPro）

# idea一直刷新索引：updating index

File ->Invalidate Caches/restart -> Invalidate and Restart

# IDEA配置tomcat日志乱码

1. 第一步（tomcat7/8）tomcat：
   找到tomcat文件夹下的conf文件夹，去修改里面的logging.properties文件两种修改方式（第一种方法不行再用第二种）：

   1. 将文件中的5个UTF-8修改为GBK

      ![image-20211125093935522](D:\0_Notes\Hexo\hmxyl\source\_images\image-20211125093935522-1667118386300-64.png) 

   2. 将文件中的有关UTF-8的行用’#'注释掉

      ![image-20211125094101237](D:\0_Notes\Hexo\hmxyl\source\_images\image-20211125094101237-1667118386301-65.png) 

2. 第二步（IDEA2019）
   进入文件后添加一段代码：-Dfile.encoding=UTF-8

   ![image-20211125094120088](D:\0_Notes\Hexo\hmxyl\source\_images\image-20211125094120088-1667118386301-66.png) 

3. 第三步

   ![image-20211125094148869](D:\0_Notes\Hexo\hmxyl\source\_images\image-20211125094148869-1667118386301-67.png) 

   

   ![image-20211125094213607](D:\0_Notes\Hexo\hmxyl\source\_images\image-20211125094213607-1667118386301-68.png) 

   

4. 第四步重启IDEA









# 搜狗输入法候选字不跟随光标在怎么办

ctrl+shift+a 输入 switch IDE Boot JDK, 换一个 jdk 就行了
