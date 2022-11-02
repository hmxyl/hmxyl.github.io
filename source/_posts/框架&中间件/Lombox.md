---
title:  Lombox日常使用记录
date: 2022-10-30 15:39:23
tags: 
  -  Lombox
categories: 
  - [框架&中间件,  Lombox]
description: ' Logback日常使用记录'
---



# 安装插件&项目引入

## Idea里需要安装lombok插件

![78761c9df7b21539df3352c198d274cf.png](D:\0_Notes\Hexo\hmxyl\source\_images\clip_image002-16383492487521.png)  

## pom.xml中引用

```xml
<dependency>    
    <groupId>org.projectlombok</groupId>    
    <artifactId>lombok</artifactId>    
    <optional>true</optional>  
</dependency>
```

# Lombok工作原理分析

会发现在Lombok使用的过程中，只需要添加相应的注解，无需再为此写任何代码。自动生成的代码到底是如何产生的呢？核心之处就是对于注解的解析上。JDK5引入了注解的同时，也提供了两种解析方式。

1. 运行时解析

```
运行时能够解析的注解，必须将@Retention设置为RUNTIME，这样就可以通过反射拿到该注解。java.lang,reflect反射包中提供了一个接口AnnotatedElement，该接口定义了获取注解信息的几个方法，Class、Constructor、Field、Method、Package等都实现了该接口，对反射熟悉的朋友应该都会很熟悉这种解析方式。
```

2. 编译时解析
   编译时解析有两种机制，分别简单描述下：
   - Annotation Processing Tool
     apt自JDK5产生，JDK7已标记为过期，不推荐使用，JDK8中已彻底删除，自JDK6开始，可以使用Pluggable Annotation Processing API来替换它，apt被替换主要有2点原因：
     1. api都在com.sun.mirror非标准包下
     2. 没有集成到javac中，需要额外运行
   - Pluggable Annotation Processing API
     JSR 269自JDK6加入，作为apt的替代方案，它解决了apt的两个问题，javac在执行的时候会调用实现了该API的程序，这样我们就可以对编译器做一些增强

```
Lombok本质上就是一个实现了“JSR 269 API”的程序。在使用javac的过程中，它产生作用的具体流程如下：
1. javac对源代码进行分析，生成了一棵抽象语法树（AST）

2. 运行过程中调用实现了“JSR 269 API”的Lombok程序
此时Lombok就对第一步骤得到的AST进行处理，找到@Data注解所在类对应的语法树（AST），然后修改该语法树（AST），增加getter和setter方法定义的相应树节点

3. javac使用修改后的抽象语法树（AST）生成字节码文件，即给class增加新的节点（代码块）
```

# Lombok的优缺点

```
优点：
1. 能通过注解的形式自动生成构造器、getter/setter、equals、hashcode、toString等方法，提高了一定的开发效率
2. 让代码变得简洁，不用过多的去关注相应的方法
3. 属性做修改时，也简化了维护为这些属性所生成的getter/setter方法等
 
缺点：
1. 不支持多种参数构造器的重载
2. 虽然省去了手动创建getter/setter方法的麻烦，但大大降低了源代码的可读性和完整性，降低了阅读源代码的舒适度
```

# 注解

## @Slf4j

```
注解在类上；为类提供一个 属性名为log 的 log4j 日志对像
```

## @Data

```
@Data注解在类上，会为类的所有属性自动生成setter/getter、equals、canEqual、hashCode、toString方法，如为final属性，则不会为该属性生成setter方法。
```

官方实例如下

```java
import lombok.AccessLevel;
import lombok.Data;
import lombok.Setter;
import lombok.ToString;

@Data
public class DataExample {

  private final String name;

  @Setter(AccessLevel.PACKAGE)
  private int age;

  private double score;
  private String[] tags;

  @ToString(includeFieldNames = true)
  @Data(staticConstructor = "of")
  public static class Exercise<T> {

    private final String name;
    private final T value;
  }
}
```

如不使用Lombok，则实现如下

```java
import java.util.Arrays;

public class DataExample {
    private final String name;
    private int age;
    private double score;

    private String[] tags;

    public DataExample(String name) {
        this.name = name;
    }

    public String getName() {
        return this.name;
    }

    void setAge(int age) {
        this.age = age;
    }

    public int getAge() {
        return this.age;
    }

    public void setScore(double score) {
        this.score = score;
    }

    public double getScore() {
        return this.score;
    }

    public String[] getTags() {
        return this.tags;
    }

    public void setTags(String[] tags) {
        this.tags = tags;
    }

    @Override
    public String toString() {
        return "DataExample(" + this.getName() + ", " + this.getAge() + ", " + this.getScore() + ", " + Arrays.deepToString(this.getTags());
    }

    protected boolean canEqual(Object other) {
        return other instanceof DataExample;
    }

    @Override
    public boolean equals(Object o) {
        if (o == this) return true;
        if (!(o instanceof DataExample)) return false;
        DataExample other = (DataExample) o;
        if (!other.canEqual((Object) this)) return false;
        if (this.getName() == null ? other.getName() != null : !this.getName().equals(other.getName())) return false;
        if (this.getAge() != other.getAge()) return false;
        if (Double.compare(this.getScore(), other.getScore()) != 0) return false;
        if (!Arrays.deepEquals(this.getTags(), other.getTags())) return false;
        return true;
    }

    @Override
    public int hashCode() {
        final int PRIME = 59;
        int result = 1;
        final long temp1 = Double.doubleToLongBits(this.getScore());
        result = (this.getName() == null ? 43 : this.getName().hashCode());
        result = this.getAge();
        result = (int) (temp1 ^ (temp1 >>> 32));
        result = Arrays.deepHashCode(this.getTags());
        return result;
    }

    public static class Exercise<T> {
        private final String name;
        private final T value;

        private Exercise(String name, T value) {
            this.name = name;
            this.value = value;
        }

        public static <T> Exercise<T> of(String name, T value) {
            return new Exercise<T>(name, value);
        }

        public String getName() {
            return this.name;
        }

        public T getValue() {
            return this.value;
        }

        @Override
        public String toString() {
            return "Exercise(name=" + this.getName() + ", value=" + this.getValue() + ")";
        }

        protected boolean canEqual(Object other) {
            return other instanceof Exercise;
        }

        @Override
        public boolean equals(Object o) {
            if (o == this) return true;
            if (!(o instanceof Exercise)) return false;
            Exercise<?> other = (Exercise<?>) o;
            if (!other.canEqual((Object) this)) return false;
            if (this.getName() == null ? other.getValue() != null : !this.getName().equals(other.getName()))
                return false;
            if (this.getValue() == null ? other.getValue() != null : !this.getValue().equals(other.getValue()))
                return false;
            return true;
        }

        @Override
        public int hashCode() {
            final int PRIME = 59;
            int result = 1;
            result = (this.getName() == null ? 43 : this.getName().hashCode());
            result = (this.getValue() == null ? 43 : this.getValue().hashCode());
            return result;
        }
    }
}
```

## @Getter/@Setter

如果觉得``@Data``太过残暴（因为``@Data``集合了``@ToString``、``@EqualsAndHashCode``、``@Getter/@Setter``、``@RequiredArgsConstructor``的所有特性）`` ``不够精细，可以使用``@Getter/@Setter``注解，此注解在属性上，可以为相应的属性自动生成``Getter/Setter``方法，示例如下：

```java
import lombok.AccessLevel;
import lombok.Getter;
import lombok.Setter;

public class GetterSetterExample {

  @Getter
  @Setter
  private int age = 10;

  @Setter(AccessLevel.PROTECTED)
  private String name;

  @Override
  public String toString() {
    return String.format("%s (age: %d)", name, age);
  }
}
```

如果不使用Lombok：

```java
public class GetterSetterExample {

  private int age = 10;
  private String name;

  @Override
  public String toString() {
    return String.format("%s (age: %d)", name, age);
  }

  public int getAge() {
    return age;
  }

  public void setAge(int age) {
    this.age = age;
  }

  protected void setName(String name) {
    this.name = name;
  }
}
```

## @NonNull

该注解用在属性或构造器上，Lombok会生成一个非空的声明，可用于校验参数，能帮助避免空指针。**主要作用于成员变量和参数中，标识不能为空，否则抛出空指针异常**。 示例如下：

```java
import lombok.NonNull;

public class NonNullExample extends Something {

  private String name;

  public NonNullExample(@NonNull Person person) {
    super("Hello");
    this.name = person.getName();
  }
}
```

不使用Lombok

```java
import lombok.NonNull;

public class NonNullExample extends Something {

  private String name;

  public NonNullExample(@NonNull Person person) {
    super("Hello");
    if (person == null) {
      throw new NullPointerException("person");
    }
    this.name = person.getName();
  }
}
```

## @Cleanup

@Cleanup：自动关闭资源，针对实现了**java.io.Closeable**接口的对象有效，如：典型的**IO流对象**
 示例如下：

```java
import java.io.*;
import lombok.Cleanup;

public class CleanupExample {
  public static void main(String[] args) throws IOException {
    @Cleanup
    InputStream in = new FileInputStream(args[0]);
    @Cleanup
    OutputStream out = new FileOutputStream(args[1]);
    byte[] b = new byte[10000];
    while (true) {
      int r = in.read(b);
      if (r == -1) break;
      out.write(b, 0, r);
    }
  }
}
```

如不使用Lombok，则需如下：

```java
import java.io.*;

public class CleanupExample {

  public static void main(String[] args) throws IOException {
    InputStream in = new FileInputStream(args[0]);
    try {
      OutputStream out = new FileOutputStream(args[1]);
      try {
        byte[] b = new byte[10000];
        while (true) {
          int r = in.read(b);
          if (r == -1) break;
          out.write(b, 0, r);
        }
      } finally {
        if (out != null) {
          out.close();
        }
      }
    } finally {
      if (in != null) {
        in.close();
      }
    }
  }
}
```

## @EqualsAndHashCodes

​		默认情况下，会使用所有非静态（non-static）和非瞬态（non-transient）属性来生成equals和hasCode，也能通过exclude注解来排除一些属性。**作用于类，覆盖默认的equals和hashCode**
示例如下

```java
import lombok.EqualsAndHashCode;

@EqualsAndHashCode(exclude = { "id", "shape" })
public class EqualsAndHashCodeExample {

  private transient int transientVar = 10;
  private String name;
  private double score;
  private Shape shape = new Square(5, 10);
  private String[] tags;
  private int id;

  public String getName() {
    return this.name;
  }

  @EqualsAndHashCode(callSuper = true)
  public static class Square extends Shape {

    private final int width, height;

    public Square(int width, int height) {
      this.width = width;
      this.height = height;
    }
  }
}
```

## @ToString

类使用@ToString注解，Lombok会生成一个toString()方法，默认情况下，会输出类名、所有属性（会按照属性定义顺序），用逗号来分割。
 通过将includeFieldNames参数设为true，就能明确的输出toString()属性。这一点是不是有点绕口，通过代码来看会更清晰些。

使用Lombok的示例：

```java
import lombok.ToString;

@ToString(exclude = "id")
public class ToStringExample {

  private static final int STATIC_VAR = 10;
  private String name;
  private Shape shape = new Square(5, 10);
  private String[] tags;
  private int id;

  public String getName() {
    return this.getName();
  }

  @ToString(callSuper = true, includeFieldNames = true)
  public static class Square extends Shape {

    private final int width, height;

    public Square(int width, int height) {
      this.width = width;
      this.height = height;
    }
  }
}
```



不使用Lombok的示例如下

```java
package com.hots;

import java.util.Arrays;

public class ToStringExample {
    private static final int STATIC_VAR = 10;
    private String name;
    private Shape shape = new Square(5, 10);
    private String[] tags;
    private int id;

    public String getName() {
        return this.getName();
    }

    public static class Square extends Shape {
        private final int width, height;

        public Square(int width, int height) {
            this.width = width;
            this.height = height;
        }

        @Override
        public String toString() {
            return "Square(super=" + super.toString() + ", width=" + this.width + ", height=" + this.height + ")";
        }
    }

    @Override
    public String toString() {
        return "ToStringExample(" + this.getName() + ", " + this.shape + ", " + Arrays.deepToString(this.tags) + ")";
    }
}

```



## @NoArgsConstructor、@RequiredArgsConstructor、@AllArgsConstructor

无参构造器、部分参数构造器、全参构造器。作用于类上，用于生成构造函数。有staticName、access等属性。staticName属性一旦设定，将采用静态方法的方式生成实例，access属性可以限定访问权限。

```java
@NoArgsConstructor：生成无参构造器；
@RequiredArgsConstructor：生成包含final和@NonNull注解的成员变量的构造器；
@AllArgsConstructor：生成全参构造器
```

Lombok没法实现多种参数构造器的重载。
 Lombok示例代码如下：

```java
import lombok.AccessLevel;
import lombok.AllArgsConstructor;
import lombok.NonNull;
import lombok.RequiredArgsConstructor;

@RequiredArgsConstructor(staticName = "of")
@AllArgsConstructor(access = AccessLevel.PROTECTED)
public class ConstructorExample<T> {

  private int x, y;

  @NonNull
  private T description;

  @NoArgsConstructor
  public static class NoArgsExample {

    @NonNull
    private String field;
  }
}

```

不使用Lombok的示例如下

```java
public class ConstructorExample<T> {

  private int x, y;

  @NonNull
  private T description;

  private ConstructorExample(T description) {
    if (description == null) throw new NullPointerException("description");
    this.description = description;
  }

  public static <T> ConstructorExample<T> of(T description) {
    return new ConstructorExample<T>(description);
  }

  @java.beans.ConstructorProperties({ "x", "y", "description" })
  protected ConstructorExample(int x, int y, T description) {
    if (description == null) throw new NullPointerException("description");
    this.x = x;
    this.y = y;
    this.description = description;
  }

  public static class NoArgsExampmle {

    @NonNull
    private String field;

    public NoArgsExample() {}
  }
}

```

## @Log

作用于类上，生成日志变量。针对不同的日志实现产品，有不同的注解