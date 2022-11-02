#  Atomic包

## CAS(Compare And Swap)：比较并交换

`CAS`即`Compare And Swap`的缩写，翻译成中文就是**比较并交换**，其作用是让CPU比较内存中某个值是否和预期的值相同，如果相同则将这个值更新为新值，不相同则不做更新，也就是CAS是**原子性**的操作(读和写两者同时具有原子性)，其实现方式是通过借助`C/C++`调用CPU指令完成的，所以效率很高。(使用的是最快失败策略)
 `CAS`的原理很简单，这里使用一段`Java`代码来描述

```java
public boolean compareAndSwap(int value, int expect, int update) {
    // 如果内存中的值value和期望值expect一样 则将值更新为新值update
    if (value == expect) {
        value = update;
        return true;
    } else {
        return false;
    }
}
```

大致过程是将内存中的值、我们的期望值、新值交给CPU进行运算，如果内存中的值和我们的期望值相同则将值更新为新值，否则不做任何操作。这个过程是在CPU中完成的，这里不好描述CPU的工作过程，就拿Java代码来描述了。

### Unsafe源码分析

​    Java是在`Unsafe(sun.misc.Unsafe)`类实现`CAS`的操作，而我们知道Java是无法直接访问操作系统底层的API的（原因是Java的跨平台性限制了Java不能和操作系统耦合），所以Java并没有在`Unsafe`类直接实现`CAS`的操作，而是通过**JDI(Java Native Interface)**本地调用`C/C++`语言来实现`CAS`操作的。

 `Unsafe`有很多个`CAS`操作的相关方法，这里举例几个

```java
public final native boolean compareAndSwapObject(Object var1, long var2, Object var4, Object var5);

public final native boolean compareAndSwapInt(Object var1, long var2, int var4, int var5);

public final native boolean compareAndSwapLong(Object var1, long var2, long var4, long var6);
```

我们拿`public final native boolean compareAndSwapInt(Object var1, long var2, int var4, int var5);`进行分析

这个方法是比较内存中的一个值（整型）和我们的期望值（var4）是否一样，如果一样则将内存中的这个值更新为`var5`，参数中的`var1`是值所在的对象，`var2`是值在对象(var1)中的内存偏移量，**参数var1和参数var2是为了定位出值所在内存的地址**。

![img](D:\0_Notes\Hexo\hmxyl\source\_images\4007291d-9a19-4bf0-96e9-4cf088f7a077.awebp)

**Unsafe.java在这里发挥的作用有：**

1. 将对象引用、值在对象中的偏移量、期望的值和欲更新的新值传递给`Unsafe.cpp`
2. 如果值更新成功则返回`true`给开发者，没有更新则返回`false`

**Unsafe.cpp在这里发挥的作用有：**

1. 接受从`Unsafe`传递过来的对象引用、偏移量、期望的值和欲更新的新值，根据对象引用和偏移量**计算出值的地址**，然后将值的地址、期望的值、欲更新的新值传递给CPU
2. 如果值更新成功则返回`true`给`Unsafe.java`，没有更新则返回`false`

**CPU在这里发挥的作用：**

1. 接受从`Unsafe.cpp`传递过来的地址、期望的值和欲更新的新值，执行指令`cmpxchg`，比较地址中的值是否和期望的值一样，一样则将值更新为新的值，不一样则不做任何操作
2. 将操作结果返回给`Unsafe.cpp`

### CAS的缺点：ABA

**`ABA`说明**

> 在多线程场景下`CAS`会出现`ABA`问题，关于ABA问题这里简单科普下，例如有2个线程同时对同一个值(初始值为A)进行CAS操作，这三个线程如下
>
> 1. 线程1，期望值为A，欲更新的值为B
> 2. 线程2，期望值为A，欲更新的值为B
> 3. 线程3，期望值为B，欲更新的值为A
>
> 线程`1`抢先获得CPU时间片，而线程`2`因为其他原因阻塞了；线程`1`取值与期望的A值比较，发现相等然后将值更新为B；
>
> 这个时候**出现了线程`3`**，线程3取值与期望的值B比较，发现相等则将值更新为A；
>
> 此时线程`2`从阻塞中恢复，并且获得了CPU时间片，这时候线程`2`取值与期望的值A比较，发现相等则将值更新为B
>
> 虽然线程`2`也完成了操作，但是线程`2`并不知道值已经经过了`A->B->A`的变化过程。

**`ABA`问题带来的危害**

 > 小明在提款机，提取了50元，因为提款机问题，有两个线程，同时把余额从100变为50
 >
 >
 > - 线程1（提款机）：获取当前值100，期望更新为50，
 >
 > - 线程2（提款机）：获取当前值100，期望更新为50，
 >
 > 线程1成功执行，线程2某种原因block了，这时，某人给小明汇款50
 >
 > - 线程3（某人）：获取当前值50，期望更新为100，
 >
 > 这时候线程3成功执行，余额变为100，
 > 线程2从Block中恢复，获取到的也是100，compare之后，继续更新余额为50
 >
 > **此时可以看到，实际余额应该为100（100-50+50），但是实际上变为了50（100-50+50-50）这就是ABA问题带来的成功提交。**

```java
@Test
public void testAtomicReference() throws InterruptedException {
  final Integer moneyTotal = 100;
  AtomicInteger money = new AtomicInteger(moneyTotal);

  Thread t1 = new Thread(() -> {
    money.getAndAdd(-50);
    System.out.printf(Thread.currentThread().getName() + "-更新成功（%d->%d）.\n", moneyTotal, money.get());
  });
  t1.start();

  Thread t2 = new Thread(() -> {
    try {
      TimeUnit.SECONDS.sleep(2);
    } catch (InterruptedException e) {
      e.printStackTrace();
    }
    money.getAndAdd(-50);
    System.out.printf(Thread.currentThread().getName() + "-更新成功（%d->%d）.\n", moneyTotal, money.get());
  });
  t2.start();

  Thread t3 = new Thread(() -> {
    money.getAndAdd(50);
    System.out.printf(Thread.currentThread().getName() + "-更新成功（%d->%d）.\n", moneyTotal, money.get());
  });
  t3.start();

  t1.join();
  t2.join();
  t3.join();

  // 输出50，钱数错误
  System.out.println(money.get());
}

输出：
Thread-0-更新成功（100->50）.
Thread-2-更新成功（100->100）.
Thread-1-更新成功（100->50）.
50
```



**`ABA`问题解决：AtomicStampedReference**

**解决方法**： 在变量前面加上版本号（int），每次变量更新的时候变量的**版本号都`+1`**，即`A->B->A`就变成了`1A->2B->3A`。

```java
@Test
public void testAtomicStampedReference() throws InterruptedException {
    final AtomicInteger stamp = new AtomicInteger();
    final Integer moneyTotal = 100;
    AtomicStampedReference money = new AtomicStampedReference(moneyTotal, 0);
    // step1：取款50
    final Integer moneyStep1 = moneyTotal - 50;
    Thread t1 = new Thread(() -> {
        if (money.compareAndSet(moneyTotal, moneyStep1, stamp.get(), stamp.incrementAndGet())) {
            System.out.printf(Thread.currentThread().getName() + "-更新成功（%d->%d）:%d.\n", moneyTotal,
                              money.getReference(), money.getStamp());
        } else {
            System.out.printf(Thread.currentThread().getName() + "-更新失败（%d->%d）:%d.\n", moneyTotal,
                              money.getReference(), money.getStamp());
        }
    }, "T1");
    t1.start();

    Thread t2 = new Thread(() -> {

        TaskFactory.spend(2, TimeUnit.SECONDS);

        if (money.compareAndSet(moneyTotal, moneyStep1, stamp.get(), stamp.incrementAndGet())) {
            System.out.printf(Thread.currentThread().getName() + "-更新成功（%d->%d）:%d.\n", moneyTotal,
                              money.getReference(), money.getStamp());
        } else {
            System.out.printf(Thread.currentThread().getName() + "-更新失败（%d->%d）:%d.\n", moneyTotal,
                              money.getReference(), money.getStamp());
        }
    }, "T2");
    t2.start();
    // step2. 他人转入50
    final Integer moneyStep2 = moneyStep1 + 50;
    Thread t3 = new Thread(() -> {
        if (money.compareAndSet(moneyStep1, moneyStep2, 1, 2)) {
            System.out.printf(Thread.currentThread().getName() + "-更新成功（%d->%d）:%d.\n", moneyTotal,
                              money.getReference(), money.getStamp());
        } else {
            System.out.printf(Thread.currentThread().getName() + "-更新成功（%d->%d）:%d.\n", moneyTotal,
                              money.getReference(), money.getStamp());
        }
    }, "T3");
    t3.start();

    t1.join();
    t2.join();
    t3.join();
    System.out.printf(Thread.currentThread().getName() + "-最终（%d）:%d.\n", money.getReference(), money.getStamp());

}

输出
T1-更新成功（100->50）:1.
T3-更新成功（100->100）:2.
T2-更新失败（100->100）:2.
main-最终（100）:2.
```

### CAS的缺点：循环时间长开销大

如果`CAS`操作失败，就需要循环进行`CAS`操作(循环同时将期望值更新为最新的)，如果长时间都不成功的话，那么会造成CPU极大的开销。

> 这种循环也称为自旋

**解决方法**： 限制自旋次数，防止进入死循环。

###  CAS的缺点：只能保证一个共享变量的原子操作

`CAS`的原子操作只能针对一个共享变量。

**解决方法**： 如果需要对多个共享变量进行操作，可以使用加锁方式(悲观锁)保证原子性，或者可以把多个共享变量合并成一个共享变量进行`CAS`操作。



### CAS的应用

我们知道`CAS`操作并不会锁住共享变量，也就是一种**非阻塞**的同步机制，`CAS`就是乐观锁的实现。

1.  **乐观锁**总是假设最好的情况，每次去操作数据都认为不会被别的线程修改数据，**所以在每次操作数据的时候都不会给数据加锁**，即在线程对数据进行操作的时候，**别的线程不会阻塞**仍然可以对数据进行操作，只有在需要更新数据的时候才会去判断数据是否被别的线程修改过，如果数据被修改过则会拒绝操作并且返回错误信息给用户。
2.  **悲观锁**总是假设最坏的情况，每次去操作数据时候都认为会被的线程修改数据，**所以在每次操作数据的时候都会给数据加锁**，让别的线程无法操作这个数据，别的线程会一直阻塞直到获取到这个数据的锁。这样的话就会影响效率，比如当有个线程发生一个很耗时的操作的时候，别的线程只是想获取这个数据的值而已都要等待很久。

`Java`利用`CAS`的乐观锁、原子性的特性高效解决了多线程的安全性问题，例如JDK1.8中的集合类`ConcurrentHashMap`、关键字`volatile`、`ReentrantLock`等。

## AtomicLong

- 区别于AtomicInteger：VM_SUPPORTS_LONG_CAS：虚拟机是否支持 CAS 操作

  ```java
      /**
       * Records whether the underlying JVM supports lockless
       * compareAndSwap for longs. While the Unsafe.compareAndSwapLong
       * method works in either case, some constructions should be
       * handled at Java level to avoid locking user-visible locks.
       */
      static final boolean VM_SUPPORTS_LONG_CAS = VMSupportsCS8();
  
      /**
       * Returns whether underlying JVM supports lockless CompareAndSet
       * for longs. Called only once and cached in VM_SUPPORTS_LONG_CAS.
       */
      private static native boolean VMSupportsCS8();
  ```



## AtomicReference

reference的地址为int类型

## AtomicXXXFieldUpdater

使用AtomicXXXFieldUpdater的原因：

- 想让类的操作属性具备原子性的条件
  1. 类的属性是volatile（ Must be volatile type）
  2. ==非当前类调用，则非private、protected==
  3. 类型必须一致


- 不想使用锁（包括显示锁、重量级锁Synchronized）
- 大量需要原子类型修饰的对象，比较消耗资源



```java
public class AtomicIntegerFieldUpdaterTest {
    @Test
    public void test() {
        AtomicIntegerFieldUpdater<TestBean> updater = AtomicIntegerFieldUpdater.newUpdater(TestBean.class, "param");
        TestBean test = new TestBean();
        updater.incrementAndGet(test);
        System.out.println(updater.get(test));
    }

    class TestBean {
        // 非本类调用，param 不可设置未private、protected
        volatile int param;
    }
}
```

## Unsafe

​	 java 调用C++/C 再 调用汇编

### 几种Counter方案的性能对比。

```java
import sun.misc.Unsafe;

import java.lang.reflect.Field;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.atomic.AtomicInteger;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;
import java.util.function.Consumer;

public class UnsafeTest {
    private static final int THREAD_COUNT = 1000;
    private static final int MAX_NUM = 10000;

    public static void main(String[] args) {
        doAction(getAction(), new VolatileCounter(), "Volatile");
        doAction(getAction(), new AtomicCounter(), "Executors");
        doAction(getAction(), new SynCounter(), "Sync");
        doAction(getAction(), new LockCounter(), "Lock");
        doAction(getAction(), new CasCounter(), "Cas");

    }

    private static Consumer<Counter> getAction() {
        Consumer<Counter> action = param -> {
            ExecutorService executorService = Executors.newFixedThreadPool(THREAD_COUNT);
            for (int i = 0; i < THREAD_COUNT; i++) {
                executorService.submit(new CounterRunnable(param, MAX_NUM));
            }
            executorService.shutdown();
            try {
                // 不可省略，需要等待执行线程运行结束
                // 一般情况下awaitTermination和shutdown配合使用，shutdown之后调用awaitTermination
                // 如果注释掉shutdown方法，则awaitTermination不会监视到线程池关闭的信息 所以在这个地方代码会堵塞，
                // 如果注释掉awaitTermination方法，则后面的代码不会得到线程执行过的结果
                executorService.awaitTermination(1, TimeUnit.HOURS);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        };
        return action;
    }

    /**
     * 统计运行时长
     *
     * @param action
     * @param counter
     */
    private static void doAction(Consumer<Counter> action, Counter counter, String tag) {
        long begin = System.currentTimeMillis();
        // 任务执行
        action.accept(counter);
        long end = System.currentTimeMillis();
        System.out.println(tag + " counter result: " + counter.getCounter() + " and time passed in ms: " + (end - begin));
    }

    interface Counter {
        void increment();

        long getCounter();
    }


    static class VolatileCounter implements Counter {
        private volatile int counter;

        @Override
        public void increment() {
            ++counter;
        }

        @Override
        public long getCounter() {
            return counter;
        }
    }

    static class AtomicCounter implements Counter {
        private AtomicInteger counter = new AtomicInteger(0);

        @Override
        public void increment() {
            counter.incrementAndGet();
        }

        @Override
        public long getCounter() {
            return counter.get();
        }
    }

    static class SynCounter implements Counter {
        private int counter = 0;

        @Override
        public synchronized void increment() {
            ++counter;
        }

        @Override
        public long getCounter() {
            return counter;
        }
    }


    static class LockCounter implements Counter {
        private int counter = 0;
        private Lock lock = new ReentrantLock();

        @Override
        public void increment() {
            try {
                lock.lock();
                ++counter;
            } finally {
                lock.unlock();
            }
        }

        @Override
        public long getCounter() {
            return counter;
        }
    }

    static class CasCounter implements Counter {
        private int counter = 0;
        private static final Unsafe unsafe = getUnsafe();
        private static final long valueOffset;

        static {
            try {
                valueOffset = unsafe.objectFieldOffset(CasCounter.class.getDeclaredField("counter"));
            } catch (Exception ex) {
                throw new Error(ex);
            }
        }


        @Override
        public void increment() {
            int expect = counter;
            while (!unsafe.compareAndSwapInt(this, valueOffset, expect, expect + 1)) {
                expect = counter;
            }
        }

        @Override
        public long getCounter() {
            return counter;
        }

        private static Unsafe getUnsafe() {
            try {
                Field unsafe = Unsafe.class.getDeclaredField("theUnsafe");
                unsafe.setAccessible(true);
                return (Unsafe) unsafe.get(null);
            } catch (Exception e) {
                throw new RuntimeException(e);
            }
        }
    }

    static class CounterRunnable implements Runnable {
        private final Counter counter;
        private final int num;

        CounterRunnable(Counter counter, int num) {
            this.counter = counter;
            this.num = num;
        }

        @Override
        public void run() {
            synchronized (counter) {
                for (int i = 0; i < num; i++) {
                    counter.increment();
                }
            }
        }
    }
}
```

执行结果

```java
Volatile counter result: 10000000 and time passed in ms: 177
Executors counter result: 10000000 and time passed in ms: 183
Sync counter result: 10000000 and time passed in ms: 1111
Lock counter result: 10000000 and time passed in ms: 204
Cas counter result: 10000000 and time passed in ms: 114
```

### Java 调用 C 流程（JNI ）

1. 创建目录`jni`

2. 创建文件`Hello.java`

   ```java
   public class Hello{
   	static{
   		// 加载动态链接库
   		System.loadLibrary("hello");
   	}
   	
   	// 本地方法
   	public native void hi();
   	
   	public static void main(String[] args){
   		new Hello().hi();
   	}
   }
   ```

3. 编译Java文件`javac Hello.java`

   ![image-20220222105628308](D:\0_Notes\Hexo\hmxyl\source\_images\image-20220222105628308-1667231117133-83.png) 

4. 使用命令`javah -jni Hello`生成头文件`Hello.h`（C的header文件）

   ![image-20220222105835195](D:\0_Notes\Hexo\hmxyl\source\_images\image-20220222105835195-1667231117133-84.png) 

   

   Hello.h内容如下

   ```C
   /* DO NOT EDIT THIS FILE - it is machine generated */
   #include <jni.h>
   /* Header for class Hello */
   
   #ifndef _Included_Hello
   #define _Included_Hello
   #ifdef __cplusplus
   extern "C" {
   #endif
   /*
    * Class:     Hello
    * Method:    hi
    * Signature: ()V
    */
   JNIEXPORT void JNICALL Java_Hello_hi
     (JNIEnv *, jobject);
   
   #ifdef __cplusplus
   }
   #endif
   #endif
   
   ```

5. 编写C程序：`Hello.c`，也就是上面header文件中方法的实现

   ```c
   #include <jni.h>
   #include "Hello.h"
   
   JNIEXPORT void JNICALL Java_Hello_hi (JNIEnv * env, jobject o){
   	printf("Say hi.\n");
   };
   ```

6. 查看`ls -l $JAVA_HOME/include`

7. 编译C文件`gcc -fPIC  -I"$JAVA_HOME/include" -I"$JAVA_HOME/include/linux" -c Hello.c`，生成了`Hello.o`的目标文件

8. 生成"hello" 的动态链接库 `gcc -shared Hello.o -o libhello.so`， 生成了`libhello.so`（`lib` 是linux约定俗成的前缀）

9. 运行java文件：`java Hello`，报错

   ```java
   Exception in thread "main" java.lang.UnsatisfiedLinkError: no hello in java.library.path
   	at java.lang.ClassLoader.loadLibrary(ClassLoader.java:1867)
   	at java.lang.Runtime.loadLibrary0(Runtime.java:870)
   	at java.lang.System.loadLibrary(System.java:1122)
   	at Hello.<clinit>(Hello.java:4)
   ```

10. 配置`java.library.path`. 临时生效：`export LD_LIBRARY_PATH=.`

    ![image-20220222113453875](D:\0_Notes\Hexo\hmxyl\source\_images\image-20220222113453875-1667231117133-86.png) 

11. 重新运行：`java Hello`

    ![image-20220222113625291](D:\0_Notes\Hexo\hmxyl\source\_images\image-20220222113625291-1667231117133-85.png) 



## 底层汇编相关指令

> compareAndSwapInt -> cmpxchg1
>
> compareAndSwapLong -> cmpxchg
>
> putOrderedInt -> xchg1
>
> compareAndSwapObject -> cmpxchgq

# CountDownLatch

```java
A synchronization aid that allows one or more threads to wait until a set of operations being performed in other threads completes.
“允许一个或多个线程等待，直到在其他线程中执行的一组操作完成”的同步算法
```

## 退出条件

1. countDown() 减到0：`await()`
2. 等待时间到了截止时间：`await(long timeout, TimeUnit unit)`

## 使用场景

### 等待所有线程执行完成

```java
package hots.utils;

import java.util.Random;
import java.util.concurrent.CountDownLatch;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.atomic.AtomicInteger;
import java.util.stream.IntStream;

public class CountDownLatchTest {
    private final static AtomicInteger count = new AtomicInteger();
    private final static Random random = new Random();

    private static CountDownLatch latch;
    private static ExecutorService executorService = Executors.newFixedThreadPool(2);

    public static void main(String[] args) throws InterruptedException {
        // step1: 获取查询数据
        int[] data = IntStream.rangeClosed(1, 5).map(e -> random.nextInt(1_000)).toArray();
        latch = new CountDownLatch(data.length);

        // step2：根据查询数据分配多个线程执行
        for (int i = 0; i < data.length; i++) {
            executorService.submit(new SimpleRunnable(latch, count, i, data[i]));
        }
        System.out.printf("All works submitted.\n");
        latch.await();

        executorService.shutdown();
        // step3
        System.out.printf("All works finished. Support with [%d] threads.\n", count.get());

    }

    private static class SimpleRunnable implements Runnable {
        private final int index;
        private final int param;
        private final CountDownLatch latch;
        private final AtomicInteger count;

        SimpleRunnable(CountDownLatch latch, AtomicInteger count, int index, int param) {
            this.index = index;
            this.param = param;
            this.count = count;
            this.latch = latch;
        }

        @Override
        public void run() {
            try {
                Thread.sleep(100);
                System.out.printf("%s deal with [%d]-[%d] \n", Thread.currentThread().getName(), index, param);
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                count.incrementAndGet();
                latch.countDown();
            }
        }
    }
}
```

```java
All works submitted.
pool-1-thread-2 deal with [1]-[546] 
pool-1-thread-1 deal with [0]-[833] 
pool-1-thread-2 deal with [2]-[11] 
pool-1-thread-1 deal with [3]-[247] 
pool-1-thread-2 deal with [4]-[191] 
All works finished. Support with [5] threads.
```





### 任务拆分离散并行化处理

业务流程如下

![CountDown.drawio](D:\0_Notes\Hexo\hmxyl\source\_images\CountDown.drawio-1667231117133-87.png) 

#### 基本信息定义

- 统计表

```java
@Getter
@Setter
class Table {
    // 表名
    private String tableName;
    // 原始记录条数
    private long sourceRecordCount = 10;
    // 传输完成后的记录条数：验证1
    private long targetCount;
    // 原始schema
    private String sourceColumnSchema = "<table name='a'><column name='c1' type='varchar'></column></table>";
    // 传输完成之后的schema：验证2
    private String targetColumnSchema = "";

    public Table(String tableName, long sourceRecordCount) {
        this.tableName = tableName;
        this.sourceRecordCount = sourceRecordCount;
    }
}
```

- 监控工具

```java
abstract class Watcher {
    final CountDownLatch countDownLatch;

    Watcher(CountDownLatch countDownLatch) {
        this.countDownLatch = countDownLatch;
    }

    abstract void done();
}
```

- 事件定义（对应一次批处理任务）

```java
@Getter
@Setter
class Event {
    private String eventName;

    public Event(String eventName) {
        this.eventName = eventName;
    }
}
```

- 批处理任务完成验证

```java
public class EventTaskBatch extends Watcher {
    private final Event event;

    EventTaskBatch(Event event, int taskSize) {
        super(new CountDownLatch(taskSize));
        this.event = event;
    }

    @Override
    void done() {
        countDownLatch.countDown();
        if (countDownLatch.getCount() == 0) {
            // Event涉及到的所有Table任务完成
            System.out.println("All table of event " + event.getEventName() + " is finished verify and update continue.");
            System.out.println();
        }
    }
}
```

- 批处理表验证任务全部完成

```java
class TableTaskBatch extends Watcher {
    private final Table table;

    /* 每张表存在多个验证任务 */
    private EventTaskBatch eventTaskBatch;

    TableTaskBatch(EventTaskBatch eventTaskBatch, Table table, int taskSize) {
        super(new CountDownLatch(taskSize));
        this.table = table;
        this.eventTaskBatch = eventTaskBatch;
    }

    @Override
    public void done() {
        countDownLatch.countDown();
        if (countDownLatch.getCount() == 0) {
            // Table相关所有任务完成
            System.out.println("All tasks of " + table.getTableName() + " is finished verify and update continue.");
            eventTaskBatch.done();
        }
    }
}
```

- 表数据验证行为

```java
abstract class TableVerify implements Runnable {

    protected final Table table;

    protected final TableTaskBatch tableTaskBatch;

    public TableVerify(Table table, TableTaskBatch tableTaskBatch) {
        this.table = table;
        this.tableTaskBatch = tableTaskBatch;
    }
}
```

​		a) 验证1：验证数据量

```java
class TrustSourceRecordCount extends TableVerify{
    TrustSourceRecordCount(Table table, TableTaskBatch tableTaskBatch) {
        super(table, tableTaskBatch);
    }

    @Override
    public void run() {
        TaskFactory.spend(ThreadLocalRandom.current().nextInt(10), TimeUnit.SECONDS);
        // 设置传输完成之后的数据量
        table.setTargetCount(table.getSourceRecordCount());
        // 完成一次一张表的验证完成计数
        tableTaskBatch.done();
    }
}
```

​		b) 验证2：验证表结构

```java
class TrustSourceColumns extends TableVerify {
    TrustSourceColumns(Table table, TableTaskBatch tableTaskBatch) {
        super(table, tableTaskBatch);
    }

    @Override
    public void run() {
        TaskFactory.spend(ThreadLocalRandom.current().nextInt(10), TimeUnit.SECONDS);
        table.setTargetColumnSchema(table.getSourceColumnSchema());
        // 完成一次一张表的验证完成计数
        tableTaskBatch.done();
    }
}
```

- 测试类

```java
public class CountDownLatchTest {

    public static void main(String[] args) {
        ExecutorService executorService = Executors.newFixedThreadPool(10);
        try {
            // 不同数据源的数据，批处理
            Event[] events = {new Event("Event-1"), new Event("Event-2")};
            for (Event event : events) {
                // 获取数据源表资源概况
                List<Table> tables = capture(event);
                EventTaskBatch eventTaskBatch = new EventTaskBatch(event, tables.size());
                for (Table table : tables) {
                    // 与Table相关的任务技术监控。
                    TableTaskBatch tableTaskBatch = new TableTaskBatch(eventTaskBatch, table, 2);
                    executorService.submit(new TrustSourceRecordCount(table, tableTaskBatch));
                    executorService.submit(new TrustSourceColumns(table, tableTaskBatch));
                }
            }
        } finally {
            executorService.shutdown();
        }

    }

    private static List<Table> capture(Event event) {
        List<Table> list = new ArrayList<>();
        for (int i = 0; i < 10; i++) {
            list.add(new Table(event.getEventName() + "-Table-" + i, i * 1000));
        }
        return list;
    }
}
```

 测试结果

```
All tasks of Event-1-Table-3 is finished verify and update continue.
All tasks of Event-1-Table-1 is finished verify and update continue.
All tasks of Event-1-Table-4 is finished verify and update continue.
All tasks of Event-1-Table-6 is finished verify and update continue.
All tasks of Event-1-Table-5 is finished verify and update continue.
All tasks of Event-1-Table-2 is finished verify and update continue.
All tasks of Event-1-Table-0 is finished verify and update continue.
All tasks of Event-1-Table-7 is finished verify and update continue.
All tasks of Event-1-Table-8 is finished verify and update continue.
All tasks of Event-2-Table-4 is finished verify and update continue.
All tasks of Event-2-Table-2 is finished verify and update continue.
All tasks of Event-2-Table-6 is finished verify and update continue.
All tasks of Event-2-Table-3 is finished verify and update continue.
All tasks of Event-2-Table-0 is finished verify and update continue.
All tasks of Event-1-Table-9 is finished verify and update continue.
All table of event Event-1 is finished verify and update continue.

All tasks of Event-2-Table-1 is finished verify and update continue.
All tasks of Event-2-Table-7 is finished verify and update continue.
All tasks of Event-2-Table-9 is finished verify and update continue.
All tasks of Event-2-Table-8 is finished verify and update continue.
All tasks of Event-2-Table-5 is finished verify and update continue.
All table of event Event-2 is finished verify and update continue.
```

结果分析

1. 每个**table**的所有验证完成，执行`TableTaskBatch`的`done`中的后续操作
2. 每个**event**的所有**table**的验证完成，执行`EventTaskBatch`的`done`中的后续操作

# CyclicBarrier

```tex
A synchronization aid that allows a set of threads to all wait for each other to reach a common barrier point. 
“允许一组线程互相等待到达一个共同的屏障点”的同步算法
```



```java
public class CyclicBarrierExample1 {

    public static void main(String[] args) {
        CyclicBarrier barrier = new CyclicBarrier(4, () -> {
            System.out.println("All parties action finished");
        });
        new Thread(new ActionRunnable(barrier), "T1").start();
        new Thread(new ActionRunnable(barrier), "T2").start();
        new Thread(new ActionRunnable(barrier), "T3").start();
        new Thread(new ActionRunnable(barrier), "T4").start();
    }

    private static class ActionRunnable implements Runnable {
        private final CyclicBarrier barrier;

        public ActionRunnable(CyclicBarrier barrier) {
            this.barrier = barrier;
        }

        @Override
        public void run() {
            try {
                TaskFactory.spend(ThreadLocalRandom.current().nextInt(10), TimeUnit.SECONDS, false, true);
                barrier.await();
                System.out.println(Thread.currentThread().getName() + "- await finished");
            } catch (InterruptedException e) {
                e.printStackTrace();
            } catch (BrokenBarrierException e) {
                e.printStackTrace();
            }
        }
    }
}
```

测试结果：

```
T1 finnish task（1651682474323）
T4 finnish task（1651682475329）
T3 finnish task（1651682480338）
T2 finnish task（1651682482329）
All parties action finished
T2- await finished
T1- await finished
T4- await finished
T3- await finished
```

## 示例2：使用reset()重置

```java
public static void main(String[] args) {
    CyclicBarrier barrier = new CyclicBarrier(2, () -> {
        System.out.println("All parties action finished");
    });
    new Thread(new ActionRunnable(barrier), "T1").start();
    new Thread(new ActionRunnable(barrier), "T2").start();

    TaskFactory.spendSeconds(6);
    System.out.println(barrier.getNumberWaiting());

    barrier.reset();
    TaskFactory.spendSeconds(2);
    System.out.println(barrier.getNumberWaiting());
}
```

## CountDownLatch 和 CyclicBarrier 的区别

| CountDownLatch                     | CyclicBarrier                        |
| ---------------------------------- | ------------------------------------ |
| 不可 reset                         | 可以循环使用的                       |
| CountDownLatch工作线程之间互不关心 | 工作线程互相等待到达一个共同的屏障点 |

# 	Exchanger

1. 需要成对出现，否则单出来的一个线程同样会进入阻塞状态
2. 如果成对的线程，其中一个无法到达“`交换点（Exchange Point）`”，另一个会一直等待，直到超时/一直阻塞。
3. **线程对之间交换的对象，是同一个地址的引用，会存在线程不安全的问题**，可以考虑使用Atomic包装。

```java
一个同步点，在这个同步点上，线程之间可以组队并互相交换数据。每个线程会在进入交换方法时提供给伙伴线程匹配一些对象，并在返回时接收其伙伴的提供的对象。一个交换器可以被看作是一个同步队列的双向形式。交换器在遗传算法和流水线设计等应用中可能是有用的。
```

```java
import java.util.concurrent.Exchanger;
import java.util.concurrent.TimeUnit;

public class ExchangerText {
    public static void main(String[] args) {
        final Exchanger<String> exchanger = new Exchanger<>();

        new Thread(() -> {
            TaskFactory.spend(3, TimeUnit.SECONDS, true);
            try {
                // 交换点，成对的线程同时达到这个交换点才会交换数据
                String msg = exchanger.exchange("（message from " + Thread.currentThread().getName() + ".）");
                System.out.println(Thread.currentThread().getName() + " got " + msg + "[" + System.currentTimeMillis() + "]");
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
        }, "T-A").start();

        new Thread(() -> {
            TaskFactory.spend(10, TimeUnit.SECONDS, true);
            try {
                // 交换点，成对的线程同时达到这个交换点才会交换数据
                String msg = exchanger.exchange("（message from " + Thread.currentThread().getName() + ".）");
                System.out.println(Thread.currentThread().getName() + " got " + msg + "[" + System.currentTimeMillis() + "]");
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
        }, "T-B").start();
    }
}

输出结果：
T-A finnish task（1651715733432）
T-B finnish task（1651715740432）
T-B got （message from T-A.）[1651715740432]
T-A got （message from T-B.）[1651715740432]
```

# Semaphore

注册/回收许可证

1. `acquire()/release()` ： 相当于 acquire(1)/release(1)

2. `acquire(int permits) /release(int permits)`

3. `acquireUninterruptibly()/acquireUninterruptibly(int permits) ` 不可打断，不会抛出InterruptedException异常

4. `drainPermits()` 获取所有的许可证

5. `tryAcquire()/tryAcquire(int permits)`  不可打断，不会抛出InterruptedException异常，拿不到许可证，不会阻塞，放弃获取，继续执行

6. `getQueueLength()` 返回等待获取的线程数的评估值

7. `availablePermits()`返回此信号量中可用的当前许可数（评估值）

- DEMO-1：可中断的许可证请求（会抛出InterruptedException）

```java
public class SemaphoreExample {
    public static void main(String[] args) {
        final Semaphore semaphore = new Semaphore(1);
        new Thread(new TaskRunnable(semaphore), "T1").start();
        new Thread(new TaskRunnable(semaphore), "T2").start();
    }

    static class TaskRunnable implements Runnable {
        private final Semaphore semaphore;
        TaskRunnable(Semaphore semaphore) {
            this.semaphore = semaphore;
        }
        @Override
        public void run() {
            try {
                System.out.println(Thread.currentThread().getName() +  " ask for permits");
                // 请求执行许可证
                semaphore.acquire();
                System.out.println(Thread.currentThread().getName() +  " got permits");
                TaskFactory.spend(10, TimeUnit.SECONDS, true);
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            } finally {
                // 释放许可证
                semaphore.release();
            }
        }
    }
}

```

```tcl
T1 ask for permits
T1 got permits
T2 ask for permits
T1 finnish task [1651772544344]
T2 got permits
T2 finnish task [1651772549361]
```

- DEMO-2

```java
public class SemaphoreExample2 {
    public static void main(String[] args) {
        final Semaphore semaphore = new Semaphore(1);
        new Thread(new TaskRunnable(semaphore), "T1").start();
        new Thread(new TaskRunnable(semaphore), "T2").start();
    }

    static class TaskRunnable implements Runnable {
        private final Semaphore semaphore;
        TaskRunnable(Semaphore semaphore) {
            this.semaphore = semaphore;
        }
        @Override
        public void run() {
            try {
                // 请求执行许可证
                System.out.println(Thread.currentThread().getName() +  " ask for permits");
                boolean tryResult = semaphore.tryAcquire();
                System.out.println(Thread.currentThread().getName() +  (tryResult ? " got permits" : " ignore permits and continue"));
                TaskFactory.spend(2, TimeUnit.SECONDS, true);
            } finally {
                // 释放许可证
                semaphore.release();
            }
        }
    }
}
```

```
T2 ask for permits
T1 ask for permits
T2 got permits
T1 ignore permits and continue
T1 finnish task [1651774206609]
T2 finnish task [1651774206609]
```

# Lock包

> >  java中常见锁分类
>
> - 公平锁和非公平锁
>
>   根据多线程竞争时是否排队依次获取锁，synchronized和ReentrantLock实现默认都是非公平锁，非公平锁可以提高效率，避免线程唤醒带来的空档期
>
> - 可重入锁和不可重复锁
>
>   根据同一个线程是否能重复获取同一把锁
>
> - 共享锁和独占锁(排他锁)
>
>   根据多线程是否能共享一把锁，典型的比如ReentrantReadWriteLock，其中读锁是共享锁，写锁是排他锁
>
> - 可中断锁和不可中断锁
>
>   根据正在尝试获取锁的线程是否可中断
>
> - 悲观锁和乐观锁
>
>   根据线程是否锁住共享资源
>
> - 自旋锁和阻塞锁
>
>   根据线程等待的过程

## ReentrantLock

ReentrantLock特点：作用同Synchronized，但是拥有一些独有的特性

- 可重入：ReentrantLock同步块对同一条线程来说是可重入的，不会出现自己把自己锁死的问题
- 阻塞同步：在成功获取锁的线程执行完之前，会阻塞后面其它线程进入
- <font color='red'>等待可中断</font>：持有锁的线程长期不释放锁时，正在等待获取锁的线程可以选择放弃等待，改为处理其它事情，主要是tryLock(time)、lockInterruptibly()方法响应支持

- <font color="red">实现公平锁</font>：通过new ReentrantLock(true)可以实现多线程在等待同一个锁时，严格按照申请锁的顺序来依次获取锁
- <font color="red">锁可以绑定多个条件</font>：一个ReentrantLock对象锁可以同时绑定多个Condition对象

ReentrantLock核心方法解析

- lock()：尝试获取锁，如果锁已被其它线程获取则等待，lock()方法不能被中断，在死锁情况下会无限等待
- tryLock()：尝试获取锁，如果锁已被其它线程获取则放弃，立即返回boolean类型标识位
- tryLock(long var1, TimeUnit var3)：尝试获取锁，如果锁已被其它线程持有则等待var1时间，超时再放弃
- lockInterruptibly()：相当于把tryLock(long var1, TimeUnit var3)的时间设置成了无限长，但是在等待获取锁的过程中，线程可以被中断
- unlock()：释放锁

ReentrantLock注意事项

- ReentrantLock在异常发生时候不会像synchronized锁一样自动释放锁，所以在使用ReentrantLock时候一定要配合try finally使用来进行释放锁（lock.unlock()）
- <font color="red">tryLock()方法自带插队属性</font>，也就是说即使设置了new ReentrantLock(true)，使用tryLock()方法获取锁仍然是不公平的

```java
public class ReentrantLockTest {

    public static final ReentrantLock lock = new ReentrantLock();

    public static void main(String[] args) {
        IntStream.rangeClosed(1, 2).forEach(i -> new Thread(() -> needLock()).start());
        System.out.println("-----------------------------------");
        TaskFactory.spend(10, TimeUnit.SECONDS);
        System.out.println("-----------------------------------");
        IntStream.rangeClosed(1, 2).forEach(i -> new Thread(() -> tryLock()).start());
    }

    static void needLock() {
        // 不允许打断
        lock.lock();
        try {
            TaskFactory.spend(2, TimeUnit.SECONDS, true);
            System.out.println(Thread.currentThread().getName() + " - 取得锁 ：" + lock.isHeldByCurrentThread());
        } finally {
            lock.unlock();
            System.out.println(Thread.currentThread().getName() + " - 释放锁资源：" + !lock.isLocked());
        }
    }

    static void tryLock() {
        if (lock.tryLock()) {
            // got the lock
            try {
                TaskFactory.spend(5, TimeUnit.SECONDS, true);
                System.out.println(Thread.currentThread().getName() + " - 取得锁 ：" + lock.isHeldByCurrentThread());
            } finally {
                lock.unlock();
                System.out.println(Thread.currentThread().getName() + " - 释放锁资源：" + !lock.isLocked());
            }
        } else {
            // do other things
            System.out.println(Thread.currentThread().getName() + " - 未取得锁.");
        }
    }
}
```

```
-----------------------------------
Thread-0 finnish task [1652021430165]
Thread-0 - 取得锁 ：true
Thread-0 - 释放锁资源：true
Thread-1 finnish task [1652021432181]
Thread-1 - 取得锁 ：true
Thread-1 - 释放锁资源：true
-----------------------------------
Thread-3 - 未取得锁.
Thread-2 finnish task [1652021443166]
Thread-2 - 取得锁 ：true
Thread-2 - 释放锁资源：true

Process finished with exit code 0
```

## ReadWriteLock

需要解决同时读的排他性

```java
public class ReentrantReadWriteLockExample {
    public static final ReentrantReadWriteLock lock = new ReentrantReadWriteLock();
    public static final ReentrantReadWriteLock.ReadLock readLock = lock.readLock();
    public static final ReentrantReadWriteLock.WriteLock writeLock = lock.writeLock();

    public static void main(String[] args) {
        // new Thread(ReadWriteLockExample::doWriteAction, "A1").start();
        // 同时读，不会排他
        new Thread(ReentrantReadWriteLockExample::readFiles, "A1").start();
        new Thread(ReentrantReadWriteLockExample::readFiles, "A2").start();
    }

    static void readFiles() {
        try {
            readLock.lock();
            TaskFactory.spend(5, TimeUnit.SECONDS, true);
            System.out.println(Thread.currentThread().getName() + " 开始读操作");
            TaskFactory.spend(3, TimeUnit.SECONDS);
        } finally {
            readLock.unlock();
            System.out.println(Thread.currentThread().getName() + " 完成读操作");
        }
    }

    static void writeFiles() {
        try {
            writeLock.lock();
            System.out.println(Thread.currentThread().getName() + " 开始写操作");
            TaskFactory.spend(3, TimeUnit.SECONDS);
        } finally {
            writeLock.unlock();
            System.out.println(Thread.currentThread().getName() + " 完成写操作");
        }
    }
}
```

## Condition

作用：monitor对象的wait、notify

使用：condition.await()/ condition.signal()，需要配合lock使用

### 当个等待锁队列

```java
package hots.utils.condition;

import java.util.Optional;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.ReentrantLock;

/**
 * @author: DH
 * @date: 2022/3/5
 * @desc:
 */
public class ConditionExample {
    private final static ReentrantLock sourceLock = new ReentrantLock();
    //  condition 是由lock创建
    private final static Condition condition = sourceLock.newCondition();

    private static int data = 0;

    private static boolean isUsed = false;


    public static void main(String[] args) {
        new Thread(() -> {
            while (true) {
                buildData();
            }
        }).start();

        new Thread(() -> {
            while (true) {
                useData();
            }
        }).start();
    }

    private static void buildData() {
        try {
            sourceLock.lock(); // synchronized 关键词 (monitor enter)
            while (!isUsed) {
                condition.await();　// monitor await
            }

            TimeUnit.SECONDS.sleep(1);	
            data++;
            Optional.of("P：" + data).ifPresent(System.out::println);
            isUsed = false;
            condition.signal(); // monitor notify
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            sourceLock.unlock(); // monitor end
        }
    }


    private static void useData() {
        try {
            sourceLock.lock();
            while (isUsed) {
                condition.await();
            }
            TimeUnit.SECONDS.sleep(1);
            Optional.of("C：" + data).ifPresent(System.out::println);
            isUsed = true;
            condition.signal();
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            sourceLock.unlock();
        }
    }
}
```

###  多个等待锁队列

```
package practice.util.lock.condition;

import practice.common.TaskFactory;

import java.util.LinkedList;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;
import java.util.stream.IntStream;

/**
 * 多线程生产、多线程消费
 *
 * @author: DH
 * @date: 2022/6/12
 * @desc:
 */
public class ConditionExample3 {

    private static Lock lock = new ReentrantLock();

    private static final Condition PRODUCE_COND = lock.newCondition();

    private static final Condition CONSUMER_COND = lock.newCondition();

    private static final LinkedList<Long> TIMESTAMP_POOL = new LinkedList<>();

    private static final int MAX_SIZE = 100;

    public static void main(String[] args) {
        // 包装多名生产者
        IntStream.rangeClosed(1, 5).boxed().forEach(ConditionExample3::doBuildData);
        // 包装多名消费者
        IntStream.rangeClosed(1, 8).boxed().forEach(ConditionExample3::doConsumeData);
    }

    private static void doBuildData(int index) {
        // 生产者不间断生产数据
        new Thread(() -> {
            while (true) {
                buildData();
                TaskFactory.spend(1, TimeUnit.SECONDS);
            }
        }, "P(" + index + ")").start();
    }

    private static void doConsumeData(int index) {
        // 消费者不间断消费数据
        new Thread(() -> {
            while (true) {
                useData();
                TaskFactory.spend(1, TimeUnit.SECONDS);
            }
        }, "C(" + index + ")").start();
    }

    private static void buildData() {
        try {
            lock.lock();
            while (TIMESTAMP_POOL.size() > MAX_SIZE) {
                PRODUCE_COND.await();
            }
            TaskFactory.spend(1, TimeUnit.SECONDS);
            long value = System.currentTimeMillis();
            TIMESTAMP_POOL.addLast(value);
            System.out.println(Thread.currentThread() + "->" + value);
            CONSUMER_COND.signalAll();
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        } finally {
            lock.unlock();
        }
    }

    private static void useData() {
        try {
            lock.lock();
            while (TIMESTAMP_POOL.isEmpty()) {
                CONSUMER_COND.await();
            }
            TaskFactory.spend(1, TimeUnit.SECONDS);
            long value = TIMESTAMP_POOL.removeFirst();
            System.out.println(Thread.currentThread() + "->" + value);
            PRODUCE_COND.signalAll();
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        } finally {
            lock.unlock();
        }
    }
}
```

## StampedLock

### 产生背景

ReentrantReadWriteLock使得多个读线程同时持有读锁（只要写锁未被占用），而写锁是独占的。

但是，读写锁如果使用不当，很容易产生<font color='red'>**“写饥饿”**</font>问题

比如在读线程非常多，写线程很少的情况下，很容易导致写线程“饥饿”，虽然使用“公平”策略可以一定程度上缓解这个问题，但是“公平”策略是以牺牲系统吞吐量为代价的。

### StampedLock的主要特点

1. 所有获取锁的方法，都返回一个邮戳（Stamp），Stamp为0表示获取失败，其余都表示成功；

2. 所有释放锁的方法，都需要一个邮戳（Stamp），这个Stamp必须是和成功获取锁时得到的Stamp一致；

3. StampedLock是不可重入的；（如果一个线程已经持有了写锁，再去获取写锁的话就会造成死锁）

4. StampedLock有三种访问模式：

   ① Reading（读模式）：功能和ReentrantReadWriteLock的读锁类似

   ② Writing（写模式）：功能和ReentrantReadWriteLock的写锁类似

   ③ Optimistic reading（乐观读模式）：这是一种优化的读模式。

   ​	我们知道，在ReentrantReadWriteLock中，当读锁被使用时，如果有线程尝试获取写锁，该写线程会阻塞。

   ​	但是，在Optimistic reading中，即使读线程获取到了读锁，写线程尝试获取写锁也不会阻塞，这相当于对读模式的优化，但是可能会导致数据不一致的问题。

   ​	所以，**当使用Optimistic reading获取到读锁时，必须对获取结果进行校验**。

5. StampedLock支持读锁和写锁的相互转换

   我们知道RRW中，当线程获取到写锁后，可以降级为读锁，但是读锁是不能直接升级为写锁的。
   StampedLock提供了读锁和写锁相互转换的功能，使得该类支持更多的应用场景。

6. 无论写锁还是读锁，都不支持Conditon等待

### 悲观读（读锁和写锁互斥）

```java
package practice.util.lock.stamp;

import practice.common.TaskFactory;

import java.util.LinkedList;
import java.util.Optional;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.StampedLock;
import java.util.stream.Collectors;
import java.util.stream.IntStream;

/**
 * @author: DH
 */
public class StampedLockTest {

    private static final StampedLock STAMPED_LOCK = new StampedLock();

    private static final LinkedList<Long> DATA = new LinkedList<>();

    public static void main(String[] args) {
        final ExecutorService executorService = Executors.newFixedThreadPool(10);

        IntStream.rangeClosed(1, 10).forEach(index -> {
            if (index % 9 == 0) {
                // 写数据
                executorService.submit(() -> {
                    while (true) {
                        write();
                    }
                });
            } else {
                // 读数据
                executorService.submit(() -> {
                    while (true) {
                        read();
                    }
                });
            }
        });
    }

    public static void read() {
        long stamp = -1;
        try {
            // 获取锁，并获取时间戳
            stamp = STAMPED_LOCK.readLock();
            Optional.of(DATA.stream().map(String::valueOf).collect(Collectors.joining("、", "R-", "")))
                    .ifPresent(System.out::println);

            TaskFactory.spend(1, TimeUnit.SECONDS);
        } finally {
            // 按照时间戳释放锁
            STAMPED_LOCK.unlockRead(stamp);
        }
    }

    public static void write() {
        long stamp = -1;
        try {
            stamp = STAMPED_LOCK.writeLock();
            long value = System.currentTimeMillis();
            DATA.addLast(value);
            System.out.println("C:" + value);
        } finally {
            STAMPED_LOCK.unlockWrite(stamp);
        }
    }
}
```



### 乐观读：Optimistic reading

“Optimistic reading”的使用必须遵循以下模式：

```csharp
long stamp = lock.tryOptimisticRead();  // 非阻塞获取版本信息
copyVaraibale2ThreadMemory();           // 拷贝变量到线程本地堆栈
if(!lock.validate(stamp)){              // 校验在拷贝过程中有没有排他锁抢占，如果有则悲观读
    long stamp = lock.readLock();       // 获取读锁
    try {
        copyVaraibale2ThreadMemory();   // 拷贝变量到线程本地堆栈
     } finally {
       lock.unlock(stamp);              // 释放悲观锁
    }
}
useThreadMemoryVarables();              // 使用线程本地堆栈里的数据进行操作
```

以下为乐观读DEMO：

```java
package practice.util.lock.stamp;

import practice.common.TaskFactory;

import java.util.ArrayList;
import java.util.List;
import java.util.Optional;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.StampedLock;
import java.util.stream.Collectors;
import java.util.stream.IntStream;

/**
 * 乐观读
 *
 * @author: DH
 * @date: 2022/6/12
 * @desc:
 */
public class StampedLockOptimisticTest {
    private static final StampedLock stampedLock = new StampedLock();

    private static final List<Long> DATA = new ArrayList<>();

    public static void main(String[] args) {
        final ExecutorService executorService = Executors.newFixedThreadPool(10);
        IntStream.rangeClosed(1, 10).forEach(index -> {
            if (index % 8 == 0) {
                // 写数据
                executorService.submit(() -> {
                    while (true) {
                        write();
                    }
                });
            } else {
                // 读数据
                executorService.submit(() -> {
                    while (true) {
                        optimisticRead();
                    }
                });
            }
        });
    }

    public static void optimisticRead() {
        // 获取锁，并获取时间戳
        long stamp = stampedLock.tryOptimisticRead();
        // 乐观读，必须先拷贝一份数据到在方法中
        List<Long> local = new ArrayList<>();
        local.addAll(DATA);
        // 检查在拷贝过程中有没有排他锁抢占，如果有则悲观读
        if (!stampedLock.validate(stamp)) {
            stamp = stampedLock.readLock();
            try {
                System.out.println(">>>>>>>> 重新读取数据到本地 >>>>>>>>");
                local.clear();
                local.addAll(DATA);
            } finally {
                stampedLock.unlockRead(stamp);
            }
        }

        // 使用数据
        Optional.of(local.stream().map(String::valueOf).collect(Collectors.joining("、", "R-", "")))
                .ifPresent(System.out::println);
        TaskFactory.spend(1, TimeUnit.SECONDS);
    }

    public static void write() {
        long stamp = stampedLock.writeLock();
        try {
            long value = System.currentTimeMillis();
            System.out.println("W-" + value);
            System.out.println();
            DATA.add(value);
        } finally {
            stampedLock.unlockWrite(stamp);
        }
    }
}
```

#  Fork/Join框架



1. Fork/Join任务的原理：判断一个任务是否足够小，如果是，直接计算，否则，就分拆成几个小任务分别计算。这个过程可以反复“裂变”成一系列小任务。

2. 基于工作窃取算法（work-stealing）

3. Fork/Join机制可能只能在单个jvm上运行

## RecursiveTask：有返回

```java
public abstract class RecursiveTask<V> extends ForkJoinTask<V>
```

实验demo : 1~10000 求和

```java
import java.util.concurrent.ExecutionException;
import java.util.concurrent.ForkJoinPool;
import java.util.concurrent.ForkJoinTask;
import java.util.concurrent.RecursiveTask;
import java.util.stream.IntStream;

/**
 * @author: DH
 * @date: 2022/6/20
 * @desc:
 */
public class RecursiveTaskDemo {
    // 可执行容量
    private static final int TASK_CAPACITY = 3;

    public static void main(String[] args) {
        final ForkJoinPool forkJoinPool = new ForkJoinPool();
        ForkJoinTask<Integer> future = forkJoinPool.submit(new CalculateRecursiveTask(1, 100));
        System.out.println("================== other tasks ====================");
        try {
            System.out.println("================== action results " + future.get());
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        } finally {
            forkJoinPool.shutdown();
        }
    }

    // 计算
    private static class CalculateRecursiveTask extends RecursiveTask<Integer> {
        private final int start;
        private final int end;

        public CalculateRecursiveTask(int start, int end) {
            if (end >= start) {
                this.start = start;
                this.end = end;
            } else {
                this.end = start;
                this.start = end;
            }
        }

        @Override
        protected Integer compute() {
            if (end - start <= TASK_CAPACITY) {
                // 执行任务
                return IntStream.rangeClosed(start, end).sum();
            } else {
                // 拆分任务
                int middle = (end + start) / 2;
                CalculateRecursiveTask taskLeft = new CalculateRecursiveTask(start, middle);
                CalculateRecursiveTask taskRight = new CalculateRecursiveTask(middle + 1, end);
                // 加入ForkJoinPool.WorkQueue 
                taskLeft.fork();
                // 加入ForkJoinPool.WorkQueue 
                taskRight.fork();

                // 等待任务执行完成并返回
                return taskLeft.join() + taskRight.join();
            }
        }
    }
}

```

## RecursiveAction：无返回

如果有返回值，需要构造一个共同访问区域。

```java
import java.util.Optional;
import java.util.concurrent.ForkJoinPool;
import java.util.concurrent.RecursiveAction;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.atomic.AtomicInteger;
import java.util.stream.IntStream;

/**
 * @author: DH
 * @date: 2022/6/20
 * @desc:
 */
public class RecursiveActionDemo {
    //  线程共享
    private static final AtomicInteger SUM_RESULT = new AtomicInteger();
    // 可执行容量
    private static final int TASK_CAPACITY = 3;

    public static void main(String[] args) {
        final ForkJoinPool forkJoinPool = new ForkJoinPool();
        try {
            forkJoinPool.submit(new CalculateRecursiveActon(1, 100));
            while (forkJoinPool.getActiveThreadCount() > 0) {
                // 等待执行完成
                System.out.println(forkJoinPool.getActiveThreadCount());
                TaskFactory.spend(1, TimeUnit.NANOSECONDS);
            }
            Optional.of(SUM_RESULT).ifPresent(System.out::println);
        } finally {
            forkJoinPool.shutdown();
        }
    }

    private static class CalculateRecursiveActon extends RecursiveAction {
        private final int start;
        private final int end;

        public CalculateRecursiveActon(int start, int end) {
            if (end >= start) {
                this.start = start;
                this.end = end;
            } else {
                this.end = start;
                this.start = end;
            }

        }

        @Override
        protected void compute() {
            if (end - start <= TASK_CAPACITY) {
                SUM_RESULT.getAndAdd(IntStream.rangeClosed(start, end).sum());
            } else {
                int middle = (end + start) / 2;
                // 任务拆分
                CalculateRecursiveActon left = new CalculateRecursiveActon(start, middle);
                CalculateRecursiveActon right = new CalculateRecursiveActon(middle + 1, end);
                // 任务入池
                left.fork();
                right.fork();
            }
        }
    }
}
```



# Phaser

## 监控方法

- `public int getRegisteredParties()`返回在当前phase上注册的party数目
- `public int getArrivedParties()`返回已经到达当前phase的party的数量，如果这个phaser已经终止，返回值是无意义和任意的
- `public int getUnarrivedParties()`返回还未到达当前phase的party的数量，如果这个phaser已经终止，返回值是无意义和任意的
- `public final int getPhase()`返回当前阶段号, 最大值是Integer.MAX_VALUE，到达最大值之后，从0重新计数

```java
public class PhaserExample4 {
    public static void main(String[] args) {
        final Phaser phaser = new Phaser(5);
        phaser.bulkRegister(5);
        monitor(1, "phaser.getRegisteredParties", phaser.getRegisteredParties());
        monitor(1, "phaser.getArrivedParties", phaser.getArrivedParties());
        new Thread(phaser::arriveAndAwaitAdvance).start();
        monitor(2, "phaser.getArrivedParties", phaser.getArrivedParties());
        monitor(2, "phaser.getUnarrivedParties", phaser.getUnarrivedParties());
    }

    static void monitor(int index, String item, Object object) {
        System.out.printf("【%s】【monitor-%d】【%30s】%s\n", Thread.currentThread().getName(), index, item, object);
    }
}
```

## 动态注册特性

`public int register()` 动态注册

`public int bulkRegister(int parties)`批量注册

`public int arriveAndAwaitAdvance()` 类似CyclicBarrier 的await方法

```java
import practice.common.TaskFactory;

import java.util.concurrent.Phaser;
import java.util.concurrent.TimeUnit;
import java.util.stream.IntStream;

public class PhaserExample1 {

    public static void main(String[] args) {
        final Phaser phaser = new Phaser();
        IntStream.rangeClosed(1, 5).boxed().map(i -> phaser).forEach(Task::new);
        System.out.println("【BEGIN-RegisteredParties】" + phaser.getRegisteredParties());
        // 注册main线程
        phaser.register();
        // 到达并且等待前行
        phaser.arriveAndAwaitAdvance();
        // 等待所有线程全部到达隔离点之后执行
        System.out.println("【END-RegisteredParties】" + phaser.getRegisteredParties());

        System.out.println("【" + Thread.currentThread().getName() + " 】 all threads finished the work.");

        System.out.println("【other work】");
    }

    private static class Task extends Thread {
        private Phaser phaser;

        public Task(Phaser phaser) {
            this.phaser = phaser;
            // 动态追加party
            this.phaser.register();
            this.start();
        }

        @Override
        public void run() {
            TaskFactory.spend(2, TimeUnit.SECONDS);
            System.out.println(Thread.currentThread().getName() + "：finished and continue");
            // 到达并且等待前行
            this.phaser.arriveAndAwaitAdvance();
        }
    }
}

```

输出结果

```java
Thread-2：finished and continue
Thread-3：finished and continue
Thread-4：finished and continue
Thread-0：finished and continue
Thread-1：finished and continue
【END-RegisteredParties】6
【main 】 all threads finished the work.
【other work】
```

## 重复使用计数器

`public final int getPhase()` ：获取已执行阶段数（从0开始，每执行一轮，计数器加1）

```java
package practice.util.phaser;

import practice.common.TaskFactory;

import java.util.Random;
import java.util.concurrent.Phaser;
import java.util.stream.IntStream;

public class PhaserExample2 {
    private static final Random random = new Random(System.currentTimeMillis());

    public static void main(String[] args) {
        // 监控5个运动员（指定parties）
        final Phaser phaser = new Phaser(5);
        IntStream.rangeClosed(1, 5).boxed().map(i -> phaser).forEach(Athlete::new);
    }

    public static class Athlete extends Thread {
        // 运动员编号
        private final Phaser phaser;

        public Athlete(Phaser phaser) {
            this.phaser = phaser;
            this.start();
        }

        @Override
        public void run() {
            // step1: 游泳
            monitor(0, "phaser.getPhase", phaser.getPhase());
            actionDeal("swimming");
            // 等待所有运动员完成游泳任务，继续执行
            phaser.arriveAndAwaitAdvance();
            // step2: 自行车
            monitor(1, "phaser.getPhase", phaser.getPhase());
            actionDeal("bicycle");
            // 等待所有运动员完成自行车任务，继续执行
            phaser.arriveAndAwaitAdvance();
            // step3: 长跑
            monitor(2, "phaser.getPhase", phaser.getPhase());
            actionDeal("running");
            // 等待所有运动员完成长跑任务，继续执行
            phaser.arriveAndAwaitAdvance();

            System.out.println("【" + Thread.currentThread().getName() + "】 finish all tasks.");
        }

        static void monitor(int index, String item, Object object) {
            System.out.printf("\t【monitor】【%s】【%d】【%s】%s\n", Thread.currentThread().getName(), index, item, object);
        }

        static void actionDeal(String taskName) {
            TaskFactory.spendSeconds(random.nextInt(6));
            System.out.println("【" + Thread.currentThread().getName() + "】 finish  " + taskName + ".");
        }
    }
}
```

执行结果

```java
	【monitor】【Thread-0】【0】【phaser.getPhase】0
	【monitor】【Thread-4】【0】【phaser.getPhase】0
	【monitor】【Thread-3】【0】【phaser.getPhase】0
	【monitor】【Thread-1】【0】【phaser.getPhase】0
	【monitor】【Thread-2】【0】【phaser.getPhase】0
【Thread-0】 finish  swimming.
【Thread-2】 finish  swimming.
【Thread-1】 finish  swimming.
【Thread-3】 finish  swimming.
【Thread-4】 finish  swimming.
	【monitor】【Thread-2】【1】【phaser.getPhase】1
	【monitor】【Thread-0】【1】【phaser.getPhase】1
	【monitor】【Thread-3】【1】【phaser.getPhase】1
	【monitor】【Thread-1】【1】【phaser.getPhase】1
	【monitor】【Thread-4】【1】【phaser.getPhase】1
【Thread-0】 finish  bicycle.
【Thread-2】 finish  bicycle.
【Thread-4】 finish  bicycle.
【Thread-3】 finish  bicycle.
【Thread-1】 finish  bicycle.
	【monitor】【Thread-1】【2】【phaser.getPhase】2
	【monitor】【Thread-3】【2】【phaser.getPhase】2
	【monitor】【Thread-2】【2】【phaser.getPhase】2
	【monitor】【Thread-4】【2】【phaser.getPhase】2
	【monitor】【Thread-0】【2】【phaser.getPhase】2
【Thread-0】 finish  running.
【Thread-2】 finish  running.
【Thread-3】 finish  running.
【Thread-1】 finish  running.
【Thread-4】 finish  running.
【Thread-4】 finish all tasks.
【Thread-2】 finish all tasks.
【Thread-3】 finish all tasks.
【Thread-1】 finish all tasks.
【Thread-0】 finish all tasks.
```

## 减少计数器（动态销户）

需要注意：销户之后的<font color='red'>return</font>，否则仍然会参与后续流程的计数

```java
import practice.common.TaskFactory;

import java.util.Random;
import java.util.concurrent.Phaser;
import java.util.stream.IntStream;

/**
 * Phaser 减少计数器（动态销户）
 */
public class PhaserExample3 {
    private static final Random random = new Random(System.currentTimeMillis());

    public static void main(String[] args) {
        // 监控5个运动员
        final Phaser phaser = new Phaser(5);
        IntStream.rangeClosed(1, 5).boxed().map(i -> phaser).forEach(Athlete::new);
    }

    public static class Athlete extends Thread {
        // 运动员编号
        private final Phaser phaser;

        public Athlete(Phaser phaser) {
            this.phaser = phaser;
            this.start();
        }

        @Override
        public void run() {
            // step1: 游泳
            monitor(0, phaser);
            actionDeal("swimming");
            phaser.arriveAndAwaitAdvance();
            // step2: 自行车
            if (Thread.currentThread().getName().endsWith("2")) {
                monitor(1, phaser);
                actionFailed("bicycle");
                // 运动员退出比赛（退出计数）
                phaser.arriveAndDeregister();
                // 退出计数之后，后续流程不在参与重新参与计数
                return;
            } else {
                monitor(1, phaser);
                actionDeal("bicycle");
                phaser.arriveAndAwaitAdvance();
            }
            // step3: 长跑
            monitor(2, phaser);
            actionDeal("running");
            phaser.arriveAndAwaitAdvance();

            System.out.println("【" + Thread.currentThread().getName() + "】 finish all tasks.");
        }

        static void monitor(int index, Phaser phaser) {
            String formatter = "\t【monitor】【%s】【%d】【%s】%s\n";
            String threadName = Thread.currentThread().getName();
            System.out.printf(formatter, threadName, index, "RegisteredParties", phaser.getRegisteredParties());
        }

        static void actionDeal(String taskName) {
            TaskFactory.spendSeconds(random.nextInt(6));
            System.out.println("【" + Thread.currentThread().getName() + "】 finish  " + taskName + ".");
        }

        static void actionFailed(String taskName) {
            System.out.println("【" + Thread.currentThread().getName() + "】 failed  " + taskName + ".");
        }
    }
}
```

输出结果

```java
【monitor】【Thread-0】【0】【RegisteredParties】5
	【monitor】【Thread-3】【0】【RegisteredParties】5
	【monitor】【Thread-4】【0】【RegisteredParties】5
	【monitor】【Thread-2】【0】【RegisteredParties】5
	【monitor】【Thread-1】【0】【RegisteredParties】5
【Thread-2】 finish  swimming.
【Thread-3】 finish  swimming.
【Thread-1】 finish  swimming.
【Thread-0】 finish  swimming.
【Thread-4】 finish  swimming.
	【monitor】【Thread-4】【1】【RegisteredParties】5
	【monitor】【Thread-0】【1】【RegisteredParties】5
	【monitor】【Thread-3】【1】【RegisteredParties】5
	【monitor】【Thread-1】【1】【RegisteredParties】5
	【monitor】【Thread-2】【1】【RegisteredParties】5
【Thread-2】 failed  bicycle.
【Thread-2】 withdrew the game.
【Thread-4】 finish  bicycle.
【Thread-3】 finish  bicycle.
【Thread-1】 finish  bicycle.
【Thread-0】 finish  bicycle.
	【monitor】【Thread-0】【2】【RegisteredParties】4
	【monitor】【Thread-4】【2】【RegisteredParties】4
【Thread-4】 finish  running.
	【monitor】【Thread-3】【2】【RegisteredParties】4
	【monitor】【Thread-1】【2】【RegisteredParties】4
【Thread-0】 finish  running.
【Thread-1】 finish  running.
【Thread-3】 finish  running.
【Thread-3】 finish all tasks.
【Thread-0】 finish all tasks.
【Thread-1】 finish all tasks.
【Thread-4】 finish all tasks.
```



## 人为控制Phase的终结：onAdvance

1. 使用方法：覆写`onAdvance`方法

   ```java
   final Phaser phaser = new Phaser(2) {
     @Override
     protected boolean onAdvance(int phase, int registeredParties) {
       // return registeredParties == 0; 原始写法
       return true; 
     }
   };
   ```

2. `onAdvance `的返回结果直接设置为`returen true` ，则`arriveAndAwaitAdvance`不会阻塞等待所有的`parties`

   ```java
   package practice.util.phaser;
   
   import practice.common.TaskFactory;
   
   import java.util.concurrent.Phaser;
   import java.util.stream.IntStream;
   
   /**
    * 人为控制phase的终结：onAdvance
    */
   public class PhaserExample5 {
       public static void main(String[] args) {
           final Phaser phaser = new Phaser(2) {
               @Override
               protected boolean onAdvance(int phase, int registeredParties) {
                   // 无论执行情况，都默认，phase 执行结束。
                   return true;
               }
           };
           IntStream.rangeClosed(1, 2).boxed().map(i -> phaser).forEach(OnAdvanceTask::new);
       }
   
       static class OnAdvanceTask extends Thread {
           private final Phaser phaser;
   
           public OnAdvanceTask(Phaser phaser) {
               this.phaser = phaser;
               this.start();
           }
   
           @Override
           public void run() {
               System.out.println("【" + this.getName() + "】 arrived part one");
               phaser.arriveAndAwaitAdvance();
               System.out.println("【" + this.getName() + "】 passed part one");
               TaskFactory.spendSeconds(1);
               monitor(1, phaser);
               if (this.getName().endsWith("1")) {
                   System.out.println("【" + this.getName() + "】 arrived part two");
                   // onAdvance 设置为true，arriveAndAwaitAdvance不会阻塞
                   // onAdvance 设置false/ 使用默认的onAdvance，Thread-1 会阻塞在此处
                   phaser.arriveAndAwaitAdvance();
                   System.out.println("【" + this.getName() + "】 passed part two");
               }
               monitor(2, phaser);
               // onAdvance 设置false/ 使用默认的onAdvance，Thread-0 会阻塞在此处
               phaser.arriveAndAwaitAdvance();
               monitor(3, phaser);
           }
       }
   
       static void monitor(int index, Phaser phaser) {
           String template = "\t【%s】【monitor-%d】【%30s】%s\n";
           String actionName = Thread.currentThread().getName();
           System.out.printf(template, actionName, index, "phaser.getPhase", phaser.getPhase());
           System.out.printf(template, actionName, index, "phaser.getRegisteredParties", phaser.getRegisteredParties());
           System.out.printf(template, actionName, index, "phaser.getArrivedParties", phaser.getArrivedParties());
           System.out.printf(template, actionName, index, "phaser.getUnarrivedParties", phaser.getUnarrivedParties());
           System.out.printf(template, actionName, index, "phaser.isTerminated", phaser.isTerminated());
       }
   }
   ```

   输出结果

   ```java
   【Thread-0】 arrived part one
   【Thread-1】 arrived part one
   【Thread-1】 passed part one
   【Thread-0】 passed part one
   	【Thread-0】【monitor-1】【               phaser.getPhase】-2147483647
   	【Thread-1】【monitor-1】【               phaser.getPhase】-2147483647
   	【Thread-0】【monitor-1】【   phaser.getRegisteredParties】2
   	【Thread-1】【monitor-1】【   phaser.getRegisteredParties】2
   	【Thread-1】【monitor-1】【      phaser.getArrivedParties】2
   	【Thread-0】【monitor-1】【      phaser.getArrivedParties】2
   	【Thread-0】【monitor-1】【    phaser.getUnarrivedParties】0
   	【Thread-1】【monitor-1】【    phaser.getUnarrivedParties】0
   	【Thread-1】【monitor-1】【           phaser.isTerminated】true
   	【Thread-0】【monitor-1】【           phaser.isTerminated】true
   【Thread-1】 arrived part two
   	【Thread-0】【monitor-2】【               phaser.getPhase】-2147483647
   	【Thread-0】【monitor-2】【   phaser.getRegisteredParties】2
   	【Thread-0】【monitor-2】【      phaser.getArrivedParties】2
   	【Thread-0】【monitor-2】【    phaser.getUnarrivedParties】0
   	【Thread-0】【monitor-2】【           phaser.isTerminated】true
   	【Thread-0】【monitor-3】【               phaser.getPhase】-2147483647
   	【Thread-0】【monitor-3】【   phaser.getRegisteredParties】2
   	【Thread-0】【monitor-3】【      phaser.getArrivedParties】2
   	【Thread-0】【monitor-3】【    phaser.getUnarrivedParties】0
   	【Thread-0】【monitor-3】【           phaser.isTerminated】true
   【Thread-1】 passed part two
   	【Thread-1】【monitor-2】【               phaser.getPhase】-2147483647
   	【Thread-1】【monitor-2】【   phaser.getRegisteredParties】2
   	【Thread-1】【monitor-2】【      phaser.getArrivedParties】2
   	【Thread-1】【monitor-2】【    phaser.getUnarrivedParties】0
   	【Thread-1】【monitor-2】【           phaser.isTerminated】true
   	【Thread-1】【monitor-3】【               phaser.getPhase】-2147483647
   	【Thread-1】【monitor-3】【   phaser.getRegisteredParties】2
   	【Thread-1】【monitor-3】【      phaser.getArrivedParties】2
   	【Thread-1】【monitor-3】【    phaser.getUnarrivedParties】0
   	【Thread-1】【monitor-3】【           phaser.isTerminated】true
   ```

## 到达之后，不阻塞等待：arrive

`public int arrive()`

使用场景：仅==监控线程==关心任务完成，执行线程无需相互等待

![arrive.drawio](D:\0_Notes\Hexo\hmxyl\source\_images\arrive.drawio-1667231117133-89.png) 

```java
import practice.common.TaskFactory;

import java.util.concurrent.Phaser;
import java.util.stream.IntStream;

/**
 * @author: DH
 * @date: 2022/6/28
 * @desc:
 */
public class PhaserExample6 {
    public static void main(String[] args) {
        final Phaser phaser = new Phaser(3);
        IntStream.rangeClosed(1, 2).boxed().map(i -> phaser).forEach(ArriveTask::new);
        // 此处main线程会阻塞，等待part one全部完成
        phaser.arriveAndAwaitAdvance();
        System.out.println("【" + Thread.currentThread().getName() + "】 part one all done");
    }

    static class ArriveTask extends Thread {
        private final Phaser phaser;

        public ArriveTask(Phaser phaser) {
            this.phaser = phaser;
            this.start();
        }

        @Override
        public void run() {
            TaskFactory.spendSecondsRandom(10, true, true);
            monitor(1, phaser);
            // parties 参与计数，但是不会阻塞等待
            phaser.arrive();
            System.out.println("【" + Thread.currentThread().getName() + "】 part one all done");
            TaskFactory.spendSecondsRandom(2, true, true);
        }
    }

    static void monitor(int index, Phaser phaser) {
        String template = "\t【%s】【monitor-%d】【%30s】%s\n";
        String actionName = Thread.currentThread().getName();
        System.out.printf(template, actionName, index, "phaser.getPhase", phaser.getPhase());
        System.out.printf(template, actionName, index, "phaser.getRegisteredParties", phaser.getRegisteredParties());
        System.out.printf(template, actionName, index, "phaser.getArrivedParties", phaser.getArrivedParties());
        System.out.printf(template, actionName, index, "phaser.getUnarrivedParties", phaser.getUnarrivedParties());
        System.out.printf(template, actionName, index, "phaser.isTerminated", phaser.isTerminated());
    }
}
```

输出结果

```java
	【Thread-1】【monitor-1】【               phaser.getPhase】0
	【Thread-1】【monitor-1】【   phaser.getRegisteredParties】3
	【Thread-1】【monitor-1】【      phaser.getArrivedParties】1
	【Thread-1】【monitor-1】【    phaser.getUnarrivedParties】2
	【Thread-1】【monitor-1】【           phaser.isTerminated】false
【Thread-1】 part one all done
	【Thread-0】【monitor-1】【               phaser.getPhase】0
	【Thread-0】【monitor-1】【   phaser.getRegisteredParties】3
	【Thread-0】【monitor-1】【      phaser.getArrivedParties】2
	【Thread-0】【monitor-1】【    phaser.getUnarrivedParties】1
	【Thread-0】【monitor-1】【           phaser.isTerminated】false
【Thread-0】 part one all done
【main】 part one all done

```

##  仅完成监控任务：awaitAdvance

awaitAdvance方法 不占用 `party` 数量，在所有parties全部完成后执行

```java
/**
 * 仅完成监控任务：awaitAdvance
 */
public class PhaserExample7 {
    public static void main(String[] args) {
        // 若将phaser的parties注册为3，程序会加入阻塞状态
        final Phaser phaser = new Phaser(2);
        actionArriveAndAwaitAdvance(phaser).start();
        actionArriveAndAwaitAdvance(phaser).start();

        new Thread(() -> {
            phaser.awaitAdvance(phaser.getPhase());
            // 监听到指定phase的parties全部完成后执行
            System.out.println("all parties finished：" + phaser.getPhase());
        }).start();

        TaskFactory.spendSeconds(12);

        actionArriveAndAwaitAdvance(phaser).start();
        actionArriveAndAwaitAdvance(phaser).start();

        new Thread(() -> {
            phaser.awaitAdvance(phaser.getPhase());
            // 监听到指定phase的parties全部完成后执行
            System.out.println("all parties finished：" + phaser.getPhase());
        }).start();

        TaskFactory.spendSeconds(12);
        System.out.println("\t【" + Thread.currentThread().getName() + "】" + " done");
    }

    private static Thread actionArriveAndAwaitAdvance(Phaser phaser) {
        return new Thread(() -> {
            TaskFactory.spendSecondsRandom(5);
            phaser.arriveAndAwaitAdvance();
            TaskFactory.spendSeconds(1);
            System.out.println("\t【" + Thread.currentThread().getName() + "】" + " done");
        });
    }
}
```

输出结果

```java
all parties finished：1
	【Thread-0】 done
	【Thread-1】 done
all parties finished：2
	【Thread-4】 done
	【Thread-3】 done
```

- 配合`arrive`使用

```java
package practice.util.phaser;

import practice.common.TaskFactory;

import java.util.concurrent.Phaser;
import java.util.stream.IntStream;

/**
 * 测试 利用 {@link java.util.concurrent.Phaser#awaitAdvance} 监控所有party完成指定任务，才允许后续操作
 */
public class PhaserExample8 {
    public static void main(String[] args) throws InterruptedException {
        final Phaser phaser = new Phaser(3);
        IntStream.rangeClosed(1, 2).boxed().map(i -> phaser).forEach(AwaitAdvanceTask::new);
        phaser.awaitAdvance(phaser.getPhase());
        System.out.println("【" + Thread.currentThread().getName() + "】 all part one finished.");
    }

    static class AwaitAdvanceTask extends Thread {
        private final Phaser phaser;

        public AwaitAdvanceTask(Phaser phaser) {
            this.phaser = phaser;
            this.start();
        }

        @Override
        public void run() {
            // 需要监控完成的工作
            actionDeal("part one", 2);
            phaser.arrive();
            // 非阻塞等待，完成其他工作
            actionDeal("part two", 4);
        }
    }

    static void actionDeal(String actionName, int seconds) {
        System.out.println("【" + Thread.currentThread().getName() + "】 start " + actionName + ".");
        TaskFactory.spendSeconds(seconds);
        System.out.println("\t【" + Thread.currentThread().getName() + "】 finish  " + actionName + ".");
    }
}
```

输出结果

```java
【Thread-1】 start part one.
【Thread-0】 start part one.
	【Thread-1】 finish  part one.
【Thread-1】 start part two.
	【Thread-0】 finish  part one.
【Thread-0】 start part two.
【main】 all part one finished.
	【Thread-0】 finish  part two.
	【Thread-1】 finish  part two.
```

## 打断/超时 终止：awaitAdvanceInterruptibly

等待此Phaser的阶段从给定的phase值前进，如果在等待期间被中断，则抛出 InterruptedException，或者如果当前phase不等于给定的phase值或此Phaser终止，则立即返回。



`Phaser.awaitAdvanceInterruptibly(int)` ，调用interrupt，抛出InterruptedException

`Phaser.awaitAdvanceInterruptibly(int, long, TimeUnit)`: 调用interrupt/给定超时时间，抛出InterruptedException

- 不占用 `party` 数量，在所有parties全部完成后执行

- 打断了其中一个`party`，其他的 party 仍然能够继续执行

  ```java
  import practice.common.TaskFactory;
  
  import java.util.concurrent.Phaser;
  import java.util.stream.IntStream;
  
  public class PhaserExample9 {
      final static int finishTime = 2;
      // phase未结束，可以被打断，其他的 party 仍然能够继续执行
      final static int waitTimeBeforeInterrupt = 1;
      // phase已结束，不会抛出打断异常
      //final static int waitTimeBeforeInterrupt = 4;
  
      public static void main(String[] args) {
          final Phaser phaser = new Phaser(2);
          IntStream.rangeClosed(1, 2).forEach(i -> {
              new Thread(() -> {
                  TaskFactory.spendSeconds(finishTime);
                  phaser.arriveAndAwaitAdvance();
                  System.out.println(Thread.currentThread().getName() + ": continue.");
              }).start();
          });
  
          Thread thread = new Thread(() -> {
              try {
                  // 允许被打断的await
                  phaser.awaitAdvanceInterruptibly(phaser.getPhase());
                  System.out.println(Thread.currentThread().getName() + ": continue.");
              } catch (InterruptedException e) {
                  e.printStackTrace();
                  System.out.println(Thread.currentThread().getName() + ": 未完成party数：" + phaser.getUnarrivedParties());
              }
          });
          thread.start();
          TaskFactory.spendSeconds(waitTimeBeforeInterrupt);
          thread.interrupt();
          System.out.println("=================================");
      }
  }
  ```

  输出结果

  ```java
  =================================
  Thread-2: 未完成party数：2
  java.lang.InterruptedException
  	at java.util.concurrent.Phaser.awaitAdvanceInterruptibly(Phaser.java:760)
  	at practice.util.phaser.PhaserExample9.lambda$main$2(PhaserExample9.java:28)
  	at java.lang.Thread.run(Thread.java:748)
  Thread-1: continue.
  Thread-0: continue.
  ```

## 强制销毁：forceTermination

强制此Phaser进入终止状态。注册方的数量不受影响。如果此Phaser是分层Phaser集的成员，则该集中的所有Phaser都将终止。如果此Phaser已终止，则此方法无效。

此方法可用于在一个或多个任务遇到意外异常后协调恢复。

```java
import practice.common.TaskFactory;

import java.util.concurrent.Phaser;
import java.util.stream.IntStream;

public class PhaserExample10 {
    public static void main(String[] args) {
        final Phaser phaser = new Phaser(2);
        IntStream.rangeClosed(1, 1).forEach(i -> {
            new Thread(() -> {
                TaskFactory.spendSeconds(5);
                phaser.arriveAndAwaitAdvance();
                monitor(1, phaser);
                System.out.println(Thread.currentThread().getName() + ": continue.");
            }).start();
        });
        phaser.forceTermination();
        monitor(1, phaser);
        TaskFactory.spendSeconds(6);
        System.out.println(">>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>");
        IntStream.rangeClosed(1, 2).forEach(i -> {
            new Thread(() -> {
                TaskFactory.spendSeconds(5);
                phaser.arriveAndAwaitAdvance();
                monitor(1, phaser);
                System.out.println(Thread.currentThread().getName() + ": continue.");
            }).start();
        });
        monitor(1, phaser);
    }

    static void monitor(int index, Phaser phaser) {
        String template = "\t【%s】【monitor-%d】【%30s】%s\n";
        String actionName = Thread.currentThread().getName();
        System.out.printf(template, actionName, index, "phaser.getPhase", phaser.getPhase());
        System.out.printf(template, actionName, index, "phaser.getRegisteredParties", phaser.getRegisteredParties());
        System.out.printf(template, actionName, index, "phaser.getArrivedParties", phaser.getArrivedParties());
        System.out.printf(template, actionName, index, "phaser.getUnarrivedParties", phaser.getUnarrivedParties());
        System.out.printf(template, actionName, index, "phaser.isTerminated", phaser.isTerminated());
    }
}
```

输出结果

```java
	【main】【monitor-1】【               phaser.getPhase】-2147483648
	【main】【monitor-1】【   phaser.getRegisteredParties】2
	【main】【monitor-1】【      phaser.getArrivedParties】0
	【main】【monitor-1】【    phaser.getUnarrivedParties】2
	【main】【monitor-1】【           phaser.isTerminated】true
	【Thread-0】【monitor-1】【               phaser.getPhase】-2147483648
	【Thread-0】【monitor-1】【   phaser.getRegisteredParties】2
	【Thread-0】【monitor-1】【      phaser.getArrivedParties】0
	【Thread-0】【monitor-1】【    phaser.getUnarrivedParties】2
	【Thread-0】【monitor-1】【           phaser.isTerminated】true
Thread-0: continue.
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
	【main】【monitor-1】【               phaser.getPhase】-2147483648
	【main】【monitor-1】【   phaser.getRegisteredParties】2
	【main】【monitor-1】【      phaser.getArrivedParties】0
	【main】【monitor-1】【    phaser.getUnarrivedParties】2
	【main】【monitor-1】【           phaser.isTerminated】true
	【Thread-2】【monitor-1】【               phaser.getPhase】-2147483648
	【Thread-2】【monitor-1】【   phaser.getRegisteredParties】2
	【Thread-2】【monitor-1】【      phaser.getArrivedParties】0
	【Thread-2】【monitor-1】【    phaser.getUnarrivedParties】2
	【Thread-2】【monitor-1】【           phaser.isTerminated】true
Thread-2: continue.
	【Thread-1】【monitor-1】【               phaser.getPhase】-2147483648
	【Thread-1】【monitor-1】【   phaser.getRegisteredParties】2
	【Thread-1】【monitor-1】【      phaser.getArrivedParties】0
	【Thread-1】【monitor-1】【    phaser.getUnarrivedParties】2
	【Thread-1】【monitor-1】【           phaser.isTerminated】true
Thread-1: continue.

Process finished with exit code 0
```



[Toc]

# Executor框架

## ExecutorService接口

### ExecutorService 继承树

[Executor框架.drawio](D:\0_Notes\Hexo\hmxyl\source\_images\Executor框架.drawio)

![Executor框架](D:\0_Notes\Hexo\hmxyl\source\_images\Executor框架-16588299109821-1667231117133-88.png)



### ExecutorService的创建

创建一个什么样的ExecutorService的实例（即线程池）需要g根据具体应用场景而定，不过Java给我们提供了一个Executors工厂类，它可以帮助我们很方便的创建各种类型ExecutorService线程池，Executors一共可以创建下面这四类线程池

- ThreadPoolExecutor 核心构造函数

```java
  import java.util.concurrent.*;
import java.util.stream.IntStream;

/**
 * 测试ThreadPoolExecutor
 */
public class ThreadPoolExecutorBuild {
    public static void main(String[] args) {
        ThreadPoolExecutor threadPoolExecutor = (ThreadPoolExecutor) buildThreadPoolExecutor();

        IntStream.rangeClosed(1, 50).forEach(index -> threadPoolExecutor.submit(() -> {
            doAction(3);
            monitorThreadPool(threadPoolExecutor, index);
            System.out.println();
            System.out.println();
        }));

        threadPoolExecutor.shutdown();
    }

    /** 
    * ThreadPoolExecutor 核心构造函数
    */
    private static ExecutorService buildThreadPoolExecutor() {
        int corePoolSize = 2;
        int maximumPoolSize = 10;
        // 当线程数大于核心时，这是多余的空闲线程在终止前等待新任务的最长时间
        long keepAliveTime = 1;
        TimeUnit timeUnit = TimeUnit.SECONDS;
        // 用于在任务完成之前保存任务的队列
        BlockingQueue<Runnable> blockingQueue = new ArrayBlockingQueue<>(30);
        // 线程创建工厂
        ThreadFactory threadFactory = r -> new Thread(r);
        // 拒绝策略
        RejectedExecutionHandler rejectedExecutionHandler = new ThreadPoolExecutor.AbortPolicy();
        return new ThreadPoolExecutor(corePoolSize, maximumPoolSize, keepAliveTime, timeUnit, blockingQueue,
                threadFactory, rejectedExecutionHandler);
    }

    private static void monitorThreadPool(ThreadPoolExecutor threadPoolExecutor, int index) {
        System.out.println(index + "【getPoolSize】:" + threadPoolExecutor.getPoolSize());
        System.out.println(index + "【getActiveCount】:" + threadPoolExecutor.getActiveCount());
        System.out.println(index + "【getMaximumPoolSize】:" + threadPoolExecutor.getMaximumPoolSize());
        System.out.println(index + "【getCompletedTaskCount】:" + threadPoolExecutor.getCompletedTaskCount());
        System.out.println(index + "【getCorePoolSize】:" + threadPoolExecutor.getCorePoolSize());
        System.out.println(index + "【getLargestPoolSize】:" + threadPoolExecutor.getLargestPoolSize());
    }

    private static void doAction(int seconds) {
        try {
            TimeUnit.SECONDS.sleep(seconds);
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
    }
}
```

Executors只是一个工厂类，它所有的方法返回的都是ThreadPoolExecutor、ScheduledThreadPoolExecutor这两个类的实例

### ExecutorService的执行

ExecutorService有如下几个执行方法：

```java
- execute(Runnable)
- submit(Runnable)
- submit(Callable)
- invokeAny(...)
- invokeAll(...)
```

- execute(Runnable)

这个方法接收一个Runnable实例，并且异步的执行，请看下面的实例：

```java
ExecutorService executorService = Executors.newSingleThreadExecutor();

executorService.execute(new Runnable() {
  public void run() {
     System.out.println("Asynchronous task");
  }
});

executorService.shutdown();
```

这个方法有个问题，就是没有办法获知task的执行结果。

- submit(Runnable)

submit(Runnable)和execute(Runnable) 

区别是前者可以返回一个Future对象，通过返回的Future对象，我们可以检查提交的任务是否执行完毕 

```java
Future future = executorService.submit(new Runnable() {
  public void run() {
      System.out.println("Asynchronous task");
  }
});
future.get();  //returns null if the task has finished correctly.
```

如果任务执行完成，future.get()方法会返回一个null。注意，future.get()方法会产生阻塞。

- submit(Callable)

submit(Callable) 和submit(Runnable)类似，也会返回一个Future对象，但是除此之外，submit(Callable)接收的是一个Callable的实现，Callable接口中的call()方法有一个返回值，可以返回任务的执行结果，而Runnable接口中的run()方法是void的，没有返回值。


```java
Future future = executorService.submit(new Callable(){
	public Object call() throws Exception {
    	System.out.println("Asynchronous Callable");
    	return "Callable Result";
	}
});

System.out.println("future.get() = " + future.get());
```

future.get()方法会返回Callable任务的执行结果。注意，future.get()方法会产生阻塞。

- invokeAny(…)

invokeAny(...)方法接收的是一个Callable的集合，执行这个方法不会返回Future，但是会返回所有Callable任务中其中一个任务的执行结果。这个方法也无法保证返回的是哪个任务的执行结果，反正是其中的某一个

```java
ExecutorService executorService = Executors.newSingleThreadExecutor();

Set<Callable<String>> callables = new HashSet<Callable<String>>();

callables.add(new Callable<String>() {
	public String call() throws Exception {
 	   return "Task 1";
	}
});
callables.add(new Callable<String>() {
	public String call() throws Exception {
	    return "Task 2";
	}
});
callables.add(new Callable<String>() {
    public String call() throws Exception {
    return "Task 3";
	}
});

String result = executorService.invokeAny(callables);
System.out.println("result = " + result);
executorService.shutdown();
```

每次执行都会返回一个结果，并且返回的结果是变化的，可能会返回“Task2”也可是“Task1”或者其它。

- nvokeAll(…)

invokeAll(...)与 invokeAny(...)类似也是接收一个Callable集合，但是前者执行之后会返回一个Future的List，其中对应着每个Callable任务执行后的Future对象

```java
ExecutorService executorService = Executors.newSingleThreadExecutor();

List<Callable<String>> callables = new ArrayList<Callable<String>>();

callables.add(new Callable<String>() {
public String call() throws Exception {
    return "Task 1";
}
});
callables.add(new Callable<String>() {
    public String call() throws Exception {
    return "Task 2";
}
});
callables.add(new Callable<String>() {
public String call() throws Exception {
    return "Task 3";
}
});

List<Future<String>> futures = executorService.invokeAll(callables);

for(Future<String> future : futures){
System.out.println("future.get = " + future.get());
}

executorService.shutdown();
```

`List<Callable<String>> callables` 返回的结果集是无序的。



### ExecutorService的关闭

当我们使用完成ExecutorService之后应该关闭它，否则它里面的线程会一直处于运行状态。

- void shutdown()

- List<Runnable> shutdownNow()

![image-20220725112237287](D:\0_Notes\Hexo\hmxyl\source\_images\image-20220725112237287-1667231117133-90.png) 

## Executors工具

[Executors 工具.xlsx ](D:\0_Notes\Hexo\hmxyl\source\_images\Executors.xlsx)

**利用：ThreadPoolExecutor(int corePoolSize, int maximumPoolSize,  long keepAliveTime, TimeUnit unit, BlockingQueue<Runnable> workQueue,  ThreadFactory threadFactory, RejectedExecutionHandler handler)**

| Executors模板方法                                            | 特性                                                         | corePoolSize | maximumPoolSize   | keepAliveTime | unit                  | workQueue                                                    | threadFactory                    | handler                              |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ------------ | ----------------- | ------------- | --------------------- | ------------------------------------------------------------ | -------------------------------- | ------------------------------------ |
| newFixedThreadPool(int  nThreads)                            | 线程池中的线程不会被销毁                                     | nThreads     | nThreads          | 0L            | TimeUnit.MILLISECONDS | <font color='red'> new LinkedBlockingQueue<Runnable>(Integer.MAX_VALUE)</font> | Executors.defaultThreadFactory() | defaultHandler（new  AbortPolicy()） |
| newFixedThreadPool(int  nThreads, ThreadFactory threadFactory) | 线程池中的线程不会被销毁                                     | nThreads     | nThreads          | 0L            | TimeUnit.MILLISECONDS | new LinkedBlockingQueue<Runnable>(Integer.MAX_VALUE)         | Executors.defaultThreadFactory() | defaultHandler（new  AbortPolicy()） |
| newSingleThreadExecutor                                      | 可以保留单线程需要执行的任务队列,并且将ThreadPoolExecutor中的方法屏蔽 | 1            | 1                 | 0L            | TimeUnit.MILLISECONDS | new LinkedBlockingQueue<Runnable>(Integer.MAX_VALUE)         | Executors.defaultThreadFactory() | defaultHandler（new  AbortPolicy()） |
| newSingleThreadExecutor(ThreadFactory  threadFactory)        | 可以保留单线程需要执行的任务队列,并且将ThreadPoolExecutor中的方法屏蔽 | 1            | 1                 | 0L            | TimeUnit.MILLISECONDS | new LinkedBlockingQueue<Runnable>(Integer.MAX_VALUE)         | threadFactory                    | defaultHandler（new  AbortPolicy()） |
| newCachedThreadPool()                                        | 每提交一个任务,创建一个线程                                  | 0            | Integer.MAX_VALUE | 60L           | TimeUnit.SECONDS      | <font color='red'>new SynchronousQueue<Runnable>()</font>    | Executors.defaultThreadFactory() | defaultHandler（new  AbortPolicy()） |
| newCachedThreadPool(ThreadFactory  threadFactory)            | 每提交一个任务,创建一个线程                                  | 0            | Integer.MAX_VALUE | 60L           | TimeUnit.SECONDS      | new SynchronousQueue<Runnable>()                             | threadFactory                    | defaultHandler（new  AbortPolicy()） |
| newSingleThreadScheduledExecutor()                           |                                                              | 0            | Integer.MAX_VALUE | 60L           | TimeUnit.SECONDS      | new DelayedWorkQueue()                                       | Executors.defaultThreadFactory() | defaultHandler（new  AbortPolicy()） |
| newSingleThreadScheduledExecutor(ThreadFactory  threadFactory) |                                                              | 0            | Integer.MAX_VALUE | 60L           | TimeUnit.SECONDS      | <font color='red'>new DelayedWorkQueue()</font>              | threadFactory                    | defaultHandler（new  AbortPolicy()） |
| newScheduledThreadPool(int  corePoolSize)                    |                                                              | corePoolSize | Integer.MAX_VALUE | 0             | TimeUnit.NANOSECONDS  | new DelayedWorkQueue()                                       | Executors.defaultThreadFactory() | defaultHandler（new  AbortPolicy()） |
| newScheduledThreadPool(int  corePoolSize, ThreadFactory threadFactory) |                                                              | corePoolSize | Integer.MAX_VALUE | 0             | TimeUnit.NANOSECONDS  | new DelayedWorkQueue()                                       | threadFactory                    | defaultHandler（new  AbortPolicy()） |





**利用ForkJoinPool(int parallelism,  ForkJoinWorkerThreadFactory factory,  UncaughtExceptionHandler handler,  boolean asyncMode)**

| Executors模板方法                     | int parallelism                            | ForkJoinWorkerThreadFactory  factory            | UncaughtExceptionHandler handler | boolean asyncMode |
| ------------------------------------- | ------------------------------------------ | ----------------------------------------------- | -------------------------------- | ----------------- |
| newWorkStealingPool()                 | Runtime.getRuntime().availableProcessors() | ForkJoinPool.defaultForkJoinWorkerThreadFactory | null                             | TRUE              |
| newWorkStealingPool(int  parallelism) | parallelism                                | ForkJoinPool.defaultForkJoinWorkerThreadFactory | null                             | TRUE              |

```java
package hots.utils.executor;

import java.util.List;
import java.util.concurrent.Callable;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.TimeUnit;
import java.util.stream.Collectors;
import java.util.stream.IntStream;

public class ExecutorsExample {
    public static void main(String[] args) throws InterruptedException {
        ExecutorService executorService = Executors.newWorkStealingPool();
        List<Future<String>> futures = executorService.invokeAll(getTasks());
        executorService.shutdown();
    }

    static List<Callable<String>> getTasks() {
        return IntStream.rangeClosed(0, 20).boxed().map(i -> {
            Callable<String> stringCallable = () -> {
                System.out.println(Thread.currentThread().getName());
                doAction(1);
                return "Task:" + i;
            };
            return stringCallable;
        }).collect(Collectors.toList());
    }

    private static void doAction(int seconds) {
        try {
            TimeUnit.SECONDS.sleep(seconds);
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
    }
}
```



## ThreadPoolExecutor

### 四个内置拒绝策略

- 继承自RejectedExecutionHandler

![image-20220725171152798](D:\0_Notes\Hexo\hmxyl\source\_images\image-20220725171152798-1667231117133-91.png) 

| 策略                | 说明                                                         |
| ------------------- | ------------------------------------------------------------ |
| AbortPolicy         | 抛出RejectedExecutionException异常                           |
| DiscardPolicy       | 默默地丢弃被拒绝的任务                                       |
| DiscardOldestPolicy | 丢弃最早的未处理请求，然后重试执行请求任务。若任务已被关闭，则丢弃任务 |
| CallerRunsPolicy    | 直接在执行方法的调用线程中运行被拒绝的任务。若任务已被关闭，则丢弃任务 |



### 自定义ThreadFactory

```java
import lombok.Data;

import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.ThreadFactory;
import java.util.concurrent.atomic.AtomicInteger;

@Data
public class MyThreadFactory implements ThreadFactory {
    private AtomicInteger SEQ = new AtomicInteger();
    private List<FailedAction> failed = new ArrayList<>();

    @Override
    public Thread newThread(Runnable r) {
        Thread thread = new Thread(r);
        thread.setName("Factory-" + SEQ.getAndIncrement());
        thread.setUncaughtExceptionHandler((t, e) -> {
            // 保留异常信息
            failed.add(new FailedAction(t, e));
        });
        return thread;
    }

    @Data
    static class FailedAction {
        private Thread thread;
        private Throwable throwable;

        FailedAction(Thread t, Throwable e) {
            this.thread = t;
            this.throwable = e;
        }
    }
}
```

```java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class MyThreadFactoryTest {
    public static void main(String[] args) {
        MyThreadFactory myThreadFactory = new MyThreadFactory();
        ExecutorService executorService = Executors.newFixedThreadPool(1, myThreadFactory);
        executorService.execute(() -> System.out.println(1 / 0));
        myThreadFactory.getFailed().stream().map(e -> e.getThrowable()).forEach(e -> e.printStackTrace());
        executorService.shutdown();
    }
}
```



### 允许回收执行线程：allowCoreThreadTimeOut

```java
private volatile boolean allowCoreThreadTimeOut;
```

```java
public class AllowCoreThreadTimeOutTest {
    public static void main(String[] args) {
        ThreadPoolExecutor executorService = (ThreadPoolExecutor) Executors.newFixedThreadPool(2);
        executorService.setKeepAliveTime(10, TimeUnit.SECONDS);
        executorService.submit(() -> System.out.println(Thread.currentThread().getName()));
        executorService.allowCoreThreadTimeOut(true);
    }
}

# 10秒后线程池被销毁，测试进程退出
```

```java
public class AllowCoreThreadTimeOutTest {
    public static void main(String[] args) {
        ThreadPoolExecutor executorService = (ThreadPoolExecutor) Executors.newFixedThreadPool(2);
        executorService.setKeepAliveTime(10, TimeUnit.SECONDS);
        executorService.submit(() -> System.out.println(Thread.currentThread().getName()));
        executorService.allowCoreThreadTimeOut(false);
    }
}

# 测试进程无法退出，executorService保有2个活跃的执行线程
```

### 删除任务：remove

适用于`executorService.execute(e) `提交的任务，而`executorService.submit(e)` 提交的任务，无法移除。submit 提交的任务

```java
public class RemoveTest {
    public static void main(String[] args) {
        ThreadPoolExecutor executorService = new ThreadPoolExecutor(2, 2, 0, TimeUnit.SECONDS,
                new ArrayBlockingQueue<>(2));

        List<Runnable> runnableList = IntStream.rangeClosed(0, 3).boxed().map(i -> (Runnable) () -> {
                    sleep(5);
                    System.out.println("task:" + i + " with " + Thread.currentThread().getName());
                }
        ).collect(Collectors.toList());
        runnableList.stream().forEach(e -> executorService.execute(e));

        sleep(1);
        boolean result = executorService.remove(runnableList.get(2));
        System.out.println("remove result : " + result);

        sleep(6);
        executorService.shutdown();
    }

    private static void sleep(int seconds) {
        try {
            TimeUnit.SECONDS.sleep(seconds);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

### prestartCoreThread ： 启动一个执行线程

### prestartAllCoreThreads：启动所有执行线程

### beforeExecute/afterExecute

自定义ThreadPoolExecutor的子类，覆写。

## Future与FutureTask



### Future 接口

| 接口方法                                                     |
| ------------------------------------------------------------ |
| V get() throws InterruptedException, ExecutionException;     |
| V get(long timeout, TimeUnit unit)     throws InterruptedException, ExecutionException, TimeoutException; |
| boolean isDone();                                            |
| boolean cancel(boolean mayInterruptIfRunning);               |
| boolean isCancelled();                                       |

-  `cancel`：取消任务，如果取消任务成功则返回true，如果取消任务失败则返回false。参数`mayInterruptIfRunning`表示是否允许取消正在执行却没有执行完毕的任务： 
   1. 如果设置true，则表示可以取消正在执行过程中的任务
   2. 如果任务已经完成，则无论mayInterruptIfRunning为true还是false，此方法肯定返回false
   3. 如果任务正在执行，若mayInterruptIfRunning设置为true，则返回true，若mayInterruptIfRunning设置为false，则返回false
   4. 如果任务还没有执行，则无论mayInterruptIfRunning为true还是false，肯定返回true
-  `isCancelled`：方法表示任务是否被取消成功，如果在任务正常完成前被取消成功，则返回 true 
-  `isDone`：判断任务是否已经完成，已完成则返回true； 
-  `get()`：获取执行结果，这个方法会产生阻塞，会一直等到任务执行完毕才返回；
-  ` get(long timeout, TimeUnit unit)`：用来获取执行结果，如果在指定时间内，还没获取到结果，就直接返回null



```java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Future;
import java.util.concurrent.TimeUnit;

/**
 * boolean cancel(boolean mayInterruptIfRunning);
 */
public class FutureExample {
    public static void main(String[] args) throws InterruptedException {
        ExecutorService executorService = Executors.newSingleThreadExecutor();
        Future<String> result = executorService.submit(() -> {
            while (!Thread.currentThread().isInterrupted()) {
            }
            return "done";
        });

        sleep(1);
        System.out.println(result.cancel(true));

        executorService.shutdown();
        System.out.println("all done");
    }

    private static void sleep(int seconds) {
        try {
            TimeUnit.SECONDS.sleep(seconds);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}

```

### FutureTask类

FutureTask类实现了RunnableFuture接口，RunnableFuture接口又继承了Runable和Future，可见，FutureTask既可以作为Runnable被线程执行，又可以作为Future得到Callable的返回值。

FutureTask类图如下

![img](D:\0_Notes\Hexo\hmxyl\source\_images\1620-1667231117133-93.png) 

FutureTask两个构造函数：

```java
FutureTask(Callable<V> callable);
FutureTask(Runnable runnable, V result);
```

使用FutureTask来实现Future多线程获取任务结果的场景

```java
import java.util.Date;
import java.util.concurrent.*;

public class FutureTaskTest {
    public static void main(String[] args) {
        ExecutorService executor = Executors.newCachedThreadPool();
        for (int i = 0; i < 3; i++) {
            Callable<String> callable = new Task(8 - i);
            MyFutureTask task = new MyFutureTask(callable);
            executor.submit(task);
        }
        executor.shutdown();
    }
}

class MyFutureTask extends FutureTask<String> {
    public MyFutureTask(Callable<String> callable) {
        super(callable);
    }

    @Override
    protected void done() {
        try {
            System.out.println(get() + "完成：" + new Date());
        } catch (InterruptedException | ExecutionException e) {
            e.printStackTrace();
        }
    }
}

class Task implements Callable<String> {
    private int time;
    public Task(int time) {
        this.time = time;
    }
    @Override
    public String call() throws InterruptedException {
        String name = Thread.currentThread().getName();
        System.out.println(name + "启动：" + new Date());
        TimeUnit.SECONDS.sleep(time);
        return name;
    }
}
```

输出结果：

```javascript
pool-1-thread-1启动：Fri Nov 08 17:35:26 CST 2019
pool-1-thread-3启动：Fri Nov 08 17:35:26 CST 2019
pool-1-thread-2启动：Fri Nov 08 17:35:26 CST 2019
pool-1-thread-3完成：Fri Nov 08 17:35:32 CST 2019
pool-1-thread-2完成：Fri Nov 08 17:35:33 CST 2019
pool-1-thread-1完成：Fri Nov 08 17:35:34 CST 2019
```

## CompletionService接口

 我们知道，通过 Future 和 FutureTask 可以获得线程任务的执行结果，但它们有一定的缺陷：

- Future：多个线程任务的执行结果，我们可以通过轮询的方式去获取，但普通轮询会有被阻塞的可能，升级轮询会非常消耗cpu
- FutureTask：虽然我们可以调用 done 方法，在线程任务执行结束后立即返回或做其他处理，但对批量线程任务结果的管理方面有所不足

为了更好地应对大量线程任务结果处理的问题，JDK提供了功能强大的 CompletionService。CompletionService是一个接口，使用创建时提供的 Executor 对象（通常是线程池）来执行任务，并在内部维护了一个阻塞队列`QueueingFuture`，当任务执行结束就把任务的执行结果的`Future`对象加入到阻塞队列中。

该接口只有一个实现类： `ExecutorCompletionService`



> ExecutorCompletionService  的构造函数

```java
/**
 * Creates an ExecutorCompletionService using the supplied
 * executor for base task execution and a
 * {@link LinkedBlockingQueue} as a completion queue.
 *
 * @param executor the executor to use
 * @throws NullPointerException if executor is {@code null}
 */
public ExecutorCompletionService(Executor executor) {
	if (executor == null)
		throw new NullPointerException();
	this.executor = executor;
	this.aes = (executor instanceof AbstractExecutorService) ?
		(AbstractExecutorService) executor : null;
	this.completionQueue = new LinkedBlockingQueue<Future<V>>();
}

/**
 * Creates an ExecutorCompletionService using the supplied
 * executor for base task execution and the supplied queue as its
 * completion queue.
 *
 * @param executor the executor to use
 * @param completionQueue the queue to use as the completion queue
 *        normally one dedicated for use by this service. This
 *        queue is treated as unbounded -- failed attempted
 *        {@code Queue.add} operations for completed tasks cause
 *        them not to be retrievable.
 * @throws NullPointerException if executor or completionQueue are {@code null}
 */
public ExecutorCompletionService(Executor executor,
								 BlockingQueue<Future<V>> completionQueue) {
	if (executor == null || completionQueue == null)
		throw new NullPointerException();
	this.executor = executor;
	this.aes = (executor instanceof AbstractExecutorService) ?
		(AbstractExecutorService) executor : null;
	this.completionQueue = completionQueue;
}
```

这两个构造方法都需要传入一个线程池，如果不指定 `completionQueue`，那么默认会使用无界的 `LinkedBlockingQueue`。任务执行结果的 Future 对象就是加入到 completionQueue 中。



> CompletionService 接口方法

```java
public interface CompletionService<V> {
    //提交线程任务
    Future<V> submit(Callable<V> task);
    //提交线程任务
    Future<V> submit(Runnable task, V result);
    //阻塞等待
    Future<V> take() throws InterruptedException;
    //非阻塞等待
    Future<V> poll();
    //带时间的非阻塞等待
    Future<V> poll(long timeout, TimeUnit unit) throws InterruptedException;
}
```

- submit(Callable task)：提交线程任务，交由 Executor 对象去执行，并将结果放入阻塞队列；
- take()：在阻塞队列中获取并移除一个元素，该方法是阻塞的，即获取不到的话线程会一直阻塞；
- poll()：在阻塞队列中获取并移除一个元素，该方法是非阻塞的，获取不到即返回 null ；
- poll(long timeout, TimeUnit unit)：从阻塞队列中非阻塞地获取并移除一个元素，在设置的超时时间内获取不到即返回 null ；

接下来，我们重点看一下submit 的源码：

```java
 public Future<V> submit(Callable<V> task) {
    if (task == null) throw new NullPointerException();
    RunnableFuture<V> f = newTaskFor(task);
    executor.execute(new QueueingFuture(f));
    return f;
 }
```

从submit 方法的源码中可以确认两点：

1. 线程任务确实是由 Executor 对象执行的；
2. 提交某个任务时，该任务首先将被包装为一个QueueingFuture。

继续追查 `QueueingFuture`，可以发现： 该类重写了 FutureTask 的done方法，当计算完成时，把Executor执行的计算结果放入BlockingQueue中，而==放入结果是按任务完成顺序来进行==的，即先完成的任务先放入阻塞队列。

```java
 /** 
  * FutureTask extension to enqueue upon completion 
  */  
private class QueueingFuture extends FutureTask<Void> {  
    QueueingFuture(RunnableFuture<V> task) {  
        super(task, null);  
        this.task = task;  
    }  
    protected void done() { completionQueue.add(task); }  
    private final Future<V> task;  
}  
```

由此，CompletionService 实现了生产者提交任务和消费者获取结果的解耦，任务的完成顺序由 CompletionService 来保证，消费者一定是按照任务完成的先后顺序来获取执行结果。

>  CompletionService 使用示例

```java
import java.util.Date;
import java.util.concurrent.*;

public class CompletionServiceTest {
    public static void main(String[] args) {
        ExecutorService executor = Executors.newCachedThreadPool();
        CompletionService<String> cs = new ExecutorCompletionService<>(executor);
        // 此线程池运行5个线程
        for (int i = 0; i < 5; i++) {
            final int index = i;
            cs.submit(() -> {
                String name = Thread.currentThread().getName();
                System.out.println(name + " 启动：" + new Date());
                TimeUnit.SECONDS.sleep(10 - index * 2);
                return name;
            });
        }
        executor.shutdown();

        for (int i = 0; i < 5; i++) {
            try {
                System.out.println(cs.take().get() + " 结果：" + new Date());
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }
}
```

```java
pool-1-thread-2 启动：Sun Nov 10 11:34:13 CST 2019
pool-1-thread-4 启动：Sun Nov 10 11:34:13 CST 2019
pool-1-thread-3 启动：Sun Nov 10 11:34:13 CST 2019
pool-1-thread-5 启动：Sun Nov 10 11:34:13 CST 2019
pool-1-thread-1 启动：Sun Nov 10 11:34:13 CST 2019
pool-1-thread-5 结果：Sun Nov 10 11:34:15 CST 2019
pool-1-thread-4 结果：Sun Nov 10 11:34:17 CST 2019
pool-1-thread-3 结果：Sun Nov 10 11:34:19 CST 2019
pool-1-thread-2 结果：Sun Nov 10 11:34:21 CST 2019
pool-1-thread-1 结果：Sun Nov 10 11:34:23 CST 2019
```

## ScheduledExecutorService

![image-20220726163319524](D:\0_Notes\Hexo\hmxyl\source\_images\image-20220726163319524-1667231117133-92.png) 



```java
/**
* 1秒后开始执行任务，每2秒执行一回
*/
public class ScheduledExecutorServiceTest {
    public static void main(String[] args) {
        ScheduledExecutorService scheduledExecutorService = Executors.newScheduledThreadPool(2);
        //scheduledExecutorService.schedule(() -> System.out.println(Thread.currentThread().getName()), 1, TimeUnit.SECONDS);
        AtomicLong time = new AtomicLong(-1);
        scheduledExecutorService.scheduleAtFixedRate(() -> {
            sleep(5);
            if (time.get() > 0) {
                System.out.println(Thread.currentThread().getName() + ": " + (System.currentTimeMillis() - time.get()));
            }

            time.set(System.currentTimeMillis());
        }, 1, 2, TimeUnit.SECONDS);
    }

    private static void sleep(int time) {
        try {
            TimeUnit.SECONDS.sleep(time);
        } catch (Exception e) {
        }
    }
}
```

```java
pool-1-thread-1: 5001
pool-1-thread-1: 5001
pool-1-thread-1: 5000
pool-1-thread-2: 5001
pool-1-thread-2: 5000
pool-1-thread-2: 5001
pool-1-thread-2: 5001
pool-1-thread-2: 5000
pool-1-thread-1: 5001
pool-1-thread-1: 5000
...
```

> `ScheduledThreadPoolExecutor `特殊参数说明

```java
/**
 * 允许现有周期性任务在Shutdown之后继续执行
 */
private volatile boolean continueExistingPeriodicTasksAfterShutdown;

/**
 * 允许现有延时任务在Shutdown之后继续执行
 */
private volatile boolean executeExistingDelayedTasksAfterShutdown = true;
```

```java
public class ScheduledExecutorServiceTest {
    public static void main(String[] args) {
        ScheduledThreadPoolExecutor scheduledThreadPool = (ScheduledThreadPoolExecutor) Executors.newScheduledThreadPool(2);
        scheduledThreadPool.setExecuteExistingDelayedTasksAfterShutdownPolicy(false);
        //scheduledThreadPool.setContinueExistingPeriodicTasksAfterShutdownPolicy(true);
        final AtomicLong time = new AtomicLong(-1);
        scheduledThreadPool.scheduleAtFixedRate(() -> {
            sleep(3);
            if (time.get() > 0) {
                System.out.println(Thread.currentThread().getName() + ": " + (System.currentTimeMillis() - time.get()));
            }
            time.set(System.currentTimeMillis());
        }, 1, 2, TimeUnit.SECONDS);
        scheduledThreadPool.shutdown();
    }

    private static void sleep(int time) {
        try {
            TimeUnit.SECONDS.sleep(time);
        } catch (Exception e) {
        }
    }
}
```

# CompletableFuture

### 创建对象

#### runAsync

```java
public static CompletableFuture<Void> runAsync(Runnable runnable)
public static CompletableFuture<Void> runAsync(Runnable runnable, Executor executor)
```

#### supplyAsync

```java
public static <U> CompletableFuture<U> supplyAsync(Supplier<U> supplier)
public static <U> CompletableFuture<U> supplyAsync(Supplier<U> supplier, Executor executor)
```

runAsync 方法以Runnable函数式接口类型为参数，没有返回结果，

supplyAsync 方法Supplier函数式接口类型为参数，返回结果类型为U；

没有指定Executor的方法会使用ForkJoinPool.commonPool() 作为它的线程池执行异步代码。如果指定线程池，则使用指定的线程池运行

### 结果处理

> 当CompletableFuture的计算结果完成，或者抛出异常的时候，我们可以执行特定的 Action

#### whenComplete

```java
public CompletableFuture<T> whenComplete(BiConsumer<? super T,? super Throwable> action)
public CompletableFuture<T> whenCompleteAsync(BiConsumer<? super T,? super Throwable> action)
public CompletableFuture<T> whenCompleteAsync(BiConsumer<? super T,? super Throwable> action, Executor executor)
```

#### exceptionally

```java
public CompletableFuture<T> exceptionally(Function<Throwable,? extends T> fn)
```

- Action的类型是BiConsumer<? super T,? super Throwable>，它可以处理正常的计算结果，或者异常情况。
- 方法不以Async结尾，意味着Action使用相同的线程执行，而Async可能会使用其它的线程去执行(如果使用相同的线程池，也可能会被同一个线程选中执行。
- **这几个方法都会返回CompletableFuture。当Action执行完毕后，<font color="red">返回原始的CompletableFuture的计算结果或者返回异常</font>。**

```java
public class CompletableFutureExample2 {
    public static void main(String[] args) {
        CompletableFuture<Void> future = CompletableFuture.runAsync(() -> {
            sleep(1);
            int data = ThreadLocalRandom.current().nextInt(20);
            if (data % 2 == 0) {
                System.out.println(Thread.currentThread().getName() + "：数据-" + data);
                int i = 12 / 0;
            }
            System.out.println(Thread.currentThread().getName() + "：执行结束");
        });
        future.whenComplete(new BiConsumer<Void, Throwable>() {
            @Override
            public void accept(Void t, Throwable action) {
                System.out.println(Thread.currentThread().getName() + "：执行完成");
            }
        });
        future.exceptionally(new Function<Throwable, Void>() {
            @Override
            public Void apply(Throwable t) {
                System.out.println(Thread.currentThread().getName() + "：执行失败，" + t.getMessage());
                return null;
            }
        }).join();
    }

    private static void sleep(int seconds) {
        try {
            TimeUnit.SECONDS.sleep(seconds);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

正常执行结束

```java
ForkJoinPool.commonPool-worker-1：执行结束
ForkJoinPool.commonPool-worker-1：执行完成
```

抛出异常

```java
ForkJoinPool.commonPool-worker-1：数据-6
ForkJoinPool.commonPool-worker-1：执行失败，java.lang.ArithmeticException: / by zero
ForkJoinPool.commonPool-worker-1：执行完成
```

#### handle

- 当原先的CompletableFuture的值计算完成或者抛出异常的时候，由BiFunction参数计算，<font color="red">产生新的CompletableFuture</font>

这组方法兼有whenComplete和转换的两个功能（whenComplete and reture）

```java
public <U> CompletableFuture<U> handle(BiFunction<? super T,Throwable,? extends U> fn)
public <U> CompletableFuture<U> handleAsync(BiFunction<? super T,Throwable,? extends U> fn)
public <U> CompletableFuture<U> handleAsync(BiFunction<? super T,Throwable,? extends U> fn, Executor executor)
```

测试DEMO：

```java
public static void main(String[] args) throws ExecutionException, InterruptedException {
	CompletableFuture<Integer> future = CompletableFuture.supplyAsync(() -> {
		int result = 100;
		System.out.println("一阶段：" + result);
		return result;
	}).handle((number, exception) -> {
		int result = number * 3;
		System.out.println("二阶段：" + result);
		return result;
	});
	System.out.println("最终结果：" + future.get());
}
```

```java
一阶段：100
二阶段：300
最终结果：300
```



### 结果转换（Function）

所谓结果转换，就是将上一段任务的执行结果作为下一阶段任务的入参参与重新计算，<font color="red">产生新的结果</font>

#### thenApply

1. `thenApply` 接收一个函数作为参数，使用该函数处理上一个CompletableFuture 调用的结果，并返回一个具有处理结果的Future对象。

   ```java
   public <U> CompletableFuture<U> thenApply(Function<? super T,? extends U> fn)
   public <U> CompletableFuture<U> thenApplyAsync(Function<? super T,? extends U> fn)
   public <U> CompletableFuture<U> thenApplyAsync(Function<? super T,? extends U> fn, Executor executor)
   ```

   **示例：**

   ```java
   public static void main(String[] args) throws ExecutionException, InterruptedException {
       CompletableFuture<Integer> future = CompletableFuture.supplyAsync(() -> {
       int result = 100;
       System.out.println("一阶段：" + result);
           return result;
       }).thenApply(number -> {
           int result = number * 3;
           System.out.println("二阶段：" + result);
           return result;
       });
       System.out.println("最终结果：" + future.get());
   }
   ```

   ```java
   一阶段：100
   二阶段：300
   最终结果：300
   ```

#### thenCompose

1. `thenCompose`的参数为一个返回 CompletableFuture 实例的函数，该函数的参数是先前计算步骤的结果。

   ```java
   public <U> CompletableFuture<U> thenCompose(Function<? super T, ? extends CompletionStage<U>> fn);
   public <U> CompletableFuture<U> thenComposeAsync(Function<? super T, ? extends CompletionStage<U>> fn) ;
   public <U> CompletableFuture<U> thenComposeAsync(Function<? super T, ? extends CompletionStage<U>> fn, Executor executor) ;
   ```

   示例

   ```java
   public static void main(String[] args) throws InterruptedException, ExecutionException {
       CompletableFuture<Integer> future = CompletableFuture.supplyAsync(new Supplier<Integer>() {
           @Override
           public Integer get() {
               int number = new Random().nextInt(3);
               System.out.println("第一阶段：" + number);
               return number;
           }
       }).thenCompose(new Function<Integer, CompletionStage<Integer>>() {
           @Override
           public CompletionStage<Integer> apply(Integer param) {
               return CompletableFuture.supplyAsync(new Supplier<Integer>() {
                   @Override
                   public Integer get() {
                       int number = param * 2;
                       System.out.println("第二阶段：" + number);
                       return number;
                   }
               });
           }
       });
       System.out.println("最终结果: " + future.get());
   }
   ```

   那么 `thenApply `和`thenCompose `有何区别呢：

   - `thenApply `转换的是泛型中的类型，返回的是同一个CompletableFuture；
   - `thenCompose` 使用上一个CompletableFutre 调用的结果在下一步的 CompletableFuture 调用中进行运算，是生成一个内部构造的新的CompletableFuture。

   下面用一个例子对对比：

   ```java
   public static void main(String[] args) throws InterruptedException, ExecutionException {
       CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> "Hello");
   
       CompletableFuture<String> result1 = future.thenApply(param -> param + " World");
       CompletableFuture<String> result2 = future.thenCompose(param -> CompletableFuture.supplyAsync(() -> param + " World"));
   
       System.out.println(result1.get());
       System.out.println(result2.get());
   }
   ```

   

### 结果消费（Consumer）

与结果处理和结果转换系列函数返回一个新的 CompletableFuture 不同，结果消费系列函数只对结果执行Action，而<font color="red">不返回新的计算值</font>。

根据对结果的处理方式，结果消费函数又分为：

- `thenAccept`系列：对单个结果进行消费
- `thenAcceptBoth`系列：对两个结果进行消费
- `thenRun`系列：不关心结果，只对结果执行Action

#### thenAccept

通过观察该系列函数的参数类型可知，它们是函数式接口Consumer，这个接口只有输入，没有返回值。

```java
public CompletionStage<Void> thenAccept(Consumer<? super T> action);
public CompletionStage<Void> thenAcceptAsync(Consumer<? super T> action);
public CompletionStage<Void> thenAcceptAsync(Consumer<? super T> action,Executor executor);
```

示例

```csharp
public static void main(String[] args) throws ExecutionException, InterruptedException {
    CompletableFuture<Void> future = CompletableFuture.supplyAsync(() -> {
        int number = new Random().nextInt(10);
        System.out.println("第一阶段：" + number);
        return number;
    }).thenAccept(number -> System.out.println("第二阶段：" + number * 5));
    System.out.println("最终结果：" + future.get());
}
```

#### thenAcceptBoth

thenAcceptBoth 函数的作用是，当两个 CompletionStage 都正常完成计算的时候，就会执行提供的action，消费两个异步的结果。没有返回值。

```java
public <U> CompletionStage<Void> thenAcceptBoth(CompletionStage<? extends U> other,BiConsumer<? super T, ? super U> action);
public <U> CompletionStage<Void> thenAcceptBothAsync(CompletionStage<? extends U> other,BiConsumer<? super T, ? super U> action);
public <U> CompletionStage<Void> thenAcceptBothAsync(CompletionStage<? extends U> other,BiConsumer<? super T, ? super U> action, Executor executor);
```

示例

```java
public static void main(String[] args) throws ExecutionException, InterruptedException {
    CompletableFuture<Integer> futrue1 = CompletableFuture.supplyAsync(new Supplier<Integer>() {
        @Override
        public Integer get() {
            int number = new Random().nextInt(3) + 1;
            try {
                TimeUnit.SECONDS.sleep(number);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("第一阶段：" + number);
            return number;
        }
    });

    CompletableFuture<Integer> future2 = CompletableFuture.supplyAsync(new Supplier<Integer>() {
        @Override
        public Integer get() {
            int number = new Random().nextInt(3) + 1;
            try {
                TimeUnit.SECONDS.sleep(number);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("第二阶段：" + number);
            return number;
        }
    });

    futrue1.thenAcceptBoth(future2, new BiConsumer<Integer, Integer>() {
        @Override
        public void accept(Integer x, Integer y) {
            System.out.println("最终结果：" + (x + y));
        }
    }).join();
}
```

#### thenRun

thenRun 也是对线程任务结果的一种消费函数，与thenAccept不同的是，thenRun 会在上一阶段 CompletableFuture 计算完成的时候执行一个Runnable，但是不使用该 CompletableFuture 计算的结果。

```java
public CompletionStage<Void> thenRun(Runnable action);
public CompletionStage<Void> thenRunAsync(Runnable action);
public CompletionStage<Void> thenRunAsync(Runnable action,Executor executor);
```

示例

```java
public static void main(String[] args) throws ExecutionException, InterruptedException {
    CompletableFuture<Void> future = CompletableFuture.supplyAsync(() -> {
        int number = new Random().nextInt(10);
        System.out.println("第一阶段：" + number);
        return number;
    }).thenRun(() -> System.out.println("thenRun 执行"));
    System.out.println("最终结果：" + future.get());
}
```

### 结果组合

#### thenCombine

thenCombine 方法，合并两个线程任务的结果，并进一步处理。

```java
public <U,V> CompletionStage<V> thenCombine(CompletionStage<? extends U> other, BiFunction<? super T,? super U,? extends V> fn);
public <U,V> CompletionStage<V> thenCombineAsync(CompletionStage<? extends U> other, BiFunction<? super T,? super U,? extends V> fn);
public <U,V> CompletionStage<V> thenCombineAsync(CompletionStage<? extends U> other, BiFunction<? super T,? super U,? extends V> fn,Executor executor);
```

示例

```java
public static void main(String[] args) throws ExecutionException, InterruptedException {
    CompletableFuture<Integer> future1 = CompletableFuture.supplyAsync(new Supplier<Integer>() {
        @Override
        public Integer get() {
            int number = new Random().nextInt(10);
            System.out.println("第一阶段：" + number);
            return number;
        }
    });
  
    CompletableFuture<Integer> future2 = CompletableFuture.supplyAsync(new Supplier<Integer>() {
        @Override
        public Integer get() {
            int number = new Random().nextInt(10);
            System.out.println("第二阶段：" + number);
            return number;
        }
    });
  
    CompletableFuture<Integer> result = future1.thenCombine(future2, new BiFunction<Integer, Integer, Integer>() {
        @Override
        public Integer apply(Integer x, Integer y) {
            return x + y;
        }
    });
    System.out.println("最终结果：" + result.get());
}
```



### 任务交互

线程交互，是指将两个线程任务获取结果的速度相比较，按一定的规则进行下一步处理。

#### applyToEither（转换）

两个线程任务相比较，先获得执行结果的，就对该结果进行下一步的转换操作。

```java
public <U> CompletionStage<U> applyToEither(CompletionStage<? extends T> other,Function<? super T, U> fn);
public <U> CompletionStage<U> applyToEitherAsync(CompletionStage<? extends T> other,Function<? super T, U> fn);
public <U> CompletionStage<U> applyToEitherAsync(CompletionStage<? extends T> other,Function<? super T, U> fn,Executor executor);
```

示例

```java
public static void main(String[] args) throws ExecutionException, InterruptedException {
    CompletableFuture<Integer> future1 = CompletableFuture.supplyAsync(new Supplier<Integer>() {
        @Override
        public Integer get() {
            int number = new Random().nextInt(3);
            try {
                TimeUnit.SECONDS.sleep(number);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("第一阶段：" + number);
            return number;
        }
    });
    CompletableFuture<Integer> future2 = CompletableFuture.supplyAsync(new Supplier<Integer>() {
        @Override
        public Integer get() {
            int number = new Random().nextInt(3);
            try {
                TimeUnit.SECONDS.sleep(number);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("第二阶段：" + number);
            return number;
        }
    });

    future1.applyToEither(future2, new Function<Integer, Integer>() {
        @Override
        public Integer apply(Integer number) {
            System.out.println("最快结果：" + number);
            return number * 2;
        }
    }).join();
}
```

#### acceptEither（消费）

两个线程任务相比较，先获得执行结果的，就对该结果进行下一步的消费操作。

```java
public CompletionStage<Void> acceptEither(CompletionStage<? extends T> other,Consumer<? super T> action);
public CompletionStage<Void> acceptEitherAsync(CompletionStage<? extends T> other,Consumer<? super T> action);
public CompletionStage<Void> acceptEitherAsync(CompletionStage<? extends T> other,Consumer<? super T> action,Executor executor);
```

示例

```java
public static void main(String[] args) {
    CompletableFuture<Integer> future1 = CompletableFuture.supplyAsync(new Supplier<Integer>() {
        @Override
        public Integer get() {
            int number = new Random().nextInt(3) + 1;
            try {
                TimeUnit.SECONDS.sleep(number);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("第一阶段：" + number);
            return number;
        }
    });

    CompletableFuture<Integer> future2 = CompletableFuture.supplyAsync(new Supplier<Integer>() {
        @Override
        public Integer get() {
            int number = new Random().nextInt(3) + 1;
            try {
                TimeUnit.SECONDS.sleep(number);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("第二阶段：" + number);
            return number;
        }
    });

    future1.acceptEither(future2, new Consumer<Integer>() {
        @Override
        public void accept(Integer number) {
            System.out.println("最快结果：" + number);
        }
    }).join();
}
```

#### runAfterEither

两个线程任务相比较，有任何一个执行完成，就进行下一步操作，不关心运行结果。

```java
public CompletionStage<Void> runAfterEither(CompletionStage<?> other,Runnable action);
public CompletionStage<Void> runAfterEitherAsync(CompletionStage<?> other,Runnable action);
public CompletionStage<Void> runAfterEitherAsync(CompletionStage<?> other,Runnable action,Executor executor);
```

示例

```java
import java.util.Random;
import java.util.concurrent.CompletableFuture;
import java.util.concurrent.TimeUnit;
import java.util.function.Supplier;

public class CompletableFutureTest {
    public static void main(String[] args) {
        CompletableFuture<Integer> future1 = CompletableFuture.supplyAsync(new Supplier<Integer>() {
            @Override
            public Integer get() {
                int number = new Random().nextInt(5);
                try {
                    TimeUnit.SECONDS.sleep(number);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println("第一阶段：" + number);
                return number;
            }
        });

        CompletableFuture<Integer> future2 = CompletableFuture.supplyAsync(new Supplier<Integer>() {
            @Override
            public Integer get() {
                int number = new Random().nextInt(5);
                try {
                    TimeUnit.SECONDS.sleep(number);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println("第二阶段：" + number);
                return number;
            }
        });

        future1.runAfterEither(future2, new Runnable() {
            @Override
            public void run() {
                System.out.println("已经有一个任务完成了");
            }
        }).join();
    }
}
```

#### runAfterBoth

两个线程任务相比较，两个全部执行完成，才进行下一步操作，不关心运行结果。

```java
public CompletionStage<Void> runAfterBoth(CompletionStage<?> other,Runnable action);
public CompletionStage<Void> runAfterBothAsync(CompletionStage<?> other,Runnable action);
public CompletionStage<Void> runAfterBothAsync(CompletionStage<?> other,Runnable action,Executor executor);
```

示例

```java
import java.util.concurrent.CompletableFuture;
import java.util.concurrent.TimeUnit;
import java.util.function.Supplier;

public class CompletableFutureTest {
    public static void main(String[] args) {
        CompletableFuture<Integer> future1 = CompletableFuture.supplyAsync(new Supplier<Integer>() {
            @Override
            public Integer get() {
                try {
                    TimeUnit.SECONDS.sleep(1);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println("第一阶段：1");
                return 1;
            }
        });

        CompletableFuture<Integer> future2 = CompletableFuture.supplyAsync(new Supplier<Integer>() {
            @Override
            public Integer get() {
                try {
                    TimeUnit.SECONDS.sleep(2);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println("第二阶段：2");
                return 2;
            }
        });

        future1.runAfterBoth(future2, new Runnable() {
            @Override
            public void run() {
                System.out.println("上面两个任务都执行完成了。");
            }
        }).get();
    }
}
```

#### anyOf

anyOf 方法的参数是多个给定的 CompletableFuture，当其中的任何一个完成时，返回这个任务的 CompletableFuture

```java
public static void main(String[] args) throws ExecutionException, InterruptedException {
    Random random = new Random();
    CompletableFuture<String> future1 = CompletableFuture.supplyAsync(() -> {
        try {
            TimeUnit.SECONDS.sleep(random.nextInt(5));
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return "hello";
    });
    
    CompletableFuture<String> future2 = CompletableFuture.supplyAsync(() -> {
        try {
            TimeUnit.SECONDS.sleep(random.nextInt(1));
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return "world";
    });
    CompletableFuture<Object> result = CompletableFuture.anyOf(future1, future2);
    System.out.println(result.get());
}
```

#### allOf

allOf方法用来实现监听 多个 CompletableFuture 的全部完成。

示例

```java
public static void main(String[] args) {
    CompletableFuture<String> future1 = CompletableFuture.supplyAsync(() -> {
        try {
            TimeUnit.SECONDS.sleep(2);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("future1完成！");
        return "future1完成！";
    });
    
    CompletableFuture<String> future2 = CompletableFuture.supplyAsync(() -> {
        System.out.println("future2完成！");
        return "future2完成！";
    });
    
    CompletableFuture<Void> combindFuture = CompletableFuture.allOf(future1, future2);
    try {
        combindFuture.get();
    } catch (InterruptedException e) {
        e.printStackTrace();
    } catch (ExecutionException e) {
        e.printStackTrace();
    }
    System.out.println("future1: " + future1.isDone() + "，future2: " + future2.isDone());
}
```

### CompletableFuture：其他

#### getNow：提交任务继续运行

```java
public T getNow(T valueIfAbsent)
```

 示例：

```java
public class CompletableFutureExample4 {
    public static void main(String[] args) throws InterruptedException {
        String result = CompletableFuture.supplyAsync(() -> {
            sleep(3);
            System.out.println(System.currentTimeMillis() + " 任务继续执行");
            return System.currentTimeMillis() + " HELLO";
        }).getNow(System.currentTimeMillis() + " WORLD");
        System.out.println(result);
        sleep(5);
        System.out.println(System.currentTimeMillis() + " main exit.");
    }

    private static void sleep(int seconds) {
        try {
            TimeUnit.SECONDS.sleep(seconds);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}

# Console输出
1659410799621 WORLD
1659410802622 任务继续执行
1659411545079 main exit.
```

#### complete：提交任务不会继续运行

如果尚未完成，则将 get() 和相关方法返回的值设置为给定值

```java
public boolean complete(T value)
```

示例：

```java
public class CompletableFutureExample5 {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        CompletableFuture<String> result = CompletableFuture.supplyAsync(() -> {
            sleep(3);
            System.out.println(System.currentTimeMillis() + " 任务继续执行");
            return System.currentTimeMillis() + " HELLO";
        });
        boolean status = result.complete(System.currentTimeMillis() + " WORLD");
        System.out.println(System.currentTimeMillis() + " " + status);
        System.out.println(result.get());
        sleep(5);
        System.out.println(System.currentTimeMillis() + " main exit.");
    }

    private static void sleep(int seconds) {
        try {
            TimeUnit.SECONDS.sleep(seconds);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
# Console输出
1659410986243 true
1659410986243 WORLD
1659410991249 main exit. 
```

#### completeExceptionally

如果任务尚未完成，则导致调用 get() 和相关方法抛出给定的异常

```java
public boolean completeExceptionally(Throwable ex)
```

示例

```java
public class CompletableFutureExample6 {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        CompletableFuture<String> result = CompletableFuture.supplyAsync(() -> {
            sleep(3);
            System.out.println(System.currentTimeMillis() + " 任务继续执行");
            return System.currentTimeMillis() + " HELLO";
        });
        boolean status = result.completeExceptionally(new RuntimeException("等不及返回结果"));
        System.out.println(System.currentTimeMillis() + " " + status);
        // 抛出异常
        System.out.println(result.get());
        // 不会执行
        //result.thenAccept((e) -> {
        //    System.out.println("-------------");
        //});
        // 不会执行
        //result.thenApply(e -> "-----");
        sleep(5);
        System.out.println(System.currentTimeMillis() + " main exit.");
    }

    private static void sleep(int seconds) {
        try {
            TimeUnit.SECONDS.sleep(seconds);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

```java
1659412396003 true
Exception in thread "main" java.util.concurrent.ExecutionException: java.lang.RuntimeException: 等不及返回结果
	at java.util.concurrent.CompletableFuture.reportGet(CompletableFuture.java:357)
	at java.util.concurrent.CompletableFuture.get(CompletableFuture.java:1895)
	at hots.utils.executor.future.CompletableFutureExample6.main(CompletableFutureExample6.java:22)
Caused by: java.lang.RuntimeException: 等不及返回结果
	at hots.utils.executor.future.CompletableFutureExample6.main(CompletableFutureExample6.java:19)
 
```

# 并发集合

JDK中并发队列提供了两种实现,一种是高性能队列ConcurrentLinkedQueue,一种是阻塞队列BlockingQueue,两种都继承自Queue:

## BlockingQueue集合类关系图

![BlockingDeque](D:\0_Notes\Hexo\hmxyl\source\_images\BlockingDeque-1667231117133-94.png) 

## BlockingQueue的7个子类

- Queue 方法概述

|         | Throws exception | Returns special value                                        |      |
| ------- | ---------------- | ------------------------------------------------------------ | ---- |
| Insert  | add(e)           | offer(e)：【@return：true if the element was added to this queue, else false】 |      |
| Remove  | remove()         | poll()：【@return：the head of this queue, or null if the specified waiting time elapses before an element is available】 |      |
| Examine | element()        | peek()：【@return：the head of this queue, or null if this queue is empty】 |      |



-  BlockingQueue 方法概述

|         | Throws exception | Returns special value | Blocks | Times out            |
| ------- | ---------------- | --------------------- | ------ | -------------------- |
| Insert  | add(e)           | offer(e)              | put(e) | offer(e, time, unit) |
| Remove  | remove()         | poll()                | take() | poll(time, unit)     |
| Examine | element()        | peek()                | ---    | ---                  |

-  说明

1. ArrayBlockingQueue

   ​		 **基于数组的阻塞队列实现**，在ArrayBlockingQueue内部，维护了一个定长数组，以便缓存队列中的数据对象，这是一个常用的阻塞队列，除了一个定长数组外，ArrayBlockingQueue内部还保存着两个整形变量，分别标识着队列的头部和尾部在数组中的位置。**ArrayBlockingQueue在生产者放入数据和消费者获取数据，都是共用同一个锁对象**，**由此也意味着两者无法真正并行运行，这点尤其不同于LinkedBlockingQueue；** 按照实现原理来分析，ArrayBlockingQueue完全可以采用分离锁，从而实现生产者和消费者操作的完全并行运行。Doug Lea之所以没这样去做，也许是因为ArrayBlockingQueue的数据写入和获取操作已经足够轻巧，以至于引入独立的锁机制，除了给代码带来额外的复杂性外，其在性能上完全占不到任何便宜。  **ArrayBlockingQueue和LinkedBlockingQueue间还有一个明显的不同之处在于，前者在插入或删除元素时不会产生或销毁任何额外的对象实例，而后者则会生成一个额外的Node对象。这在长时间内需要高效并发地处理大批量数据的系统中，其对于GC的影响还是存在一定的区别。而在创建ArrayBlockingQueue时，我们还可以控制对象的内部锁是否采用公平锁，默认采用非公平锁。** 

2. LinkedBlockingQueue

   - **基于链表的阻塞队列**，同ArrayBlockingQueue类似，其内部也维持着一个数据缓冲队列（该队列由一个链表构成）。当生产者往队列中放入一个数据时，队列会从生产者手中获取数据，并缓存在队列内部，而生产者立即返回；只有当队列缓冲区达到最大值缓存容量时（LinkedBlockingQueue可以通过构造函数指定该值），才会阻塞生产者队列，直到消费者从队列中消费掉一份数据，生产者线程会被唤醒，反之对于消费者这端的处理也基于同样的原理。而LinkedBlockingQueue之所以能够高效的处理并发数据，**还因为其对于生产者端和消费者端分别采用了独立的锁来控制数据同步，这也意味着在高并发的情况下生产者和消费者可以并行地操作队列中的数据，以此来提高整个队列的并发性能。**
   - **作为开发者，我们需要注意的是，如果构造一个LinkedBlockingQueue对象，而没有指定其容量大小，LinkedBlockingQueue会默认一个类似无限大小的容量（Integer.MAX_VALUE），这样的话，如果生产者的速度一旦大于消费者的速度，也许还没有等到队列满阻塞产生，系统内存就有可能已被消耗殆尽了**
   - ArrayBlockingQueue和LinkedBlockingQueue是两个最普通也是最常用的阻塞队列，一般情况下，在处理多线程间的生产者消费者问题，使用这两个类足以

3. PriorityBlockingQueue

   ​		基于优先级的阻塞队列（优先级的判断通过构造函数传入的Compator对象来决定），但需要注意的是PriorityBlockingQueue并不会阻塞数据生产者，而只会在没有可消费的数据时，阻塞数据的消费者。因此使用的时候要特别注意，生产者生产数据的速度绝对不能快于消费者消费数据的速度，否则时间一长，会最终耗尽所有的可用堆内存空间。在实现PriorityBlockingQueue时，**内部控制线程同步的锁采用的是公平锁**

4. DelayQueue

   ​		DelayQueue中的元素只有当其指定的延迟时间到了，才能够从队列中获取到该元素。DelayQueue是一个没有大小限制的队列，因此往队列中插入数据的操作（生产者）永远不会被阻塞，而只有获取数据的操作（消费者）才会被阻塞。

   ​		DelayQueue使用场景较少，但都相当巧妙，常见的例子比如使用一个DelayQueue来管理一个超时未响应的连接队列。

5. SynchronousQueue

   ​		一种无缓冲的等待队列，生产者产生的数据直接会被消费者获取并消费， 类似于无中介的直接交易，有点像原始社会中的生产者和消费者，生产者拿着产品去集市销售给产品的最终消费者，而消费者必须亲自去集市找到所要商品的直接生产者，如果一方没有找到合适的目标，那么对不起，大家都在集市等待。相对于有缓冲的BlockingQueue来说，少了一个中间经销商的环节（缓冲区），如果有经销商，生产者直接把产品批发给经销商，而无需在意经销商最终会将这些产品卖给那些消费者，由于经销商可以库存一部分商品，因此相对于直接交易模式，总体来说采用中间经销商的模式会吞吐量高一些（可以批量买卖）；但另一方面，又因为经销商的引入，使得产品从生产者到消费者中间增加了额外的交易环节，单个产品的及时响应性能可能会降低。
   　　

   ​		声明一个SynchronousQueue有两种不同的方式，它们之间有着不太一样的行为。

   公平模式和非公平模式的区别:
   　　

   - 如果采用公平模式：SynchronousQueue会采用公平锁，并**配合一个FIFO队列**来阻塞多余的生产者和消费者，从而体系整体的公平策略；

   - 但如果是非公平模式（SynchronousQueue默认）：SynchronousQueue采用非公平锁，同时**配合一个LIFO队列**来管理多余的生产者和消费者，而后一种模式，如果生产者和消费者的处理速度有差距，则很容易出现饥渴的情况，即可能有某些生产者或者是消费者的数据永远都得不到处理。

6. LinkedTransferQueue

   传入的数据，需要担保被使用了。否则放入失败/阻塞

   ```
   private static final int NOW   = 0; // for untimed poll, tryTransfer
   private static final int ASYNC = 1; // for offer, put, add
   private static final int SYNC  = 2; // for transfer, take
   private static final int TIMED = 3; // for timed poll, tryTransfer
   ```



|                                            | ArrayBlockingQueue                                           | PriorityBlockingQueue                                        | LinkedBlockingQueue                                          | LinkedBlockingDeque                                          | SynchronousQueue                                             | DelayQueue                                                   | LinkedTransferQueue                                          |
| ------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **bounded（边界）**                        | Y                                                            | N                                                            | Optional                                                     | Optional                                                     | Y                                                            | N                                                            | N                                                            |
| **add**                                    | 添加成功：返回`true`；<br />添加失败（已满）：抛出`IllegalStateException`<br /> | 调用`offer`，返回结果同offer                                 | 调用`offer`<br />添加成功：返回`true`；<br />添加失败（false）：抛出`IllegalStateException`<br /> | <font color="orange">**由`addLast`执行**</font><br /><br />调用`offerLast`<br />添加成功：返回`true`；<br />添加失败（false）：抛出`IllegalStateException`<br /> | 调用`offer`<br />添加成功：返回`true`；<br />添加失败（已满）：抛出`IllegalStateException`<br /> | 同`offer`                                                    | 在尾部插入元素<br />无边界Queue，不会抛出IllegalStateException，或者false。<br />添加成功：返回`true`；<br /> |
| <font color="red">**offer**</font>         | 添加成功：返回`true`；<br />添加失败(已满)：返回`false`<br /> | 添加成功，返回`true`；<br />无边界，不存在已满<br />抛出异常：元素无法compare<br /> | 队尾添加成功：返回`true`；<br />添加失败(已满)：返回`false`<br /> | <font color="orange">**同`offerLast`**</font><br /><br />队尾添加成功：返回`true`；<br />添加失败(已满)：返回`false`<br /> | 如果另一个线程正在等待接收，则将指定元素插入此队列，返回`true`;<br/>没有接收线程，返回`false` | 在尾部插入元素<br />添加成功，返回`true`；<br />无边界Queue，不存在已满<br /> | 同 `add`                                                     |
| <font color="blue">**put（阻塞）**</font>  | 将指定元素插入此队列的==尾部==，如果队列已满，则==等待空间可用== | 同`offer`。无边界，无需阻塞。                                | 在此队列的尾部插入指定元素，如有必要，则==等待空间可用==。   | <font color="orange">**同`putLast`**</font><br /><br />在此队列的尾部插入指定元素，如有必要，则==等待空间可用==。 | 将指定元素添加到此队列中，阻塞，等待另一个线程接收它。       | 同`offer`                                                    | 同`add`                                                      |
| **remove**                                 | `poll`头部元素，如果为null，则会抛出异常                     | `poll`头部元素，如果为null，则会抛出异常                     | `poll`头部元素，如果为null，则会抛出异常                     | <font color="orange"> **同`removeFirst`** </font> <br /><br />`pollFirst`头部元素，如果为null，则会抛出异常 | `poll`头部元素，如果为null，则会抛出异常                     | `poll`头部元素，如果为null，则会抛出异常                     | `poll`头部元素，如果为null，则会抛出异常                     |
| <font color="red">**poll**</font>          | 移除头部元素并返回<br />无元素返回`null`                     | 移除头部元素并返回<br />无元素返回`null`                     | 移除头部元素并返回<br />无元素返回`null`                     | <font color="orange"> **同`pollFirst`** </font> <br /><br />移除头部元素并返回<br />无元素返回`null` | 移除头部元素并返回<br />如果没有可用元素，则返回 `null`<br />无元素返回`null` | 移除头部元素并返回<br />如果此队列没有具有过期延迟的元素，则返回`null`<br />无元素返回`null` | 移除头部元素并返回<br />无元素返回`null`                     |
| **element**                                | `peek `头部元素<br />若结果为null，则抛出异常`NoSuchElementException` | `peek `头部元素<br />若结果为null，则抛出异常`NoSuchElementException` | `peek `头部元素<br />若结果为null，则抛出异常`NoSuchElementException` | **`peekFirst`**头部元素<br /><br />若结果为null，则抛出异常`NoSuchElementException` | `peek `头部元素<br />永远抛出`NoSuchElementException`        | `peek `头部元素<br />若结果为null，则抛出异常`NoSuchElementException` | `peek `头部元素<br />若结果为null，则抛出异常`NoSuchElementException` |
| **peek**                                   | 获取头部元素<br />不会删除元素                               | 获取头部元素<br />不会删除元素                               | 获取头部元素<br />不会删除元素                               | **`peekFirst`**头部元素<br /><br />获取头部元素<br />不会删除元素 | 直接 `return null`                                           | 获取头部元素<br />不会删除元素                               | 获取头部元素<br />不会删除元素                               |
| <font color="blue">**take（阻塞）**</font> | 移除头部元素并返回<br />队列无元素，则阻塞等待<br />         | 移除头部元素并返回<br />队列无元素，则阻塞等待<br />         | 移除头部元素并返回<br />队列无元素，则阻塞等待<br />         | 同**`takeFirst`**<br /><br />移除头部元素并返回<br />队列无元素，则阻塞等待<br /> | 一直等待，直到有另一个线程`transfer`元素                     | 一直等待，直到有线程放进元素，且头部元素过期                 | 一直等待，直到有线程放进元素                                 |
| **transfer**                               | 无                                                           | 无                                                           | 无                                                           | 无                                                           | 无                                                           | 无                                                           | 在队列尾部插入元素，若没有被消费，则一直等待                 |



- ConcurrentHashMap

- ConcurrentSkipListMap

- ConcurrentSkipListSet

- ConcurrentLinkedQueue

- ConcurrentLinkedDeque

- CopyOnWriteArrayList

- CopyOnWriteArraySet
