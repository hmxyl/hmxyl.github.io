---
title: MySQL日常使用记录
date: 2022-10-25 19:52:44
tags: MySQL
category: 
  - [数据库, MySQL]
description: 'MySQL日常使用记录'
---



# MyBaties调用MySQL数据库查询无结果,查询语句正确

1. DEBUG 进去源码，排查最终提交的SQL语句中文乱码
2. 检查数据库编码是否为UTF-8
3. 修改MySQL的连接配置：`jdbc:mysql://127.0.0.1:3306/qhswdxssplog?useUnicode=true&characterEncoding=UTF-8`

# 将多行查询结果合并到一行

```sql
GROUP_CONCAT：默认连接符号是逗号

SELECT GROUP_CONCAT(A_TABLE_ID) FRON A_TABLE

#查询某分类的所有子分类并用分号连接子分类ID
SELECT GROUP_CONCAT(A_TABLE_ID SEPARATOR ';') FRON A_TABLE

#查询某分类的所有子分类，根据p_order ASC, cat_id DESC排序后再连接
SELECT GROUP_CONCAT(cat_id ORDER BY p_order ASC, cat_id DESC) FROM goods_cat WHERE pid = 25
```



# 数据库更新操作关于不同数据库的update set from where操作

用来同步两个表的数据

```sql
(Mysql)语句：
    UPDATE A, B SET A_1 = B_1, A_2 = B_2, A_3 = B_3 WHERE A.ID = B.ID
(Oralce)语句：
    UPDATE A SET (A1, A2, A3) = (SELECT B1, B2, B3 FROM B WHERE A.ID = B.ID)
    WHERE ID IN (SELECT B.ID FROM B WHERE A.ID = B.ID)
(MS SQL Server)语句：
    UPDATE A SET A1 = B1, A2 = B2, A3 = B3 FROM A, B WHERE A.ID = B.ID
```

update set from 语句格式

​		当where和set都需要关联一个表进行查询时，整个 update执行时，就需要对被关联的表进行两次扫描，显然效率比较低。对于这种情况，Sybase和SQL SERVER的解决办法是使用UPDATE...SET...FROM...WHERE...的语法，实际上就是从源表获取更新数据。
​		在 SQL 中，表连接（left join、right join、inner join 等）常常用于 select 语句，其实在 SQL 语法中，这些连接也是可以用于update 和 delete 语句的，在这些语句中使用 join 还常常得到事半功倍的效果。

```sql
Update T_OrderForm 
SET T_OrderForm.SellerID =B.L_TUserID
FROM T_OrderForm A LEFT JOIN T_ProductInfo B ON B.L_ID=A.ProductID
```

​		

# 获取建表命令

```sql
show create table 旧表;
```

 这样会将旧表的创建命令列出。我们只需要将该命令拷贝出来，更改table的名字，就可以建立一个完全一样的表



# 将表1内容全部复制到表2

```sql
复制旧表的数据到新表(假设两个表结构一样)
   SELECT * INTO 表2 FROM 表1;
   
复制旧表的数据到新表(假设两个表结构不一样)
   INSERT INTO 新表(字段1,字段2,.......) 
   SELECT 字段1,字段2,...... FROM 旧表
```



# 修改表/字段的注释

```sql
创建表的时候写注释
   create table test1(field_name int comment '字段的注释') comment='表的注释';
   
修改表的注释
   alter table test1 comment '修改后的表的注释';

修改字段的注释--注意：字段名和字段类型照写就行
   alter table test1 modify column field_name int comment '修改后的字段注释';
   
```

# 数据表切换数据库

```sql
RENAME TABLE db_A.table_1 TO db_B.table_2
```

# MySQL自增主键列重新排列

```sql
ALTER TABLE 表名 DROP 列名;
ALTER TABLE 表名 ADD 列名 MEDIUMINT(8) NOT NULL FIRST;
ALTER TABLE 表名 MODIFY COLUMN 列名 MEDIUMINT(8) NOT NULL AUTO_INCREMENT,ADD PRIMARY KEY(列名);
```



# 自动修改更改时间

```sql
ALTER TABLE ware_resource_info MODIFY res_create_time TIMESTAMP DEFAULT CURRENT_TIMESTAMP;
ALTER TABLE ware_resource_info MODIFY modify_time TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP;
```





# linux下mysql-5.6忘记root密码

在linux平台下使用mysql过程中忘记了root密码，对于运维和DBA来讲都是一件头疼的事情，下面来讲解下怎么进行重置mysql数据库root 密码：

1. 停止mysql服务进程： **service mysqld stop** 

2. 然后编辑mysql的配置文件my.cnf  **vim /etc/my.cnf**

3. 找到 [mysqld]这个模块：在最后面添加一段代码，可忽略mysql权限问题，直接登录

   ```sql
   skip-grant-tables
   ```

​		然后保存 :wq!退出

​		启动mysql服务

```sh
service mysqld start
```

直接进入mysql数据库：

 ```sh
Starting MySQL. SUCCESS!
[root@web1 ~]# mysql
Welcome to the MySQL monitor. Commands end with ; or \g.
Your MySQL connection id is 1
Server version: 5.6.34 Source distribution

Copyright (c) 2000, 2016, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.
Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
mysql>
 ```

使用mysql表，然后进行修改mysql的root密码：

```sh
mysql> use mysql; #使用mysql数据库
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> update user set password=password("123456") where user="root";#更新密码
Query OK, 4 rows affected (0.00 sec)
Rows matched: 4 Changed: 4 Warnings: 0

mysql> flush privileges;#刷新权限
Query OK, 0 rows affected (0.00 sec)
```

补充说明：5.7的更新语句

```sql
update mysql.user set authentication_string=password('root') where user='root' ;
```

```sh
[root@web1 ~]# ps -ef |grep mysql #显示mysql现有的进程
root 56407 1 0 17:50 pts/0 00:00:00 /bin/sh /usr/local/mysql/bin/mysqld_safe --datadir=/data/mysql --pid-file=/data/mysql/web1.pid
mysql 56533 56407 0 17:50 pts/0 00:00:00 /usr/local/mysql/bin/mysqld --basedir=/usr/local/mysql --datadir=/data/mysql --plugin-dir=/usr/local/mysql/lib/plugin --user=mysql --log-error=/data/mysql/web1.err --pid-file=/data/mysql/web1.pid
root 56560 1737 0 17:55 pts/0 00:00:00 grep mysql
[root@web1 ~]# killall mysqld #删除mysql现有进程
[root@web1 ~]# ps -ef |grep mysql
root 56566 1737 0 17:56 pts/0 00:00:00 grep mysql
[root@web1 ~]# service mysqld start #重新启动mysql服务
Starting MySQL. SUCCESS!
[root@web1 ~]# mysql -uroot -p #使用新密码登录
Enter password:
Welcome to the MySQL monitor. Commands end with ; or \g.
Your MySQL connection id is 1
Server version: 5.6.34 Source distribution

Copyright (c) 2000, 2016, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>

```

# MySQL5.7.X版本only_full_group_by问题解决

1. 报错信息

   ```
   [Err] 1055 - Expression #1 of ORDER BY clause is not in GROUP BY clause and contains nonaggregated column 'information_schema.PROFILING.SEQ' which is not functionally dependent on columns in GROUP BY clause; this is incompatible with sql_mode=only_full_group_by
   ```

   可以看出是因为sql_mode中设置了only_full_group_by模式引起的。



2. sql_mode的作用是什么呢

   模式定义MySQL会支持哪些SQL语法。以及应执行哪种数据验证检查。最终达到的目标：适应在不同环境中适应mysql，因为可以根据各自的程序设置不同的操作模式。

   

   在only_full_group_by这种模式下，使用group by语句进行查询时，所要查询的语句必须依赖于group by子句中所列出的列，也就是group by要以查询的字段作为分组依据，这里是要查询的所有字段。

3. 常用的sql_mode

   | 参数                       | 说明                                                         |
   | -------------------------- | ------------------------------------------------------------ |
   | ONLY_FULL_GROUP_BY         | 对于GROUP BY聚合操作，如果在SELECT中的列，没有在GROUP  BY中出现，那么这个SQL是不合法的，因为列不在GROUP BY从句中 |
   | NO_AUTO_VALUE_ON_ZERO      | 该值影响自增长列的插入。<br>默认设置下，插入0或NULL代表生成下一个自增长值。如果用户  希望插入的值为0，而该列又是自增长的，那么这个选项就有用了。 |
   | STRICT_TRANS_TABLES        | 在该模式下，如果一个值不能插入到一个事务表中，则中断当前的操作，对非事务表不做限制NO_ZERO_IN_DATE。在严格模式下，不允许日期和月份为零。 |
   | NO_ZERO_DATE               | 设置该值，MySQL数据库不允许插入零日期，插入零日期会抛出错误而不是警告。 |
   | ERROR_FOR_DIVISION_BY_ZERO | 在INSERT或UPDATE过程中，如果数据被零除，则产生错误而非警告。如果未给出该模式，那么数据被零除时MySQL返回NULL |
   | NO_AUTO_CREATE_USER        | 禁止GRANT创建密码为空的用户                                  |
   | NO_ENGINE_SUBSTITUTION     | 如果需要的存储引擎被禁用或未编译，那么抛出错误。不设置此值时，用默认的存储引擎替代，并抛出一个异常 |
   | PIPES_AS_CONCAT            | 将”\|\|”视为字符串的连接操作符而非或运算符，这和Oracle数据库是一样的，也和字符串的拼接函数Concat相类似 |
   | ANSI_QUOTES                | 启用ANSI_QUOTES后，不能用双引号来引用字符串，因为它被解释为识别符 |

4. 查询当前的sql_mode配置

   ```sql
   1. 可以使用select @@global.sql_mode; //全局配置 查询，
   
   2. 也可以通过select @@sql_mode;//已存在数据库配置查询
   ```

5. 解决方法

   - 使用`any_value()`函数,这个函数对不需要group by的字段有效，等同于关闭only_full_group_by，但是这样难免会遗漏某个字段，所以不推荐使用。

   - 暂时性关闭（可以通过select @@sql_mode查出sql_mode以后去掉ONLY_FULL_GROUP_BY后复制过来），但是在重启服务以后失效

     ```sql
     set sql_mode=' ' //改变已经存在的数据库sql_mode 
     set @@global.sql_mode=' ' //改变全局配置sql_mode
     ```

     

   - 更改配置文件（推荐使用）

     ```
     可以通过select @@sql_mode查出sql_mode以后去掉ONLY_FULL_GROUP_BY后复制配置参数
     
     linux系统
     更改/etc/my.cnf文件
     使用vi命令打开，如果有sql_mode=...的注释就把注释打开，如果没有就加上sql_mode=...
     
     Windows系统
     修改安装目录下的my.ini文件，其余同上
     
     ```

     
