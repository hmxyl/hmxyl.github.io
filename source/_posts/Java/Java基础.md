# abstract class和interface的区别

|          | Abstract class                                               | Interface                                                    |
| -------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 实例化   | 不能                                                         | 不能                                                         |
| 类       | 一种继承关系，一个类只能使用一次继承关系。<br/>可以通过继承多个接口实现多重继承 | 一个类可以实现多个interface                                  |
| 数据成员 | 可有自己的                                                   | 静态的不能被修改即必须是static final，一般不在此定义         |
| 方法     | 可以私有的，非abstract方法，必须实现                         | 不可有私有的，默认是public abstract 类型                     |
| 变量     | 可有私有的，默认是friendly 型，其值可以在子类中重新定义，也可以重新赋值 | 不可有私有的，默认是public static final 型，且必须给其初值，实现类中不能重新定义，不能改变其值。 |
| 设计理念 | 表示的是“is-a”关系                                           | 表示的是“like-a”关系                                         |
| 实现     | 需要继承，要用 extends                                       | 要用 implements                                              |

​		`abstract class `和 `interface`在Java语言中都是用来进行抽象类（本文中的抽象类并非从abstract class翻译而来，它表示的是一个抽象体，而abstract class为Java语言中用于定义抽象类的一种方法）定义的

## 1. 定义

- 定义：abstract class

​		声明方法的存在而不去实现它的类被叫做抽象类（abstract class），它用于要创建一个体现某些基本行为的类，并为该类声明方法，但不能在该类中实现该类的情况。不能创建abstract 类的实例。然而可以创建一个变量，其类型是一个抽象类，并让它指向具体子类的一个实例。不能有抽象构造函数或抽象静态方法。

​		abstract 类的子类为它们父类中的所有抽象方法提供实现，否则它们也是抽象类。取而代之，在子类中实现该方法。知道其行为的其它类可以在类中实现这些方法。

- 定义：interface

​		接口（interface）是抽象类的变体。在接口中，所有方法都是抽象的。多继承性可通过实现 这样的接口而获得。接口中的所有方法都是抽象的，没有一个有程序体。

​		接口只可以定义`static final`成员变量。接口的实现与子类相似，除了该实现类不能从接口定义中继承行为。当类实现特殊接口时，它定义（即将程序体给予）所有这种接口的方法。 然后，它可以在实现了该接口的类的任何对象上调用接口的方法。由于有抽象类，它允许使用接口名作为引用变量的类型。通常的动态联编将生效。引用可以转换到 接口类型或从接口类型转换，`instanceof `运算符可以用来决定某对象的类是否实现了接口。



​		接口可以继承接口。抽象类可以实现(implements)接口，抽象类是可以继承实体类，但前提是实体类必须有明确的构造函数。

​		接口更关注“能实现什么功能”，而不管“怎么实现的”。

## 2. 相同点

  A. 两者都是抽象类，都不能实例化。
  B. interface实现类及abstrct class的子类都必须要实现已经声明的抽象方法。

## 3. 不同点

- `interface`需要实现，要用`implements`，而`abstract class`需要继承，要用`extends`。

- 一个类可以实现多个`interfac`e，但一个类只能继承一个`abstract class`。

- `interface`强调特定功能的实现，而`abstract class`强调所属关系

- 尽管interface实现类及`abstrct class`的子类都必须要实现相应的抽象方法，但实现的形式不同。

  `interface`中的每一个方法都是抽象方法，都只是声明的 (declaration, 没有方法体)，实现类必须要实现。

  `abstract class`的子类可以有选择地实现。 这个选择有两点含义：

  1.  `abastract class`中并非所有的方法都是抽象的。**只有那些冠有`abstract`的方法才是抽象的，子类必须实现**。那些没有`abstract`的方法，在`abstrct class`中必须定义方法体。
  2.  `abstract class`的子类在继承它时，对非抽象方法既可以直接继承，也可以覆盖；而对抽象方法，可以选择实现，也可以通过再次声明其方法为抽象的方式，无需实现，留给其子类来实现，但此类必须也声明为抽象类。既是抽象类，当然也不能实例化。

- `abstract class`是`interface`与`class`中介

  ​		`interface`是完全抽象的，只能声明方法，而且只能声明pulic的方法，不能声明`private`及`protected`的方法，不能定义方法体，也不能声明实例变量。然而，`interface`却可以声明常量变量，并且在JDK中不难找出这种例子。但将常量变量放在`interface`中违背了其作为接 口的作用而存在的宗旨，也混淆了`interface`与类的不同价值。如果的确需要，可以将其放在相应的`abstract class`或`class`中。
  ​		`abstract class`在`interface`及`class`中起到了承上启下的作用。一方面，`abstract class`是抽象的，可以声明抽象方法，以规范子类必须实现的功能；另一方面，它又可以定义缺省的方法体，供子类直接使用或覆盖。另外，它还可以定义自己 的实例变量，以供子类通过继承来使用。

## 4.  应用场合

- `interface`

  1. 类与类之前需要特定的接口进行协调，而不在乎其如何实现。
  2. 作为能够实现特定功能的标识存在，也可以是什么接口方法都没有的纯粹标识。
  3. 需要将一组类视为单一的类，而调用者只通过接口来与这组类发生联系。
  4. 需要实现特定的多项功能，而这些功能之间可能完全没有任何联系。

- `abstract class`
  ==在既需要统一的接口，又需要实例变量或缺省的方法的情况下，就可以使用它==

  最常见的有：

  1. 定义了一组接口，但又不想强迫每个实现类都必须实现所有的接口。可以用`abstract class`定义一组方法体，甚至可以是空方法体，然后由子类选择自己所感兴趣的方法来覆盖。
  2. 某些场合下，只靠纯粹的接口不能满足类与类之间的协调，还必需类中表示状态的变量来区别不同的关系。`abstract`的中介作用可以很好地满足这一点。
  3. 规范了一组相互协调的方法，其中一些方法是共同的，与状态无关的，可以共享的，无需子类分别实现；而另一些方法却需要各个子类根据自己特定的状态来实现特定的功能。

# Java命名规范

## 泛型类

在书写泛型类时，通常做以下的约定：

- E表示Element，通常用在集合中；
- ID用于表示对象的唯一标识符类型
- T表示Type(类型)，通常指代类；
- K表示Key(键),通常用于Map中；
- V表示Value(值),通常用于Map中，与K结对出现；
- N表示Number,通常用于表示数值类型；
- ？表示不确定的Java类型；
- X用于表示异常；
- U,S表示任意的类型。

##  速记Java开发中的各种O

通过一张表和图快速对Java中的`BO`,`DTO`,`DAO`,`PO`,`POJO`,`VO`之间的含义，区别以及联系进行梳理。

| 名称 | 使用范围                                       | 解释说明                                                     |
| ---- | ---------------------------------------------- | ------------------------------------------------------------ |
| BO   | 用于Service,Manager,Business等业务相关类的命名 | Business Object业务处理对象，主要作用是把业务逻辑封装成一个对象。 |
| DTO  | 经过加工后的PO对象，其内部属性可能增加或减少   | Data Transfer  Object数据传输对象，主要用于远程调用等需要大量传输数据的地方，例如，可以将一个或多个PO类的部分或全部属性封装为DTO进行传输 |
| DAO  | 用于对数据库进行读写操作的类进行命名           | Data Access  Object数据访问对象，主要用来封装对数据库的访问，通过DAO可以将POJO持久化为PO，也可以利用PO封装出VO和DTO |
| PO   | Bean,Entity等类的命名                          | Persistant  Object持久化对象，数据库表中的数据在Java对象中的映射状态，可以简单的理解为一个PO对象即为数据库表中的一条记录 |
| POJO | POJO是DO/DTO/BO/VO的统称                       | Plain Ordinary Java Object  简单Java对象，它是一个简单的普通Java对象，禁止将类命名为XxxxPOJO |
| VO   | 通常是视图控制层和模板引擎之间传递的数据对象   | Value Object  值对象，主要用于视图层，视图控制器将视图层所需的属性封装成一个对象，然后用一个VO对象在视图控制器和视图之间进行数据传输。 |
| AO   | 应用层对象                                     | Application Object，在Web层与Service层之间抽象的复用对象模型，很少用。 |

下面将通过一张图来理解上述几种O之间相互转换的关系：

![微信图片_20211213004837](D:\0_Notes\Hexo\hmxyl\source\_images\eae992c8-ce71-47df-ae94-eca9bc91dac0.jpg) 



# Java中printf的用法总结

printf支持以下格式： 

| 格式 | 说明         | 补充说明                                                     |
| ---- | ------------ | ------------------------------------------------------------ |
| %c   | 单个字符     |                                                              |
| %d   | 十进制整数   | `%d`：按整型数据的实际长度输出<br/><br/>`%md`：m为指定的输出字段的宽度。如果数据的位数小于m，则左端补以空格，若大于m，则按实际位数输出<br/><br/>`%ld`：输出长整型数据 |
| %f   | 十进制浮点数 | `%f`不指定宽度，整数部分全部输出，并输出6位小数<br/>`%m.nf`输出共占m列，其中有n位小数，如数值宽度小于m左端补空格<br/>`%-m.nf`输出共占n列，其中有n位小数，如数值宽度小于m右端补空格 |
| %o   | 八进制数     | `%o、%#o`： “#”号会将八进制符号“0X”显示出来。大写“X”，则会显示大写英文字符 |
| %s   | 字符串       | `%s`例如:printf("%s", "CHINA") 输出"CHINA"字符串（不包括双引号）<br/><br/>`%ms`输出的字符串占m列，如字符串本身长度大于m，则突破获m的限制,将字符串全部输出。若串长小于m，则左补空格<br/><br/>`%-ms`如果串长小于m，则在m列范围内，字符串向左靠，右补空格<br/><br/>`%m.ns`输出占m列，但只取字符串中左端n个字符。这n个字符输出在m列的右侧，左补空格<br/><br/>`%-m.ns`其中m、n含义同上，n个字符输出在m列范围的左侧，右补空格。如果n>m，则自动取n值，即保证n个字符正常输出 |
| %x   | 十六进制数   | `%x、%X、%#x、%#X`： “#”号会将十六进制符号“0X”显示出来。大写“X”，则会显示大写英文字符 |
| %%   | 输出百分号%  |                                                              |
|      |              |                                                              |

# 对象之间相同属性的赋值

参考：[Java 对象之间相同属性的赋值](https://blog.csdn.net/power0405hf/article/details/78521286)

## Java中clone()与new的区别

区别

（1）在java中clone()与new都能创建对象。
（2）clone()不会调用构造方法；new会调用构造方法。
（3）clone()能快速创建一个已有对象的副本，即创建对象并且将已有对象中所有属性值克隆；

new只能在JVM中申请一个空的内存区域，对象的属性值要通过构造方法赋值。

注意：
（1）使用clone()类必须实现**java.lang.Cloneable**接口并**重写Object类的clone()方法**，如果没有实现Cloneable()接口将会抛出CloneNotSupportedException异常。（此类实现java.lang.Cloneable接口，指示Object.clone()方法可以合法的对该类实例进行按字段复制。）
（2）默认的Object.clone()方法是浅拷贝，创建好对象的副本然后通过“赋值”拷贝内容，如果类包含引用类型变量，那么原始对象和克隆对象的引用将指向相同的引用内容。

面试题：什么是浅拷贝？什么是深拷贝？

- “浅拷贝”：默认的Object.clone()方法,对于引用类型成员变量拷贝只是拷贝“值”即地址，没有在堆中开辟新的内存空间。
- “深拷贝”：重写clone()方法，对于引用类型成员变量，重新在堆中开辟新的内存空间。



## BeanUtils.copyProperties：可以进行类型转换

```java
import org.springframework.beans.BeanUtils;
User src = new User(); 
User dest = new User(); 
BeanUtils.copyProperties(dest, src);
```

## PropertyUtils.copyProperties：不会进行类型转换

## 使用Dozer

在pom.xml中增加依赖

```xml
<dependency>
  <groupId>net.sf.dozer</groupId>
  <artifactId>dozer</artifactId>
  <version>5.5.1</version>
</dependency>
```

Spring集成dozer

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE beans PUBLIC "-//SPRING//DTD BEAN//EN" "http://www.springframework.org/dtd/spring-beans.dtd">
<beans>
    <bean id="baseMapper" class="org.dozer.spring.DozerBeanMapperFactoryBean">
        <property name="mappingFiles">
            <list>
                <value>classpath:mapping/dozer-mapping.xml</value>
            </list>
        </property>
    </bean>
</beans>
```

使用baseMapper进行Bean的转换

```java
@Autowired 
private Mapper baseMapper; 
private UserVO doToVo(UserDO userDO){     
  if(userDO == null) return null;     
  UserVO vo = baseMapper.map(userDO, UserVO.getClass());     
  if(userDO.getCompanyId != null) getCompany(vo);     
  return vo; 
}
```

​		通过以上的代码加配置，我们就实现了从DO转换到VO的部分操作，之所以说是部分操作，是因为我们在dozer-mapping.xml并没有做多余的配置，只是使用dozer将DO中和VO中共有的属性转换了过来。对于其他的类型不同或者名称不同等的转换可以参考官网例子通过设置dozer-mapping.xml文件来实现。
 	上面还有一个getCompany()没有实现。这个方法其实就是通过companyId查询出company实体，然后在赋值给UserVO中的company属性。

# 枚举类型(enum) 详解7种常见的用法

参考：[Java 枚举(enum) 详解7种常见的用法](http://blog.csdn.net/qq_27093465/article/details/52180865)

JDK1.5引入了枚举类型

## 用法一：常量

在JDK1.5 之前，我们定义常量都是： public static fianl.... 。现在好了，有了枚举，可以把相关的常量分组到一个枚举类型里，而且枚举提供了比常量更多的方法。 

```java
public enum Color {  
	  RED, GREEN, BLANK, YELLOW  
} 
```

## 用法二：switch

JDK1.6之前的switch语句只支持int,char,enum类型，使用枚举，能让我们的代码可读性更强。 

```java
enum Signal {  
    GREEN, YELLOW, RED  
}  
public class TrafficLight {  
    Signal color = Signal.RED;  
    public void change() {  
        switch (color) {  
        case RED:  
            color = Signal.GREEN;  
            break;  
        case YELLOW:  
            color = Signal.RED;  
            break;  
        case GREEN:  
            color = Signal.YELLOW;  
            break;  
        }  
    }  
} 
```

## 用法三：向枚举中添加新方法

如果打算自定义自己的方法，那么必须在enum实例序列的最后添加一个分号。而且 Java 要求必须先定义 enum 实例。 

```java
public enum Color {  
    RED("红色", 1), GREEN("绿色", 2), BLANK("白色", 3), YELLO("黄色", 4);  
    // 成员变量  
    private String name;  
    private int index;  
    // 构造方法  
    private Color(String name, int index) {  
        this.name = name;  
        this.index = index;  
	}  
	// 普通方法  
	public static String getName(int index) {  
		for (Color c : Color.values()) {  
			if (c.getIndex() == index) {  
				return c.name;  
			}  
		}  
		return null;  
	}  
	// get set 方法  
	public String getName() {  
		return name;  
	}  
	public void setName(String name) {  
		this.name = name;  
	}  
	public int getIndex() {  
		return index;  
	}  
	public void setIndex(int index) {  
		this.index = index;  
	}  
}  
```



## 用法四：覆盖枚举的方法

下面给出一个toString()方法覆盖的例子。 

```java
public enum Color {  
    RED("红色", 1), GREEN("绿色", 2), BLANK("白色", 3), YELLO("黄色", 4);  
    // 成员变量  
    private String name;  
    private int index;  
    // 构造方法  
    private Color(String name, int index) {  
        this.name = name;  
        this.index = index;  
	}  
	//覆盖方法  
	@Override  
	public String toString() {  
		return this.index+"_"+this.name;  
	}  
}  

```

## 用法五：实现接口

所有的枚举都继承自java.lang.Enum类。由于Java 不支持多继承，所以枚举对象不能再继承其他类。 

```java
public interface Behaviour {  
    void print();  
    String getInfo();  
}  
public enum Color implements Behaviour{  
    RED("红色", 1), GREEN("绿色", 2), BLANK("白色", 3), YELLO("黄色", 4);  
    // 成员变量  
    private String name;  
    private int index;  
	// 构造方法  
	private Color(String name, int index) {  
		this.name = name;  
		this.index = index;  
	}  
	//接口方法  
	@Override  
	public String getInfo() {  
		return this.name;  
	}  
	//接口方法  
	@Override  
	public void print() {  
		System.out.println(this.index+":"+this.name);  
	}  
}  
```

## 用法六：使用接口组织枚举

```java
public interface Food {  
    enum Coffee implements Food{  
        BLACK_COFFEE,DECAF_COFFEE,LATTE,CAPPUCCINO  
    }  
    enum Dessert implements Food{  
        FRUIT, CAKE, GELATO  
    }  
} 
```

## 用法七：关于枚举集合的使用

java.util.EnumSet和java.util.EnumMap是两个枚举集合。EnumSet保证集合中的元素不重复；EnumMap中的 key是enum类型，而value则可以是任意类型。关于这个两个集合的使用就不在这里赘述，可以参考JDK文档。

关于枚举的实现细节和原理请参考：

参考资料：《Thinking In Java》第四版

http://softbeta.iteye.com/blog/1185573

# 系统方法

## System.getProperty()

| code                              | 说明                            |
| --------------------------------- | ------------------------------- |
| java.versionJava                  | 运行时环境版本                  |
| java.vendorJava                   | 运行时环境供应商                |
| java.vendor.urlJava               | 供应商的URL                     |
| java.homeJava                     | 安装目录                        |
| java.vm.specification.versionJava | 虚拟机规范版本                  |
| java.vm.specification.vendorJava  | 虚拟机规范供应商                |
| java.vm.specification.nameJava    | 虚拟机规范名称                  |
| java.vm.versionJava               | 虚拟机实现版本                  |
| java.vm.vendorJava                | 虚拟机实现供应商                |
| java.vm.nameJava                  | 虚拟机实现名称                  |
| java.specification.versionJava    | 运行时环境规范版本              |
| java.specification.vendorJava     | 运行时环境规范供应商            |
| java.specification.nameJava       | 运行时环境规范名称              |
| java.class.versionJava            | 类格式版本号                    |
| java.class.pathJava               | 类路径                          |
| java.library.path                 | 加载库时搜索的路径列表          |
| java.io.tmpdir                    | 默认的临时文件路径              |
| java.compiler                     | 要使用的JIT编译器的名称         |
| java.ext.dirs                     | 一个或多个扩展目录的路径        |
| os.name                           | 操作系统的名称                  |
| os.arch                           | 操作系统的架构                  |
| os.version                        | 操作系统的版本                  |
| file.separator                    | 文件分隔符（在UNIX系统中是“/”） |
| path.separator                    | 路径分隔符（在UNIX系统中是“:”） |
| line.separator                    | 行分隔符（在UNIX系统中是“/n”）  |
| user.name                         | 用户的账户名称                  |
| user.home                         | 用户的主目录                    |
| user.dir                          | 用户的当前工作目录              |