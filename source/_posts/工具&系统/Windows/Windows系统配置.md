---

title: Windows系统配置
date: 2022-10-30 15:39:23
tags: 
  -  系统配置
categories: 
  - [系统, Windows, 系统配置]
description: 'Window系统设置'
---



# 设置系统护眼色

## 豆沙绿：RGB（202，234，206），#CAEACE

## 淡黄色：RGB（253，246，227），#FDF6E3 【选用】

1. 首先使用 Win + R 组合快捷键，打开“运行”，然后键入打开注册表命令「regedit」，按回车键确认打开，如图所示 

2. 打开Win10注册表之后，依次在左侧树状菜单中展开：HKEY_CURRENT_USER\Control Panel\Colors然后再右侧找到「Windows」值，并双击打开，将默认的255 255 255（默认是白色背景）三组颜色数值改成 253，246，227完成后，点击下方的“确定”保存，如图所示。 

    

![image.png](D:\0_Notes\Hexo\hmxyl\source\_images\image-20211124002345774.png) 

3. 继续找到注册表的路径：HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Themes\DefaultColors\Standard同样再在右侧找到「Windows」双击打开，将默认的数据值 ffffff 改成  FDF6E3 完成后，点击下方的确定保存如下图所示。

![image.png](D:\0_Notes\Hexo\hmxyl\source\_images\image-20211124002408921.png) 



完成以上三步操作后，需要重启电脑生效





# Windows查看端口占用

1. 获取占用端口的进程ID  `netstat -ano|findstr 10001`
2. 获取进程信息 `tasklist|findstr 9352`
3. 关闭进程（强行关闭） `taskkill /PID 9140 /T /F`

# windows开机启动程序路径

```
C:\Users\Administrator\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup
```

# windows的host文件位置

```
C:\Windows\System32\drivers\etc
```

# Windows刷新DNS命令

```
ipconfig /flushdns
```

win+r可执行的命令

| 命令         | 说明             |
| ------------ | ---------------- |
| mstsc        | 打开远程连接     |
| services.msc | 本地服务设置     |
| calc         | 计算器           |
| dxdiag       | 系统配置查看命令 |

# Windows命令行打开防火墙

```
win10：netsh advfirewall firewall add rule name="ES Port 9300" dir=in action=allow protocol=TCP localport=9300
```



# Windows 开放服务器端口

win7下打开端口测试端口时 可用telnet 命令侦听端口：`C:\Documents and Settings\administrator>netstat -na`

![image-20211124112102739](D:\0_Notes\Hexo\hmxyl\source\_images\image-20211124112102739.png) 

测试端口是否开放：
 `C:\Documents and Settings\administrator>telnet 127.0.0.1 8500`
 正在连接到127.0.0.1...不能打开到主机的连接， 在端口 8500: 连接失败



下面开始打开端口（Win7）

Win7的防火墙做了比较大升级 设置已经分为入站和出站。下面以开通Tomcat的远程访问8080作为例子。**控制面板--所有控制面板项--Windows 防火墙** 进入

入站规则设置
 第一步 选择 入站规则 然后 新建规则，选择 端口，然后下一步
 第二步 选择TCP 选择特定端口 然后输入端口，如有多个端口需要用逗号隔开了 例如:88,8080
 第三步，选择允许连接
 第四步 选择应用规则的范围
 第五步 输入规则名称

 出站规则设置
 第一步 选择 入站规则 然后 新建规则，选择 端口，然后下一步
 第二步 选择TCP 选择特定端口 然后输入端口，如有多个端口需要用逗号隔开了 例如:88,8080
 第三步，选择允许连接
 第四步 选择应用规则的范围
 第五步 输入规则名称
 至此，防火墙规则设置完毕，启用即可！
 另外win7的 IIS7，只需启用 入站规则：BranchCache 内容检索(HTTP-In)
 出站规则： BranchCache 内容检索(HTTP-Out) 即可。



![image-20211124112849477](D:\0_Notes\Hexo\hmxyl\source\_images\image-20211124112849477.png) 
![image-20211124113626250](D:\0_Notes\Hexo\hmxyl\source\_images\image-20211124113626250.png) 
![image-20211124113803236](D:\0_Notes\Hexo\hmxyl\source\_images\image-20211124113803236.png)  
![image-20211124113911637](D:\0_Notes\Hexo\hmxyl\source\_images\image-20211124113911637.png )

![image-20211124114010436](D:\0_Notes\Hexo\hmxyl\source\_images\image-20211124114010436.png) 

![image-20211124114015918](D:\0_Notes\Hexo\hmxyl\source\_images\image-20211124114015918.png) 

![image-20211124114021360](D:\0_Notes\Hexo\hmxyl\source\_images\image-20211124114021360.png) 

![image-20211124114030488](D:\0_Notes\Hexo\hmxyl\source\_images\image-20211124114030488.png) 

![image-20211124114034567](D:\0_Notes\Hexo\hmxyl\source\_images\image-20211124114034567.png) 

输出规则也一样的设置 可以看到

 ![image-20211124114043841](D:\0_Notes\Hexo\hmxyl\source\_images\image-20211124114043841.png)   

![image-20211124114050299](D:\0_Notes\Hexo\hmxyl\source\_images\image-20211124114050299.png)   



# Windows系统快捷键记录

## 常用记录

| **快捷键**     | **说明**                                       |
| -------------- | ---------------------------------------------- |
| Ctrl+Shift+Esc | 打开任务管理器                                 |
| Esc            | 取消当前任务                                   |
| Shift+delete   | 永久删除所选的项目（删除之后无法从回收站还原） |
| Ctrl+shiff+tab | 在选项卡上向后移动                             |
| Tab            | 在选项上向后移动                               |
| Shift+Tab      | 在选项卡上向前移动                             |

# Windows资源管理器中的快捷键

| **快捷键** | **说明**                   |
| ---------- | -------------------------- |
| Alt+P      | 显示预览窗格               |
| Alt+←      | 切换到前一次打开的文件夹   |
| Alt+→      | 切换到下一次后打开的文件夹 |
| Alt+↑      | 打开上层文件夹             |
| Backspace  | 打开上层文件夹             |

# Windows徽标键相关的快捷键

| **快捷键**  | **说明**                         |
| ----------- | -------------------------------- |
| Win         | 打开或者关闭开始菜单             |
| Win+Pause   | 显示系统属性对话框               |
| Win+d       | 显示桌面                         |
| Win+m       | 最小化所有窗口                   |
| Win+Shift+m | 还原最小化窗口到桌面上           |
| Win+E       | 打开我的电脑                     |
| Win+F       | 搜索文件或文件夹                 |
| Win+L       | 锁定您的计算机或切换用户         |
| Win+R       | 打开运行对话框                   |
| Win+↓       | 最小化窗口                       |
| Win+↑       | 最大化当前窗口                   |
| Win+←       | 最大化到窗口左侧的屏幕上         |
| Win+→       | 最大化到窗口右侧的屏幕上         |
| Win+home    | 最小化所有窗口，除了当前激活窗口 |

# Windows定时执行任务脚本

1. 第一步：打开控制面板-》系统和安全-》管理工具
2. 
