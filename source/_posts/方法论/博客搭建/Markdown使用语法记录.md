---
title: Markdown使用语法记录
date: 2022-10-27 17:48:27
tags: Markdown
categories: 
  - [方法论, 博客搭建]
description: Markdown常用语法
---



# 目录

```markdown
开头输入` [TOC]`， 根据标题生成目录
```

# 标题

```markdown
# 一级标题
## 二级标题
### 三级标题
#### 四级标题
##### 五级标题
###### 六级标题
```

# 段落

```markdown
段落的换行是使用两个以上空格加上回车。
```

# 字体

```markdown
*斜体文本*
**粗体文本**
***粗斜体文本***
_斜体文本_
__粗体文本__
___粗斜体文本___
```

# 分隔线

```markdown
三个以上的星号、减号、底线来建立一个分隔线，行内不能有其他东西。
也可以在星号或是减号中间插入空格。
下面每种写法都可以建立分隔线：

***
* * *
*****
- - -
----------
```

# 删除线

```markdown
~~删除文字~~
```

# 高亮

```markdown
==highlight==
```

# 下划线

```markdown
<u>带下划线文本</u>
```

# 脚注

```
[^要注明的文本]


举例：
下面是要标记脚注[^脚注]的测试段落。可以在多个位置使用同一个脚注[^脚注]。脚注[^脚注]内容会自动到页面底部。
[^脚注]:脚注说明内容，需要独立一行。下面一行空着。
1
2
3
再次使用同一个脚注[^脚注]

```

效果：
下面是要标记脚注[^脚注]的测试段落。可以在多个位置使用同一个脚注[^脚注]。脚注[^脚注]内容会自动到页面底部。

[^脚注]: 脚注说明内容，需要独立一行。下面一行空着。

1
2
3
再次使用同一个脚注[^脚注]

# 无序列表

```markdown
星号(*)、加号(+)或是减号(-)作为列表标记，标记后面添加一个空格

举例：

* 第一项
* 第二项
* 第三项

+ 第一项
+ 第二项
+ 第三项


- 第一项
- 第二项
- 第三项
```

# 有序列表

```markdown
有序列表使用数字并加上 . 号来表示，如：

1. 第一项
2. 第二项
3. 第三项
```

# 列表嵌套

```markdown
列表嵌套只需在子列表中的选项前面添加四个空格即可：
举例：
1. 第一项：
    - 第一项嵌套的第一个元素
    - 第一项嵌套的第二个元素
2. 第二项：
    - 第二项嵌套的第一个元素
    - 第二项嵌套的第二个元素
```

# 区块

```markdown
段落开头使用 > 符号 ，然后后面紧跟一个空格符号：

> 区块引用
> 菜鸟教程
> 学的不仅是技术更是梦想
```

> 最外层
>
> > 第一层嵌套
> >
> > > 第二层嵌套

```markdown
另外区块是可以嵌套的，一个 > 符号是最外层，两个 > 符号是第一层嵌套：

> 最外层
> > 第一层嵌套
> > > 第二层嵌套

```

> 最外层
>
> > 第一层嵌套
> >
> > > 第二层嵌套

# 区块中使用列表

```markdown
> 区块中使用列表
> 1. 第一项
> 2. 第二项
> + 第一项
> + 第二项
> + 第三项
```

> 区块中使用列表
>
> 1. 第一项
> 2. 第二项
>
> + 第一项
> + 第二项
> + 第三项

# 复选框

```markdown
使用 - [ ] 和 - [x] 语法可以创建复选框，实现 todo-list 等功能。例如：

 - [x] 已完成事项
 - [ ] 待办事项1
 - [ ] 待办事项2
```

  - [ ] 已完成事项
 - [ ] 待办事项1
 - [x] 待办事项2

# 列表中使用区块

```markdown
需要在 > 前添加四个空格的缩进。

列表中使用区块实例如下：

* 第一项
    > 菜鸟教程
    > 学的不仅是技术更是梦想
* 第二项

```

* 第一项

  > 菜鸟教程
  > 学的不仅是技术更是梦想

* 第二项

# 段落中代码

```markdown
用 `代码内容` 包裹一段代码
```

用 `代码内容` 包裹一段代码

# 代码区块

```
用 ``` 包裹一段代码。可以指定一种语言
```

举例如下：

```javascript
$(document).ready(function () {
    alert('RUNOOB');
});
```


# 链接

```markdown
1. [链接名称](链接地址)  ：[百度](www.baidu.com)
2. <链接地址>  ：<www.baidu.com>  
3. 我们可以通过变量来设置一个链接，变量赋值在文档末尾进行：

这个链接用 1 作为网址变量 [Google][1]。这个链接用 runoob 作为网址变量 [Runoob][runoob]。下面要空一行

然后在文档的结尾为变量赋值（网址）

  [1]: http://www.google.com/
  [runoob]: http://www.runoob.com/
```



+ [链接名称](www.baidu.com)

+ <www.baidu.com>  

+ 这个链接用 1 作为网址变量 [Google][1]。这个链接用 runoob 作为网址变量 [Runoob][runoob]。下面要空一行

  [1]: http://www.google.com/
  [runoob]: http://www.runoob.com/

# 文档内部锚点引用

```markdown
# 标题

----
## 目录
1. [目录1](#jump1)
2. [目录2](#jump2)

---
### <span id="jump1">1. 目录1</span>
---
### <span id="jump2">2. 目录2</span>

```





# 图片

```markdown
![alt 属性文本](图片地址)

![alt 属性文本](图片地址 "图片的title属性文字")


开头一个感叹号 !
接着一个方括号，里面放上图片的替代文字
接着一个普通括号，里面放上图片的网址（最后还可以用引号包住并加上选择性的 'title' 属性的文字）


Markdown 还没有办法指定图片的高度与宽度，如果你需要的话，你可以使用普通的 <img> 标签。
<img src="http://static.runoob.com/images/runoob-logo.png" width="20%">
```

如果希望图片左对齐，左对齐很简单，**单行图片的情况下在前面输入一个空格就解决了**，右对齐就需要靠css了

# 表格

```markdown
|  表头   | 表头  |
|  ----  | ----  |
| 单元格  | 单元格 |
| 单元格  | 单元格 |



对齐方式

我们可以设置表格的对齐方式：

-: 设置内容和标题栏居右对齐。
:- 设置内容和标题栏居左对齐。
:-: 设置内容和标题栏居中对齐。
```

| 左对齐 | 右对齐 | 居中对齐 |
| :----- | -----: | :------: |
| 单元格 | 单元格 |  单元格  |
| 单元格 | 单元格 |  单元格  |

# 支持的 HTML 元素

```markdown
不在 Markdown 涵盖范围之内的标签，都可以直接在文档里面用 HTML 撰写。

目前支持的 HTML 元素有：
<kbd> 定义键盘文本。它表示文本是从键盘上键入的
<b> 加粗
<i>斜体文本
<em>呈现为被强调的文本
<sup> 上标文本
<sub> 下标文本
<br> 换行
<video src="xxx.mp4" /> 引入视频
```

这是使用 <kbd>Ctrl</kbd>+<kbd>Alt</kbd>+<kbd>Del</kbd> 重启电脑
这是<b> 加粗</b>
这是<i>斜体文本</i>
这是<em>呈现为被强调的文本</em>
这是<sup> 上标文本</sup>
这是<sub> 下标文本</sub>
这是<br> 换行

# 转义

```markdown
使用反斜杠转义特殊字符
    **文本加粗** 
    \*\* 正常显示星号 \*\*

Markdown 支持以下这些符号前面加上反斜杠来帮助插入普通的符号：

    \   反斜线
    `   反引号
    *   星号
    _   下划线
    {}  花括号
    []  方括号
    ()  小括号
    #   井字号
    +   加号
    -   减号
    .   英文句点
    !   感叹号
```

# 公式

```markdown
当你需要在编辑器中插入数学公式时，可以使用两个美元符 $$ 包裹 TeX 或 LaTeX 格式的数学公式来实现
```

# 流程图、时序图(顺序图)、甘特图

1. 横向流程图源码格式

   ```
   graph LR
   A[方形] -->B(圆角)
       B --> C{条件a}
       C -->|a=1| D[结果1]
       C -->|a=2| E[结果2]
   ```

   

   ```mermaid
   graph LR
   A[方形] -->B(圆角)
       B --> C{条件a}
       C -->|a=1| D[结果1]
       C -->|a=2| E[结果2]
   ```

   

2. 竖向流程图源码格式：

   ```
   graph TD
   A[方形] --> B(圆角)
       B --> C{条件a}
       C --> |a=1| D[结果1]
       C --> |a=2| E[结果2]
       F[竖向流程图]
   ```

   效果：

   ```mermaid
   graph TD
   A[方形] --> B(圆角)
       B --> C{条件a}
       C --> |a=1| D[结果1]
       C --> |a=2| E[结果2]
       F[竖向流程图]
   ```

   

   

3. 标准流程图源码格式：

   ```
   st=>start: 开始框
   op=>operation: 处理框
   cond=>condition: 判断框(是或否?)
   sub1=>subroutine: 子流程
   io=>inputoutput: 输入输出框
   e=>end: 结束框
   st->op->cond
   cond(yes)->io->e
   cond(no)->sub1(right)->op
   ```

   效果：

   

   ```flow
   st=>start: 开始框
   op=>operation: 处理框
   cond=>condition: 判断框(是或否?)
   sub1=>subroutine: 子流程
   io=>inputoutput: 输入输出框
   e=>end: 结束框
   st->op->cond
   cond(yes)->io->e
   cond(no)->sub1(right)->op
   
   ```


4. 标准流程图源码格式（横向）：

   ```
   st=>start: 开始框
   op=>operation: 处理框
   cond=>condition: 判断框(是或否?)
   sub1=>subroutine: 子流程
   io=>inputoutput: 输入输出框
   e=>end: 结束框
   st(right)->op(right)->cond
   cond(yes)->io(bottom)->e
   cond(no)->sub1(right)->op
   ```

   ```flow
   st=>start: 开始框
   op=>operation: 处理框
   cond=>condition: 判断框(是或否?)
   sub1=>subroutine: 子流程
   io=>inputoutput: 输入输出框
   e=>end: 结束框
   st(right)->op(right)->cond
   cond(yes)->io(bottom)->e
   cond(no)->sub1(right)->op
   ```

   

5. UML时序图源码样例：

   ```
   对象A->对象B: 对象B你好吗?（请求）
   Note right of 对象B: 对象B的描述
   Note left of 对象A: 对象A的描述(提示)
   对象B-->对象A: 我很好(响应)
   对象A->对象B: 你真的好吗？
   ```

   

   ```sequence
   对象A->对象B: 对象B你好吗?（请求）
   Note right of 对象B: 对象B的描述
   Note left of 对象A: 对象A的描述(提示)
   对象B-->对象A: 我很好(响应)
   对象A->对象B: 你真的好吗？
   ```

   

   6、UML时序图源码复杂样例：

   ```
   Title: 标题：复杂使用
   对象A->对象B: 对象B你好吗?（请求）
   Note right of 对象B: 对象B的描述
   Note left of 对象A: 对象A的描述(提示)
   对象B-->对象A: 我很好(响应)
   对象B->小三: 你好吗
   小三-->>对象A: 对象B找我了
   对象A->对象B: 你真的好吗？
   Note over 小三,对象B: 我们是朋友
   participant C
   Note right of C: 没人陪我玩
   ```

   

   ```sequence
   Title: 标题：复杂使用
   对象A->对象B: 对象B你好吗?（请求）
   Note right of 对象B: 对象B的描述
   Note left of 对象A: 对象A的描述(提示)
   对象B-->对象A: 我很好(响应)
   对象B->小三: 你好吗
   小三-->>对象A: 对象B找我了
   对象A->对象B: 你真的好吗？
   Note over 小三,对象B: 我们是朋友
   participant C
   Note right of C: 没人陪我玩
   ```

   

   7、UML标准时序图样例：

   ```
   %% 时序图例子,-> 直线，->>实线箭头，-->虚线，-->>虚线箭头
     sequenceDiagram
       participant 张三
       participant 李四
       张三->王五: 王五你好吗？
       loop 健康检查
           王五->王五: 与疾病战斗
       end
       Note right of 王五: 合理 食物 <br/>看医生...
       李四-->>张三: 很好!
       王五->李四: 你怎么样?
       李四-->王五: 很好!
   ```

   

   ```mermaid
   %% 时序图例子,-> 直线，->>实线箭头，-->虚线，-->>虚线箭头
     sequenceDiagram
       participant 张三
       participant 李四
       张三->王五: 王五你好吗？
       loop 健康检查
           王五->王五: 与疾病战斗
       end
       Note right of 王五: 合理 食物 <br/>看医生...
       李四-->>张三: 很好!
       王五->李四: 你怎么样?
       李四-->王五: 很好!
   ```




   8、甘特图样例：

   ```
   %% 语法示例
     gantt
     dateFormat  YYYY-MM-DD
     title 软件开发甘特图
     section 设计
     需求:done, des1, 2014-01-06,2014-01-08
     原型 :active,  des2, 2014-01-09, 3d
     UI设计:des3, after des2, 5d
   未来任务:des4, after des3, 5d
     section 开发
     学习准备理解需求 :crit, done, 2014-01-06,24h
     设计框架  :crit, done, after des2, 2d
     开发:crit, active, 3d
     未来任务:crit, 5d
     耍  :2d
     section 测试
     功能测试:active, a1, after des3, 3d
     压力测试 :after a1  , 20h
     测试报告 : 48h
   ```

   

   ```mermaid
   %% 语法示例
     gantt
     dateFormat  YYYY-MM-DD
     title 软件开发甘特图
     section 设计
     需求:done, des1, 2014-01-06,2014-01-08
     原型 :active,  des2, 2014-01-09, 3d
     UI设计:des3, after des2, 5d
   未来任务:des4, after des3, 5d
     section 开发
     学习准备理解需求 :crit, done, 2014-01-06,24h
     设计框架  :crit, done, after des2, 2d
     开发:crit, active, 3d
     未来任务:crit, 5d
     耍  :2d
     section 测试
     功能测试:active, a1, after des3, 3d
     压力测试 :after a1  , 20h
     测试报告 : 48h
   ```

   
