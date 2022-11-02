---
title: UML类图与接口图的表示
date: 2022-10-27 17:57:45
tags: UML
categories: 
  - [方法论, 博客搭建]
description: UML类图类之间的关系
---



# 类图



![image-20221027182132103](D:\0_Notes\Hexo\hmxyl\source\_images\image-20221027182132103.png) 

 

```
①访问修饰符

	+ ：public
	-： private
	# : protected（friendly也归入这类）

② + a : int = defaultValue解读

	a：成员变量名
	int： 变量名的类型
	defaultValue： 为a的默认值

③ + operation1(int params):returnType解读
	operation1： 方法名
	params： 方法参数名
	reternType： 返回值类型
```

# 抽象类图

注意，名称为斜体

![image-20221027182148176](D:\0_Notes\Hexo\hmxyl\source\_images\image-20221027182148176.png)  

# 接口图

![image-20221027182201862](D:\0_Notes\Hexo\hmxyl\source\_images\image-20221027182201862.png) 

# 类之间的关系

## 泛化（继承）关系

![image-20221027182221807](D:\0_Notes\Hexo\hmxyl\source\_images\image-20221027182221807.png)  

## 实现关系

![image-20221027182231520](D:\0_Notes\Hexo\hmxyl\source\_images\image-20221027182231520.png)  

## 关联关系

### 单关联关系

![在这里插入图片描述](D:\0_Notes\Hexo\hmxyl\source\_images\20200908213344266.png) 

### 双向关联关系

![在这里插入图片描述](D:\0_Notes\Hexo\hmxyl\source\_images\20200908213410395.png) 

### 自关联

![image-20221027182248199](D:\0_Notes\Hexo\hmxyl\source\_images\image-20221027182248199.png)  

### 聚合关系

![在这里插入图片描述](D:\0_Notes\Hexo\hmxyl\source\_images\20200908213520120.png) 

## 组合关系

![在这里插入图片描述](D:\0_Notes\Hexo\hmxyl\source\_images\20200908213538273.png)  
