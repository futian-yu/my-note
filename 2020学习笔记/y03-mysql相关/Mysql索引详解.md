**Mysql索引详解：**

```java
mysql查询会产生读锁.(影响插入和更新，不影响其他查询)

会锁表，但分情况，一般是加S锁，如果是有索引，会给行锁，如果没有索引，则给的是表锁。

S锁不会影响到其他的查询，但是会影响到插入和更新，也就是说，如果你有一个查询很慢，且进行了表锁，你的插入和更新都会被影响到，但不会影响其他的查询，但如果有索引，走的行锁，又不会影响到其他的插入和更新。
```





​	**1.mysql组合索引：**

​		[mysql组合索引与字段顺序](https://www.cnblogs.com/sunss/archive/2010/09/14/1826112.html)

​		  很多时候，我们在mysql中创建了索引，但是某些查询还是很慢，根本就没有使用到索引！
一般来说，可能是某些字段没有创建索引，或者是组合索引中字段的顺序与查询语句中字段的顺序不符。

看下面的例子：
假设有一张订单表(orders)，包含order_id和product_id二个字段。
一共有31条数据。符合下面语句的数据有5条。

执行下面的sql语句：

```sql
select product_id
from orders
where order_id in (123, 312, 223, 132, 224);
```

这条语句要mysql去根据order_id进行搜索，然后返回匹配记录中的product_id。

所以组合索引应该按照以下的顺序创建：

```sql
create index orderid_productid on orders(order_id, product_id)
mysql> explain select product_id from orders where order_id in (123, 312, 223, 132, 224) \G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: orders
         type: range
possible_keys: orderid_productid
          key: orderid_productid
      key_len: 5
          ref: NULL
         rows: 5
        Extra: Using where; Using index
1 row in set (0.00 sec)
```

可以看到，这个组合索引被用到了,扫描的范围也很小，只有5行。

如果把组合索引的顺序换成product_id, order_id的话，
mysql就会去索引中搜索 *123 *312 *223 *132 *224，必然会有些慢了。

```sql
mysql> create index orderid_productid on orders(product_id, order_id);                                                      
Query OK, 31 rows affected (0.01 sec)
Records: 31  Duplicates: 0  Warnings: 0

mysql> explain select product_id from orders where order_id in (123, 312, 223, 132, 224) \G

*************************** 1. row ***************************

​           id: 1
  select_type: SIMPLE
​        table: orders
​         type: index
possible_keys: NULL
​          key: orderid_productid
​      key_len: 10
​          ref: NULL
​         rows: 31
​        Extra: Using where; Using index
1 row in set (0.00 sec)
```

这次索引搜索的性能显然不能和上次相比了。

rows:31，我的表中一共就31条数据。

索引被使用部分的长度：key_len:10，比上一次的key_len:5多了一倍。

不知道是这样在索引里面查找速度快，还是直接去全表扫描更快呢？

```sql
mysql> alter table orders add modify_a char(255) default 'aaa';
Query OK, 31 rows affected (0.01 sec)
Records: 31  Duplicates: 0  Warnings: 0

mysql>
mysql>
mysql> explain select modify_a from orders where order_id in (123, 312, 223, 132, 224) \G         
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: orders
         type: ALL
possible_keys: NULL
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 31
        Extra: Using where
1 row in set (0.00 sec)
```

这样就不会用到索引了。 刚才是因为select的product_id与where中的order_id都在索引里面的。  

###### 为什么要创建组合索引呢？这么简单的情况直接创建一个order_id的索引不就行了吗？

如果只有一个order_id索引，没什么问题，会用到这个索引，然后mysql要去磁盘上的表里面取到product_id。

如果有组合索引的话，mysql可以完全从索引中取到product_id，速度自然会快。

###### 再多说几句组合索引的最左优先原则：

组合索引的第一个字段必须出现在查询组句中，这个索引才会被用到。
如果有一个组合索引(col_a,col_b,col_c)

下面的情况都会用到这个索引：

```sql
col_a = "some value";
col_a = "some value" and col_b = "some value";
col_a = "some value" and col_b = "some value" and col_c = "some value";
col_b = "some value" and col_a = "some value" and col_c = "some value";

对于最后一条语句，mysql会自动优化成第三条的样子~~。

下面的情况就不会用到索引：
col_b = "aaaaaa";
col_b = "aaaa" and col_c = "cccccc";
```


下列转自：http://hi.baidu.com/liuzhiqun/blog/item/4957bcb1aed1b5590823023c.html

通过实例理解单列索引、多列索引以及最左前缀原则

实例：现在我们想查出满足以下条件的用户id：
mysql>SELECT ｀uid｀ FROM people WHERE lname｀='Liu'  AND ｀fname｀='Zhiqun' AND ｀age｀=26
因为我们不想扫描整表，故考虑用索引。

  单列索引：
ALTER TABLE people ADD INDEX lname (lname);
将lname列建索引，这样就把范围限制在lname='Liu'的结果集1上，之后扫描结果集1，产生满足fname='Zhiqun'的结果集2，再扫描结果集2，找到 age=26的结果集3，即最终结果。

由 于建立了lname列的索引，与执行表的完全扫描相比，效率提高了很多，但我们要求扫描的记录数量仍旧远远超过了实际所需 要的。虽然我们可以删除lname列上的索引，再创建fname或者age 列的索引，但是，不论在哪个列上创建索引搜索效率仍旧相似。

2.多列索引：
ALTER TABLE people ADD INDEX lname_fname_age (lame,fname,age);
为了提高搜索效率，我们需要考虑运用多列索引,由于索引文件以B－Tree格式保存，所以我们不用扫描任何记录，即可得到最终结果。

注：在mysql中执行查询时，只能使用一个索引，如果我们在lname,fname,age上分别建索引,执行查询时，只能使用一个索引，mysql会选择一个最严格(获得结果集记录数最少)的索引。

3.最左前缀：顾名思义，就是最左优先，上例中我们创建了lname_fname_age多列索引,相当于创建了(lname)单列索引，(lname,fname)组合索引以及(lname,fname,age)组合索引。

注：在创建多列索引时，要根据业务需求，where子句中使用最频繁的一列放在最左边。  

**为什么要使用联合索引**

减少开销。建一个联合索引(col1,col2,col3)，实际相当于建了(col1),(col1,col2),(col1,col2,col3)三个索引。每多一个索引，都会增加写操作的开销和磁盘空间的开销。对于大量数据的表，使用联合索引会大大的减少开销！

覆盖索引。对联合索引(col1,col2,col3)，如果有如下的sql: select col1,col2,col3 from test where col1=1 and col2=2。那么MySQL可以直接通过遍历索引取得数据，而无需回表，这减少了很多的随机io操作。减少io操作，特别的随机io其实是dba主要的优化策略。所以，在真正的实际应用中，覆盖索引是主要的提升性能的优化手段之一。

效率高。索引列越多，通过索引筛选出的数据越少。有1000W条数据的表，有如下sql:select from table where col1=1 and col2=2 and col3=3,假设假设每个条件可以筛选出10%的数据，如果只有单值索引，那么通过该索引能筛选出1000W10%=100w条数据，然后再回表从100w条数据中找到符合col2=2 and col3= 3的数据，然后再排序，再分页；如果是联合索引，通过索引筛选出1000w10% 10% *10%=1w，效率提升可想而知！



**2.mysql覆盖索引**

​	**2.1 索引的有哪些种类？**

索引的种类这里只罗列出InnoDB支持的索引：主键索引(PRIMARY)，普通索引(INDEX)，唯一索引(UNIQUE)，组合索引，总体划分为两类，主键索引也被称为聚簇索引（clustered index），其余都称呼为非主键索引也被称为二级索引（secondary index）。

**2.2 InnoDB的不同的索引组织结构是怎样的呢？**

众所周知在InnoDB引用的是B+树索引模型，这里对B+树结构暂时不做过多阐述，很多文章都有描述，在第二问中我们对索引的种类划分为两大类主键索引和非主键索引，那么问题就在于比较两种索引的区别了，我们这里建立一张学生表，其中包含字段id设置主键索引、name设置普通索引、age(无处理)，并向数据库中插入4条数据：（"小赵", 10）（"小王", 11）（"小李", 12）（"小陈", 13）

```sql
CREATE TABLE `student` (
  `id` int(11) NOT NULL AUTO_INCREMENT COMMENT '自增主键',
  `name` varchar(32) COLLATE utf8_bin NOT NULL COMMENT '名称',
  `age` int(3) unsigned NOT NULL DEFAULT '1' COMMENT '年龄',
  PRIMARY KEY (`id`),
  KEY `I_name` (`name`)
) ENGINE=InnoDB;

INSERT INTO student (name, age) VALUES("小赵", 10),("小王", 11),("小李", 12),("小陈", 13);

```

这里我们设置了主键为自增，那么此时数据库里数据为：

![img](https://user-gold-cdn.xitu.io/2019/10/15/16dcff55c1ff558f?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

每一个索引在 InnoDB 里面对应一棵B+树，那么此时就存着两棵B+树。

![img](https://user-gold-cdn.xitu.io/2019/10/15/16dd012fccb51ee0?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

可以发现区别在与叶子节点中，主键索引存储了整行数据，而非主键索引中存储的值为主键id, 在我们执行如下sql后



```sql
SELECT age FROM student WHERE name = '小李'；

```

流程为：

1. 在name索引树上找到名称为小李的节点 id为03
2. 从id索引树上找到id为03的节点 获取所有数据
3. 从数据中获取字段命为age的值返回 12

**在流程中从非主键索引树搜索回到主键索引树搜索的过程称为：回表**，在本次查询中因为查询结果只存在主键索引树中，我们必须回表才能查询到结果，那么如何优化这个过程呢？引入正文覆盖索引

## 4. 什么是覆盖索引？

覆盖索引（covering index ，或称为索引覆盖）即从非主键索引中就能查到的记录，而不需要查询主键索引中的记录，避免了回表的产生减少了树的搜索次数，显著提升性能。

## 5. 如何使用是覆盖索引？

之前我们已经建立了表student，那么现在出现的业务需求中要求根据名称获取学生的年龄，并且该搜索场景非常频繁，那么先在我们删除掉之前以字段name建立的普通索引，以name和age两个字段建立联合索引，sql命令与建立后的索引树结构如下

```sql
ALTER TABLE student DROP INDEX I_name;
ALTER TABLE student ADD INDEX I_name_age(name, age);
复制代码
```



![img](https://user-gold-cdn.xitu.io/2019/10/16/16dd033b2a2c7c24?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

 那在我们再次执行如下sql后



```sql
SELECT age FROM student WHERE name = '小李'；
```

流程为：

1. 在name,age联合索引树上找到名称为小李的节点
2. 此时节点索引里包含信息age 直接返回 12

## 6. 如何确定数据库成功使用了覆盖索引呢？

当发起一个索引覆盖查询时，在explain的extra列可以看到using index的信息 

![img](https://user-gold-cdn.xitu.io/2019/10/16/16dd03e788c92e7e?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

 这里我们很清楚的看到Extra中Using index表明我们成功使用了覆盖索引



# 30多条mysql数据库优化方法,千万级数据库记录查询轻松解决（利用索引）

（http://www.ihref.com/read-16422.html）

1.对查询进行优化，应尽量避免全表扫描，首先应考虑在 where 及 order by 涉及的列上建立索引。

2.应尽量避免在 where 子句中对字段进行 null 值判断，否则将导致引擎放弃使用索引而进行全表扫描，

```
Sql 代码 : select id from t where num is null;
```

可以在 num 上设置默认值 0,确保表中 num 列没有 null 值，然后这样查询：

```
Sql 代码 : select id from t where num=0;
```

3.应尽量避免在 where 子句中使用!=或<>操作符，否则将引擎放弃使用索引而进行全表扫描。

4.应尽量避免在 where 子句中使用 or 来连接条件，否则将导致引擎放弃使用索引而进行全表扫描，

```
Sql 代码 : select id from t where num=10 or num=20;
```

可以这样查询：

```
Sql 代码 : select id from t where num=10 union all select id from t where num=20;
```

5.in 和 not in 也要慎用，否则会导致全表扫描，如：

```
Sql 代码 : select id from t where num in(1,2,3);
```

对于连续的数值，能用 between 就不要用 in 了：

```
Sql 代码 : select id from t where num between 1 and 3;
```

6.下面的查询也将导致全表扫描：

```
Sql 代码 : select id from t where name like '%c%';
```

若要提高效率，可以考虑全文检索。

7.如果在 where 子句中使用参数，也会导致全表扫描。因为 SQL 只有在运行时才会解析局部变量，但优 化程序不能将访问计划的选择推迟到运行时;它必须在编译时进行选择。然 而，如果在编译时建立访问计 划，变量的值还是未知的，因而无法作为索引选择的输入项。如下面语句将进行全表扫描：

```
Sql 代码 : select id from t where num=@num ;
```

可以改为强制查询使用索引：

```
Sql 代码 : select id from t with(index(索引名)) where num=@num ;
```

8.应尽量避免在 where 子句中对字段进行表达式操作， 这将导致引擎放弃使用索引而进行全表扫描。

```
Sql 代码 : select id from t where num/2=100;
```

可以这样查询：

```
Sql 代码 : select id from t where num=100*2;
```

9.应尽量避免在 where 子句中对字段进行函数操作，这将导致引擎放弃使用索引而进行全表扫描。如：

```
Sql 代码 : select id from t where substring(name,1,3)='abc';#name 以 abc 开头的 id
```

应改为：

```
Sql 代码 : select id from t where name like 'abc%';
```

10.不要在 where 子句中的“=”左边进行函数、算术运算或其他表达式运算，否则系统将可能无法正确使用 索引。

11.在使用索引字段作为条件时，如果该索引是复合索引，那么必须使用到该索引中的第一个字段作为条件 时才能保证系统使用该索引， 否则该索引将不会 被使用， 并且应尽可能的让字段顺序与索引顺序相一致。

12.不要写一些没有意义的查询，如需要生成一个空表结构：

```
Sql 代码 : select col1,col2 into #t from t where 1=0;
```

这类代码不会返回任何结果集，但是会消耗系统资源的，应改成这样：

```
Sql 代码 : create table #t(…);
```

13.很多时候用 exists 代替 in 是一个好的选择：

```
Sql 代码 : select num from a where num in(select num from b);
```

用下面的语句替换：

```
Sql 代码 : select num from a where exists(select 1 from b where num=a.num);
```

14.并不是所有索引对查询都有效，SQL 是根据表中数据来进行查询优化的，当索引列有大量数据重复时， SQL 查询可能不会去利用索引，如一表中有字段 ***,male、female 几乎各一半，那么即使在 *** 上建 了索引也对查询效率起不了作用。

15.索引并不是越多越好，索引固然可以提高相应的 select 的效率，但同时也降低了 insert 及 update 的效率，因为 insert 或 update 时有可能会重建索引，所以怎样建索引需要慎重考虑，视具体情况而定。一个表的索引数最好不要超过 6 个，若太多则应考虑一些不常使用到的列上建的索引是否有必要。

16.应尽可能的避免更新 clustered 索引数据列， 因为 clustered 索引数据列的顺序就是表记录的物理存储顺序，一旦该列值改变将导致整个表记录的顺序的调整，会耗费相当大的资源。若应用系统需要频繁更新 clustered 索引数据列，那么需要考虑是否应将该索引建为 clustered 索引。

17.尽量使用数字型字段，若只含数值信息的字段尽量不要设计为字符型，这会降低查询和连接的性能，并 会增加存储开销。这是因为引擎在处理查询和连接时会逐个比较字符串中每一个字符，而对于数字型而言 只需要比较一次就够了。

18.尽可能的使用 varchar/nvarchar 代替 char/nchar , 因为首先变长字段存储空间小， 可以节省存储空间， 其次对于查询来说，在一个相对较小的字段内搜索效率显然要高些。

19.任何地方都不要使用 select * from t ,用具体的字段列表代替“*”,不要返回用不到的任何字段。

20.尽量使用表变量来代替临时表。如果表变量包含大量数据，请注意索引非常有限(只有主键索引)。

21.避免频繁创建和删除临时表，以减少系统表资源的消耗。

22.临时表并不是不可使用，适当地使用它们可以使某些例程更有效，例如，当需要重复引用大型表或常用 表中的某个数据集时。但是，对于一次性事件， 最好使用导出表。

23.在新建临时表时，如果一次性插入数据量很大，那么可以使用 select into 代替 create table,避免造成大量 log ,以提高速度;如果数据量不大，为了缓和系统表的资源，应先 create table,然后 insert.

24.如果使用到了临时表， 在存储过程的最后务必将所有的临时表显式删除， 先 truncate table ,然后 drop table ,这样可以避免系统表的较长时间锁定。

25.尽量避免使用游标，因为游标的效率较差，如果游标操作的数据超过 1 万行，那么就应该考虑改写。

26.使用基于游标的方法或临时表方法之前，应先寻找基于集的解决方案来解决问题，基于集的方法通常更 有效。

27.与临时表一样，游标并不是不可使用。对小型数据集使用 FAST_FORWARD 游标通常要优于其他逐行处理方法，尤其是在必须引用几个表才能获得所需的数据时。在结果集中包括“合计”的例程通常要比使用游标执行的速度快。如果开发时间允许，基于游标的方法和基于集的方法都可以尝试一下，看哪一种方法的效果更好。

28.在所有的存储过程和触发器的开始处设置 SET NOCOUNT ON ,在结束时设置 SET NOCOUNT OFF .无需在执行存储过程和触发器的每个语句后向客户端发送 DONE_IN_PROC 消息。

29.尽量避免大事务操作，提高系统并发能力。 sql 优化方法使用索引来更快地遍历表。 缺省情况下建立的索引是非群集索引，但有时它并不是最佳的。在非群集索引下，数据在物理上随机存放在数据页上。合理的索引设计要建立在对各种查询的分析和预测上。一般来说：

a.有大量重复值、且经常有范围查询( > ,< ,> =,< =)和 order by、group by 发生的列，可考虑建立集群索引;

b.经常同时存取多列，且每列都含有重复值可考虑建立组合索引;

c.组合索引要尽量使关键查询形成索引覆盖，其前导列一定是使用最频繁的列。索引虽有助于提高性能但 不是索引越多越好，恰好相反过多的索引会导致系统低效。用户在表中每加进一个索引，维护索引集合就 要做相应的更新工作。

30.定期分析表和检查表。

```
分析表的语法：ANALYZE [LOCAL | NO_WRITE_TO_BINLOG] TABLE tb1_name[, tbl_name]...
```

以上语句用于分析和存储表的关键字分布，分析的结果将可以使得系统得到准确的统计信息，使得SQL能够生成正确的执行计划。如果用户感觉实际执行计划并不是预期的执行计划，执行一次分析表可能会解决问题。在分析期间，使用一个读取锁定对表进行锁定。这对于MyISAM，DBD和InnoDB表有作用。

```
例如分析一个数据表：analyze table table_name
检查表的语法：CHECK TABLE tb1_name[,tbl_name]...[option]...option = {QUICK | FAST | MEDIUM | EXTENDED | CHANGED}
```

检查表的作用是检查一个或多个表是否有错误，CHECK TABLE 对MyISAM 和 InnoDB表有作用，对于MyISAM表，关键字统计数据被更新

CHECK TABLE 也可以检查视图是否有错误，比如在视图定义中被引用的表不存在。

31.定期优化表。

```
优化表的语法：OPTIMIZE [LOCAL | NO_WRITE_TO_BINLOG] TABLE tb1_name [,tbl_name]...
```

如果删除了表的一大部分，或者如果已经对含有可变长度行的表(含有 VARCHAR、BLOB或TEXT列的表)进行更多更改，则应使用OPTIMIZE TABLE命令来进行表优化。这个命令可以将表中的空间碎片进行合并，并且可以消除由于删除或者更新造成的空间浪费，但OPTIMIZE TABLE 命令只对MyISAM、 BDB 和InnoDB表起作用。

```
例如： optimize table table_name
```

注意： analyze、check、optimize执行期间将对表进行锁定，因此一定注意要在MySQL数据库不繁忙的时候执行相关的操作。