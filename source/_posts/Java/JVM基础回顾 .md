

## 线程生命周期

当线程被创建并启动以后，它既不是一启动就进入了执行状态，也不是一直处于执行状态。在线程的生命周期中，它要经过 **新建（New）、就绪（Runnable）、运行（Running）、阻塞（Blocked）和死亡（Dead）5种状态**。尤其是当线程启动以后，它不可能一直"霸占"着CPU独自运行，所以CPU需要在多条线程之间切换，于是 **线程状态也会多次在运行、阻塞之间切换**。



### 0. 生命周期图

![7126254-55d47d5ebef3b1e2](D:\0_Notes\Hexo\hmxyl\source\_images\7126254-55d47d5ebef3b1e2-16380810237142.webp)  





### 1. 新建（New）状态

当程序使用new关键字创建了一个线程之后，该线程就处于 **新建状态**，此时的线程情况如下：

- 此时JVM**为其分配内存，并初始化其成员变量的值**；
- 此时**线程对象**没有表现出任何线程的动态特征，程序也不会执行线程的线程执行体；



### 2. 就绪（Runnable）状态

- 此时JVM会为其**创建方法调用栈和程序计数器**；
- 该状态的线程一直处于**线程就绪队列**（尽管是采用队列形式，事实上，把它称为可运行池而不是可运行队列。因为CPU的调度不一定是按照先进先出的顺序来调度的），线程并没有开始运行；
- 此时线程**等待系统为其分配CPU时间片**，并不是说执行了start()方法就立即执行；

#### 调用start()方法与run()方法对比

1. **调用start()方法来启动线程，系统会把该run()方法当成线程执行体来处理**。
2. 直接调用线程对象的run()方法，则run()方法立即就会被执行，而且在run()方法返回之前其他线程无法并发执行。也就是说，**系统把线程对象当成一个普通对象，而run()方法也是一个普通方法，而不是线程执行体**；

3. 需要指出的是，调用了线程的run()方法之后，**该线程已经不再处于新建状态**，不要再次调用线程对象的start()方法。**只能对处于新建状态的线程调用start()方法，否则将引发==IllegaIThreadStateExccption==异常**；

#### 如何让子线程调用start()方法之后立即执行而非"等待执行"：

程序可以使用Thread.sleep(1) 来让当前运行的线程（主线程）睡眠1毫秒，1毫秒就够了，**因为在这1毫秒内CPU不会空闲，它会去执行另一个处于就绪状态的线程，这样就可以让子线程立即开始执行**；



### 3. 运行（Running）状态

当CPU开始调度处于 **就绪状态** 的线程时，此时线程获得了CPU时间片才得以真正开始执行run()方法的线程执行体，则该线程处于 **运行状态**。

```
如果计算机只有一个CPU，那么在任何时刻只有一个线程处于运行状态；
如果在一个多处理器的机器上，将会有多个线程并行执行，处于运行状态；
当线程数大于处理器数时，依然会存在多个线程在同一个CPU上轮换的现象；
```

处于运行状态的线程最为复杂，它 **不可能一直处于运行状态（除非它的线程执行体足够短，瞬间就执行结束了）**，线程在运行过程中需要被中断，**目的是使其他线程获得执行的机会，线程调度的细节取决于底层平台所采用的策略**。线程状态可能会变为 **阻塞状态、就绪状态和死亡状态**。比如：

1. 对于采用 **抢占式策略** 的系统而言，系统会给每个可执行的线程分配一个时间片来处理任务；当该时间片用完后，系统就会剥夺该线程所占用的资源，让其他线程获得执行的机会。线程就会又 **从运行状态变为就绪状态**，重新等待系统分配资源；

2. 对于采用 **协作式策略**的系统而言，只有当一个线程调用了它的yield()方法后才会放弃所占用的资源—**也就是必须由该线程主动放弃所占用的资源**，线程就会又 **从运行状态变为就绪状态**。



### 4. 阻塞（Blocked）状态

处于运行状态的线程在某些情况下，让出CPU并暂时停止自己的运行，进入 **阻塞状态**。

#### 线程将会进入阻塞状态可能原因

1. **线程调用sleep()方法**，主动放弃所占用的处理器资源，暂时进入中断状态（**不会释放持有的对象锁**），时间到后等待系统分配CPU继续执行；

2. **线程调用一个阻塞式IO方法**，在该方法返回之前，该线程被阻塞；

3. **线程试图获得一个同步监视器**，但该同步监视器正被其他线程所持有;

4. **程序调用了线程的suspend方法将线程挂起**；

5. **线程调用wait**，等待notify/notifyAll唤醒时(会释放持有的对象锁)；

#### 阻塞状态分类

1. **等待阻塞**：运行状态中的 **线程执行wait()方法**，使本线程进入到等待阻塞状态；

2. **同步阻塞**：线程在 **获取synchronized同步锁失败**（因为锁被其它线程占用），它会进入到同步阻塞状态；

3. **其他阻塞**：通过调用线程的 **sleep()或join()或发出I/O请求** 时，线程会进入到阻塞状态。当 **sleep()状态超时、join()等待线程终止或者超时、或者I/O处理完毕** 时，线程重新转入就绪状态；



**在阻塞状态的线程只能进入就绪状态，无法直接进入运行状态**。而就绪和运行状态之间的转换通常不受程序控制，**而是由系统线程调度所决定**。当处于就绪状态的线程获得处理器资源时，该线程进入运行状态；**当处于运行状态的线程失去处理器资源时，该线程进入就绪状态**。

但有一个方法例外，**调用yield()方法可以让运行状态的线程转入就绪状态**。



### 4.1 无限等待（WAITING）状态

线程处于 **无限制等待状态**，等待一个特殊的事件来重新唤醒，如：

> 1. 通过wait()方法进行等待的线程等待一个notify()或者notifyAll()方法；
> 2. 通过join()方法进行等待的线程等待目标线程运行结束而唤醒；

以上两种一旦通过相关事件唤醒线程，线程就进入了 **就绪（RUNNABLE）状态** 继续运行。

#### 4.2 时限等待（TIMED_WAITING）状态

线程进入了一个 **时限等待状态**，如：**sleep(3000)**，等待3秒后线程重新进行 **就绪（RUNNABLE）状态** 继续运行。

### 5. 死亡（Dead）状态

线程会以如下3种方式结束，结束后就处于 **死亡状态**：

1. **run()或call()方法执行完成**，线程正常结束（**TERMINATED状态**）

2. **线程抛出一个未捕获的Exception或Error**；

3. **直接调用该线程stop()方法来结束该线程**（该方法容易导致死锁，通常不推荐使用）

   

**处于死亡状态的线程对象也许是活的，但是，它已经不是一个单独执行的线程**。线程一旦死亡，就不能复生。 **如果在一个死去的线程上调用start()方法，会抛出java.lang.IllegalThreadStateException异常**。所以，需要注意的是：**一旦线程通过start()方法启动后就再也不能回到新建（NEW）状态，线程终止后也不能再回到就绪（RUNNABLE）状态**。



#### 5.1 终止（TERMINATED）状态

线程执行完毕后，进入终止（TERMINATED）状态。

### 6. 线程相关方法

```java
public class Thread{
    // 线程的启动
    public void start(); 
    // 线程体
    public void run(); 
    // 已废弃
    public void stop(); 
    // 已废弃
    public void resume(); 
    // 已废弃
    public void suspend(); 
    // 在指定的毫秒数内让当前正在执行的线程休眠
    public static void sleep(long millis); 
    // 同上，增加了纳秒参数
    public static void sleep(long millis, int nanos); 
    // 测试线程是否处于活动状态
    public boolean isAlive(); 
    // 中断线程
    public void interrupt(); 
    // 测试线程是否已经中断
    public boolean isInterrupted(); 
    // 测试当前线程是否已经中断
    public static boolean interrupted(); 
    // 等待该线程终止
    public void join() throws InterruptedException; 
    // 等待该线程终止的时间最长为 millis 毫秒
    public void join(long millis) throws InterruptedException; 
    // 等待该线程终止的时间最长为 millis 毫秒 + nanos 纳秒
    public void join(long millis, int nanos) throws InterruptedException; 
}
```

#### 6.1 线程就绪、运行和死亡状态转换

1. **就绪状态转换为运行状态**：此线程得到CPU资源；
2. **运行状态转换为就绪状态**：此线程主动调用yield()方法或在运行过程中失去CPU资源。
3. **运行状态转换为死亡状态**：此线程执行执行完毕或者发生了异常；

注意：

当调用线程中的yield()方法时，线程从运行状态转换为就绪状态，**但接下来CPU调度就绪状态中的那个线程具有一定的随机性**，因此，可能会出现A线程调用了yield()方法后，接下来CPU仍然调度了A线程的情况。



#### 6.2 run & start

通过调用start启动线程，线程执行时会执行run方法中的代码。

```java
start()：线程的启动；
run()：线程的执行体；
```

#### 6.3 sleep & yield

##### sleep()

```java
通过sleep(millis)使线程进入休眠一段时间，该方法在指定的时间内无法被唤醒，同时也不会释放对象锁。

sleep是静态方法，最好不要用Thread的实例对象调用它，因为它睡眠的始终是当前正在运行的线程，而不是调用它的线程对象，它只对正在运行状态的线程对象有效

Java线程调度是Java多线程的核心，只有良好的调度，才能充分发挥系统的性能，提高程序的执行效率。但是不管程序员怎么编写调度，只能最大限度的影响线程执行的次序，而不能做到精准控制。因为使用sleep方法之后，线程是进入阻塞状态的，只有当睡眠的时间结束，才会重新进入到就绪状态，而就绪状态进入到运行状态，是由系统控制的，我们不可能精准的去干涉它，所以如果调用Thread.sleep(1000)使得线程睡眠1秒，可能结果会大于1秒。
```

##### yield()

```
与sleep类似，也是Thread类提供的一个静态的方法，它也可以让当前正在执行的线程暂停，让出CPU资源给其他的线程。但是和sleep()方法不同的是，它不会进入到阻塞状态，而是进入到就绪状态。

yield()方法只是让当前线程暂停一下，重新进入就绪线程池中，让系统的线程调度器重新调度器重新调度一次，完全可能出现这样的情况：当某个线程调用yield()方法之后，线程调度器又将其调度出来重新进入到运行状态执行。

实际上，当某个线程调用了yield()方法暂停之后，优先级与当前线程相同，或者优先级比当前线程更高的就绪状态的线程更有可能获得执行的机会，当然，只是有可能，因为我们不可能精确的干涉cpu调度线程。
```

**关于sleep()方法和yield()方的区别如下**

```
sleep方法暂停当前线程后，会进入阻塞状态，只有当睡眠时间到了，才会转入就绪状态。而yield方法调用后 ，是直接进入就绪状态，所以有可能刚进入就绪状态，又被调度到运行状态；

sleep方法声明抛出了InterruptedException，所以调用sleep方法的时候要捕获该异常，或者显示声明抛出该异常。而yield方法则没有声明抛出任务异常；

sleep方法比yield方法有更好的可移植性，通常不要依靠yield方法来控制并发线程的执行；
```

#### 6.4 join

线程的合并的含义就是 **将几个并行线程的线程合并为一个单线程执行**，应用场景是 **当一个线程必须等待另一个线程执行完毕才能执行时**，Thread类提供了join方法来完成这个功能，**注意，它不是静态方法**。

join有3个重载的方法：

```
void join()    
    当前线程等该加入该线程后面，等待该线程终止。    
void join(long millis)    
    当前线程等待该线程终止的时间最长为 millis 毫秒。 如果在millis时间内，该线程没有执行完，那么当前线程进入就绪状态，重新等待cpu调度   
void join(long millis,int nanos)    
    等待该线程终止的时间最长为 millis 毫秒 + nanos 纳秒。如果在millis时间内，该线程没有执行完，那么当前线程进入就绪状态，重新等待cpu调度

```



举例：

```java
/**
 * 在主线程中调用thread.join(); 就是将主线程加入到thread子线程后面等待执行。不过有时间限制，为1毫秒。
 */
public class Test1 {
    public static void main(String[] args) throws InterruptedException {
        MyThread t = new MyThread();
        t.start();
        //将主线程加入到子线程后面，不过如果子线程在1毫秒时间内没执行完，则主线程便不再等待它执行完，进入就绪状态，等待cpu调度  
        t.join(1);
        for (int i = 0; i < 30; i++) {
            System.out.println(Thread.currentThread().getName() + "线程第" + i + "次执行");
        }
    }
}

class MyThread extends Thread {
    @Override
    public void run() {
        for (int i = 0; i < 1000; i++) {
            System.out.println(this.getName() + "线程，第" + i + "次执行");
        }
    }
}
```

**在JDK中join方法的源码，如下：**

```java
public final synchronized void join(long millis)  throws InterruptedException {  
    long base = System.currentTimeMillis();  
    long now = 0;  
  
    if (millis < 0) {  
        throw new IllegalArgumentException("timeout value is negative");  
    }  
          
    if (millis == 0) {  
        while (isAlive()) {  
           wait(0);  
        }  
    } else {  
        while (isAlive()) {  
            long delay = millis - now;  
            if (delay <= 0) {  
                break;  
            }  
            wait(delay);  
            now = System.currentTimeMillis() - base;  
        }  
    }  
}  
```



**join方法实现是通过调用wait方法实现**。当main线程调用t.join时候，**main线程会获得线程对象t的锁（wait 意味着拿到该对象的锁)，调用该对象的wait(等待时间)，直到该对象唤醒main线程**，比如退出后。**这就意味着main 线程调用t.join时，必须能够拿到线程t对象的锁**。



#### 6.5 suspend & resume (已过时)

suspend-**线程进入阻塞状态，但不会释放锁**。此方法已不推荐使用，**因为同步时不会释放锁，会造成死锁的问题**。

resume-**使线程重新进入可执行状态**。

为什么 Thread.suspend 和 Thread.resume 被废弃了？

Thread.suspend 天生容易引起死锁。**如果目标线程挂起时在保护系统关键资源的监视器上持有锁，那么其他线程在目标线程恢复之前都无法访问这个资源。如果要恢复目标线程的线程在调用 resume 之前试图锁定这个监视器，死锁就发生了**。这种死锁一般自身表现为“冻结（ frozen ）”进程。

**其他相关资料：**

> https://blog.csdn.net/dlite/article/details/4212915

#### 6.6 stop（已过时）

**不推荐使用，且以后可能去除，因为它不安全**。为什么 Thread.stop 被废弃了？

因为其天生是不安全的。**停止一个线程会导致其解锁其上被锁定的所有监视器（监视器以在栈顶产生ThreadDeath异常的方式被解锁）**。如果之前被这些监视器保护的任何对象处于不一致状态，其它线程看到的这些对象就会处于不一致状态。**这种对象被称为受损的 （damaged）**。当线程在受损的对象上进行操作时，会导致任意行为。这种行为可能微妙且难以检测，也可能会比较明显。

**不像其他未受检的（unchecked）异常， ThreadDeath 悄无声息的杀死及其他线程**。因此，用户得不到程序可能会崩溃的警告。崩溃会在真正破坏发生后的任意时刻显现，甚至在数小时或数天之后。

**其他相关资料：**

> https://blog.csdn.net/dlite/article/details/4212915

#### 6.7 wait & notify/notifyAll

wait & notify/notifyAll这三个都是Object类的方法。使用 wait ，notify 和 notifyAll **前提是先获得调用对象的锁**。

> 1. 调用 wait 方法后，释放持有的对象锁，**线程状态有 Running 变为 Waiting**，并将当前线程放置到对象的 **等待队列**；
> 2. 调用notify 或者 notifyAll 方法后，**等待线程依旧不会从 wait 返回，需要调用 noitfy 的线程释放锁之后，等待线程才有机会从 wait 返回**；
> 3. notify 方法：**将等待队列的一个等待线程从等待队列中移到同步队列中** ，而 notifyAll 方法：**将等待队列中所有的线程全部移到同步队列，被移动的线程状态由 Waiting 变为 Blocked**。

前面一直提到两个概念，**等待队列（等待池），同步队列（锁池）**，这两者是不一样的。具体如下：

> **同步队列（锁池）**：假设线程A已经拥有了某个对象（注意:不是类）的锁，而其它的线程想要调用这个对象的某个synchronized方法(或者synchronized块)，由于这些线程在进入对象的synchronized方法之前必须先获得该对象的锁的拥有权，但是该对象的锁目前正被线程A拥有，**所以这些线程就进入了该对象的同步队列（锁池）中，这些线程状态为Blocked**。
>
> **等待队列（等待池）**：假设一个线程A调用了某个对象的wait()方法，线程A就会释放该对象的锁（因为wait()方法必须出现在synchronized中，这样自然在执行wait()方法之前线程A就已经拥有了该对象的锁），同时 **线程A就进入到了该对象的等待队列（等待池）中，此时线程A状态为Waiting**。如果另外的一个线程调用了相同对象的notifyAll()方法，那么 **处于该对象的等待池中的线程就会全部进入该对象的同步队列（锁池）中，准备争夺锁的拥有权**。如果另外的一个线程调用了相同对象的notify()方法，那么 **仅仅有一个处于该对象的等待池中的线程（随机）会进入该对象的同步队列（锁池）**。



#### 6.8 线程优先级

每个线程执行时都有一个优先级的属性，**优先级高的线程可以获得较多的执行机会，而优先级低的线程则获得较少的执行机会**。与线程休眠类似，线程的优先级仍然无法保障线程的执行次序。只不过，**优先级高的线程获取CPU资源的概率较大，优先级低的也并非没机会执行**。

> **每个线程默认的优先级都与创建它的父线程具有相同的优先级，在默认情况下，main线程具有普通优先级**；

Thread类提供了setPriority(int newPriority)和getPriority()方法来设置和返回一个指定线程的优先级，其中setPriority方法的参数是一个整数，范围是1~10之间，也可以使用Thread类提供的三个静态常量：

```
MAX_PRIORITY   =10
MIN_PRIORITY   =1
NORM_PRIORITY   =5
```

**注意一点**：

> 虽然Java提供了10个优先级别，但这些优先级别需要操作系统的支持。**不同的操作系统的优先级并不相同，而且也不能很好的和Java的10个优先级别对应**。所以我们应该使用MAX_PRIORITY、MIN_PRIORITY和NORM_PRIORITY三个静态常量来设定优先级，**这样才能保证程序最好的可移植性**。



#### 6.9 守护线程

守护线程与普通线程写法上基本没啥区别，**调用线程对象的方法setDaemon(true)**，则可以将其设置为守护线程。

守护线程使用的情况较少，但并非无用，举例来说，**JVM的垃圾回收、内存管理等线程都是守护线程**。还有就是在做数据库应用时候，使用的数据库连接池，**连接池本身也包含着很多后台线程，监控连接个数、超时时间、状态等等**。

**setDaemon方法详细说明**：

> public final void setDaemon(boolean on)：将该线程标记为守护线程或用户线程。**当正在运行的线程都是守护线程时，Java 虚拟机退出**。
>
> **该方法必须在启动线程前调用**。 该方法首先调用该线程的 checkAccess 方法，且不带任何参数。这可能抛出 SecurityException（在当前线程中）。
>
> **参数：**
>
> ```
> on - 如果为 true，则将该线程标记为守护线程。
> ```
>
> **抛出：**
>
> ```
> IllegalThreadStateException - 如果该线程处于活动状态。
> SecurityException - 如果当前线程无法修改该线程。
> ```

```
/** 
* Java线程：线程的调度-守护线程 
*/  
public class Test {  
        public static void main(String[] args) {  
                Thread t1 = new MyCommon();  
                Thread t2 = new Thread(new MyDaemon());  
                t2.setDaemon(true);        //设置为守护线程  
  
                t2.start();  
                t1.start();  
        }  
}  
  
class MyCommon extends Thread {  
        public void run() {  
                for (int i = 0; i < 5; i++) {  
                        System.out.println("线程1第" + i + "次执行！");  
                        try {  
                                Thread.sleep(7);  
                        } catch (InterruptedException e) {  
                                e.printStackTrace();  
                        }  
                }  
        }  
}  
  
class MyDaemon implements Runnable {  
        public void run() {  
                for (long i = 0; i < 9999999L; i++) {  
                        System.out.println("后台线程第" + i + "次执行！");  
                        try {  
                                Thread.sleep(7);  
                        } catch (InterruptedException e) {  
                                e.printStackTrace();  
                        }  
                }  
        }  
}  
```

执行结果：

```
后台线程第0次执行！  
线程1第0次执行！  
线程1第1次执行！  
后台线程第1次执行！  
后台线程第2次执行！  
线程1第2次执行！  
线程1第3次执行！  
后台线程第3次执行！  
线程1第4次执行！  
后台线程第4次执行！  
后台线程第5次执行！  
后台线程第6次执行！  
后台线程第7次执行！ 
复制代码
```

从上面的执行结果可以看出：**前台线程是保证执行完毕的，后台线程还没有执行完毕就退出了**。

> 实际上：**JRE判断程序是否执行结束的标准是所有的前台执线程行完毕了，而不管后台线程的状态，因此，在使用后台线程时候一定要注意这个问题**。



#### 6.10 如何结束一个线程

**Thread.stop()、Thread.suspend、Thread.resume、Runtime.runFinalizersOnExit** 这些终止线程运行的方法已经被废弃了，使用它们是极端不安全的！想要安全有效的结束一个线程，可以使用下面的方法。

> 1. 正常执行完run方法，然后结束掉；
> 2. 控制循环条件和判断条件的标识符来结束掉线程；

**比如run方法这样写**：只要保证在一定的情况下，run方法能够执行完毕即可。而不是while(true)的无限循环。

```
class MyThread extends Thread {  
    int i=0;  
    @Override  
    public void run() {  
        while (true) {  
            if(i==10)  
                break;  
            i++;  
            System.out.println(i);  
              
        }  
    }  
}  
或者
class MyThread extends Thread {  
    int i=0;  
    boolean next=true;  
    @Override  
    public void run() {  
        while (next) {  
            if(i==10)  
                next=false;  
            i++;  
            System.out.println(i);  
        }  
    }  
}  
或者
class MyThread extends Thread {  
    int i=0;  
    @Override  
    public void run() {  
        while (true) {  
            if(i==10)  
                return;  
            i++;  
            System.out.println(i);  
        }  
    }  
}  
复制代码
```

诚然，使用上面方法的标识符来结束一个线程，是一个不错的方法，但其也有弊端，如果 **该线程是处于sleep、wait、join的状态时候，while循环就不会执行**，那么我们的标识符就无用武之地了，**当然也不能再通过它来结束处于这3种状态的线程了**。

**所以，此时可以使用interrupt这个巧妙的方式结束掉这个线程**。我们先来看看sleep、wait、join方法的声明：

```
public final void wait() throws InterruptedException 
public static native void sleep(long millis) throws InterruptedException
public final void join() throws InterruptedException
复制代码
```

可以看到，这三者有一个共同点，都抛出了一个InterruptedException的异常。**在什么时候会产生这样一个异常呢**？

> **每个Thread都有一个中断状状态，默认为false**。可以通过Thread对象的isInterrupted()方法来判断该线程的中断状态。可以通过Thread对象的interrupt()方法将中断状态设置为true。
>
> 当一个线程处于sleep、wait、join这三种状态之一的时候，**如果此时他的中断状态为true，那么它就会抛出一个InterruptedException的异常**，并将中断状态重新设置为false。

看下面的简单的例子：

```
public class Test1 {  
    public static void main(String[] args) throws InterruptedException {  
        MyThread thread=new MyThread();  
        thread.start();  
    }  
}  
  
class MyThread extends Thread {  
    int i=1;  
    @Override  
    public void run() {  
        while (true) {  
            System.out.println(i);  
            System.out.println(this.isInterrupted());  
            try {  
                System.out.println("我马上去sleep了");  
                Thread.sleep(2000);  
                this.interrupt();  
            } catch (InterruptedException e) {  
                System.out.println("异常捕获了"+this.isInterrupted());  
                return;  
            }  
            i++;  
        }  
    }  
}  
```

测试结果：

```
1  
false  
我马上去sleep了  
2  
true  
我马上去sleep了  
异常捕获了false 
```

可以看到，首先执行第一次while循环，在第一次循环中，睡眠2秒，然后将中断状态设置为true。**当进入到第二次循环的时候，中断状态就是第一次设置的true，当它再次进入sleep的时候，马上就抛出了InterruptedException异常，然后被我们捕获了**。然后中断状态又被重新自动设置为false了（从最后一条输出可以看出来）。

所以，我们可以使用interrupt方法结束一个线程。具体使用如下：

```
public class Test1 {  
    public static void main(String[] args) throws InterruptedException {  
        MyThread thread=new MyThread();  
        thread.start();  
        Thread.sleep(3000);  
        thread.interrupt();  
    }  
}  
  
class MyThread extends Thread {  
    int i=0;  
    @Override  
    public void run() {  
        while (true) {  
            System.out.println(i);  
            try {  
                Thread.sleep(1000);  
            } catch (InterruptedException e) {  
                System.out.println("中断异常被捕获了");  
                return;  
            }  
            i++;  
        }  
    }  
} 
```

多测试几次，会发现一般有两种执行结果：

```
0  
1  
2  
中断异常被捕获了
```

或者

```
0  
1  
2  
3  
中断异常被捕获了 
```

这两种结果恰恰说明了，**只要一个线程的中断状态一旦为true，只要它进入sleep等状态，或者处于sleep状态，立马回抛出InterruptedException异常**。

> **第一种情况**，是当主线程从3秒睡眠状态醒来之后，调用了子线程的interrupt方法，此时子线程正处于sleep状态，立马抛出InterruptedException异常。
>
> **第二种情况**，是当主线程从3秒睡眠状态醒来之后，调用了子线程的interrupt方法，此时子线程还没有处于sleep状态。然后再第3次while循环的时候，在此进入sleep状态，立马抛出InterruptedException异常。



----



##  JVM Class Loader



### 结束一个JVM的生命周期

1. System.exit：`Runtime.getRuntime().exit(status)`

2. 正常执行结束

3. 程序抛出异常

4. JVM Crash（异常）

5. 操作系统/硬件异常

   

### 类加载的三个过程

查看class文件的二进制代码`javap -v BasicThread`

```sh
cd target/classes/com/hots/part1/chapter1
javap -v BasicThread
```



![类加载的三个过程](D:\0_Notes\Hexo\hmxyl\source\_images\类加载的三个过程.png)  

- 加载（Loading）: 查找并且加载类的二进制数据

- 链接（Linking）

  1. 验证：确保被加载类的准确性
  2. 准备：为类的静态变量分配内存，**并将其初始化为默认值**
  3. 解析：把类中的符号引用转换为直接引用

- 初始化（Initialize）：为类的静态变量赋予正确的初始值

  

#### Java程序对类的使用方式：主动使用和被动使用

 除了下述六个外，其余的都是被动使用，不会导致类的初始化。

1. new 直接使用

2. 初始化一个子类，其父类也会被初始化

3. 访问某个类或者接口的静态变量，或者对该静态变量进行赋值
   - 对类的静态变量进行读写
   - 对接口的静态变量进行读取（默认 final static）

4. 调用静态方法

5. 反射某个类

6. 启动类。比如 java HelloWorld



特例说明几个非主动引用：

1. 通过子类访问父类的static变量，不会导致子类的初始化（主动引用的是父类）

   ```java
   System.out.println(Child.salary);
   ```

2. 定义引用数组，不会初始化类

   ```java
   Obj[] arrays = new Obj[10];
   ```

3. `final`修饰的==**常量**==会在编译期间放到常量池中，不会初始化类

4. `final`修饰的==**复杂类型**==，在编译期间无法计算得出，会初始化类

   ```java
   import java.util.Random;
   
   public class ClassActiveUse {
       static {
           System.out.println("ClassActiveUse");
       }
   
       public static void main(String[] args) throws ClassNotFoundException {
           // final修饰的常量会在编译期间放到常量池中，不会初始化类
           System.out.println(Obj.salary);
           System.out.println("--------------------------------------");
           // final修饰的复杂类型，在编译期间无法计算得出，会初始化类
           System.out.println(Obj.x);
       }
   }
   
   class Obj {
   
       public static final long salary = 100000L;
   
       public static final int x = new Random().nextInt(100);
   
       static {
           System.out.println("Obj 被初始化.");
       }
   }
   ```

   输出

   ```java
   ClassActiveUse
   100000
   --------------------------------------
   Obj 被初始化.
   27
   ```

#### 加载（Loading）

##### 加载的定义

将class文件中的二进制数据读取到内存中，将其放在方法区中，然后在堆中创建一个java.lang.Class对象，用来封装方法区的数据结构。

#####  加载类的方式

- 本地磁盘中直接加载

- 内存中直接加载

- 通过网络加载class

- 从zip、jar等归档文件中加载.class文件

- 数据库中提取.class文件

- 动态编译

  

### Class和Object对象在内存中

![image-20220222143225196](D:\0_Notes\Hexo\hmxyl\source\_images\image-20220222143225196.png) 

##### 创建一个对象的过程

![image-20220211165432806](D:\0_Notes\Hexo\hmxyl\source\_images\image-20220211165432806.png) 

或者

![image-20220211165501885](D:\0_Notes\Hexo\hmxyl\source\_images\image-20220211165501885.png) 

#### 链接（Linking）

#### 初始化（Initialize）

- 类加载过程中的最后一步
- 初始化阶段是执行构造函数`<cinit>`方法的过程

- `<cinit>`方法是由编译器自动收集类中的所有变量的赋值动作和静态语句块中的语句合并产生的
- 静态语句块中只能访问到定义在静态语句块之前的变量，定义在他之后的变量，只能赋值，不能访问。
- `<cinit>`方法与类的构造函数的区别：他不需要显式的调用父类的构造函数，虚拟机会保证在子类的`<cinit>`执行之前，先执行父类的`<cinit>`，因此，在虚拟机中首先被执行的是Object的`<cinit>`方法
- 由于父类的`<cinit>`方法先被执行，也就意味着父类中定义的静态语句块，要优于子类
- `<cinit>`方法对于一个类来说并不是必须的
- 接口中照样存在`<cinit>`方法
- 虚拟机有义务保证`<cinit>`方法的线程安全

### JVM类加载器

#### 父委托机制

1. 类加载器的委托是优先交给父亲加载器先去尝试加载
2. 父加载器和子加载器其实是一种包装关系，或者包含关系

![image-20220211172948276](D:\0_Notes\Hexo\hmxyl\source\_images\image-20220211172948276.png) 



```java
public class BootClassLoader {

    public static void main(String[] args) throws ClassNotFoundException {
        // 根（Bootstrap）类加载器加载的内容
        System.out.println(System.getProperty("sun.boot.class.path"));
        System.out.println("------------------");
        // 扩展（Extension）类加载器加载的内容
        System.out.println(System.getProperty("java.ext.dirs"));
        System.out.println("------------------");

        Class<?> klass = Class.forName("example.chapter2.SimpleObject");
        // 系统(Application)类加载器
        // sun.misc.Launcher$AppClassLoader@18b4aac2
        System.out.println(klass.getClassLoader());
        // 扩展类加载器
        // sun.misc.Launcher$ExtClassLoader@6d6f6e28
        System.out.println(klass.getClassLoader().getParent());
        // 根加载器是由C++写的，输出为null。
        System.out.println(klass.getClassLoader().getParent().getParent());

        // 无法获取到自定义的String类
        // 原因：父加载器中存在，优先返回父加载器
        Class<?> clazz = Class.forName("java.lang.String");
        System.out.println(clazz);
        System.out.println(clazz.getClassLoader());
    }
}
```

#### 类加载器的命名空间

- 每个类的加载器都有子命名空间。命名空间由该加载器和其所有父加载器的类组成

- 在同一个命名空间中，不会出现完整的名字

  ![image-20220212115459354](D:\0_Notes\Hexo\hmxyl\source\_images\image-20220212115459354.png) 

#### 运行时包

- 父类加载器看不到子类加载器加载的类

- 不同命名空间下的类加载器之间的类互相不可访问

  ```java
  可以理解为：运行时，类的实际包名 = 类加载器名称 + 包名
  
  //Boot.Ext.App.com.wangwenjun.concurrent.classloader.chapter5.SimpleClassLoaderTest
  //Boot.Ext.App.SimpleClassLoader.com.wangwenjun.concurrent.classloader.chapter5.SimpleClassLoaderTest
  ```

### 类的卸载以及ClassLoader的卸载（GC）

JVM中的Class只有满足以下三个条件，才能被GC回收，也就是该Class被卸载（unload）

- 该类所有的实例都已经被GC
- 加载该类的ClassLoader实例被GC
- 该类的java.lang.Class对象没有在任何地方被引用

GC的时机我们是不可控的，那么同样的我们对于Class的卸载也是不可控的

### 线程上下文类加载器

举例

```java
Class.forName("com.mysql.jdbc.Driver")
```





## JVM内存结构

<img src="D:\0_Notes\Hexo\hmxyl\source\_images\v2-abefb713de46f1e6dd241246c0afe263_720w.jpg" alt="img"/>    

JVM的内存结构大概分为：

- 堆（Heap）：线程共享。所有的==对象实例==以及==数组==都要在堆上分配。**回收器主要管理的对象**。
- 方法区（Method Area）：线程共享。存储类信息、常量、静态变量、即时编译器编译后的代码。（非堆）
- 虚拟机栈（JVM Stack）：线程私有。存储局部变量表、操作栈、动态链接、方法出口，对象指针。
- 本地方法栈（Native Method Stack）：线程私有。为虚拟机使用到的Native 方法服务。如Java使用c或者c++编写的接口服务时，代码在此区运行。
- 程序计数器（Program Counter Register）：线程私有。有些文章也翻译成PC寄存器（PC Register）。可看作是当前线程所执行的字节码的行号指示器。指向下一条要执行的指令。

先看一张图，这张图能很清晰的说明JVM内存结构的布局和相应的控制参数：

<img src="D:\0_Notes\Hexo\hmxyl\source\_images\v2-8845236d1ab9f22fcc658375967d53fb_720w.jpg" alt="img" style="zoom:150%;" /> 

### 堆

堆的作用是存放对象实例和数组。从结构上来分，可以分为新生代和老年代。而新生代又可以分为Eden 空间、From Survivor 空间（s0）、To Survivor 空间（s1）。

所有新生成的对象首先都是放在新生代的。

需要注意，Survivor的两个区是对称的，没先后关系，所以同一个区中可能同时存在从Eden复制过来的对象，和从前一个Survivor复制过来的对象，而复制到老年代的只有从第一个Survivor区过来的对象。而且，Survivor区总有一个是空的。

- 控制参数

-Xms设置堆的最小空间大小。-Xmx设置堆的最大空间大小。

-XX:NewSize设置新生代最小空间大小。-XX:MaxNewSize设置新生代最大空间大小。



- 垃圾回收

此区域是垃圾回收的主要操作区域。



- 异常情况

如果在堆中没有内存完成实例分配，并且堆也无法再扩展时，将会抛出OutOfMemoryError 异常

### 方法区

方法区（Method Area）与Java 堆一样，**是各个线程共享的内存区域**，它用于存储已被虚拟机加载的类信息、常量、静态变量、即时编译器编译后的代码等数据。

虽然Java 虚拟机规范把方法区描述为堆的一个逻辑部分，但是它却有一个别名叫做`Non-Heap（非堆）`，目的应该是与Java 堆区分开来。



很多人愿意把方法区称为“永久代”（Permanent Generation），本质上两者并不等价，仅仅是因为HotSpot虚拟机的设计团队选择把GC 分代收集扩展至方法区，或者说使用永久代来实现方法区而已。对于其他虚拟机（如BEA JRockit、IBM J9 等）来说是不存在永久代的概念的。在Java8中永生代彻底消失了。



- 控制参数

-XX:PermSize 设置最小空间 -XX:MaxPermSize 设置最大空间。



- 垃圾回收

对此区域会涉及但是很少进行垃圾回收。这个区域的内存回收目标主要是针对常量池的回收和对类型的卸载，一般来说这个区域的回收“成绩”比较难以令人满意。



- 异常情况

根据Java 虚拟机规范的规定， 当方法区无法满足内存分配需求时，将抛出OutOfMemoryError。

### 方法栈（虚拟机栈）

每个线程会有一个私有的栈。每个线程中方法的调用又会在本栈中创建一个**栈帧**。在方法栈中会存放编译期可知的各种基本数据类型（boolean、byte、char、short、int、float、long、double）、对象引用（reference 类型，它不等同于对象本身。**局部变量表所需的内存空间在编译期间完成分配**，当进入一个方法时，这个方法需要在帧中分配多大的局部变量空间是完全确定的，在方法运行期间不会改变局部变量表的大小。



- 控制参数

-Xss控制每个线程栈的大小。



- 异常情况

在Java 虚拟机规范中，对这个区域规定了两种异常状况：

\- StackOverflowError： 异常线程请求的栈深度大于虚拟机所允许的深度时抛出；

\- OutOfMemoryError 异常： 虚拟机栈可以动态扩展，当扩展时无法申请到足够的内存时会抛出。

### 本地方法栈

本地方法栈（Native Method Stacks）与虚拟机栈所发挥的作用是非常相似的，其区别不过是虚拟机栈为虚拟机执行Java 方法（也就是字节码）服务，而本地方法栈则

是为虚拟机使用到的Native 方法服务。

- 控制参数

在Sun JDK中本地方法栈和方法栈是同一个，因此也可以用-Xss控制每个线程的大小。

- 异常情况

与虚拟机栈一样，本地方法栈区域也会抛出StackOverflowError 和OutOfMemoryError异常。



### 程序计数器

它的作用可以看做是当前线程所执行的字节码的行号指示器。

- 异常情况：

  此内存区域是唯一一个在Java 虚拟机规范中没有规定任何OutOfMemoryError 情况的区域。



----



##  常见内存溢出错误说明

有了对内存结构清晰的认识，就可以帮助我们理解不同的OutOfMemoryErrors，下面列举一些比较常见的内存溢出错误，通过查看冒号“：”后面的提示信息，基本上就能断定是JVM运行时数据的哪个区域出现了问题。

```text
Exception in thread “main”: java.lang.OutOfMemoryError: Java heap space

原因：对象不能被分配到堆内存中。
```



```text
Exception in thread “main”: java.lang.OutOfMemoryError: PermGen space

原因：类或者方法不能被加载到老年代。它可能出现在一个程序加载很多类的时候，比如引用了很多第三方的库。
```



```text
Exception in thread “main”: java.lang.OutOfMemoryError: Requested array size exceeds VM limit

原因：创建的数组大于堆内存的空间。
```



```text
Exception in thread “main”: java.lang.OutOfMemoryError: request <size> bytes for <reason>. Out of swap space?

原因：方法内存分配失败。JNI、本地库或者Java虚拟机都会从本地堆中分配内存空间。
```



```text
Exception in thread “main”: java.lang.OutOfMemoryError: <reason> <stack trace>（Native method）

原因：同样是本地方法内存分配失败，只不过是JNI或者本地方法或者Java虚拟机发现。
```



关于OutOfMemoryError的更多信息可以查看：

[Troubleshooting Memory Leaksdocs.oracle.com/javase/7/docs/webnotes/tsg/TSG-VM/html/memleaks.html](https://link.zhihu.com/?target=https%3A//docs.oracle.com/javase/7/docs/webnotes/tsg/TSG-VM/html/memleaks.html)



## **Java内存模型**



​		由上述对JVM内存结构的描述中，我们知道了堆和方法区是线程共享的。而局部变量，方法定义参数和异常处理器参数就不会在线程之间共享，它们不会有内存可见性问题，也不受内存模型的影响。



**Java线程之间的通信由Java内存模型（本文简称为JMM）控制**

​		JMM决定一个线程对共享变量的写入何时对另一个线程可见。

​		从抽象的角度来看，JMM定义了线程和主内存之间的抽象关系：线程之间的共享变量存储在主内存（main memory）中，每个线程都有一个私有的本地内存（local memory），`本地内存中存储了该线程以读/写共享变量的副本`。

​		本地内存是JMM的一个抽象概念，并不真实存在。它涵盖了缓存，写缓冲区，寄存器以及其他的硬件和编译器优化。

​		Java内存模型的抽象示意图如下：

![img](D:\0_Notes\Hexo\hmxyl\source\_images\v2-b098a84eb7598d70913444a991d1759b_720w.jpg) 

从上图来看，线程A与线程B之间如要通信的话，必须要经历下面2个步骤：

1. 首先，线程A把本地内存A中更新过的共享变量刷新到主内存中去。
2. 通知本地内存共享变量的副本失效，再次读取时需要强制从主内存读取。
3. 然后，线程B到主内存中去读取线程A之前已更新过的共享变量。

下面通过示意图来说明这两个步骤：

![img](D:\0_Notes\Hexo\hmxyl\source\_images\v2-2c452d147bf0d09b14b770d3990740cb_720w.jpg) 

如上图所示，本地内存A和B有主内存中共享变量x的副本。假设初始时，这三个内存中的x值都为0。线程A在执行时，把更新后的x值（假设值为1）临时存放在自己的本地内存A中。当线程A和线程B需要通信时，线程A首先会把自己本地内存中修改后的x值刷新到主内存中，此时主内存中的x值变为了1。随后，线程B到主内存中去读取线程A更新后的x值，此时线程B的本地内存的x值也变为了1。



从整体来看，这两个步骤实质上是线程A在向线程B发送消息，而且**这个通信过程必须要经过主内存**。JMM通过控制主内存与每个线程的本地内存之间的交互，来为java程序员提供内存可见性保证。

## 指令重排序

在执行程序时为了提高性能，编译器和处理器常常会对指令做重排序。

这里说的重排序可以发生在好几个地方：编译器、运行时、JIT等，比如编译器会觉得把一个变量的写操作放在最后会更有效率，编译后，这个指令就在最后了（前提是只要不改变程序的语义，编译器、执行器就可以这样自由的随意优化），一旦编译器对某个变量的写操作进行优化（放到最后），那么在执行之前，另一个线程将不会看到这个执行结果。



当然了，写入动作可能被移到后面，那也有可能被挪到了前面，这样的“优化”有什么影响呢？这种情况下，其它线程可能会在程序实现“发生”之前，看到这个写入动作（这里怎么理解，指令已经执行了，但是在代码层面还没执行到）。通过`内存屏障`的功能，我们可以禁止一些不必要、或者会带来负面影响的重排序优化，在内存模型的范围内，实现更高的性能，同时保证程序的正确性。



下面我们来看一个重排序的例子：

```java
Class Reordering {
  int x = 0, y = 0;
  public void writer() {
    x = 1;
    y = 2;
  }
  public void reader() {
    int r1 = y;
    int r2 = x;
  }
}
```

假设这段代码有2个线程并发执行，线程A执行writer方法，线程B执行reader方法，线程B看到y的值为2，因为把y设置成2发生在变量x的写入之后（代码层面），所以能断定线程B这时看到的x就是1吗？

当然不行！ 因为在writer方法中，可能发生了重排序，y的写入动作可能发在x写入之前，这种情况下，线程B就有可能看到x的值还是0。

在Java内存模型中，描述了在多线程代码中，哪些行为是正确的、合法的，以及多线程之间如何进行通信，代码中变量的读写行为如何反应到内存、CPU缓存的底层细节。

在Java中包含了几个关键字：volatile、final和synchronized，帮助程序员把代码中的并发需求描述给编译器。JMM中定义了它们的行为，确保正确同步的Java代码在所有的处理器架构上都能正确执行。

