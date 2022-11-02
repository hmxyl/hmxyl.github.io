---
title: Linux系统命令
date: 2022-10-30 15:39:23
tags: 
  -  系统命令
categories: 
  - [系统,  Linux, 系统命令]
description: 'Linux系统命令记录'
---

# 查看信息

##  “！”命令

! 符号在 Linux 中不但可以用作否定符号，还可以用来从历史命令记录中取出命令或不加修改的执行之前运行的命令。



1. 执行上一条命令：`!!`

   ```sh
   [root@hots java]# whereis java
   java: /usr/bin/java /usr/lib/java /etc/java /usr/share/java /opt/soft/jdk1.8.0_211/bin/java /usr/share/man/man1/java.1.gz
   [root@hots java]# !! 
   whereis java
   java: /usr/bin/java /usr/lib/java /etc/java /usr/share/java /opt/soft/jdk1.8.0_211/bin/java /usr/share/man/man1/java.1.gz
   [root@hots java]#
   ```

   ==!!代表了上一条执行的命令。可以看到，当输入两个感叹号时，它显示上条命令的同时会执行上一条命令。==当然了，通常我们还会想到使用“UP”键来完成这个事情。但是如果是基于上条命令扩充，!!就来得更加方便了。

   ```sh
   [root@hots java]# ExitCapture #忘记输入java
   bash: ExitCapture: command not found...
   [root@hots java]# java !!
   java ExitCapture 
   ```

2. 使用上条命令最后一个参数：`!$`

   ```java
   root@hots java]# ls /root/java/
   ExitCapture.class  ExitCapture.java  nohup.out
   [root@hots java]# ls -al !$
   ls -al /root/java/
   total 16
   drwxr-xr-x.  2 root root   72 Dec 28 14:43 .
   dr-xr-x---. 16 root root 4096 Dec 28 14:13 ..
   -rw-r--r--.  1 root root  934 Dec 28 14:46 test.java
   
   ```

   

3. 使用上条命令第一个参数：`!^`

   ```java
   [root@hots java]# ls /root/java/
   ExitCapture.class  ExitCapture.java  nohup.out
   [root@hots java]# ls -al !^
   ls -al /root/java/
   total 16
   drwxr-xr-x.  2 root root   72 Dec 28 14:43 .
   dr-xr-x---. 16 root root 4096 Dec 28 14:13 ..
   -rw-r--r--.  1 root root  934 Dec 28 14:46 test.java
   ```

4. 去掉上一条命令最后一个参数，再次执行 `!:-`

   ```sh
   [root@hots java]# ls -al /root/java/
   total 16
   drwxr-xr-x.  2 root root   72 Dec 28 14:43 .
   dr-xr-x---. 16 root root 4096 Dec 28 14:13 ..
   -rw-r--r--.  1 root root  934 Dec 28 14:46 test.java
   -rw-------.  1 root root  369 Dec 28 14:44 nohup.out
   [root@hots java]# !:-
   ls -al
   total 16
   drwxr-xr-x.  2 root root   72 Dec 28 14:43 .
   dr-xr-x---. 16 root root 4096 Dec 28 14:13 ..
   -rw-r--r--.  1 root root  934 Dec 28 14:46 test.java
   ```

5. 使用上一条命令的所有参数：`!*`

   ```sh
   [root@hots java]# finsd -name "foo.zip" # 这里特意输错了find命令
   bash: finsd: command not found...
   [root@hots java]# find !*
   find -name "foo.zip"
   [root@hots java]# 
   ```

6. 使用上条命令指定的参数：`![命令名]:[参数号]`

   ```sh
   [root@hots java]# ls -al /root/java/
   total 16
   drwxr-xr-x.  2 root root   72 Dec 28 14:43 .
   dr-xr-x---. 16 root root 4096 Dec 28 14:13 ..
   -rw-r--r--.  1 root root  934 Dec 28 14:46 test.java
   [root@hots java]# pwd !ls:2
   pwd /root/java/
   /root/java
   [root@hots java]# 
   ```

7. 执行上一条以关键字开头的命令：`!关键字`

   ```sh
   [root@hots java]# !find #执行上条以find开头的命令
   find -name "foo.zip"
   ```

8. 逻辑非的作用

   这个是它最为人所熟悉的作用，例如删除除了cfg结尾以外的所有文件：`rm !(*.cfg)`

**小结**

| 命令            | 说明                          |
| --------------- | ----------------------------- |
| !!              | 上一条命令                    |
| !$              | 上一条命令中的最后一个参数    |
| !:-             | 上一命令除了最后一个参数      |
| !*              | 上一条命令中的所有参数        |
| !str            | 最近一条以str开头的命令       |
| !?str?          | 最近一条包含str的命令         |
| !n              | 顺数第n条命令                 |
| !-n             | 倒数第n条命令                 |
| `^old^new`      | 将上一命令中的old替换为new    |
| !!:gs/old/new   | 将上一命令中的old替换为new    |
| !scp:gs/old/new | 将上一scp命令中的old替换为new |
|                 |                               |

## 查看系统内存条信息

```shell
dmidecode -t memory
```

![image-20211125171541505](D:\0_Notes\Hexo\hmxyl\source\_images\image-20211125171541505.png) 

## 查看linux机器是32位还是64位

```shell
  getconf LONG_BIT
```

![image-20211125172057979](D:\0_Notes\Hexo\hmxyl\source\_images\image-20211125172057979.png) 

## 查看系统版本号

```sh
lsb_release -a
```



## 查看进程状态：ps

```sh
格式：ps [options] [--help]
参数：
-A 列出所有的线程
-e 列出所有的进程
-f 显示详细的信息（包括命令行参数）

例子: 查看java进程信息
ps -ef|grep java
```

##  查看系统磁盘信息：df

```shell
df -lm
```

##  查看内存使用情况

```sh
free -g
```

##  查看系统内核信息

```sh
uname -a
```

##  显示系统用户信息：who

```sh
格式 : who - [husfV] [user]
说明 : 显示有哪些用户登录到系统中，显示的信息包含用户ID，使用的终端，上线时间，呆滞时间，CPU使用量，动作等等。
参数说明 :
    -H : 显示标题列
    -u : 显示用户的闲置时间
    -s : 使用简短的格式来显示
    --version : 显示程式版本
    -r 查看当前系统运行时间
    -b 查看最后一次系统启动的时间。

相关命令 : who am i  显示当前用户是谁

例子： 查看最后启动时间
          who -b
```



# 文件操作命令

##  du：查看文件和目录磁盘使用的空间

```sh
格式：du [选项] [文件]
参数:
    -a或-all 显示目录中个别文件的大小
    -b或-bytes 显示目录或文件大小时，以byte为单位
    -c或--total 除了显示个别目录或文件的大小外，同时也显示所有目录或文件的总和
    -k或--kilobytes 以KB(1024bytes)为单位输出
    -m或--megabytes 以MB为单位输出
    -s或--summarize 仅显示总计，只列出最后加总的值
    -h或--human-readable 以K，M，G为单位，提高信息的可读性
    -x或--one-file-xystem 以一开始处理时的文件系统为准，若遇上其它不同的文件系统目录则略过
    -L<符号链接>或--dereference<符号链接> 显示选项中所指定符号链接的源文件大小
    -S或--separate-dirs 显示个别目录的大小时，并不含其子目录的大小
    -X<文件>或--exclude-from=<文件> 在<文件>指定目录或文件
    --exclude=<目录或文件> 略过指定的目录或文件
    -D或--dereference-args 显示指定符号链接的源文件大小
    -H或--si 与-h参数相同，但是K，M，G是以1000为换算单位
    -l或--count-links 重复计算硬件链接的文件。
范例：du -hm 目录名称
```

## 文件操作-find 将符合 expression 的文件列出来

```sh
格式 : find [path...] [expression]
说明 : 将符合 expression 的文件列出来。                    
  -amin n : 在过去 n 分钟内被读取过的文件
  -anewer file : 比文件 file 更晚被读取过的文件
  -atime n : 在过去 n 天被读取过的文件
  -cmin n : 在过去 n 分钟内被修改过的文件
  -cnewer file :比文件 file 更新的文件
  -ctime n : 在过去 n 天过修改过的文件
  -name filename, -iname filename : 符合 filename 的文件。iname 会忽略大小写
  -size n : 档案大小 是 n 单位，b 代表 512 位元组的区块，c 表示字元数，k表示 kilo bytes，w 是二个位元组。
  -type c : 档案类型是 c 的档案。
```

范例:

1. 将当前目录及其子目录下所有扩展名是 c 的文件列出来。

   ```sh
   find . -name "*.c"
   ```

   

2. 将当前目录及其子目录下所有最近 20 分钟内更新过的文件列出

   ```sh
   find . -cmin -20
   ```

   

3. 查找包含字符串的文件

   ```sh
   find /opt/Tomcat7  -name "system.properties"|xargs grep -ri "BASCI_SCI_UPDATE"
   ```

   

4. 将/usr/local/backups目录下所有10天前带"."的文件删除

   ```sh
   find /usr/local/backups -mtime +10 -name "*.*" |xargs rm -rf
   或者
   find /usr/local/backups -mtime +10 -name "*.*" -exec rm -rf {} \;
   
   
   
   说明： 　　
   find：linux的查找命令，用户查找指定条件的文件 　　
   /usr/local/backups：想要进行清理的任意目录 　　
   -mtime：标准语句写法 　　
   ＋10：查找10天前的文件，这里用数字代表天数，＋30表示查找30天前的文件
   "*.*"：希望查找的数据类型，"*.jpg"表示查找扩展名为jpg的所有文件，"*"表示查找所有文件，这个可以灵活运用，举一反三 　　
   
   
   -exec rm -rf {} \; ：find发现的结果一次性传给exec选项，删除
   |xargs rm -rf : 分批次的处理删除（推荐）
   ```



## 文件操作-grep： 搜索字符串命令

```sh
格式：grep [-no] pattern files
参数：
     -n 显示行号
     -o 只显示匹配的串
```

范例：

```
grep  printf *
    file1.c:   printf("\nHello\n");
    file2.c:   printf("\nSample\n");

grep -n  printf *
    file1.c:4   printf("\nHello\n");
    file2.c:9   printf("\nSample\n");

grep -o  printf *
   file1.c:   printf
   file2.c:   printf

如果搜索的串中有空格，则用引号括起来

grep "asd abc" *
```

## wc：统计指定文件中的字节数、字数、行数，并将统计结果显示输出

```sh
格式：wc [选项] 文件名称
选项 ：
   -c 统计字节数。
   -l 统计行数。
   -m 统计字符数。这个标志不能与 -c 标志一起使用。
   -w 统计字数。一个字被定义为由空白、跳格或换行字符分隔的字符串。
   -L 打印最长行的长度。
   -help 显示帮助信息
   --version 显示版本信息
```

## cp：(源文件存在)将一个文件拷贝至另一文件，或将数个文件拷贝至另一目录

```sh
格式：cp [-arf] source dest
参数:
   -a 将文件状态、权限等信息都照原状予以复制。
   -r  若 source 中含有目录名，则将目录下的文件顺序拷贝至目的地。
   -f  若目的地已经有相同的文件名存在，则在复制前先予以删除再行复制。
```

范例： 

```sh
 1. 将文件 aaa 复制一份名字为 bbb 的文件:         
    cp aaa bbb   
 2. 将当前目录下的所有C程序拷贝到当前目录下的Finished 子目录中：
    cp *.c Finished
```

## mv：(源文件不存在)将一个文件改名为另一文件，或将数个文件移至另一目录。

```sh
格式： mv [-i] source dest
        mv [-i] source... directory
参数：-i 若目的地已有同名文件，则先询问是否覆盖旧文件
```

范例：

```sh
1. 将文件 aaa 改名为 bbb :
   mv aaa bbb
2. 将所有的C程序移至 Finished 子目录中：
   mv -i *.c  Finished
```

## rm：删除文件及目录

```sh
格式：rm [-ifr] name...
   -i  删除前逐一询问确认。
   -f  即使原文件属性设为只读，也直接删除，无需逐一确认。
   -r  将目录及以下之文件逐一删除。
```

范例

```sh
1.  删除所有C程序文件并删除前逐一询问确认 :
rm -i *.c
2. 将 Finished 子目录及子目录中所有文件删除 :
rm -r Finished
```

## tail条件查看文件内容

```sh
格式：tail [ -f ] [ -c Number | -n Number | -m Number | -b Number | -k Number ]  文件名称
-f  该参数用于监视File文件增长。
-c  Number 从 Number 字节位置读取指定文件
-n  Number 从 Number 行位置读取指定文件。
-m  Number 从 Number 多字节字符位置读取指定文件，比方你的文件假设包括中文字，假设指定-c参数，可能导致截断，但使用-m则会避免该问题。
-b  Number 从 Number 表示的512字节块位置读取指定文件。
-k  Number 从 Number 表示的1KB块位置读取指定文件。
上述命令中，都涉及到number，假设不指定，默认显示10行。
Number前面可使用正负号，表示该偏移从顶部还是从尾部開始计算。
tail可运行文件一般在/usr/bin/以下。
```

示例

```sh
1、  tail -f filename 
说明：监视filename文件的尾部内容（默认10行，相当于增加参数 -n 10），刷新显示在屏幕上。退出，按下CTRL+C。 
2、  tail -n 20 filename 
说明：显示filename最后20行。
```

## cat： 查看文件(文件拼接)

```sh
格式：cat [-AbeEnstTuv] [--help] [--version] fileName
说明：把文件串连接后输出到荧幕或加 > fileName 到另一个档案
参数：
    -A 等价于 -vET
    -n或 --number 由 1 开始对所有输出的行数编号
    -b或 --number-nonblank和 -n 相似，只不过对于空白行不编号
    -e 等价于 –vE
    -E 每行末尾显示一个$符号
    -s或 --squeeze-blank 当遇到有连续两行以上的空白行，就代换为一行的空白行
    -t 等价于 –vT
    -T 显示制表符为 ^I
    -v或 --show-nonprinting,  dos格式的回车换行显示为^M
```

范例：

```sh
1. 把 textfile1 的文件内容加上行号后输入到 textfile2 文件里：
 cat -n textfile1 > textfile2
2. 把 textfile1 和 textfile2 的文件内容加上行号（空白行不加）之后将内容附加到 textfile3 ：
 cat -b textfile1 textfile2 >> textfile3
（ > 为重定向操作符， >>为重定向追加操作符 ）
```

## more：文件查看

```sh
格式：more  [-num] [+linenum] [fileNames..]
说明：类似 cat ，不过是以一页一页的方式显示。而最基本的指令就是按空白键（space）就往下一页显示，按 b 键就会往回（back）一页显示。
参数：
   -num 一次显示的行数
    +linenum 从第 num 行开始显示
         fileNames 欲显示内容的文件，可为多个文件
```

范例：

```sh
从第 20 行开始显示 testfile 之文件内容。

more +20 testfile
```

## less：文件查看

```sh
格式： less [Option]  filename
说明： less 的作用与 more 十分相似，都可以用来浏览文本文件的内容，不同的是 less 允许使用者往回卷动（PageUp PageDown）以浏览已经看过的部份，同时因为 less 并未在一开始就读入整个文件，因此在遇上大型文件的开启时，会比一般的文本编辑器(如  vi)来的快速。
```

## touch：新建文件

```sh
touch 文件名
```

## mkdir：创建目录

```sh
mkdir  dirName
```

# Linux系统其他命令

## systemctl

```
以firewalld.service为例

启动一个服务：systemctl start firewalld.service
关闭一个服务：systemctl stop firewalld.service
重启一个服务：systemctl restart firewalld.service

显示一个服务的状态：systemctl status firewalld.service

在开机时启用一个服务：systemctl enable firewalld.service
在开机时禁用一个服务：systemctl disable firewalld.service

查看服务是否开机启动：systemctl is-enabled firewalld.service
查看开机启动的服务列表：systemctl list-unit-files|grep enabled
查看启动失败的服务列表：systemctl --failed
```

## ssh：远程登陆

```sh
格式: ssh 用户名@机器名
范例: ssh rd@build01
```

## chown：变更文件夹的所有者

```sh
chown 用户名 文件名 -R
```

## 命令行执行sh文件

```sh
【步骤一】cd到.sh文件所在目录
【步骤二】给.sh文件添加x执行权限 chmod u+x hello.sh
【步骤三】 ./hello.sh 或者 sh hello.sh
```

## curl：查看网页结果

```sh
curl ipinfo.io/ip
curl cip.cc
```

## scp：从其他机器拷贝文件夹

```sh
scp -r 文件夹名(源) 用户名@机器名:/路径（目的）
之后输入，目标机器的用户密码。
举例： scp -r /index1/DAYAIR/Lucene/20160930  root@117.122.222.74:/index1/dayalib
```

## SFTP：上传文件/下载

```sh
上传： put -r  本地文件夹/上传文件夹
put local-file  [remote-file]
下载： get remote-file  [local-file]
```

## kill

```sh
格式： kill [ -s signal ] pid ...
      kill -l [ signal ]
说明：kill 送出一个特定的信号 (signal) 给进程号为 pid 的进程。根据该信号而做特定的动作, 若没有指定,默认是送出终止 (TERM) 信号

参数：
    -s (signal) : 其中常用的一个信号(9) 杀死进程; 详细的信号可以用 kill -l
    -l (signal) : 列出所有可用的信号名称

范例：
  1. 将 pid 为 323 的进程杀死 ： kill -9 323
  2. 将 pid 为 456 的进程重跑 (restart) : kill -HUP 456
```

# ulimit：控制shell程序的资源#

```sh
语　　法：ulimit [-aHS][-c <core文件上限>][-d <数据节区大小>][-f <文件大 小>][-m <内存大小>][-n <文件数目>][-p <缓冲区大小>][-s <堆栈大小>][-t <CPU时间>][-u <程序数目>][-v <虚拟内存大小>]
补充说明：ulimit为shell内建指令，可用来控制shell执行程序的资源。
参　　数：
        -a 显示目前资源限制的设定。
        -c <core文件上限> 　设定core文件的最大值，单位为区块。
        -d <数据节区大小> 　程序数据节区的最大值，单位为KB。
        -f <文件大小> 　shell所能建立的最大文件，单位为区块。
        -H 　设定资源的硬性限制，也就是管理员所设下的限制。
        -m <内存大小> 　指定可使用内存的上限，单位为KB。
        -n <文件数目> 　指定同一时间最多可打开的文件数。
        -p <缓冲区大小> 　指定管道缓冲区的大小，单位512字节。
        -s <堆栈大小> 　指定堆叠的上限，单位为KB。
        -S 　设定资源的弹性限制。
        -t <CPU时间> 　指定CPU使用时间的上限，单位为秒。
        -u <进程数目> 　用户最多可启动的进程数目。
        -v <虚拟内存大小> 　指定可使用的虚拟内存上限，单位为KB。
```

## top：实时显示系统中各个进程的资源占用状况

top命令是Linux下常用的性能分析工具，能够实时显示系统中各个进程的资源占用状况，类似于Windows的任务管理器。

>  下面详细介绍它的使用方法。

top是一个动态显示过程,即可以通过用户按键来不断刷新当前状态.如果在前台执行该命令,它将独占前台,直到用户终止该程序为止.比较准确的说,top命令提供了实时的对系统处理器的状态监视.它将显示系统中CPU最“敏感”的任务列表.该命令可以按CPU使用.内存使用和执行时间对任务进行排序；而且该命令的很多特性都可以通过交互式命令或者在个人定制文件中进行设定

```sh
命令说明
1.  命令格式： top [参数]
2.  命令功能： 显示当前系统正在执行的进程的相关信息，包括进程ID、内存占用率、CPU占用率等
3.  命令参数
        -b 批处理
        -c 显示完整的治命令
        -I 忽略失效过程
        -s 保密模式
        -S 累积模式
        -i<时间> 设置更新间隔时间
        -u<用户名> 指定用户名
        -p<进程号> 指定进程
        -n<次数>
```

> 补充top使用技巧

- 多核CPU监控在top基本视图中，按键盘数字“1”，可监控每个逻辑CPU的状况：

- 高亮显示当前运行进程敲击键盘“b”（打开/关闭加亮效果 ）

- 通过”shift + >”或”shift + <”可以向右或左改变排序列下图是按一次”shift + >”的效果图,视图现在已经按照%MEM来排序。



> 使用实例     

- 实例1：显示指定的进程信息

  ```sh
  top -p 2885
  ```

  ![image-20211125180715469](D:\0_Notes\Hexo\hmxyl\source\_images\image-20211125180715469.png) 

- 实例2: 循环显示的次数

  ```sh
  命令：  top -n 2
  说明：表示更新两次后终止更新显示
  ```

- 实例3：显示进程信息，并具体说明

  ![20211125181146934](D:\0_Notes\Hexo\hmxyl\source\_images\20211125181146934.png) 



> 下面我们看每一行信息的具体意义

1. 第一行，任务队列信息，同 **uptime** 命令的执行结果

```sh
top - 18:10:32 up  2:39,  2 users,  load average: 0.23, 0.11, 0.06
```

| 参数                           | 说明                                                         |
| ------------------------------ | ------------------------------------------------------------ |
| 18:10:32                       | 当前系统时间                                                 |
| up   2:39                      | 系统已经运行了2小时39分钟（在这期间系统没有重启过）          |
| 2 users                        | 当前有2个用户登录系统                                        |
| load average: 0.23, 0.11, 0.06 | load average后面的三个数分别是1分钟、5分钟、15分钟的负载情况。<br>load average数据是每隔5秒钟检查一次活跃的进程数，然后按特定算法计算出的数值。如果这个数除以逻辑CPU的数量，结果高于5的时候就表明系统在超负荷运转了。 |



2. 第二行，Tasks — 任务（进程）

```sh
Tasks: 213 total,   1 running, 210 sleeping,   0 stopped,   2 zombie

具体信息说明如下：
系统现在共有213个进程，其中处于运行中的有1个，210个在休眠，stoped状态的有0个，zombie状态（僵尸）的有2个
```

3. 第三行：cpu状态信息

```sh
%Cpu(s):  0.7 us,  0.6 sy,  0.0 ni, 98.7 id,  0.0 wa,  0.0 hi,  0.1 si,  0.0 st

备注：在这里CPU的使用比率和windows概念不同，需要理解linux系统用户空间和内核空间的相关知识
```

| 参数    | 说明                                         |
| ------- | -------------------------------------------- |
| 0.7 us  | 用户空间占用CPU的百分比。                    |
| 0.6 sy  | 内核空间占用CPU的百分比。                    |
| 0.0 ni  | 改变过优先级的进程占用CPU的百分比            |
| 98.7 id | 空闲CPU百分比                                |
| 0.0 wa  | IO等待占用CPU的百分比                        |
| 0.0 hi  | 硬中断（Hardware IRQ）占用CPU的百分比        |
| 0.1 si  | 软中断（Software Interrupts）占用CPU的百分比 |
| 0.0 st  | 虚拟机占用百分比                             |

4. 第四行,内存状态

```sh
KiB Mem : 16212604 total,  4243632 free, 10135032 used,  1833940 buff/cache

第四行中使用中的内存总量（used）指的是现在系统内核控制的内存数，空闲内存总量（free）是内核还未纳入其管控范围的数量。
纳入内核管理的内存不见得都在使用中，还包括过去使用过的现在可以被重复利用的内存，内核并不把这些可被重新使用的内存交还到free中去，因此在linux上free内存会越来越少，但不用为此担心。

若需要计算可用内存数，这里有个近似的计算公式：第四行的free + 第四行的buffers + 第五行的cached/avail Mem

对于内存监控，在top里我们要时刻监控第五行swap交换分区的used，如果这个数值在不断的变化，说明内核在不断进行内存和swap的数据交换，这是真正的内存不够用了
```

| 参数               | 说明             |
| ------------------ | ---------------- |
| 16212604 total     | 物理内存总量     |
| 10135032 used      | 使用中的内存总量 |
| 4243632 free       | 空闲内存总量     |
| 1833940 buff/cache | 缓存的内存量     |

5. 第五行，swap交换分区信息

```sh
KiB Swap:  8126460 total,  8126460 free,        0 used.  5577340 avail Mem
```

| 参数              | 说明             |
| ----------------- | ---------------- |
| 8126460 total     | 交换区总量       |
| 0 used            | 使用的交换区总量 |
| 8126460 free      | 空闲交换区总量   |
| 5577340 avail Mem | 可用交换取总量   |

6. 第六行，空行

7. 第七行以下：各进程（任务）的状态监控： 

   ```sh
     PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND                                                                                                                                                                     
     944 root      20   0  565168  10532   7440 S   0.7  0.1   0:36.70 NetworkManager
   ```

   | 参数    | 说明                                                         |
   | ------- | ------------------------------------------------------------ |
   | PID     | 进程id                                                       |
   | USER    | 进程所有者                                                   |
   | PR      | 进程优先级                                                   |
   | NI      | nice值。负值表示高优先级，正值表示低优先级                   |
   | VIRT    | 进程使用的虚拟内存总量，单位kb。VIRT=SWAP+RES                |
   | RES     | 进程使用的、未被换出的物理内存大小，单位kb。RES=CODE+DATA    |
   | SHR     | 共享内存大小，单位kb                                         |
   | S       | 进程状态。D=不可中断的睡眠状态 R=运行 S=睡眠 T=跟踪/停止 Z=僵尸进程 |
   | %CPU    | 上次更新到现在的CPU时间占用百分比                            |
   | %MEM    | 进程使用的物理内存百分比                                     |
   | TIME+   | 进程使用的CPU时间总计，单位1/100秒                           |
   | COMMAND | 进程名称（命令名/命令行）                                    |



## netstat:显示各种网络相关信息

Netstat 命令用于显示各种网络相关信息如网络连接，路由表，接口状态，端口信息，masquerade 连接，多播成员 等等。
例如：

```sh
找出程序运行的端口：netstat -ap|grep ssh
找出端口运行的精细化进程: netstat -an|grep ':80'
```


执行**netstat**后，其输出结果为

![image-20211125185454741](D:\0_Notes\Hexo\hmxyl\source\_images\image-20211125185454741.png) 



从整体上看，netstat的输出结果可以分为两个部分：

一个是**Active Internet connections**，称为有源TCP连接，其中"Recv-Q"和"Send-Q"指%0A的是接收队列和发送队列。这些数字一般都应该是0。如果不是则表示软件包正在队列中堆积。这种情况只能在非常少的情况见到。

另一个是**Active UNIX domain sockets**，称为有源Unix域套接口（和网络套接字一样，但是只能用于本机通信，性能可以提高一倍）

| 参数   | 说明                                     |
| ------ | ---------------------------------------- |
| Proto  | 显示连接使用的协议                       |
| RefCnt | 表示连接到本套接口上的进程号             |
| Types  | 显示套接口的类型                         |
| State  | 显示套接口当前的状态                     |
| Path   | 表示连接到套接口的其它进程使用的路径名。 |

常见参数

```sh
-a (all)显示所有选项，默认不显示LISTEN相关
-t (tcp)仅显示tcp相关选项
-u (udp)仅显示udp相关选项
-n 拒绝显示别名，能显示数字的全部转化成数字。
-l 仅列出有在 Listen (监听) 的服務状态
-p 显示建立相关链接的程序名
-r 显示路由信息，路由表
-e 显示扩展信息，例如uid等
-s 按各个协议进行统计
-c 每隔一个固定时间，执行该netstat命令。

提示：LISTEN和LISTENING的状态只有用-a或者-l才能看到
```

实用命令实例

```sh
netstat -a ：列出所有端口 (包括监听和未监听的) 
netstat -at ：列出所有 tcp 端口
netstat -au : 列出所有 udp 端口
netstat -p : 在 netstat 输出中显示 PID 和进程名称 

netstat -p 可以与其它开关一起使用，就可以添加 “PID/进程名称” 到 netstat 输出中，这样 debugging 的时候可以很方便的发现特定端口运行的程序。
```

## nmcli

nmcli使用方法非常类似linux ip命令、cisco交换机命令，并且支持tab补全（详见本文最后的Tips），也可在命令最后通过-h、--help、help查看帮助。

```properties
nmcli --help
Usage: nmcli [OPTIONS] OBJECT { COMMAND | help }
 
OPTIONS
-o[verview] overview mode (hide default values)
-t[erse] terse output
-p[retty] pretty output
-m[ode] tabular|multiline output mode
-c[olors] auto|yes|no whether to use colors in output
-f[ields] <field1,field2,...>|all|common specify fields to output
-g[et-values] <field1,field2,...>|all|common shortcut for -m tabular -t -f
-e[scape] yes|no escape columns separators in values
-a[sk] ask for missing parameters
-s[how-secrets] allow displaying passwords
-w[ait] <seconds> set timeout waiting for finishing operations
-v[ersion] show program version
-h[elp] print this help
 
OBJECT
g[eneral] NetworkManager's general status and operations
n[etworking] overall networking control
r[adio] NetworkManager radio switches
c[onnection] NetworkManager's connections
d[evice] devices managed by NetworkManager
a[gent] NetworkManager secret agent or polkit agent
m[onitor] monitor NetworkManager changes
```

- 常用命令

> nmcli connection

译作连接，可理解为配置文件，相当于ifcfg-ethX。可以简写为nmcli c

> nmcli device

译作设备，可理解为实际存在的网卡（包括物理网卡和虚拟网卡）。可以简写为nmcli d

在NM里，有2个维度：连接（connection）和设备（device），这是多对一的关系。想给某个网卡配ip，首先NM要能纳管这个网卡。设备里存在的网卡（即nmcli d可以看到的），就是NM纳管的。接着，可以为一个设备配置多个连接（即nmcli c可以看到的），每个连接可以理解为一个ifcfg配置文件。同一时刻，一个设备只能有一个连接活跃。可以通过nmcli c up切换连接。

connection有2种状态：

▷ 活跃（带颜色字体）：表示当前该connection生效

▷ 非活跃（正常字体）：表示当前该connection不生效



- nmcli常用命令一览

```properties
# 查看ip（类似于ifconfig、ip addr）
nmcli

# 创建connection，配置静态ip（等同于配置ifcfg，其中BOOTPROTO=none，并ifup启动）
nmcli c add type ethernet con-name ens32 ifname ens32 ipv4.addr 192.168.1.100/24 ipv4.gateway 192.168.1.1 ipv4.method manual

# 创建connection，配置动态ip（等同于配置ifcfg，其中BOOTPROTO=dhcp，并ifup启动）
nmcli c add type ethernet con-name ens32 ifname ens32 ipv4.method auto

# 修改ip（非交互式）
nmcli c modify ens32 ipv4.addr '192.168.1.200/24'
nmcli c up ens32


# 修改ip（交互式）
nmcli c edit ens32
nmcli> goto ipv4.addresses
nmcli ipv4.addresses> change
Edit 'addresses' value: 192.168.1.200/24
Do you also want to set 'ipv4.method' to 'manual'? [yes]: yes
nmcli ipv4> save
nmcli ipv4> activate
nmcli ipv4> quit


# 启用connection（相当于ifup）
nmcli c up ens32

# 停止connection（相当于ifdown）
nmcli c down

# 删除connection（类似于ifdown并删除ifcfg）
nmcli c delete ens32

# 查看connection列表
nmcli c show

# 查看connection详细信息
nmcli c show ens32


# 重载所有ifcfg或route到connection（不会立即生效）
nmcli c reload


# 重载指定ifcfg或route到connection（不会立即生效）
nmcli c load /etc/sysconfig/network-scripts/ifcfg-ens32
nmcli c load /etc/sysconfig/network-scripts/route-ens32


# 立即生效connection，有3种方法
nmcli c up ens32
nmcli d reapply ens32
nmcli d connect ens32

# 查看device列表
nmcli d

# 查看所有device详细信息
nmcli d show<br><br># 查看指定device的详细信息
nmcli d show ens3　　

# 激活网卡
nmcli d connect ens3 

# 关闭无线网络（NM默认启用无线网络）
nmcli r all off

# 查看NM纳管状态
nmcli n

# 开启NM纳管
nmcli n on

# 关闭NM纳管（谨慎执行）
nmcli n off
　　

# 监听事件
nmcli m

# 查看NM本身状态
nmcli

# 检测NM是否在线可用
nm-online
```
