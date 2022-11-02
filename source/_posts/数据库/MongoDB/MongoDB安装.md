---
title: MongoDB安装
date: 2022-10-25 19:52:44
tags: MongoDB
category: 
  - [数据库, MongoDB]
description: 'MongoDB安装记录'
---

# Windows 安装

## 安装文件

现官网只含64位的安装文件，或者去以下路径下载： http://dl.mongodb.org/dl/win32/x86_64

## 环境变量

```
MONGODB_HOME：D:\ProgramSoft\MongoDB

path追加：%MONGODB_HOME%\bin;
```



## 安装服务

 前提：创建文件夹 `E:\ProgramData\db` 和 `E:\ProgramData\log`

系统管理员操作

1. 创建服务

```bash
sc create mongodb binPath= "D:\ProgramSoft\MongoDB\bin\mongod.exe --service --dbpath E:\ProgramData\db --logpath=E:\ProgramData\log\mongodb.log --logappend"
```

2. 删除服务

```bash
sc delete mongodb
```

![460c99600f3e8207ce486a218ed21620.png](D:\0_Notes\Hexo\hmxyl\source\_images\clip_image001-16383252343726.png)  

错误记录： 

![6495b50cee3c37ebdbff17f02d31dd12.png](D:\0_Notes\Hexo\hmxyl\source\_images\clip_image002-16383252343727.png)  

异常1： 服务开启不了 发生服务特定错误: 100，发生服务特定错误: 48
原因，service安装语句

```
sc create mongodb binPath= "D:\ProgramSoft\MongoDB\bin\mongod.exe --service --dbpath D:\mongodb\data --logpath=E:\ProgramData\log\mongodb.log --logappend --directoryperdb"
```

 解决方案：

1. 删除`E:\ProgramData\db\mongod.lock文件`

2. 删除服务

   ```
   net stop mongodb;
   net delete mongodb;
   ```

3. 重新安装 注意：去除` --directoryperdb `命令

   ```
   sc create mongodb binPath= "D:\ProgramSoft\MongoDB\bin\mongod.exe --service --dbpath D:\mongodb\data --logpath=E:\ProgramData\log\mongodb.log --logappend"
   ```

   

# 
