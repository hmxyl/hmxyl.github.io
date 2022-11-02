---
title: IDEA使用技巧
date: 2022-10-30 15:35:57
tags: 
  - IDEA
categories: 
  - [工具, 开发工具, IDEA]
description: 'IDEA日常使用技巧记录'
---

# 插件

## CamelCase

<!-- 使用快捷键转换驼峰、下划线等命名规则-->

![image-20211125092234688](D:\0_Notes\Hexo\hmxyl\source\_images\image-20211125092234688.png)  

光标放在要修改的名称上（如：变量名，或者mapper.xml里的字段名,会自动识别光标所在单词），按control+alt+U,则进行按命名规则进行转换，会按配置中选择的命名规则列表来回切换。 

如图，如果只选择了CamelCase to camelCase、camelCase to snake_case，则可以在两者之间来回切换，适合公司对命名的要求。

![image-20211125092300247](D:\0_Notes\Hexo\hmxyl\source\_images\image-20211125092300247.png)  



## RestfulToolkit

<!--  API查找工具 --> 

![image-20211125092703437](D:\0_Notes\Hexo\hmxyl\source\_images\image-20211125092703437.png)  

## Free Mybatis plugin

<!-- 快速从代码跳转到mapper及从mapper返回代码-->



## EasyCode

代码生成插件

1. 下载插件，安装后重启

   ![image-20211125092413929](D:\0_Notes\Hexo\hmxyl\source\_images\image-20211125092413929.png) 	

2. 在idea右侧选择Database，选择自己的数据库

   ![image-20211125092433296](D:\0_Notes\Hexo\hmxyl\source\_images\image-20211125092433296.png) 

3. 输入账号密码，连接成功

   ![image-20211125092518334](D:\0_Notes\Hexo\hmxyl\source\_images\image-20211125092518334.png) 

4. 选择自己所需的表，鼠标右键->EasyCode->Generate Code

   ![image-20211125092538684](D:\0_Notes\Hexo\hmxyl\source\_images\image-20211125092538684.png)  

5. 选择自己需要生成的，勾选，然后OK就行

   ![image-20211125092616622](D:\0_Notes\Hexo\hmxyl\source\_images\image-20211125092616622.png)  

6. 系统自己生成了entity包，以及实体类。

   ![image-20211125092639951](D:\0_Notes\Hexo\hmxyl\source\_images\image-20211125092639951.png)   



## MybatisLog

<!-- 程序运行的SQL语句可以直接拷贝运行 -->







# 快捷键

## Editing

| 快捷键                    | 英文说明                                                     | 中文说明                                                     |
| ------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| ~~Ctrl + Space~~ Ctrl + , | Basic code completion (the name of any class, method or variable) | 基本代码完成（任何类、方法或变量的名称）                     |
| Ctrl + Shift + Space      | Smart code completion (filters the list of methods and variables by expected type) | 智能代码完成（按预期类型筛选方法和变量列表）                 |
| Ctrl + Shift + Enter      | Complete statement                                           | 完整声明                                                     |
| Ctrl + P                  | Parameter info (within method call arguments)                | 调用方法时，列出全部参数信息                                 |
| Ctrl + Q                  | Quick documentation lookup                                   | 光标放在类/方法上，快速显示Java doc 信息                     |
| Ctrl +hover               | Brief Info                                                   | 鼠标放到类/方法上，提示基本信息                              |
| Ctrl + F1                 | Show descriptions of error or warning at caret               | 显示光标处的错误或警告的处理提示信息                         |
| Alt + Insert              | Generate code... (Getters, Setters, Constructors, hashCode/equals, toString) | 弹出可生成代码列表（getter、setter、constructor、hashCode/equals、toString） |
| Ctrl + O                  | Override methods                                             | 弹出可覆写的方法列表                                         |
| Ctrl + I                  | Implement methods                                            | 弹出要实现的方法列表                                         |
| Ctrl + Alt + T            | Surround with…(if..else,try..catch, for,  synchronized, etc.) | 代码块快选（要放在插入点上的代码结束分号上）                 |
| Ctrl + /                  | Comment/uncomment with line comment                          | 代码行注释                                                   |
| Ctrl + Shift + /          | Comment/uncomment with block comment                         | 代码块注释                                                   |
| Ctrl + W                  | Select successively increasing code blocks                   | 选择连续递增的代码块                                         |
| Ctrl + Shift + W          | Decrease current selection to previous state                 | 取消选择代码块                                               |
| Alt + Enter               | Show intention actions and quick-fixes                       | 展示意图行动<br/>快速修复                                    |
| Ctrl + Alt + L            | Reformat code                                                | 代码格式化                                                   |
| Ctrl + Alt + O            | Optimize imports                                             | 优化类的import                                               |
| Ctrl + Alt + I            | Auto-indent line(s)                                          | 自动缩进行                                                   |
| Tab / Shift + Tab         | Indent/unindent selected lines                               | 缩进/取消缩进选定行                                          |
| Ctrl+X                    | Cut current line or selected block to clipboard              | 将当前行或选定块剪切到剪贴板                                 |
| Ctrl+C                    | Copy current line or selected block to clipboard             | 将当前行或选定块复制到剪贴板                                 |
| Ctrl+V                    | Paste from clipboard                                         | 粘贴                                                         |
| Ctrl+Shift + V            | Paste from recent buffers...                                 | 打开剪贴板，选择要粘贴的内容                                 |
| Ctrl+D                    | Duplicate current line or selected block                     | 复制并粘贴当前行或选定块                                     |
| Ctrl+Y                    | Delete line at caret                                         | 删除光标处的行                                               |
| Ctrl+Shift + J            | Smart line join                                              | 光标所在行和其下一行，或者选择的代码块，合并到一行           |
| Ctrl+Enter                | Smart line split                                             | 行拆分                                                       |
| Shift + Enter             | Start new line                                               | 打开新的一行                                                 |
| Ctrl + Shift + U          | Toggle case for word at caret or selected block              | 在光标所在词语或选定块中切换单词的大小写                     |
| Ctrl + Shift + ]/[        | Select till code block end/start                             | 选择直到代码块结束/开始                                      |
| Ctrl + Delete/Backspace   | Delete to word end/start                                     | 删除至单词结束/开始                                          |
| Ctrl + NumPad+/-          | Expand/collapse code block                                   | 展开/折叠代码块                                              |
| Ctrl + Shift+NumPad+      | Expand all                                                   | 展开所有代码块                                               |
| Ctrl + Shift+NumPad-      | Collapse all                                                 | 折叠所有代码块                                               |
| Ctrl + F4                 | Close active editor tab                                      | 关闭当前活动的编辑窗口                                       |
| Ctrl + F12                |                                                              | 当前类内搜索方法                                             |

| 快捷键     | 英文说明     | 中文说明     |
| ---------- | ------------ | ------------ |
| Alt + Q    | Context info | *上下文信息* |
| Shift + F1 | External Doc | *外部文件*   |

## Usage Search

| 快捷键             | 英文说明                        | 中文说明                                       |
| ------------------ | ------------------------------- | ---------------------------------------------- |
| Alt + F7/Ctrl + F7 | Find usages/Find usages in file | 查找用法/在文件中查找【参数/方法】使用到的地方 |
| Ctrl + Shift + F7  | Highlight usages in file        | 高亮【参数/方法】使用到的地方                  |
| Ctrl + Alt + F7    | Show usages                     | 弹出【参数/方法】使用到的列表                  |

## Navigation

| 快捷键                  | 英文说明                                     | 中文说明                                                     |
| ----------------------- | -------------------------------------------- | ------------------------------------------------------------ |
| Alt + Right/Left        | Go to next / previous editor tab             | 左边的编辑框/右边的编辑框                                    |
| F12                     | Go back to previous tool window              | 项目结构中，选中当前文件                                     |
| Esc                     | Go to editor (from tool window)              | 光标回到编辑栏                                               |
| Shift + Esc             | Hide active or last active window            | 关闭最新打开的工具栏                                         |
| Ctrl+Shift+F4           | Close active run / messages / find / ... tab | 关闭活动的工具栏                                             |
| Ctrl+G                  | Go to line                                   | 跳转到指定行                                                 |
| Ctrl+E                  | Recent files popup                           | 弹框显示最近使用过的文件                                     |
| Ctrl+Alt + Left/Right   | Navigate back / forward                      | 后退/前进                                                    |
| Ctrl+Shift+Backspace    | Navigate to last edit location               | 关闭工具窗口，光标回到最后编辑的文件（类似Esc）              |
| Alt + F1                | Select current file or symbol in any view    | 选择一个视图，显示当前文件或符号（资源管理器、浏览器、结构图等） |
| Ctrl + B , Ctrl + Click | Go to declaration                            | 跳转到声明或者用例处                                         |
| Ctrl + Alt + B          | Go to implementation(s)                      | 跳转到实现处                                                 |
| Ctrl + Shift + I        | Open quick definition lookup                 | 弹框中查看类文件                                             |
| Ctrl + Shift + B        | Go to type declaration                       | 跳转类型声明处                                               |
| Ctrl + Shift + T        |                                              | 转到测试类                                                   |
| Ctrl + U                | Go to super-method / super-class             | 跳转父方法/父类                                              |
| Alt + Up/Down           | Go to previous / next method                 | 光标转到前/后一个方法                                        |
| Ctrl + ]/[              | Move to code block end/start                 | 光标跳转到代码块的结尾/开头                                  |
| Ctrl + F12              | File structure popup                         | 弹出类结构                                                   |
| Ctrl + H                | Type hierarchy                               | 打开类型层次结构                                             |
| Ctrl + Shift + H        | Method hierarchy                             | 打开方法层次结构                                             |
| Ctrl + Alt + H          | Call hierarchy                               | 打开调用层次结构                                             |
| F2 / Shift + F2         | Next/previous highlighted error              | 下一个/前一个 error                                          |
| F4                      | Edit source / View source                    | 编辑/查看文件                                                |
| Alt + Home              | Show navigation bar                          | 显示代码导航栏                                               |
| F11                     | Toggle bookmark                              | 加上/去掉书签                                                |
| Ctrl + F11              | Toggle bookmark with mnemonic                | 添加标记书签                                                 |
| Ctrl + #[0-9]           | Go to numbered bookmark                      | 转到编号书签                                                 |
| Shift + F11             | Show bookmarks                               | 弹框，列出书签                                               |

## Search/Replace

| 快捷键                 | 英文说明                  | 中文说明                         |
| ---------------------- | ------------------------- | -------------------------------- |
| Double Shift           | Search everywhere         | 搜索任意文件                     |
| Ctrl + F               | Find                      | 当前文件查找                     |
| F3 / Shift + F3        | Find next / Find previous | 配合“Ctrl + F” 查找下一个/前一个 |
| Ctrl + R               | Replace                   | 当前文件替换                     |
| Ctrl + Shift + F       | Find in pathqu            | 全局查找                         |
| Ctrl + Shift + R       | Replace in path           | 全局替换                         |
| Ctrl + N               | Go to class               | 搜索类                           |
| Ctrl + Shift + N       | Go to file                | 搜索文件                         |
| Ctrl + Alt + Shift + N | Go to symbol              | 搜索符号                         |
| Ctrl + Shift + A       | Find Action               | 搜索操作                         |

## Live Templates

| 快捷键         | 英文说明                                        | 中文说明                                                     |
| -------------- | ----------------------------------------------- | ------------------------------------------------------------ |
| Ctrl + Alt + J | Surround with Live Template                     |                                                              |
| Ctrl + J       | Insert Live Template                            | 弹出模板选择需要的代码片段                                   |
| iter           | Iteration according to Java SDK 1.5 style       | for (Object o : ) {      }                                   |
| inst           | Checkobjecttype with instanceof and downcast it | if (name instanceof Object) {     Object o = (Object) name;      } |
| itco           | Iterate elements of java.util.Collection        | for (Iterator iterator = collection.iterator(); iterator.hasNext(); ) {     Object next =  iterator.next();      } |
| itit           | Iterate elements of java.util.Iterator          | while (iterator.hasNext()) {     Object next =  iterator.next();      } |
| itli           | Iterate elements of java.util.List              | for (int i = 0; i < list.size(); i++) {     Object o =  list.get(i); |
| thr            | throw new                                       | throw new                                                    |

## Refactoring：重构

| 快捷键         | 英文说明          | 中文说明                                                     |
| -------------- | ----------------- | ------------------------------------------------------------ |
| F5             | Copy              | 复制文件                                                     |
| F6             | Move              | 移动文件                                                     |
| Alt + Delete   | Safe Delete       |                                                              |
| Shift + F6     | Rename            | 重命名（F12 先选中）                                         |
| Ctrl + F6      | Change Signature  | 更改签名                                                     |
| Ctrl + Alt + N | Inline            |                                                              |
| Ctrl + Alt + M | Extract Method    | 提取代码块为一个method                                       |
| Ctrl + Alt + V | Extract Variable  | 提取变量（定义一个新变量，保留当前变量的值）<br/>修改前：<br/>return name;<br/>修改后：<br/>String name1 = name; <br/>return name1; |
| Ctrl + Alt + F | Extract Field     | 提取变量为类的一个字段。                                     |
| Ctrl + Alt + C | Extract Constant  | 提取静态变量                                                 |
| Ctrl + Alt + P | Extract Parameter | 提取方法入参                                                 |
|                |                   |                                                              |

## Debugging

| 快捷键                  | 英文说明                 | 中文说明            |
| ----------------------- | ------------------------ | ------------------- |
| F7 / F8                 | Step into / Step over    | 步入/步出（F5/F6）  |
| Shift + F7 / Shift + F8 | Smart step into/Step out |                     |
| Alt + F9                | Run to cursor            | 跑到下一断点        |
| Alt + F8                | Evaluate expression      | 计算表达式          |
| F9                      | Resume program           | 重新开始程序（F11） |
| Ctrl + F8               | Toggle breakpoint        | 加上/去掉 断点      |
| Ctrl + Shift + F8       | View breakpoints         | 查看断点            |

## Compile and Run

| 快捷键               | 英文说明                                     | 中文说明                |
| -------------------- | -------------------------------------------- | ----------------------- |
| Ctrl + F9            | Make project (compile modifed and dependent) | 构建项目                |
| Ctrl + Shift + F9    | Compile selected file, package or module     | 重新编译                |
| Alt + Shift + F10/F9 | Select configuration and run/and debug       | 打开run/debug的配置文件 |
| Shift + F10/F9       | Run/Debug                                    | Run/Debug 程序          |
| Ctrl + Shift + F10   | Run context configuration from editor        | 从编辑器运行上下文配置  |
| Ctrl + F2            |                                              | stop 程序               |

## VCS/Local History

| 快捷键              | 英文说明              | 中文说明             |
| ------------------- | --------------------- | -------------------- |
| Ctrl + K            | Commit project to VCS | 提交                 |
| Ctrl + T            | Update from VCS       | 更新                 |
| Alt + Shift + C     | View recent changes   | 打开最近变更内容列表 |
| Alt + BackQuote (`) | VCS Operations Popup  | 弹出版本控制选项框   |

## General

| 快捷键                 | 英文说明                                  | 中文说明                            |
| ---------------------- | ----------------------------------------- | ----------------------------------- |
| Alt + #[0-9]           | Open corresponding tool window            | 打开相应的工具窗口                  |
| Ctrl + S               | Save all                                  | 保存                                |
| Ctrl + Alt + Y         | Synchronize                               | 同步（reload）                      |
| Ctrl + Shift + F12     | Toggle maximizing editor                  | 最大化/恢复 编辑窗口                |
| Alt + Shift + F        | Add to Favorites                          | 添加收藏                            |
| Alt + Shift + I        | Inspect current file with current profile | 使用当前配置文件检查当前文件        |
| Ctrl + BackQuote (`)   | Quick switch current scheme               | 切换IDEA主题模式                    |
| Ctrl + Alt + S         | Open Settings dialog                      | 打开设置                            |
| Ctrl + Alt + Shift + S | Open Project Structure dialog             | 打开项目结构                        |
| Ctrl + Tab             | Switch between tabs and tool window       | 打开切换器（类似于windows的任务栏） |

## 查看当前类的父类（Ctrl+Alt+Shift+u）

下面看这个编辑器怎么以图解的形式，查看这种继承关系。

![image-20211127223845078](D:\0_Notes\Hexo\hmxyl\source\_images\image-20211127223845078.png)   

 ![image-20211127223909961](D:\0_Notes\Hexo\hmxyl\source\_images\image-20211127223909961.png) 



## 打开子类工具栏：F4

利用的： 顶部菜单 `Navigate --> Type Hierarchy`

## 弹出子类UML图：Ctrl+Alt+u

 ![image-20211127224042688](D:\0_Notes\Hexo\hmxyl\source\_images\image-20211127224042688.png)  

设置相关快捷键

![image-20211127225955330](D:\0_Notes\Hexo\hmxyl\source\_images\image-20211127225955330.png)  

## 弹出层显示的记录文件个数（Ctrl + E）

![image-20211127231134179](D:\0_Notes\Hexo\hmxyl\source\_images\image-20211127231134179.png)   



## 行拷贝（Ctrl+Alt+⬇）

 Duplicate Line Or Selection

![30f63f77a3c8b29426f7f6411d13f4b9.png](D:\0_Notes\Hexo\hmxyl\source\_images\clip_image002-16380260866347.png)  



# 基础配置

## 编码配置

### 全局编码

![57826dd953a5c863292bed5462025d60.png](D:\0_Notes\Hexo\hmxyl\source\_images\clip_image001-163802678409512.png)   

### 文件编码

打开需要设置编码的文件，在右下角进行设置



### 编码统一

#### File Encodings

 ![image-20211125093711347](D:\0_Notes\Hexo\hmxyl\source\_images\image-20211125093711347.png)



## 文件默认打开方式

![fce0704d161bda7eec8a5cc4743cc062.png](D:\0_Notes\Hexo\hmxyl\source\_images\clip_image002-16380266644219.png)   

如上图标注 1 所示，该区域的后缀类型文件在 IntelliJ IDEA 中将以标注 2 的方式进行打开。

如上图标注 3 所示，我们可以在 IntelliJ IDEA 中忽略某些后缀的文件或是文件夹，比如我一般会把 .idea 这个文件夹忽略。



 



## 字体设置

### 界面字体

Settings->Appearance

![e8ad5d669969dbce8a1e35d6269013df.png](D:\0_Notes\Hexo\hmxyl\source\_images\clip_image001-163802676145910.png) 

### 程序字体

Editor -> Colors & Fonts -> Font

![5e3013e5dad6b0e05e97274216472a90.png](D:\0_Notes\Hexo\hmxyl\source\_images\clip_image002-163802676145911.png) 







# 使用技巧



## 代码提示快捷键（Ctrl+逗号）

![image-20211127232203726](D:\0_Notes\Hexo\hmxyl\source\_images\image-20211127232203726.png)   

如上所示，默认 Ctrl + 空格 快捷键是基础代码提示、补充快捷键，但是由于我们中文系统基本这个快捷键都被输入法占用了，所以我们发现不管怎么按都是没有提示代码效果的，原因就是在此。我个人建议修改此快捷键为 Ctrl + 逗号。



## 鼠标放上去提示参数

![0e6d5bfa29683800dc3eae4602afc8a8.png](D:\0_Notes\Hexo\hmxyl\source\_images\clip_image002-163802681052513.png)   

 软分行查看代码

右键

  ![9fff17978c4752eeb8d4c127c9e2a554.png](D:\0_Notes\Hexo\hmxyl\source\_images\clip_image001.png)  



## 编译增加堆内存

![image-20211127225848686](D:\0_Notes\Hexo\hmxyl\source\_images\image-20211127225848686.png)  

## 格式化代码，后多行空行转为一行

idea格式化代码后会出现最多2行空行，不能像eclipse一样最多只保留一行空行，要想设置的和eclipse效果一样，设置如下

File --> setting --> 搜索 code style --> 选择 blank lines标签项 --> 保留最大空行数设置为1

![image-20211127230244828](D:\0_Notes\Hexo\hmxyl\source\_images\image-20211127230244828.png)  



## 文件打开超过一行，多行显示

![image-20211127230844516](D:\0_Notes\Hexo\hmxyl\source\_images\image-20211127230844516.png)  



如上图标注 1 所示，在打开很多文件的时候，IntelliJ IDEA 默认是把所有打开的文件名 Tab 单行显示的。可以修改为多行 



## 修改类注释模板

`File->Settings->File and Code Templates 找到Includes`



## 添加自定义方法

1. 首先，点击File–>Settings–>Editor–>Live Templates 

   ![image-20211127230339392](D:\0_Notes\Hexo\hmxyl\source\_images\image-20211127230339392.png)  

2. 接着，点击右上角“+”添加“Template Group”模板组，如Java

​        ​![image-20211127230357335](D:\0_Notes\Hexo\hmxyl\source\_images\image-20211127230357335.png)   



3. 在新增的模板组内添加模板，点击右上角“+”添加“Live Template”

   ![image-20211127230437624](D:\0_Notes\Hexo\hmxyl\source\_images\image-20211127230437624.png)   

4. 填写模板内容，定义出发快捷键选择 Enter

​        ![image-20211127230500953](D:\0_Notes\Hexo\hmxyl\source\_images\image-20211127230500953.png)   

5. 定义作用域

   ![image-20211127230524523](D:\0_Notes\Hexo\hmxyl\source\_images\image-20211127230524523.png)   

   

   这样就OK了，可以仿照这种方式，自定义很多快捷输入的语句，比如输入，输出等：

    ![image-20211127230606315](D:\0_Notes\Hexo\hmxyl\source\_images\image-20211127230606315.png)   

   

   

## 默认代码展示形式（折叠/展开）

 ![image-20211127230751897](D:\0_Notes\Hexo\hmxyl\source\_images\image-20211127230751897.png)  

如上图标注红圈所示，我们可以对指定代码类型进行默认折叠或是展开的设置，勾选上的表示该类型的代码在文件被打开的时候默认是被折叠的，去掉勾选则反之。

## 行号和方法分割线

![2c9d0deb2e32bea240016e879acd4cb7.png](D:\0_Notes\Hexo\hmxyl\source\_images\clip_image002.png)  
 如上图红圈所示，默认 IntelliJ IDEA 是没有勾选 Show line numbers 显示行数的，但是我建议一般这个要勾选上。
 如上图红圈所示，默认 IntelliJ IDEA 是没有勾选 Show method separators 显示方法线的，这种线有助于我们区分开方法，所以也是建议勾选上的。

## 单行注释、注释块

搜索：`Add a space at line comment start`

![image-20220304151518515](D:\0_Notes\Hexo\hmxyl\source\_images\image-20220304151518515.png)  



## 显示内存回收

![2bf7deb86f42bcbdd32876423cdbe6b5.png](D:\0_Notes\Hexo\hmxyl\source\_images\clip_image002-16380258373592.png)  

IntelliJ IDEA 14 版本默认是不显示内存使用情况的，对于大内存的机器来讲不显示也无所谓，但是如果是内存小的机器最好还是显示下。如上图演示，点击后可以进行部分内存的回收。



## 拼写检查

![29916ac8772ea1af0367ee55106b713d.png](D:\0_Notes\Hexo\hmxyl\source\_images\clip_image002-16380258506663.png)   







## 增加打开的文件 Tab 个数

Tab limit

![3a13ec2347321bd1c23131c75e5f6808.png](D:\0_Notes\Hexo\hmxyl\source\_images\clip_image002-16380259713575.png)   





## 单文件多窗口打开

![a1f9f345240dd5190a929ffc0fe14de1.png](D:\0_Notes\Hexo\hmxyl\source\_images\clip_image002-16380260699266.png)   









## SpringBoot实现服务器自动重启

- 导入devtools依赖即可

  ```xml
  <dependency> 
    <groupId>org.springframework.boot</groupId> 
    <artifactId>spring-boot-devtools</artifactId> 
    <optional>true</optional> 
  </dependency>
  ```

- 然后到setting框中，输入compiler，然后勾选**Build project automatically**

- 然后按住**shift+alt+ctrl+/**，进入**maintenance**,然后选择进入**Registry**

- 勾选c**ompiler.automake.when.app.running**



## IDEA 开启RunDashboard

- 修改 .idea/workspace.xml 文件

![b8b40c372c3114cb88d98abdc1621768.png](D:\0_Notes\Hexo\hmxyl\source\_images\clip_image002-163802692934114.png)   

```xml
找到<component name="RunDashboard"> 添加配置：


<option name="configurationTypes"> 
  <set> 
    <option value="SpringBootApplicationConfigurationType" /> 
  </set> 
</option>
```

最终配置：
  ![c8e483b781cfd443e68a15b94345844b.png](D:\0_Notes\Hexo\hmxyl\source\_images\clip_image004.png)  
 显示效果：
  ![511e587a5be1655e22f23d7981c2b8c8.png](D:\0_Notes\Hexo\hmxyl\source\_images\clip_image005.png)  





## 忽略提交文件到Git（.ignore）

1. 点击**File->Settings->Plugins**，点击Browse repositories…

   ![3b2f62bb2f51c955400f9d9ff9945a6e.png](D:\0_Notes\Hexo\hmxyl\source\_images\clip_image002-163802704819915.png)  

2. 搜索**.ignore**，点击**Install**，安装完成后，重启IDEA

3. 在 项目上 右键->New ->.ignore file ->.gitignore file(Git)

   ![bdd020c028cf2b207232d1c58ac45f66.png](D:\0_Notes\Hexo\hmxyl\source\_images\clip_image004-163802704820016.png)  

4. 先选择Example user template好了，以后有什么想过滤的可以自行添加，~最后点击Generate生成

   ![3e4705d5b72a8e3944b7e70d4f967e89.png](D:\0_Notes\Hexo\hmxyl\source\_images\clip_image006.png)  

5. 然后就会发现被忽略的文件名变成了灰色 

6. 也可以右键文件将其加入忽略的名单中 

   ![89c2f940c462b9558a9158f433a6e13b.png](D:\0_Notes\Hexo\hmxyl\source\_images\clip_image008.png)   

7. 以下是一些.gitignore文件忽略的匹配规则：

| 通配符   | 说明                                                 |
| -------- | ---------------------------------------------------- |
| .a       | 忽略所有 .a 结尾的文件                               |
| !lib.a   | 但 lib.a 除外                                        |
| /TODO    | 仅仅忽略项目根目录下的 TODO 文件，不包括 subdir/TODO |
| build/   | 忽略 build/ 目录下的所有文件                         |
| doc/.txt | 会忽略 doc/notes.txt 但不包括 doc/server/arch.txt    |

8. 注意

.gitignore只能忽略那些原来没有被track的文件，如果某些文件已经被纳入了版本管理中，则修改.gitignore是无效的。
 那么解决方法就是先把本地缓存删除（改变成未track状态），然后再提交：

 ```
输入：
git rm -r –cached filePath
git commit -m “remove xx”

或者：
git rm -r –cached .
git add .
git commit -m “update .gitignore”


来解释下几个参数
-r 是删除文件夹及其子目录
–cached 是删除暂存区里的文件而不删除工作区里的文件，

第一种是删除某个文件，第二种方法就把所有暂存区里的文件删了，再加一遍，相当于更新了一遍。

 ```



