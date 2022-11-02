---
title: MySQL查询优化策略
date: 2022-10-25 19:52:44
tags: MySQL
category: 
  - [数据库, MySQL]
description: 'MySQL查询优化策略'
---

# 查询优化策略

| 建议                                                         | 原因                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| 不要使用 select * from t                                     | 任何地方都不要使用  select * from t ，用具体的字段列表代替“*”，不要返回用不到的任何字段 |
| 控制查询结果的大小                                           | 尽量避免向客户端返回大数据量，若数据量过大，应该考虑相应需求是否合理。 |
| 考虑在 where 及  order by 涉及的列上建立索引                 |                                                              |
| 避免在 where 子句中使用!=或<>操作符                          | 否则将引擎放弃使用索引而进行全表扫描                         |
| 避免在 where 子句中对字段进行 null 值判断<br>（可以用空白或0这样的默认值替代W） | 否则将引擎放弃使用索引而进行全表扫描                         |
| LEFT JOIN 将可能的组合条件放到Where中                        |                                                              |
| 查询时候的like条件，减少前置百分号                           | 考虑全文检索？                                               |
| 尽量避免在 where 子句中使用 or 来连接条件                    | 否则将引擎放弃使用索引而进行全表扫描： <br>select id from t where num=10 or num=20<br>可以这样查询：<br>select id from t where num=10<br>union all<br>select id from t where num=20 |
| in 和 not in 也要慎用，否则会导致全表扫描                    | select id from  t where num in(1,2,3)<br>对于连续的数值，能用 between 就不要用 in 了<br>select id from t where num between 1 and 3 |
| 很多时候可以用  exists 代替 in                               | select num from  a where num in(select num from b) <br>用下面的语句替换：<br>select num from a where exists(select 1 from b where num=a.num) |
| 使用数字型字段（数据只含有数字）                             | 若只含数值信息的字段尽量不要设计为字符型，这会降低查询和连接的性能，并会增加存储开销。<br>这是因为引擎在处理查询和连接时会 逐个比较字符串中每一个字符，而对于数字型而言只需要比较一次就够了。 |
| 避免在 where 子句中对字段进行表达式操作或者函数操作等        | 否则将导致引擎放弃使用索引而进行全表扫描。<br>如：select id from  t where num/2=100<br>应改为: select id from  t where num=100*2<br><br/>select id from t where substring(name,1,3)=’abc’;<br/>select id from t where datediff(day,createdate,’2005-11-30′)=0<br/>应改为:<br/>select id from t where name like ‘abc%’<br/>select id from t where createdate >= ’2005-11-30′ and createdate < ’2005-12-1′<br/> |
| 使用符合索引，查询条件必须使用到其第一个字段。               | 在使用索引字段作为条件时，如果该索引是复合索引，那么必须使用到该索引中的第一个字段作为条件时才能保证系统使用该索引，否则该索引将不会被使 用，并且应尽可能的让字段顺序与索引顺序相一致。 |
| 不要写一些没有意义的查询，减少资源占用                       | 如需要生成一个空表结构：<br/>select col1,col2 into #t from t where 1=0<br/>这类代码不会返回任何结果集，但是会消耗系统资源的，应改成这样：<br/>create table #t(…) |
| 对大量重复的数据列建索引无太大意义                           | 并不是所有索引对查询都有效<br/>SQL是根据表中数据来进行查询优化的，当索引列有大量数据重复时，SQL查询可能不会去利用索引<br/>如一表中有字段 sex，male、female几乎各一半，那么即使在sex上建了索引也对查询效率起不了作用。<br/> |
| 使用索引会提高select的效率，降低insert 及 update 的效率      | 索引并不是越多越好，索引固然可以提高相应的 select 的效率，<br/>但同时也降低了 insert 及 update 的效率，因为 insert 或 update 时有可能会重建索引<br/> |
| 尽可能的使用 varchar/nvarchar 代替 char/nchar                | 因为首先变长字段存储空间小，可以节省存储空间，<br/>其次对于查询来说，在一个相对较小的字段内搜索效率显然要高些。 |
| 应尽可能的避免更新 clustered 索引数据列                      | 因为 clustered 索引数据列的顺序就是表记录的物理存储顺序，一旦该列值改变将导致整个表记录的顺序的调整，会耗费相当大的资源。若应用系统需要频繁更新 clustered 索引数据列，那么需要考虑是否应将该索引建为 clustered 索引。 |

# Explain关键字

​		explain显示了MySQL如何使用索引来处理select语句以及连接表。可以帮助选择更好的索引和写出更优化的查询语句。简单讲，它的作用就是分析查询性能。

​		explain关键字的使用方法很简单，就是把它放在select查询语句的前面。

​		mysql查看是否使用索引，简单的看type类型就可以。如果它是all，那说明这条查询语句遍历了所有的行，并没有使用到索引。

```sql
explain select * from company_info where cname like '%小%'
```

![image-20211201013210968](D:\0_Notes\Hexo\hmxyl\source\_images\image-20211201013210968.png)  

```sql
explain select * from company_info where cname like '小%'
```

 ![image-20211201013229904](D:\0_Notes\Hexo\hmxyl\source\_images\image-20211201013229904.png)   

Explain查询结果说明

![image-20211201013654229](D:\0_Notes\Hexo\hmxyl\source\_images\image-20211201013654229.png)   



1. id列数字越大越先执行，如果说数字一样大，那么就从上往下依次执行，id列为null的就表是这是一个结果集，不需要使用它来进行查询。

2. select_type列常见的有

   | 参数               | 说明                                                         |
   | ------------------ | ------------------------------------------------------------ |
   | simple             | 表示不需要union操作或者不包含子查询的简单select查询。有连接查询时，外层的查询为simple，且只有一个 |
   | primary            | 一个需要union操作或者含有子查询的select，位于最外层的单位查询的select_type即为primary。且只有一个 |
   | union              | union连接的两个select查询，第一个查询是dervied派生表，除了第一个表外，第二个以后的表select_type都是union |
   | dependent union    | 与union一样，出现在union 或union all语句中，但是这个查询要受到外部查询的影响 |
   | union result       | 包含union的结果集，在union和union all语句中,因为它不需要参与查询，所以id字段为null |
   | subquery           | 除了from字句中包含的子查询外，其他地方出现的子查询都可能是subquery |
   | dependent subquery | 与dependent union类似，表示这个subquery的查询要受到外部表查询的影响 |
   | derived            | from字句中出现的子查询，也叫做派生表，其他数据库中可能叫做内联视图或嵌套select |

3. table

   - 如果查询使用了别名，那么这里显示的是别名

   - 如果不涉及对数据表的操作，那么这显示为null，
   - 如果显示为尖括号括起来的<derived N>就表示这个是临时表，后边的N就是执行计划中的id，表示结果来自于这个查询产生。
   - 如果是尖括号括起来的<union M,N>，与<derived N>类似，也是一个临时表，表示这个结果来自于union查询的id为M,N的结果集。

4. type

   依次从好到差：<font color='red'>system，const，eq_ref，ref，fulltext，ref_or_null，unique_subquery，index_subquery，range，index_merge，index，ALL</font>，

   除了all之外，其他的type都可以使用到索引，除了index_merge之外，其他的type只可以用到一个索引

   | 参数            | 说明                                                         |
   | --------------- | ------------------------------------------------------------ |
   | system          | 表中只有一行数据或者是空表，且只能用于myisam和memory表。如果是Innodb引擎表，type列在这个情况通常都是all或者index |
   | const           | 使用唯一索引或者主键，返回记录一定是1行记录的等值where条件时，通常type是const。其他数据库也叫做唯一索引扫描 |
   | eq_ref          | 出现在要连接过个表的查询计划中，驱动表只返回一行数据，且这行数据是第二个表的主键或者唯一索引，且必须为not  null，唯一索引和主键是多列时，只有所有的列都用作比较时才会出现eq_ref |
   | ref             | 不像eq_ref那样要求连接顺序，也没有主键和唯一索引的要求，只要使用相等条件检索时就可能出现，常见与辅助索引的等值查找。或者多列主键、唯一索引中，使用第一个列之外的列作为等值查找也会出现，总之，返回数据不唯一的等值查找就可能出现。 |
   | fulltext        | 全文索引检索，要注意，全文索引的优先级很高，若全文索引和普通索引同时存在时，mysql不管代价，优先选择使用全文索引 |
   | ref_or_null     | 与ref方法类似，只是增加了null值的比较。实际用的不多。        |
   | unique_subquery | 用于where中的in形式子查询，子查询返回不重复值唯一值          |
   | index_subquery  | 用于in形式子查询使用到了辅助索引或者in常数列表，子查询可能返回重复值，可以使用索引将子查询去重。 |
   | range           | 索引范围扫描，常见于使用>,<,is null,between ,in ,like等运算符的查询中。 |
   | index_merge     | 表示查询使用了两个以上的索引，最后取交集或者并集，常见and  ，or的条件使用了不同的索引，官方排序这个在ref_or_null之后，但是实际上由于要读取所个索引，性能可能大部分时间都不如range |
   | index           | 索引全表扫描，把索引从头到尾扫一遍，常见于使用索引列就可以处理不需要读取数据文件的查询、可以使用索引排序或者分组的查询。 |
   | all             | 这个就是全表扫描数据文件，然后再在server层进行过滤返回符合要求的记录。 |

5. possible_keys
   查询可能使用到的索引都会在这里列出来

6. key
   查询真正使用到的索引，select_type为index_merge时，这里可能出现两个以上的索引，其他的select_type这里只会出现一个。

7. key_len

   用于处理查询的索引长度，如果是单列索引，那就整个索引长度算进去，如果是多列索引，那么查询不一定都能使用到所有的列，具体使用到了多少个列的索引，这里就会计算进去，没有使用到的列，这里不会计算进去。留意下这个列的值，算一下你的多列索引总长度就知道有没有使用到所有的列了。要注意，mysql的ICP特性使用到的索引不会计入其中。另外，key_len只计算where条件用到的索引长度，而排序和分组就算用到了索引，也不会计算到key_len中。

8. ref
   如果是使用的常数等值查询，这里会显示const，如果是连接查询，被驱动表的执行计划这里会显示驱动表的关联字段，如果是条件使用了表达式或者函数，或者条件列发生了内部隐式转换，这里可能显示为func

9. rows
   这里是执行计划中估算的扫描行数，不是精确值

10. extra

​	性能从好到坏:useing index>usinh where > using temporary > using filesort

| 参数                                       | 说明                                                         |
| ------------------------------------------ | ------------------------------------------------------------ |
| distinct                                   | 在select部分使用了distinc关键字                              |
| no tables used                             | 不带from字句的查询或者From dual查询。 使用not in()形式子查询或not  exists运算符的连接查询，这种叫做反连接。<br>即，一般连接查询是先查询内表，再查询外表，反连接就是先查询外表，再查询内表 |
| using filesort                             | 排序时无法使用到索引时，就会出现这个。<br/>常见于order by和group by语句中 |
| using index                                | 查询时不需要回表查询，直接通过索引就可以获取查询的数据       |
| using_union                                | 表示使用or连接各个使用索引的条件时，该信息表示从处理结果获取并集 |
| using intersect                            | 表示使用and的各个索引的条件时，该信息表示是从处理结果获取交集 |
| using sort_union和using  sort_intersection | 与前面两个对应的类似，只是他们是出现在用and和or查询信息量大时，先查询主键，然后进行排序合并后，才能读取记录并返回 |
| using where                                | 表示存储引擎返回的记录并不是所有的都满足查询条件，需要在server层进行过滤。<br/>查询条件中分为限制条件和检查条件，<br/>5.6之前，存储引擎只能根据限制条件扫描数据并返回，然后server层根据检查条件进行过滤再返回真正符合查询的数据。<br/>5.6.x之后支持ICP特性，可以把检查条件也下推到存储引擎层，不符合检查条件和限制条件的数据，直接不读取，这样就大大减少了存储引擎扫描的记录数量。<br/>extra列显示using  index condition |
| using temporary                            | 表示使用了临时表存储中间结果。<br/>临时表可以是内存临时表和磁盘临时表，执行计划中看不出来，需要查看status变量，used_tmp_table，used_tmp_disk_table才能看出来 |
| firstmatch(tb_name)                        | 5.6.x开始引入的优化子查询的新特性之一，常见于where字句含有in()类型的子查询。如果内表的数据量比较大，就可能出现这个 |
| loosescan(m..n)                            | 5.6.x之后引入的优化子查询的新特性之一，在in()类型的子查询中，子查询返回的可能有重复记录时，就可能出现这个 |
| filtered                                   | 使用explain extended时会出现这个列，<br/>5.7之后的版本默认就有这个字段，不需要使用explain  extended了。<br/>这个字段表示存储引擎返回的数据在server层过滤后，剩下多少满足查询的记录数量的比例，注意是百分比，不是具体记录数 |