---
title: Java 8 in Action【汪文君】
date: 2022-10-25 19:52:44
tags: Java8
category: 
  - [Java, Java8基础]
description: 'Java 8 in Action'
---



源码： [Java8 In Action](https://github.com/java8/Java8InAction)
笔记来源：[https://waltyou.github.io](https://waltyou.github.io) 

# 全书脑图

![ Java 8 in Action](D:\0_Notes\Hexo\hmxyl\source\_images\Java+8+In+Action-16376831319681.png)

 

# 梳理脉络

通过脑图可以看出，全书分为四个部分：

1. 基础知识，重点是为何关心java8，行为参数化和lambda

2. 函数式编程，重点是全面系统的介绍Stream

3. Java8的其他改善点

   +  重构/测试/调试
   +  默认方法（Default Function）
   +  Optional替代null
   +  CompletableFuture 组合式异步编程
   +  日期时间API

4. Java8之上：对函数式编程的思考，函数编程的技巧，与Scala的比较

# 行为参数化

1. Why
   应对不断变化的需求，避免啰嗦，而且不打破DRY（Don’t Repeat Yourself）规则。
2. What
   简单讲：把方法（你的代码）作为参数传递给另一个方法。
   复杂讲： 让方法接受多种行为（或战略）作为参数，并在内部使用，来完成不同的行为。
3. How
   Example：用一个Comparator排序Apple，使用Java 8中List默认的sort方法。

```java
// java.util.Comparator
public interface Comparator<T> {
    public int compare(T o1, T o2);
}
// 匿名类写法
inventory.sort(new Comparator<Apple>() {
    public int compare(Apple a1, Apple a2){
        return a1.getWeight().compareTo(a2.getWeight());
    }
});
// lambda写法
inventory.sort(
    (Apple a1, Apple a2) ->
    a1.getWeight().compareTo(a2.getWeight()));
```

# 匿名函数lambda

使用lambda之前，需要了解两个概念：**函数式接口 **和**函数描述符**

## 1. 函数式接口

定义：只定义一个抽象方法的接口
`@FunctionalInterface`：标注用于表示该接口会设计成一个函数式接口。
Java 8 提供了一些新的函数式接口， 位置：java.util.function

 [函数式接口.xmind](D:\0_Notes\Hexo\hmxyl\source\_images\函数式接口.xmind)  

![image-20220502170433104](D:\0_Notes\Hexo\hmxyl\source\_images\image-20220502170433104.png) 

##  2. 函数描述符

函数式接口的抽象方法的签名基本上就是Lambda表达式的签名。如下：

```java
() -> void
(Apple) -> int
(Apple, Apple) -> boolean
```

## 3. 实现细节

### 类型检查

 上下文中Lambda表达式需要的类型称为目标类型

1. 同样的Lambda，不同的函数式接口
2. 特殊的void兼容规则：如果一个Lambda的主体是一个语句表达式 它就和一个返回void的函数描述符兼容。

### 类型推断

编译器可以了解Lambda表达式的参数类型，这样就可以在Lambda语法中省去标注参数类型。

### 使用局部变量

```java
int portNumber = 1337; 
Runnable r = () -> System.out.println(portNumber); 
```

注意： Lambda可以没有限制地捕获（也就是在其主体中引用）实例变量和静态变量。但局部变量必须显式声明为final，或事实上是final。

原因： 
1）局部变量保存在栈上，并且隐式表示它们仅限于其所在线程，如果允许捕获可改变的局部变量，就会引发造成线程不安全新的可能性；
 2）不鼓励你使用改变外部变量的典型命令式编程模式

### 方法引用（method reference）

目标引用放在分隔符` :: `前, 方法的名称放在后面。

```java
inventory.sort(comparing(Apple::getWeight));
```

方法引用主要有三类:

+ 指向静态方法的方法引用: Integer::parseInt
+ 指向任意类型实例方法的方法引用: String::length
+ 指向现有对象的实例方法的方法引用: expensiveTransaction::getValue

```java
//改写
Function<String, Integer> stringToInteger = (String s) -> Integer.parseInt(s);
Function<String, Integer> stringToInteger = Integer::parseInt;

BiPredicate<List<String>, String> contains = (list, element) -> list.contains(element);
BiPredicate<List<String>, String> contains = List::contains;
```

### 复合Lambda表达式 (因为引入了默认方法)

#### 1. 比较器复合

``` java
Comparator<Apple> c = Comparator.comparing(Apple::getWeight);
// 逆序
inventory.sort(comparing(Apple::getWeight).reversed());
// 比较器链
inventory.sort(comparing(Apple::getWeight)
    .reversed()
    .thenComparing(Apple::getCountry))
```

#### 2. 谓词复合 `negate`、`and`和`or`

```java
//取非
Predicate<Apple> notRedApple = redApple.negate();
//and操作
Predicate<Apple> redAndHeavyApple = redApple.and(a -> a.getWeight() > 150);
//and + or操作
Predicate<Apple> redAndHeavyAppleOrGreen = redApple.and(a -> a.getWeight() > 150).or(a -> "green".equals(a.getColor()));
```

从左向右确定优先级.

#### 3. 函数复合

Function提供了`andThen()`, `compose()`

```java
Function<Integer, Integer> f = x -> x + 1; 
Function<Integer, Integer> g = x -> x * 2; 
// expect: (2 + 1) * 2 = 4
// f(g(x))
System.out.println(f.andThen(g).apply(1));
// expect: 1 * 2 + 1 = 3
// g(f(x))
System.out.println(f.compose(g).apply(1));
```

复合Lambda表达式可以用来创建各种转型流水线。



# Stream定义

从（支持数据处理操作的源）生成的（元素序列）

元素序列：流提供了一个接口,可以访问特定元素类型的一组有序值。
源：集合、数组或输入/输出资源
数据处理操作：filter 、 map 、 reduce 、 find 、 match 、 sort等，可顺序，可并行。
两个重要特点：

+ 流水线：多个操作可以链接起来
+ 内部迭代：流的迭代操作是在背后进行的，优点：透明地并行处理;优化处理顺序
  注意：链中的方法调用都在排队等待,直到调用 collect 。


# Stream操作

## 概念

两大类操作
    1. 中间操作：会返回另一个流，map，filter等
    2. 终端操作：从流的流水线生成结果，collect，foreach， count等
使用三要素
    + 一个数据源(如集合)来执行一个查询;
    + 一个中间操作链,形成一条流的流水线;
    + 一个终端操作,执行流水线,并能生成结果。

```Java
dishes.stream()
    .filter(d -> d.getCalories() < 400)
    .sorted(comparing(Dish::getCalories))
    .map(Dish::getName)
    .collect(toList());
```

## 常用函数

1. 筛选和切片 Filtering and slicing
   `filter()`，`distinct()`，`limit(n)`, `skip(n)`

2. 映射 Mapping
   + map: 对流中每一个元素应用函数
   + flatmap: 把一个流中的每个值都换成另一个流,然后把所有的流连接起来成为一个流。

3. 查找和匹配 Finding and matching
   + anyMatch: 流中是否有一个元素能匹配给定的谓词, 方法返回一个 boolean
   + allMatch: 流中的元素是否都能匹配给定的谓词, 方法返回一个 boolean
   + noneMatch: 流中没有任何元素与给定的谓词匹配
   + findAny: 返回当前流中的任意元素(Optional)
   + findFirst: 找到第一个元素

4. 归约 Reducing
   将流中所有元素反复结合起来。

   + 元素求和

```java
int sum = numbers.stream().reduce(0, Integer::sum);
Optional<Integer> sum = numbers.stream().reduce(Integer::sum);
```

   + 最大值，最小值

```java
Optional<Integer> max = numbers.stream().reduce(Integer::max);
Optional<Integer> min = numbers.stream().reduce(Integer::min);
```

5. 数值流 Numeric Streams
   为了避免装箱带来的复杂性

 + 映射到数值流: mapToInt、mapToDouble 和 mapToLong
 + 转换回对象流

```java
IntStream intStream = menu.stream().mapToInt(Dish::getCalories);
Stream<Integer> stream = intStream.boxed();
```

+ 默认值 OptionalInt 、 OptionalDouble 和 OptionalLong
+ 数值范围 IntStream.rangeClosed(1, 100)

6. 构建流 Building streams

   1. 由值创建流：

   ```java
    Stream<String> stream = Stream.of("Java 8 ", "Lambdas ", "In ", "Action";
    stream.map(String::toUpperCase).forEach(System.out::println);
    //空流
    Stream<String> emptyStream = Stream.empty();
   ```

       2. 由数组创建流

     ```java
   int[] numbers = {2, 3, 5, 7, 11, 13};
   int sum = Arrays.stream(numbers).sum();
     ```

       3. 由文件生成流

     ```java
   long uniqueWords = Files.lines(Paths.get("data.txt"), Charset.defaultCharset())
   .flatMap(line -> Arrays.stream(line.split(" ")))
   .distinct().count();
     ```

   4. 由函数生成流:创建无限流

     ``` java 
    // iterate
   Stream.iterate(0, n -> n + 2)
   .limit(10)
   .forEach(System.out::println);
   // generate
   Stream.generate(Math::random)
   .limit(5)
   .forEach(System.out::println);
     ```

# 用stream收集数据

## 常用的收集器

### Collectors.reducing：广义的归约汇总

需要三个参数：

+ 归约操作的起始值
+ 获取或操作对象的属性数值(转换函数)
+ BinaryOperator，如加法

举例：

```java
int totalCalories = menu.stream().collect(reducing(
    0,                      // 归约操作的起始值
    Dish::getCalories,      // 获取或操作对象的属性数值(转换函数)
    (i, j) -> i + j));      // BinaryOperator，如加法
```


### Collectors.groupingBy ：分组

+ 一级分组

```java
Map<Dish.Type, List<Dish>> dishesByType = menu.stream().collect(groupingBy(Dish::getType));

// 自定义分组
public enum CaloricLevel { DIET, NORMAL, FAT }
Map<CaloricLevel, List<Dish>> dishesByCaloricLevel = menu.stream()
                .collect(groupingBy(dish -> {
                    if (dish.getCalories() <= 400)
                        return CaloricLevel.DIET;
                    else if (dish.getCalories() <= 700)
                        return CaloricLevel.NORMAL;
                    else
                        return CaloricLevel.FAT;
                }));
```

+ 多级分组

```java
menu.stream().collect(groupingBy(Dish::getType,
            groupingBy((Dish dish) -> {
                if (dish.getCalories() <= 400) 
                    return CaloricLevel.DIET;
                else if (dish.getCalories() <= 700) 
                    return CaloricLevel.NORMAL;
                else 
                    return CaloricLevel.FAT;
            } )
        );
```

+ 与 groupingBy 联合使用的其他收集器
  有时候在groupBy的时候，我们还想做一下其他操作，比如设定返回类型，或者只取对象中的某个属性。

``` java
// summingInt
Map<Dish.Type, Integer> totalCaloriesByType = menu.stream()
.collect(groupingBy(Dish::getType, summingInt(Dish::getCalories)));

// mapping
menu.stream().collect(groupingBy(Dish::getType, mapping(dish -> {
        if (dish.getCalories() <= 400)
            return CaloricLevel.DIET;
        else if (dish.getCalories() <= 700)
            return CaloricLevel.NORMAL;
        else
            return CaloricLevel.FAT;
    }, toSet())));

//按子组收集数据
Map<Dish.Type, Long> typesCount = menu.stream().collect(
    groupingBy(Dish::getType, counting()));

//Collectors.collectingAndThen： 把收集器返回的结果转换为另一种类型
Map<Dish.Type, Dish> mostCaloricByType = menu.stream()
.collect(groupingBy(Dish::getType, 
    collectingAndThen(maxBy(comparingInt(Dish::getCalories)), Optional::get)));
```

### Collectors.counting()

### Collectors.maxBy

### Collectors.minBy

### Collectors.summingInt，Collectors.summingLong，Collectors.summingDouble

### Collectors.averagingInt，Collectors.averagingLong，Collectors.averagingDouble

### Collectors.summarizingInt

### Collectors.joining


### Collectors.partitioningBy 分区

好处：保留了分区函数返回 true 或 false 的两套流元素列表。
与groupby的区别：需要一个谓词（返回一个布尔值的函数）

```java
Map<Boolean, List<Dish>> partitionedMenu = menu.stream().collect(partitioningBy(Dish::isVegetarian));
//二级分区
menu.stream().collect(partitioningBy(Dish::isVegetarian, partitioningBy (d -> d.getCalories() > 500)));
//联合其他收集器
menu.stream().collect(partitioningBy(Dish::isVegetarian, counting()));
```

## Collector 接口

### 基本定义：

```java
public interface Collector<T, A, R> {
    Supplier<A> supplier();
    BiConsumer<A, T> accumulator();
    Function<A, R> finisher();
    BinaryOperator<A> combiner();
    Set<Characteristics> characteristics();
}
```

+ T 是流中要收集的项目的泛型
+ A 是累加器的类型,累加器是在收集过程中用于累积部分结果的对象。
+ R 是收集操作得到的对象(通常但并不一定是集合)的类型。

方法分析：

+ 建立新的结果容器: supplier 方法
+ 将元素添加到结果容器: accumulator 方法
+ 对结果容器应用最终转换: finisher 方法
+ 合并两个结果容器: combiner 方法（并行归约）
+ characteristics 方法：返回一个不可变的 Characteristics 集合
  1.  UNORDERED：归约结果不受流中项目的遍历和累积顺序的影响
  2.  CONCURRENT：accumulator函数可以从多个线程同时调用,且该收集器可以并行归约流。如果收集器没有标为UNORDERED,那它仅在用于无序数据源时才可以并行归约
  3.  IDENTITY_FINISH：表明完成器方法返回的函数是一个恒等函数

## 自定义收集器

必要时，可以根据自己需求实现收集器， 来避免一些不必要的操作（如装箱拆箱），这样子可以获取更好的性能。

测试用例如下：

```java
 @Test
    public void testOwnCollector() {

        class ToListCollector<T> implements Collector<T, List<T>, List<T>> {
            @Override
            public Supplier<List<T>> supplier() {
                log("supplier");
                return ArrayList::new;
            }

            @Override
            public BiConsumer<List<T>, T> accumulator() {
                log("accumulator");
                return List::add;
            }

            @Override
            public BinaryOperator<List<T>> combiner() {
                log("combiner");
                return (l1, l2) -> {
                    l1.addAll(l2);
                    return l1;
                };
            }

            @Override
            public Function<List<T>, List<T>> finisher() {
                return Function.identity();
            }

            @Override
            public Set<Characteristics> characteristics() {
                log("characteristics");
                return Collections.unmodifiableSet(EnumSet.of(Characteristics.IDENTITY_FINISH, Characteristics.CONCURRENT));
            }


            private void log(final String log) {
                System.out.println(Thread.currentThread().getName() + "-" + log);
            }
        }

        Collector<String, List<String>, List<String>> collector = new ToListCollector<>();
        String[] arr = new String[]{"e1", "2r", "c", "s"};
        Optional.of(Arrays.stream(arr).filter(e -> e.matches("[a-z]")).collect(collector))
                .ifPresent(e -> {
                    System.out.println(e.getClass());
                    System.out.println(JSONObject.toJSONString(e));
                });
    }
```

# 并行数据处理与性能

## 1. 并行流处理数据

java 8 中提供了现成的并行处理流，即`parallelStream`。

### 并行流与顺序流的转换

对顺序流调用parallel方法，对并行流调用sequential方法。在合适的时候顺序流与并行流相互转换，可以提高效率。

```java
stream.parallel()
    .filter(...)
    .sequential()
    .map(...)
    .parallel()
    .reduce();
```

注意点：

+ 保证在内核中并行执行工作的时间比在内核之间传输数据的时间长。
+ 避免改变了某些共享状态

### 配置并行流使用的线程池

并行流内部使用了默认的 ForkJoinPool， 它默认的线程数量就是你的处理器数量, 这个值是由`Runtime.getRuntime().availableProcessors()` 得到的。

可以通过系统属性java.util.concurrent.ForkJoinPool.common.parallelism 来改变线程池大小：

```java
System.setProperty("java.util.concurrent.ForkJoinPool.common.parallelism","12");
```

### 如何高效使用：

+ 测量
+ 留意装箱
+ 依赖于元素顺序的操作，本身在并行流上的性能就比顺序流差
+ 流的操作流水线的总计算成本
+ 数据少
+ 数据结构是否易于分解
+ 流自身的特点，以及流水线中的中间操作修改流的方式，都可能会改变分解过程的性能
+ 终端操作中合并步骤的代价是大是小

## 2. 分支/合并框架(The fork/join framework)

### 目的

以递归方式将可以并行的任务拆分成更小的任务，然后将每个子任务的结果合并起来生成整体结果。（先拆，并行处理，合并结果）

### 定义

`RecursiveTask`是ExecutorService接口的一个实现，它把子任务分配给线程池（称为ForkJoinPool）中的工作线程。

### 使用

实现`compute()`方法，提交至ForkJoinPool.invoke
实际例子：

```java
import java.util.concurrent.ForkJoinPool;
import java.util.concurrent.ForkJoinTask;
import java.util.concurrent.RecursiveTask;
import java.util.stream.LongStream;

public class ForkJoinSumCalculator extends RecursiveTask<Long> {

    // 拆分任务的标准大小
    public static final long THRESHOLD = 10_000;

    private final long[] numbers;
    private final int start;
    private final int end;

    public ForkJoinSumCalculator(long[] numbers) {
        this(numbers, 0, numbers.length);
    }

    private ForkJoinSumCalculator(long[] numbers, int start, int end) {
        this.numbers = numbers;
        this.start = start;
        this.end = end;
    }

    // 实现compute方法
    @Override
    protected Long compute() {
        int length = end - start; // 获取当前剩余任务的大小
        if (length <= THRESHOLD) {
            return computeSequentially();
        }
        // 创建另外一个子任务 leftTask
        ForkJoinSumCalculator leftTask = new ForkJoinSumCalculator(numbers, start, start + length/2);
        // 异步执行 leftTask
        leftTask.fork();
        // 创建剩余一半任务的子任务 rightTask
        ForkJoinSumCalculator rightTask = new ForkJoinSumCalculator(numbers, start + length/2, end);
        // 递归调用获取结果
        Long rightResult = rightTask.compute();
        // 获取 leftTask 结果
        Long leftResult = leftTask.join();
        return leftResult + rightResult;
    }

    private long computeSequentially() {
        long sum = 0;
        for (int i = start; i < end; i++) {
            sum += numbers[i];
        }
        return sum;
    }

    // 如何调用fork/join框架
    public static long forkJoinSum(long n) {
        long[] numbers = LongStream.rangeClosed(1, n).toArray();
        ForkJoinTask<Long> task = new ForkJoinSumCalculator(numbers);
        return new ForkJoinPool().invoke(task);
    }
}
```

### 好的做法

+ join方法会阻塞，所以先确保两个子任务全部启动，再调用join
+ `RecursiveTask`内部不应该调用ForkJoinPool.invoke，应该直接调用compute、fork，只有顺序代码才应该用 invoke 来启动并行计算
+ 一边子任务fork，一边子任务compute，避免在线程池中多分配一个任务造成的开销
+ 调试使用分支/合并框架的并行计算可能有点棘手, 调用compute的线程并不是概念上的调用方(即调用fork的那个).
+ 不应理所当然地认为在多核处理器上使用分支/合并框架就比顺序计算快。

**工作窃取（work stealing）**
目的：为解决因为每个子任务所花的时间可能天差地别而造成的效率低下。 过程：线程把任务保存到一个双向链式队列，当一个线程的队列空了，它就随机从其他线程的队列尾部“偷”一个任务执行

###  思考

问：都是拆分任务，并行执行，为什么不使用线程池，如ThreadPoolExecutor呢？

答：Thread pool 默认期望它们所有执行的任务都是不相关的，可以尽可能的并行执行。
而fork join框架解决的问题，是一个全局问题，所有子任务拆分运行后的结果，是要合并起来的。另外，`fork-join pool`另一个特点`work stealing`，如果用ThreadPoolExecutor实现是比较麻烦的

## 3. Spliterator分割流

### 简单了解

描述：一种自动机制来拆分流。新的接口“可分迭代器”（splitable iterator）

```java
public interface Spliterator<T> {
    boolean tryAdvance(Consumer<? super T> action);
    Spliterator<T> trySplit();
    long estimateSize();
    int characteristics();
}
```

### 拆分过程

递归过程。框架不断对Spliterator调用trySplit直到它返回null,表明它处理的数据结构不能再分割。

### Spliterator 的特性

|  **特性**  |                           **含义**                           |
| :--------: | :----------------------------------------------------------: |
|  ORDERED   | 元素有既定的顺序(例如 List ),因此 Spliterator 在遍历和划分时也会遵循这一顺序 |
|  DISTINCT  |   对于任意一对遍历过的元素 x 和 y , x.equals(y) 返回 false   |
|   SORTED   |              遍历的元素按照一个预定义的顺序排序              |
|   SIZED    | 该 Spliterator 由一个已知大小的源建立(例如 Set ),因此 estimatedSize() 返回的是准确值 |
|  NONNULL   |                  保证遍历的元素不会为 null                   |
|  IMMUTABL  | Spliterator 的数据源不能修改。这意味着在遍历时不能添加、删除或修改任何元素E |
| CONCURRENT |   该 Spliterator 的数据源可以被其他线程同时修改而无需同步    |
|  SUBSIZED  |  该 Spliterator 和所有从它拆分出来的 Spliterator 都是 SIZED  |

### 各个函数及作用

|     函数名      |                             作用                             |
| :-------------: | :----------------------------------------------------------: |
|   tryAdvance    | 执行一个操作给传入的元素，并且返回一个boolean，来表示是否有剩余元素需要处理 |
|    trySplit     | 最重要的函数，如果数据可以继续分割，返回一个Spliterator，否则返回null |
|  estimateSize   |                 返回一个对剩余元素数量的估值                 |
| characteristics |            设置Spliterator的某些特性，参考上表。             |

# 默认方法

```java
    public interface Sized {
        int size();

        default boolean isEmpty() {
            return size() == 0;
        }
    }
```

这样任何一个实现了Sized接口的类都会自动继承isEmpty的实现。因此，向提供了默认实 现的接口添加方法就不是源码兼容的。

☀️ 关于继承的一些错误观点

  继承不应该成为你一谈到代码复用就试图倚靠的万精油。比如，从一个拥有100个方法及 字段的类进行
继承就不是个好主意，因为这其实会引入不必要的复杂性。你完全可以使用代理 有效地规避这种窘境，即创建一个方法通过该类的成员变量直接调用该类的方法。这就是为什 么有的时候我们发现有些类被刻意地声明为final类型：**声明为final的类不能被其他的类继 承**，避免发生这样的反模式，防止核心代码的功能被污染。注意，有的时候声明为final的类 都会有其不同的原因，比如，String类被声明为final，因为我们不希望有人对这样的核心 功能产生干扰。 

  这种思想同样也适用于使用默认方法的接口。通过精简的接口，你能获得最有效的组合， 因为你可以只选择你需要的实现。

##  多来源继承调用默认方法原则

(1) 类中的方法优先级最高。类或父类中声明的方法的优先级高于任何声明为默认方法的优 先级。 
(2) 如果无法依据第一条进行判断，那么子接口的优先级更高：函数签名相同时，优先选择 拥有最具体实现的默认方法的接口，即如果B继承了A，那么B就比A更加具体。
(3) 最后，如果还是无法判断，继承了多个接口的类必须通过显式覆盖和调用期望的方法，显式地选择使用哪一个默认方法的实现。

# 用Optional取代null

## 创建 Optional 对象

1. 声明一个空的Optional

```java
Optional<Car> optCar = Optional.empty();
```

2. 依据一个非空值创建Optional

```java
Optional<Car> optCar = Optional.of(car);
```

3. 可接受null的Optional
   最后，使用静态工厂方法Optional.ofNullable，你可以创建一个允许null值的Optional 对象：

```java
Optional<Car> optCar = Optional.ofNullable(car);
```

## 使用 map 从 Optional 对象中提取和转换值

```java
Optional<Insurance> optInsurance = Optional.ofNullable(insurance);
Optional<String> name = optInsurance.map(Insurance::getName);
```

## 使用 flatMap 链接 Optional 对象

### 原因

使用 map 从 Optional 对象中提取和转换值，遭遇嵌套式的Optional结构

```java
Optional<Person> optPerson = Optional.of(person); 
Optional<String> name = optPerson.map(Person::getCar) .map(Car::getInsurance) .map(Insurance::getName);
```

### 处理

```java
public String getCarInsuranceName(Optional<Person> person) { 
	return person.flatMap(Person::getCar)
                          .flatMap(Car::getInsurance)
                          .map(Insurance::getName)
                          .orElse("Unknown"); 
}
```

## Optional类并未实现 Serializable接口

## 默认行为及解引用 Optional 对象

|    方法     |                             描述                             |
| :---------: | :----------------------------------------------------------: |
|    empty    |                  返回一个空的 Optional 实例                  |
|   filter    | 如果值存在并且满足提供的谓词，就返回包含该值的 Optional 对象；否则返回一个空的 Optional 对象 |
|   flatMap   | 如果值存在，就对该值执行提供的 mapping 函数调用，返回一个 Optional 类型的值，否则就返 回一个空的 Optional对象 |
|     get     | 如果该值存在，将该值用 Optional 封装返回，否则抛出一个 NoSuchElementException 异常 |
|  ifPresent  |     如果值存在，就执行使用该值的方法调用，否则什么也不做     |
|  isPresent  |            如果值存在就返回 true，否则返回 false             |
|     map     |       如果值存在，就对该值执行提供的 mapping 函数调用        |
|     of      | 将指定值用 Optional 封装之后返回，如果该值为 null，则抛出一个 NullPointerException 异常 |
| ofNullable  | 将指定值用 Optional 封装之后返回，如果该值为 null，则返回一个空的 Optional 对象 |
|   orElse    |            如果有值则将其返回，否则返回一个默认值            |
|  orElseGet  | 如果有值则将其返回，否则返回一个由指定的 Supplier 接口生成的值 |
| orElseThrow | 如果有值则将其返回，否则抛出一个由指定的 Supplier 接口生成的异常 |

# CompletableFuture 组合式异步编程

## Future 接口异步

Future接口在Java 5中被引入，设计初衷是对将来某个时刻会发生的结果进行建模。它建模 了一种异步计算，返回一个执行运算结果的引用，当运算结束后，这个引用被返回给调用方。

### 模拟1

+ 使用Future以异步的方式执行一个耗时的操作

```java
import org.junit.Test;
import java.util.concurrent.atomic.AtomicBoolean;
import java.util.concurrent.atomic.AtomicReference;

public class FutureInAction {

    private interface Future<T> {

        T get();

        boolean isDone();
    }

    private interface Callable<T> {
        T action();
    }

    private static <T> Future<T> invoke(Callable<T> callable) {

        // 异步返回值
        AtomicReference<T> result = new AtomicReference<>();
        // 异步处理完成
        AtomicBoolean finished = new AtomicBoolean(false);
        Thread thread = new Thread(() -> {
            result.set(callable.action());
            finished.set(true);
        });
        thread.start();
        Future<T> future = new Future<T>() {
            @Override
            public T get() {
                return result.get();
            }

            @Override
            public boolean isDone() {
                return finished.get();
            }
        };
        return future;
    }

    @Test
    public void testFuture() {
        Future<String> result = invoke(() -> {
            try {
                Thread.sleep(10);
                return "I am finished.";
            } catch (InterruptedException e) {
                return "Error";
            }
        });

        System.out.println(result.get());
        while (!result.isDone()) {
            try {
                Thread.sleep(1);
                System.out.println("waitting...");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        System.out.println(result.get());
        System.out.println(result.isDone());
    }
}
```

输出

```
null
waitting...
waitting...
waitting...
waitting...
waitting...
waitting...
I am finished.
true
```

### 模拟2

```java
import org.junit.Test;

import java.util.concurrent.*;
import java.util.concurrent.atomic.LongAccumulator;

public class FutureInAction2 {

    @Test
    public void testExecutorService() {
        ExecutorService executorService = Executors.newSingleThreadExecutor();
        LongAccumulator longAccumulator = new LongAccumulator(Long::sum, 0);
        Future<String> result = executorService.submit(() -> {
            System.out.println(Thread.currentThread().getName());
            int begin = 1;
            while (begin < 10) {
                System.out.println("+");
                Thread.sleep(1);
                longAccumulator.accumulate(begin++);
            }
            return "I am finished.";
        });

//        // 阻塞等待执行结果
//        result.get();
//        System.out.println("result..." + longAccumulator.get());

//        //最多等待10秒, 超时未执行完成，报错
//        try {
//            result.get(10, TimeUnit.SECONDS);
//            System.out.println("result..." + longAccumulator.get());
//        } catch (Exception e) {
//            // java.util.concurrent.TimeoutException
//            e.printStackTrace();
//        }

        // 阻塞等待执行结果
        if (!result.isDone()) {
            System.out.print("running");
            while (!result.isDone()) {
                System.out.print(".");
            }
            System.out.println();
            System.out.println("result..." + longAccumulator.get());
        }

        executorService.shutdown();
    }
}
```

输出

```
running.................................................................................................pool-1-thread-1
...................................................+
...................................................................................................+
................................................................................................................................................................................................................................................+
..................................................................................................................................................................+
.........................................................................................................+
..............................................................................................................................................................................................................+
..........................................................................................................................................................+
.......................................................................................................................................................+
...............................+
....................................................................................................................................................................................................
result...45
```

### 模拟3

将阻塞等待完成后调用程序

```java
import java.util.concurrent.atomic.AtomicBoolean;
import java.util.concurrent.atomic.AtomicReference;

public class FutureInAction3 {
    private interface Callable<T> {
        T action();
    }

    private interface Completable<T> {

        void complete(T t);

        void exception(Throwable throwable);
    }

    private interface Future<T> {

        T get();

        boolean isDone();

        void setCompletable(Completable<T> completable);

        Completable<T> getCompletable();
    }

    public static <T> Future<T> invoke(Callable<T> callable) {


        AtomicReference<T> atomicReference = new AtomicReference<>();
        AtomicBoolean atomicBoolean = new AtomicBoolean(false);


        Future<T> future = new Future<T>() {
            private Completable<T> completable;

            @Override
            public T get() {
                return atomicReference.get();
            }

            @Override
            public boolean isDone() {
                return atomicBoolean.get();
            }

            @Override
            public void setCompletable(Completable<T> completable) {
                this.completable = completable;
            }

            @Override
            public Completable<T> getCompletable() {
                return completable;
            }
        };


        Thread thread = new Thread(() -> {
            try {
                T t = callable.action();
                atomicReference.set(t);
                atomicBoolean.set(true);
                if (future.getCompletable() != null) {
                    future.getCompletable().complete(t);
                }
            } catch (Throwable e) {
                if (future.getCompletable() != null) {
                    future.getCompletable().exception(e);
                }
            }
        });
        thread.start();

        return future;
    }


    public static void main(String[] args) {
        Future<String> result = invoke(() -> {
            try {
                Thread.sleep(10000);
                return "I am finished.";
            } catch (InterruptedException e) {
                e.printStackTrace();
                return "Error.";
            }
        });


        result.setCompletable(new Completable<String>() {
            @Override
            public void complete(String s) {
                System.out.println("complete..." + s);
            }

            @Override
            public void exception(Throwable cause) {
                System.out.println("exception error");
                cause.printStackTrace();
            }
        });

        System.out.println("end....");
    }
}
```

输出

```
end....
complete...I am finished.
```

## CompletableFuture 构建异步应用

### 模拟异步操作内容

```java
import java.util.Random;

public class CompletableFutureSupport {

    static final Random random = new Random(System.currentTimeMillis());

    static double getActionBody() {
        try {
            System.out.println(Thread.currentThread().getName() + "：get（begin）");
            Thread.sleep(10000L);
            double v = random.nextDouble();
            System.out.println(Thread.currentThread().getName() + "：get（done）：" + v);
            return v;
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
    }

}
```

### 模拟1 

CompletableFuture阻塞等待

```java
import java.util.Optional;
import java.util.concurrent.CompletableFuture;

public class CompletableFutureInAction1 {
    public static void main(String[] args) {
        CompletableFuture<Double> completableFuture = new CompletableFuture<>();
        new Thread(() -> {
            System.out.println("action:->" + Thread.currentThread().getName());
            double v = CompletableFutureSupport.getActionBody();
            completableFuture.complete(v);
        }).start();

        System.out.println("main.....1");

        completableFuture.whenComplete((v, t) -> {
            System.out.println("complete.....2");
            Optional.ofNullable(v).ifPresent(System.out::println);
            Optional.ofNullable(t).ifPresent(tt -> tt.printStackTrace());
        });
    }
}
```

输出

```
main.....1
action:->Thread-0
Thread-0：get（begin）
Thread-0：get（done）：0.30457757812323727
complete.....2
0.30457757812323727
```

### 模拟2 

CompletableFuture.supplyAsync 异步请求

```java
/**
     * main 线程不会等待whenComplete。
     * main结束的时候，completableFuture的Supplier 未执行完成，线程也会随main的结束而结束
     *
     * @throws ExecutionException
     * @throws InterruptedException
     */
    public static void main(String[] args) {
        CompletableFuture<Double> completableFuture = CompletableFuture.supplyAsync(CompletableFutureSupport::getActionBody);

        System.out.println(Thread.currentThread().getName() + "..........................1");

        completableFuture.whenComplete((v, t) -> {
            System.out.println(Thread.currentThread().getName() + "：complete_action（begin）");
            Optional.ofNullable(v).ifPresent(System.out::println);
            Optional.ofNullable(t).ifPresent(tt -> tt.printStackTrace());
            System.out.println(Thread.currentThread().getName() + "：complete_action（end）");
        });
    }
```

输出

```java
main..........................1
ForkJoinPool.commonPool-worker-9：get（begin）
```

### 模拟3

改写模拟2，不使用阻塞的方式

```java
public static void main(String[] args) {
        ExecutorService executorService = Executors.newFixedThreadPool(2, (r) -> {
            Thread thread = new Thread(r);
            // 不以守护进程的方式运行
            thread.setDaemon(false);
            return thread;
        });
        CompletableFuture<Double> completableFuture = CompletableFuture.supplyAsync(CompletableFutureSupport::getActionBody, executorService);

        // main
        System.out.println(Thread.currentThread().getName() + "..........................1");


        completableFuture.whenComplete((v, t) -> {
            System.out.println(Thread.currentThread().getName() + "：complete_action（begin）");
            Optional.ofNullable(v).ifPresent(System.out::println);
            Optional.ofNullable(t).ifPresent(tt -> tt.printStackTrace());
            System.out.println(Thread.currentThread().getName() + "：complete_action（end）");
        });

        // main
        System.out.println(Thread.currentThread().getName() + "..........................2");

        // 需要显示关闭线程
        // 执行先前提交的任务后，启动有序关闭。如果已经关闭，调用没有额外的影响
        executorService.shutdown();

        // main
        System.out.println(Thread.currentThread().getName() + "..........................3");
    }
```

输出

```
main..........................1
Thread-0：get（begin）
main..........................2
main..........................3
Thread-0：get（done）：0.9293436815770424
Thread-0：complete_action（begin）
0.9293436815770424
Thread-0：complete_action（end）
```

## CompletableFuture常用API

| 方法               | API                                                          |
| ------------------ | ------------------------------------------------------------ |
| supplyAsyn         | CompletableFuture<U>supplyAsync(Supplier<U>supplier);<br>CompletableFuture<U>supplyAsync(Supplier<U>supplier,Executorexecutor); |
| then**Apply**      | CompletableFuture<U>thenApply(**Function**<?superT,?extendsU>fn); |
| **handle**         | CompletableFuture<U>handle(**BiFunction**<?superT,Throwable,?extendsU>fn) |
| then**Run**        | CompletableFuture<Void>thenRun(**Runnable**action)           |
| then**Accept**     | CompletableFuture<Void>thenAccept(**Consumer**<?superT>action) |
| thenCompose        | CompletableFuture<U>thenCompose(Function<?superT,?extendsCompletionStage<U>>fn)<br>继续执行下一个CompletableFuture |
| then**Combine**    | CompletableFuture<V>thenCombine(CompletionStage<?extendsU>other,**BiFunction**<?superT,?superU,?extendsV>fn)<br/>两个CompletableFuture的返回结果经过**BiFunction**处理，进入后续操作。 |
| then**Accept**Both | CompletionStage<Void>thenAcceptBoth(CompletionStage<?extendsU>other,BiConsumer<?superT,?superU>action)<br/>两个CompletableFuture的返回结果经过**BiConsumer**处理，进入后续操作。 |
| **run**AfterBoth   | CompletableFuture<Void>runAfterBoth(CompletionStage<?>other,**Runnable**action)<br>两个CompletableFuture的返回结果经过**Runnable**处理，进入后续操作。 |
| **apply**ToEither  | CompletionStage<U>applyToEither(CompletionStage<?extendsT>other,**Function**<?superT,U>fn)<br/>两个CompletionStage任意一个执行完成即可执行其**Function**。 |
| **accept**Either   | CompletableFuture<Void>acceptEither(CompletionStage<?extendsT>other,**Consumer**<?superT>action)<br/>两个CompletionStage任意一个执行完成即可执行其**Consumer** |
| **run**AfterEither | CompletableFuture<Void>runAfterEither(CompletionStage<?>other,Runnableaction)<br/>两个CompletableFuture任意一个执行完成即可执行其**Runnable** |
| **allOf**          | public**static**CompletableFuture<Void>allOf(CompletableFuture<?>...cfs)<br/>等待所有CompletableFuture执行完成之后，执行后续操作。 |
| **anyOf**          | publicstaticCompletableFuture<Object>anyOf(CompletableFuture<?>...cfs)<br/>等待任意一个CompletableFuture执行完成之后，执行后续操作 |

### supplyAsyn

```java
API：
CompletableFuture<U> supplyAsync(Supplier<U> supplier);
举例：
CompletableFuture.supplyAsync(() -> 1);

API：
CompletableFuture<U> supplyAsync(Supplier<U> supplier,  Executor executor);
举例：
ExecutorService executor = Executors.newFixedThreadPool(2, (r) -> {
	Thread thread = new Thread(r); 
	thread.setDaemon(false);
	return thread;
});
CompletableFuture.supplyAsync(() -> 1, executor);
```

### thenApply 

```java
API:
CompletableFuture<U> thenApply(Function<? super T,? extends U> fn);

举例：
CompletableFuture.supplyAsync(() -> 1)
        .thenApply(v -> Integer.sum(v, 10))
        .whenComplete((v, t) -> System.out.println(v));
```

### handle

可以对抛出的异常进行处理

```java
API:
CompletableFuture<U> handle(BiFunction<? super T, Throwable, ? extends U> fn)

举例：
CompletableFuture.supplyAsync(() -> 1)
        .handle((v, t) -> Integer.sum(v, 10))
        .whenComplete((v, t) -> System.out.println(v));
```

### thenRun

执行完成之后的其他操作。无入参。

```java
API:
CompletableFuture<Void> thenRun(Runnable action)

举例：
CompletableFuture.supplyAsync(() -> 1)
        .whenComplete((v, t) -> System.out.println(v))
        .thenRun(() -> System.out.println());
```

### thenAccept

返回结果V的后续消费

```java
API:
CompletableFuture<Void> thenAccept(Consumer<? super T> action)

举例：
CompletableFuture.supplyAsync(() -> 1)
        .whenComplete((v, t) -> System.out.println(v))
        .thenAccept((v) -> System.out.println(v));
```

### thenCompose

继续执行下一个CompletableFuture

```java
API:
CompletableFuture<U> thenCompose(Function<? super T, ? extends CompletionStage<U>> fn)

其中：Function<? super T, ? extends CompletionStage<U>> fn是另外一个CompletableFuture。

举例：
CompletableFuture.supplyAsync(() -> 1)
        .thenCompose(v -> CompletableFuture.supplyAsync(() -> v + 10))
        .whenComplete((v, t) -> System.out.println(v));
```

### thenCombine

两个CompletableFuture的返回结果经过BiFunction处理，进入后续操作。

```java
API:
CompletableFuture<V> thenCombine(CompletionStage<? extends U> other,
        BiFunction<? super T,? super U,? extends V> fn) 

举例：
CompletableFuture.supplyAsync(() -> 1)
        .thenCombine(CompletableFuture.supplyAsync(() -> 2), (v1, v2) -> v1 + v2)
        .whenComplete((v, t) -> System.out.println(v));
```

### thenAcceptBoth

两个CompletableFuture的返回结果经过BiConsumer处理，进入后续消费。

```java
API:
CompletionStage<Void> thenAcceptBoth(CompletionStage<? extends U> other,
        BiConsumer<? super T, ? super U> action) 

举例：
CompletableFuture.supplyAsync(() -> 1)
        .thenAcceptBoth(CompletableFuture.supplyAsync(() -> 2), (v1, v2) -> {
            System.out.println(v1);
            System.out.println(v2);
        }).whenComplete((v, t) -> System.out.println(v));

输出：
1
2
null
```

### runAfterBoth

```java
API:
CompletableFuture<Void> runAfterBoth(CompletionStage<?> other, Runnable action)
```

### applyToEither

两个CompletionStage 任意一个执行完成即可执行其Function。

```java
API:
CompletionStage<U> applyToEither(CompletionStage<? extends T> other,
         Function<? super T, U> fn)

举例：
CompletableFuture.supplyAsync(() -> 1)
        .applyToEither(CompletableFuture.supplyAsync(() -> 2), v -> v + 10)
        .whenComplete((v, t) -> System.out.println(v));
输出：
11
```

### acceptEither

两个CompletionStage 任意一个执行完成即可执行其Consumer

```java
API:
CompletableFuture<Void> acceptEither(
        CompletionStage<? extends T> other, Consumer<? super T> action)

举例：
CompletableFuture.supplyAsync(() -> 1)
        .acceptEither(CompletableFuture.supplyAsync(() -> 2), v -> System.out.println(v))
        .whenComplete((v, t) -> System.out.println(v));
输出：
1
null
```

### runAfterEither

两个CompletionStage 任意一个执行完成即可执行其Runnable。

```java
CompletableFuture<Void> runAfterEither(CompletionStage<?> other,
                                                  Runnable action)
```

### allOf

等待所有CompletableFuture执行完成之后，执行后续的操作

```java
API:
public static CompletableFuture<Void> allOf(CompletableFuture<?>... cfs)

举例：
List<CompletableFuture<Double>> collect = Arrays.asList(1, 2, 3, 4)
    .stream()
    .map(i -> CompletableFuture.supplyAsync(() -> 1)
    .collect(toList());

CompletableFuture.allOf(collect.toArray(new CompletableFuture[collect.size()]))
    .thenRun(() -> System.out.println("done"));
```

### anyOf

等待任意一个CompletableFuture执行完成之后，执行后续的操作

```java
API：
public static CompletableFuture<Object> anyOf(CompletableFuture<?>... cfs)

举例：
List<CompletableFuture<Double>> collect = Arrays.asList(1, 2, 3, 4)
    .stream()
    .map(i -> CompletableFuture.supplyAsync(() -> 1)
    .collect(toList());

CompletableFuture.anyOf(collect.toArray(new CompletableFuture[collect.size()]))
    .thenRun(() -> System.out.println("done"));
```

# 新的日期和时间API

## 重新设计原因

1. java.util.Date 不够直观。
   Date date = new Date(114, 2, 18);
   它的打印输出效果为： Tue Mar 18 00:00:00 CET 2014
1. 线程不安全

```java
public static void main(String[] args) throws ParseException, InterruptedException {
        SimpleDateFormat sdf = new SimpleDateFormat("yyyyMMdd");
        for (int i = 0; i < 30; i++) {
            new Thread(() -> {
                for (int x = 0; x < 100; x++) {
                    Date parseDate = null;
                    try {
                        parseDate = sdf.parse("20160505");
                    } catch (ParseException e) {
                        e.printStackTrace();
                    }
                    System.out.println(parseDate);
                }
            }).start();
        }
    }

输出
Thu May 05 00:00:00 CST 2016
Thu May 05 00:00:00 CST 2016
...
Exception in thread "Thread-19" java.lang.NumberFormatException: For input string: ""
	at java.base/java.lang.NumberFormatException.forInputString(NumberFormatException.java:65)
	at java.base/java.lang.Long.parseLong(Long.java:702)
	at java.base/java.lang.Long.parseLong(Long.java:817)
	at java.base/java.text.DigitList.getLong(DigitList.java:195)
	at java.base/java.text.DecimalFormat.parse(DecimalFormat.java:2093)
	at java.base/java.text.SimpleDateFormat.subParse(SimpleDateFormat.java:1913)
	at java.base/java.text.SimpleDateFormat.parse(SimpleDateFormat.java:1529)
	at java.base/java.text.DateFormat.parse(DateFormat.java:386)
	at com.hots.TimeTest.lambda$main$0(TimeTest.java:15)
	at java.base/java.lang.Thread.run(Thread.java:844)
Exception in thread "Thread-17" java.lang.NumberFormatException: For input string: ""
	at java.base/java.lang.NumberFormatException.forInputString(NumberFormatException.java:65)
	at java.base/java.lang.Long.parseLong(Long.java:702)
	at java.base/java.lang.Long.parseLong(Long.java:817)
	at java.base/java.text.DigitList.getLong(DigitList.java:195)
	at java.base/java.text.DecimalFormat.parse(DecimalFormat.java:2093)
	at java.base/java.text.SimpleDateFormat.subParse(SimpleDateFormat.java:1913)
	at java.base/java.text.SimpleDateFormat.parse(SimpleDateFormat.java:1529)
	at java.base/java.text.DateFormat.parse(DateFormat.java:386)
	at com.hots.TimeTest.lambda$main$0(TimeTest.java:15)
	at java.base/java.lang.Thread.run(Thread.java:844)
Thu May 05 00:00:00 CST 2016
```

## java.time.LocalDate  此类是不可吧变更且线程安全的。并且和时间概念区分开。

```java
LocalDate localDate = LocalDate.of(2019,1,7);
System.out.println(localDate.getMonth());
System.out.println(localDate.getMonthValue());
System.out.println(localDate.lengthOfMonth());
```

| 方法                                         | 说明                                   |
| -------------------------------------------- | -------------------------------------- |
| LocalDate.now()                              | 获取当前日期                           |
| LocalDate.of(intyear,intmonth,intdayOfMonth) | 根据参数设置日期，参数分别为年，月，日 |
| localDate.getDayOfMonth()                    | 获取当前日期是所在月的第几天           |
| localDate.getDayOfWeek()                     | 获取当前日期是星期几（星期的英文全称） |
| localDate.getDayOfYear()                     | 获取当前日期是所在年的第几天           |
| localDate.getMonth()                         | 获取当前日期所在月份（月份的英文全称） |
| localDate.getMonthValue()                    | 获取当前日期所在月份的数值             |
| localDate.lengthOfMonth()                    | 获取当前日期所在月份有多少天           |
| localDate.lengthOfYear()                     | 获取当前日期所在年有多少天             |
| localDate.**isLeapYear**()                   | 获取当前日期所在年是否是闰年           |
| localDate.**with**DayOfMonth(intdayOfMonth)  | 将参数中的"日"替换localDate中的"日"    |
| localDate.**with**DayOfYear(intdayOfYear)    | 将参数中的天数替换localDate中的天数    |
| localDate.**with**Month(intmonth)            | 将参数中的"月"替换localDate中的"月"    |
| localDate.**with**Year(intyear)              | 将参数中的"年"替换localDate中的"年"    |
| localDate.minusDays(longdays)                | 将当前日期减一天                       |
| localDate.minusWeeks(longweeks)              | 将当前日期减一周                       |
| localDate.minusMonths(longmonths)            | 将当前日期减一月                       |
| localDate.minusYears(longyears)              | 将当前日期减一年                       |
| localDate.plusDays(longdays)                 | 将当前日期加一天                       |

## java.time.LocalTime

| 方法                                      | 说明                                    |
| ----------------------------------------- | --------------------------------------- |
| LocalTime.now()                           | 获取当前时间                            |
| LocalTime.of(inthour,intminute)           | 根据参数设置时间，参数分别为时，分      |
| LocalTime.of(inthour,intminute,intsecond) | 根据参数设置时间，参数分别为时，分，秒  |
| localTime.getHour()                       | 获取当前时间的小时数                    |
| localTime.getMinute()                     | 获取当前时间的分钟数                    |
| localTime.getSecond()                     | 获取当前时间的秒数                      |
| localTime.withHour(inthour)               | 将参数中的"小时"替换localTime中的"小时" |
| localTime.withMinute(intminute)           | 将参数中的"分钟"替换localTime中的"分钟" |
| localTime.withSecond(intsecond)           | 将参数中的"秒"替换localTime中的"秒"     |
| localTime.minusHours(longhours)           | 将当前时间减一小时                      |
| localTime.minusMinutes(longminutes)       | 将当前时间减一分钟                      |
| localTime.minusSeconds(longseconds)       | 将当前时间减一秒                        |
| localTime.plusHours(longhours)            | 将当前时间加一小时                      |
| localTime.plusMinutes(longminutes)        | 将当前时间加一分钟                      |
| localTime.plusSeconds(longseconds)        | 将当前时间加一秒                        |

## java.time.LocalDateTime

```java
LocalDate localDate = LocalDate.now();
LocalTime time = LocalTime.now();

LocalDateTime localDateTime = LocalDateTime.of(localDate, time);
System.out.println(localDateTime.toString());
LocalDateTime now = LocalDateTime.now();
System.out.println(now);
```

## java.time.Instant 机器时间

```java
Instant start = Instant.now();
Thread.sleep(1000L);
Instant end = Instant.now();
Duration duration = Duration.between(start, end);
System.out.println(duration.toMillis());
System.out.println(duration.toNanos());
```

## java.time.Duration

```java
LocalTime time = LocalTime.now();
LocalTime beforeTime = time.minusHours(1);
Duration duration = Duration.between(time, beforeTime);
System.out.println(duration.toHours());
```

## java.time.Period 

```java
Period period = Period.between(LocalDate.of(2014, 1, 10), LocalDate.of(2016, 1, 10));
System.out.println(period.getMonths());
System.out.println(period.getDays());
System.out.println(period.getYears());
```

## java.time.temporal.TemporalAdjusters 时间调节器

```java
LocalDateTime now = LocalDateTime.now();
//获取当月第一天
System.out.println("当月第一天："+now.with(TemporalAdjusters.firstDayOfMonth()));
//获取下月第一天
System.out.println("下月第一天："+now.with(TemporalAdjusters.firstDayOfNextMonth()));
//获取明年第一天
System.out.println("明年第一天："+now.with(TemporalAdjusters.firstDayOfNextYear()));
//获取本年第一天
System.out.println("本年第一天："+now.with(TemporalAdjusters.firstDayOfYear()));
//获取当月最后一天
System.out.println("当月最后一天："+now.with(TemporalAdjusters.lastDayOfMonth()));
//获取本年最后一天
System.out.println("本年最后一天："+now.with(TemporalAdjusters.lastDayOfYear()));
//获取当月第三周星期五
System.out.println("当月第三周星期五："+now.with(TemporalAdjusters.dayOfWeekInMonth(3, DayOfWeek.FRIDAY)));
//获取上周一
System.out.println("上周一："+now.with(TemporalAdjusters.previous(DayOfWeek.MONDAY)));
//获取下周日
System.out.println("下周日："+now.with(TemporalAdjusters.next(DayOfWeek.SUNDAY)));
```