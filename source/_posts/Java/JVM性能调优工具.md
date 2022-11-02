# 工具-jcmd

## jcmd -l

列出当前运行的所有虚拟机 

| 参数 | 说明                         |
| ---- | ---------------------------- |
| -l   | 参数-l表示列出所有java虚拟机 |

## jcmd [pid] help

针对每一个虚拟机，可以使用help命令列出该虚拟机支持的所有命令

```bash
PS C:\Users\Administrator> jcmd 13204 help
13204:
The following commands are available:
JFR.stop
JFR.start
JFR.dump
JFR.check
VM.native_memory
VM.check_commercial_features
VM.unlock_commercial_features
ManagementAgent.stop
ManagementAgent.start_local
ManagementAgent.start
GC.rotate_log
Thread.print
GC.class_stats
GC.class_histogram
GC.heap_dump
GC.run_finalization
GC.run
VM.uptime
VM.flags
VM.system_properties
VM.command_line
VM.version
help
```

其中具体举例

## jcmd  [pid]  VM.uptime

查看虚拟机启动时间

```bash
PS C:\Users\Administrator> jcmd 13204 VM.uptime
13204:
142.109 s
```

## jcmd [pid] Thread.print

 打印线程栈信息（该命令同 [jstack](https://www.jianshu.com/p/025cb069cb69) 命令）

![image-20211202105919556](D:\0_Notes\Hexo\hmxyl\source\_images\image-20211202105919556-1667231157378-120.png)    



## jstack [pid]

如何用Jstack把java进程中的堆栈信息输出到文件

```
-l long listings，会打印出额(防盗连接：本文首发自http://www.cnblogs.com/jilodream/ )外的锁信息，在发生死锁时可以用jstack -l pid来观察锁持有情况

-m mixed mode，不仅会输出Java堆栈信息，还会输出C/C++堆栈信息（比如Native方法）
```



- jps 查看进程ID

  ```bash
  C:\Users\Administrator>jps
  10904 Jps
  3512
  11228 Launcher
  12220 Bank
  ```

- jstack 打印堆栈信息

  ```bash
  C:\Users\Administrator>jstack -l 12220
  2021-12-26 22:09:55
  Full thread dump Java HotSpot(TM) 64-Bit Server VM (25.131-b11 mixed mode):
  
  "DestroyJavaVM" #16 prio=5 os_prio=0 tid=0x00000000029e2800 nid=0xf8 waiting on condition [0x0000000000000000]
     java.lang.Thread.State: RUNNABLE
  
     Locked ownable synchronizers:
          - None
  
  "四号" #15 prio=5 os_prio=0 tid=0x000000001e5c8000 nid=0x3524 waiting for monitor entry [0x000000001f63f000]
     java.lang.Thread.State: BLOCKED (on object monitor)
          at com.hots.chapter7.TickWindowRunnable.run(Bank.java:27)
          - waiting to lock <0x000000076b62bf28> (a java.lang.Object)
          at java.lang.Thread.run(Thread.java:748)
  
     Locked ownable synchronizers:
          - None
  
  "三号" #14 prio=5 os_prio=0 tid=0x000000001e5c7800 nid=0x18b4 waiting for monitor entry [0x000000001f53f000]
     java.lang.Thread.State: BLOCKED (on object monitor)
          at com.hots.chapter7.TickWindowRunnable.run(Bank.java:27)
          - waiting to lock <0x000000076b62bf28> (a java.lang.Object)
          at java.lang.Thread.run(Thread.java:748)
  ...
  ```

    

- jstack 打印堆栈信息到文件

  ```bash
  C:\Users\Administrator>jstack -l 12220 >> test.txt
  
  # 打印结果保存到目录：C:\Users\Administrator下
  ```

  

## jcmd [pid]  GC.class_histogram

查看系统中，类统计信息 。这里和jmap -histo pid的效果是一样的 。这个可以查看每个类的实例数量和占用空间大小。

![image-20211202110015127](D:\0_Notes\Hexo\hmxyl\source\_images\image-20211202110015127-1667231157378-121.png) 

## jcmd [pid] GC.heap_dump [filepath&name]

跟 jmap命令：jmap -dump:format=b,file=heapdump.phrof pid 效果一样。 

导出的 dump 文件，可以使用MAT 或者 Visual VM 等工具进行分析。

```bash
PS C:\Users\Administrator> jcmd 13204 GC.heap_dump D:\test.hprof
13204:
Heap dump file created
```



## jcmd [pid] VM.system_properties

查看 JVM 的属性信息

```bash
PS C:\Users\Administrator> jcmd 13204 VM.system_properties
13204:
#Thu Dec 02 11:03:17 CST 2021
java.vendor=Oracle Corporation
preload.project.path=D\:/Workspace/GIT/XXX
sun.java.launcher=SUN_STANDARD
sun.management.compiler=HotSpot 64-Bit Tiered Compilers
sun.nio.ch.bugLevel=
idea.config.path=C\:/Users/Administrator/AppData/Roaming/JetBrains/IntelliJIdea2021.3
idea.paths.selector=IntelliJIdea2021.3
kotlin.daemon.client.alive.path="C\:\\Users\\ADMINI~1\\AppData\\Local\\Temp\\kotlin-idea-6412110316068151676-is-running"
os.name=Windows 10
sun.boot.class.path=D\:\\Developer\\java\\jdk1.8.0_131\\jre\\lib\\resources.jar;D\:\\Developer\\java\\jdk1.8.0_131\\jre\\lib\\rt.jar;D\:\\Developer\\java\\jdk1.8.0_131\\jre\\lib\\sunrsasign.jar;D\:\\Developer\\java\\jdk1.8.0_131\\jre\\lib\\jsse.jar;D\:\\Developer\\java\\jdk1.8.0_131\\jre\\lib\\jce.jar;D\:\\Developer\\java\\jdk1.8.0_131\\jre\\lib\\charsets.jar;D\:\\Developer\\java\\jdk1.8.0_131\\jre\\lib\\jfr.jar;D\:\\Developer\\java\\jdk1.8.0_131\\jre\\classes
sun.desktop=windows
idea.plugins.path=C\:/Users/Administrator/AppData/Roaming/JetBrains/IntelliJIdea2021.3/plugins
java.vm.specification.vendor=Oracle Corporation
java.runtime.version=1.8.0_131-b11
io.netty.serviceThreadPrefix=Netty
user.name=Administrator
...
```

## jcmd [pid] VM.flags

查看 JVM 的启动参数

```bash
PS C:\Users\Administrator> jcmd 13204 VM.flags
13204:
-XX:CICompilerCount=4 -XX:InitialHeapSize=268435456 -XX:MaxHeapSize=1073741824 -XX:MaxNewSize=357564416 -XX:MinHeapDeltaBytes=524288 -XX:NewSize=89128960 -XX:OldSize=179306496 -XX:+UseCompressedClassPointers -XX:+UseCompressedOops -XX:+UseFastUnorderedTimeStamps -XX:-UseLargePagesIndividualAllocation -XX:+UseParallelGC
PS C:\Users\Administrator>
```



## jcmd [pid] PerfCounter.print

查看 JVM 性能相关的参数

```bash
PS C:\Users\Administrator> jcmd 13204 PerfCounter.print
13204:
java.ci.totalTime=26357322
java.cls.loadedClasses=3480
java.cls.sharedLoadedClasses=0
java.cls.sharedUnloadedClasses=0
java.cls.unloadedClasses=15
java.property.java.class.path="D:/Developer/JetBrains/IntelliJ IDEA 2021.1.1/plugins/java/lib/jps-launcher.jar;D:/Developer/java/jdk1.8.0_131/lib/tools.jar"
java.property.java.endorsed.dirs=""""
java.property.java.ext.dirs="D:\Developer\java\jdk1.8.0_131\jre\lib\ext;C:\Windows\Sun\Java\lib\ext"
java.property.java.home="D:\Developer\java\jdk1.8.0_131\jre"
......
```



## jcmd [pid] GC.run

对 JVM 执行 java.lang.System.gc()

```bash
PS C:\Users\Administrator> jcmd 13204 GC.run
13204:
Command executed successfully
```

## jcmd [pid] GC.run_finalization

对 JVM 执行 java.lang.System.runFinalization()

```bash
PS C:\Users\Administrator> jcmd 13204 GC.run_finalization
13204:
Command executed successfully
```



补充：

```java
system.gc()和system.runFinalization()区别作用：
  
System.gc(); //告诉垃圾收集器打算进行垃圾收集，而垃圾收集器进不进行收集是不确定的 
System.runFinalization();  //强制调用已经失去引用的对象的finalize方法 


// java中的finalize()方法
// 当垃圾收集器认为没有指向对象实例的引用时，会在销毁该对象之前调用finalize() 方法。
// 该方法最常见的作用是确保释放实例占用的全部资源。java并不保证定时为对象实例调用该方法，甚至不保证方法会被调用，所以该方法不应该用于正常内存处理。
```

#### jcmd [pid] VM.command_line

查看 JVM 的启动命令行

```bash
PS C:\Users\Administrator> jcmd 13204 VM.command_line
13204:
VM Arguments:
jvm_args: -Xmx1024m -Djava.awt.headless=true  。。。 -Dtmh.generate.line.numbers=true
java_command: org.jetbrains.jps.cmdline.Launcher D:/Developer/JetBrains/IntelliJ IDEA 2021.1.1/plugins/java/lib/jps-builders.jar;D:/Developer/JetBrains/IntelliJ IDEA 2021.1.1/plugins/java/lib/jps-builders-6.jar;D:/Developer/JetBrains/IntelliJ IDEA 2021.1.1/plugins/java/lib/jps-javac-extension-1.jar;D:/Developer/JetBrains/IntelliJ IDEA 2021.1.1/lib/util.jar;。。。D:/Developer/java/jdk1.8.0_131/lib/tools.jar
Launcher Type: SUN_STANDARD
```

# 工具- jps

1. 概述	

​	jps 命令类似与 linux 的 ps 命令，但是它只列出系统中所有的 Java 应用程序。 通过 jps 命令可以方便地查看 Java 进程的启动类、传入参数和 Java 虚拟机参数等信息。

​	如果在 linux 中想查看 java 的进程，一般我们都需要 ps -ef | grep java 来获取进程 ID。 如果只想获取 Java 程序的进程，可以直接使用 jps 命令来直接查看。

```
PS C:\Users\Administrator> jps
16208
5712 Jps
13204 Launcher
```

2. 用法

   ```
   PS C:\Users\Administrator> jps -help
   usage: jps [-help]
          jps [-q] [-mlvV] [<hostid>]
   
   Definitions:
       <hostid>:      <hostname>[:<port>]
   ```

   参数说明

   | 参数 | 说明                                        |
   | ---- | ------------------------------------------- |
   | -q   | 只输出进程 ID                               |
   | -m   | 输出传入 main 方法的参数                    |
   | -l   | 输出完全的包名，应用主类名，jar的完全路径名 |
   | -v   | 输出jvm参数                                 |
   | -V   | 输出通过flag文件传递到JVM中的参数           |

3. 示例

   - jps

     ```java
     无参数：显示进程的ID 和 启动类的名称。
     ```

   - jps -q 

     ```java
     参数 -q 只输出进程ID，而不显示出类的名称
     ```

     ```bash
     PS C:\Users\Administrator> jps -q
     16208
     13204
     6132
     ```

   - jps -m

     ```java
     可以输出传递给 Java 进程（main 方法）的参数。
     ```

     ```bash
     PS C:\Users\Administrator> jps -m
     16208
     10244 Jps -m
     13204 Launcher D:/Developer/JetBrains/IntelliJ IDEA 2021.1.1/plugins/java/lib/jps-builders.jar;D:/Developer/JetBrains/IntelliJ IDEA 2021.1.1/plugins/java/lib/jps-builders-6.jar;D:/Developer/JetBrains/IntelliJ IDEA 2021.1.1/plugins/java/lib/jps-javac-extension-1.jar;D:/Developer/JetBrains/IntelliJ IDEA 2021.1.1/lib/util.jar;D:/Developer/JetBrains/IntelliJ IDEA 2021.1.1/lib/annotations.jar;D:/Developer/JetBrains/IntelliJ IDEA 2021.1.1/lib/3rd-party-rt.jar;D:/Developer/JetBrains/IntelliJ IDEA 2021.1.1/lib/jna.jar;D:/Developer/JetBrains/IntelliJ IDEA 2021.1.1/lib/kotlin-stdlib-jdk8.jar;D:/Developer/JetBrains/IntelliJ IDEA 2021.1.1/lib/protobuf.jar;D:/Developer/JetBrains/IntelliJ IDEA 2021.1.1/lib/platform-api.jar;D:/Developer/JetBrains/IntelliJ IDEA 2021.1.1/lib/jps-model.jar;D:/Developer/JetBrains/IntelliJ IDEA 2021.1.1/plugins/java/lib/javac2.jar;D:/Developer/JetBrains/IntelliJ IDEA 2021.1.1/lib/forms_rt.jar;D:/Developer/JetBrains/IntelliJ IDEA 2021.1.1/plugins/java/lib/qdox.jar;D:/Developer
     ```

     

   - jps -l

     ```
     可以输出主函数的完整路径（类的全路径）。
     ```

     ```
     PS C:\Users\Administrator> jps -l
     16208
     13204 org.jetbrains.jps.cmdline.Launcher
     5080 sun.tools.jps.Jps
     ```

   - jsp -v

     ```
     可以显示传递给 Java 虚拟机的参数。
     ```

     ```
     PS C:\Users\Administrator> jps -v
     16208  exit -Xms128m -Xmx750m -XX:ReservedCodeCacheSize=512m -XX:+IgnoreUnrecognizedVMOptions -XX:+UseG1GC -XX:SoftRefLRUPolicyMSPerMB=50 -XX:CICompilerCount=2 -XX:+HeapDumpOnOutOfMemoryError -XX:-OmitStackTraceInFastThrow -ea -Dsun.io.useCanonCaches=false -Djdk.http.auth.tunneling.disabledSchemes="" -Djdk.attach.allowAttachSelf=true -Djdk.module.illegalAccess.silent=true -Dkotlinx.coroutines.debug=off -Xms3058m -Xmx5500m -XX:ReservedCodeCacheSize=2048m -XX:SoftRefLRUPolicyMSPerMB=100 -Djb.vmOptionsFile=C:\Users\Administrator\AppData\Roaming\JetBrains\IntelliJIdea2021.3\idea64.exe.vmoptions -Djava.system.class.loader=com.intellij.util.lang.PathClassLoader -Didea.vendor.name=JetBrains -Didea.paths.selector=IntelliJIdea2021.3 -Didea.jre.check=true -Dsplash=true -Dide.native.launcher=true -XX:ErrorFile=C:\Users\Administrator\java_error_in_idea64_%p.log -XX:HeapDumpPath=C:\Users\Administrator\java_error_in_idea64.hprof
     6708 RemoteMavenServer36 -Djava.awt.headless=true -Dmaven.defaultProjectBuilder.disableGlobalModelCache=true -Didea.version=2021.3 -Didea.maven.embedder.version=3.8.1 -Xmx1024m -Dmaven.ext.class.path=D:\Developer\JetBrains\IntelliJ IDEA 2021.1.1\plugins\maven\lib\maven-event-listener.jar -Dfile.encoding=GBK
     16968 Jps -Dapplication.home=D:\Developer\java\jdk1.8.0_131 -Xms8m
     16716 Launcher -Xmx1024m -Djava.awt.headless=true -Djava.endorsed.dirs="" -Dpreload.project.path=D:/Workspace/OWN/Learn -Dpreload.config.path=C:/Users/Administrator/AppData/Roaming/JetBrains/IntelliJIdea2021.3/options -Dexternal.project.config=C:\Users\Administrator\AppData\Local\JetBrains\IntelliJIdea2021.3\external_build_system\learn.11acc208 -Dcompile.parallel=true -Drebuild.on.dependency.change=true -Djdt.compiler.useSingleThread=true -Daether.connector.resumeDownloads=false -Dio.netty.initialSeedUniquifier=1264189494698199162 -Dfile.encoding=GBK -Duser.language=zh -Duser.country=CN -Didea.paths.selector=IntelliJIdea2021.3 -Didea.home.path=D:/Developer/JetBrains/IntelliJ IDEA 2021.1.1 -Didea.config.path=C:/Users/Administrator/AppData/Roaming/JetBrains/IntelliJIdea2021.3 -Didea.plugins.path=C:/Users/Administrator/AppData/Roaming/JetBrains/IntelliJIdea2021.3/plugins -Djps.log.dir=C:/Users/Administrator/AppData/Local/JetBrains/IntelliJIdea2021.3/log/build-log -Djps.fallback.jdk.home=D:/Developer/JetBrains/IntelliJ IDEA 202
     ```

     