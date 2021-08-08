# [explain之二：Explain 结果解读与实践，分析诊断工具之二](https://www.cnblogs.com/duanxz/p/4470607.html)

　　MySQL的EXPLAIN命令用于SQL语句的**查询执行计划(QEP)**。这条命令的输出结果能够让我们了解MySQL 优化器是如何执行SQL 语句的。这条命令并没有提供任何调整建议，但它能够提供重要的信息帮助你做出调优决策。

**语法**

MySQL 的EXPLAIN 语法可以运行在SELECT 语句或者特定表上。如果作用在表上，那么此命令等同于DESC 表命令。UPDATE和DELETE 命令也需要进行性能改进，当这些命令不是直接在表的主码上运行时，为了确保最优化的索引使用率，需要把它们改写成SELECT 语句(以便对它们执行EXPLAIN 命令)。请看下面的示例：

```
UPDATE table1  
SET col1 = X, col2 = Y  
WHERE id1 = 9  
AND dt >= '2010-01-01';
```

这个UPDATE语句可以被重写成为下面这样的SELECT语句：

```
SELECT col1, col2  
FROM table1  
WHERE id1 = 9  
AND dt >= '2010-01-01';
```

在5.6.10以后的版本里面,是可以直接对dml语句进行explain分析操作的.MySQL 优化器是基于开销来工作的，它并不提供任何的QEP的位置。这意味着QEP 是在每条SQL 语句执行的时候动态地计算出来的。在MySQL 存储过程中的SQL 语句也是在每次执行时计算QEP 的。存储过程缓存仅仅解析查询树。

![img](https://images2015.cnblogs.com/blog/285763/201704/285763-20170405135318738-849539061.png)

### DESC命令： 

查看表信息：

![img](https://images2015.cnblogs.com/blog/285763/201704/285763-20170405135736957-1566851418.png)

desc查看执行计划信息：

![img](https://images2015.cnblogs.com/blog/285763/201704/285763-20170405134540066-378849764.png)

 

Explain 结果解读与实践

注：单独一行的"%%"及"`"表示分隔内容，就象分开“第一章”“第二章”。

explain 可以分析 select 语句的执行，即 MySQL 的“执行计划”：

mysql> explain select 1;
+----+-------------+-------+------+---------------+------+---------+------+------+----------------+
| id | select_type | table | type | possible_keys | key | key_len | ref | rows | Extra |
+----+-------------+-------+------+---------------+------+---------+------+------+----------------+
| 1 | SIMPLE | NULL | NULL | NULL | NULL | NULL | NULL | NULL | No tables used |
+----+-------------+-------+------+---------------+------+---------+------+------+----------------+

用"\G"代替分号可得到竖排的格式：
mysql> explain select 1\G
*************************** 1
id: 1
select_type: SIMPLE
table: NULL
type: NULL
possible_keys: NULL
key: NULL
key_len: NULL
ref: NULL
rows: NULL
Extra: No tables used

可以用 desc 代替 explain ：desc select 1;
mysql> desc select 1;
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+----------------+
| id | select_type | table | partitions | type | possible_keys | key | key_len | ref | rows | filtered | Extra |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+----------------+
| 1 | SIMPLE | NULL | NULL | NULL | NULL | NULL | NULL | NULL | NULL | NULL | No tables used |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+----------------+
1 row in set, 1 warning (0.00 sec)

##  二、结果列详细说明：

### 2.1、id 列

用下面的例子来说明，在多张表的查询中，突出的更显著。
1、建表：
create table a(a_id int);
create table b(b_id int);
create table c(c_id int);
2、执行查询explain select * from a join b on a_id=b_id where b_id in (select c_id from c);
mysql> explain select * from a join b on a_id=b_id where b_id in (select c_id from c);
+----+--------------------+-------+...
| id | select_type | table |...
+----+--------------------+-------+...
| 1 | PRIMARY | a |...
| 1 | PRIMARY | b |...
| 2 | DEPENDENT SUBQUERY | c |...
+----+--------------------+-------+...
从 3 个表中查询，对应输出 3 行，每行对应一个表， id 列表示执行顺序，id 越大，越先执行，id 相同，由上至下执行。此处的执行顺序为（以 table 列表示）：c -> a -> b

### 2.2、select_type 列

select_type 列提供了各种表示table 列引用的使用方式的类型。最常见的值包括**SIMPLE、PRIMARY、DERIVED 和UNION**。其他可能的值还有**UNION RESULT**、**DEPENDENT SUBQUERY、DEPENDENT UNION、UNCACHEABLE UNION 以及UNCACHEABLE QUERY**。

\1. SIMPLE

对于不包含子查询和其他复杂语法的简单查询，这是一个常 见的类型。

\2. PRIMARY

这是为更复杂的查询而创建的首要表(也就是最外层的表)。这个类型通常可以在DERIVED 和UNION 类型混合使用时见到。

\3. DERIVED

当一个表不是一个物理表时，那么就被叫做DERIVED。下面的SQL 语句给出了一个QEP 中DERIVED select-type 类型的

示例：

```
mysql> EXPLAIN SELECT MAX(id)
-> FROM (SELECT id FROM users WHERE first = ‘west’) c;
```

\4. DEPENDENT SUBQUERY

这个select-type 值是为使用子查询而定义的。下面的SQL语句提供了这个值：

```
mysql> EXPLAIN SELECT p.*
-> FROM parent p
-> WHERE p.id NOT IN (SELECT c.parent_id FROM child c);
```

\5. UNION

这是UNION 语句其中的一个SQL 元素。

\6. UNION RESULT

这是一系列定义在UNION 语句中的表的返回结果。当select_type 为这个值时，经常可以看到table 的值是<unionN,M>，这说明匹配的id 行是这个集合的一部分。下面的SQL产生了一个UNION和UNION RESULT select-type：

```
mysql> EXPLAIN SELECT p.* FROM parent p WHERE p.val
LIKE ‘a%’
-> UNION
-> SELECT p.* FROM parent p WHERE p.id > 5;
```

### 2.3、table 列

table 列是EXPLAIN 命令输出结果中的一个单独行的唯一标识符。这个值可能是表名、表的别名或者一个为查询产生临时表的标识符，如派生表、子查询或集合。

### 2.4、type 列

type 列代表QEP 中指定的表使用的连接方式。下面是最常用的几种连接方式：

先从最佳类型到最差类型：NULL>**system>const>eq_ref>ref>range>index>All**  **重要且困难**

**2.4.0、NULL**在优化过程中就已得到结果，不用再访问表或索引。

```
mysql> explain select min(id) from a9;
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+------------------------------+
| id | select_type | table | partitions | type | possible_keys | key | key_len | ref | rows | filtered | Extra |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+------------------------------+
| 1 | SIMPLE | NULL | NULL | NULL | NULL | NULL | NULL | NULL | NULL | NULL | Select tables optimized away |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+------------------------------+
```

**2.4.1、system**

表仅有一行，这是const类型的特列，平时少见。

建表及插入数据：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
create table a9(id int primary key);
insert into a9 value(1);

mysql> explain select * from a9;
+----+-------------+-------+------------+-------+---------------+---------+---------+------+------+----------+-------------+
| id | select_type | table | partitions | type | possible_keys | key | key_len | ref | rows | filtered | Extra |
+----+-------------+-------+------------+-------+---------------+---------+---------+------+------+----------+-------------+
| 1 | SIMPLE | a9 | NULL | index | NULL | PRIMARY | 4 | NULL | 1 | 100.00 | Using index |
+----+-------------+-------+------------+-------+---------------+---------+---------+------+------+----------+-------------+
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

**2.4.2、const 被称为“常量”，这个词不好理解，不过出现 const 的话就表示发生下面两种情况：**

- 在整个查询过程中这个表最多只会有一条匹配的行，比如主键 id=1 就肯定只有一行；
- 只需读取一次表数据便能取得所需的结果，且表数据在分解执行计划时读取。

表最多有一个匹配行，const用于比较primary key 或者unique索引。因为只匹配一行数据，所以很快记住一定是用到primary key 或者unique，并且只检索出两条数据的 情况下才会是const,看下面这条语句

```
explain SELECT * FROM `asj_admin_log` limit 1
```

,结果是

![img](http://img.blog.csdn.net/20131108000019375?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvemh1eGluZWxp/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

虽然只搜索一条数据，但是因为没有用到指定的索引，所以不会使用const.继续看下面这个

```
explain SELECT * FROM `asj_admin_log` where log_id = 111
```

![img](http://img.blog.csdn.net/20131108000237546?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvemh1eGluZWxp/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

log_id是主键，所以使用了const。所以说可以理解为const是最优化的。

示例：

建表及插入数据：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
create table a7(id int primary key, c1 char(20) not null, c2 text not null, c3 text not null);
insert into a7 values(1, 'asdfasdf', 'asdfasdf', 'asdfasdf'), (2, 'asdfasdf', 'asdfasdf', 'asdfasdf');

mysql> explain extended select * from a7 where id=1;
+----+-------------+-------+------------+-------+---------------+---------+---------+-------+------+----------+-------+
| id | select_type | table | partitions | type | possible_keys | key | key_len | ref | rows | filtered | Extra |
+----+-------------+-------+------------+-------+---------------+---------+---------+-------+------+----------+-------+
| 1 | SIMPLE | a7 | NULL | const | PRIMARY | PRIMARY | 4 | const | 1 | 100.00 | NULL |
+----+-------------+-------+------------+-------+---------------+---------+---------+-------+------+----------+-------+
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

用 show warnings 查看 MySQL 是如何优化的：

```
mysql> show warnings\G
Message: select '1' AS `id`,'asdfasdf' AS `c1`,'asdfasdf' AS `c2`,'asdfasdf' AS
`c3` from `test`.`a` where 1
```

 

查询返回的结果为：
mysql> select * from a7 where id=1;
+----+----------+----------+----------+
| id | c1 | c2 | c3 |
+----+----------+----------+----------+
| 1 | asdfasdf | asdfasdf | asdfasdf |
+----+----------+----------+----------+

可以看出，返回结果中的字段值都以“值 AS 字段名”的形式直接出现在优化后的 select 语句中。

修改一下查询：
mysql> explain select * from a7 where id in(1,2);
+----+-------------+-------+------------+-------+---------------+---------+---------+------+------+----------+-------------+
| id | select_type | table | partitions | type | possible_keys | key | key_len | ref | rows | filtered | Extra |
+----+-------------+-------+------------+-------+---------------+---------+---------+------+------+----------+-------------+
| 1 | SIMPLE | a7 | NULL | range | PRIMARY | PRIMARY | 4 | NULL | 2 | 100.00 | Using where |
+----+-------------+-------+------------+-------+---------------+---------+---------+------+------+----------+-------------+
当返回结果超过 1 条时， type 便不再为 const 了。

重新建表及插入数据：
create table a8 (id int not null);
insert into a8 value(1),(2),(3);

mysql> explain select * from a8 where id=1;
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-------------+
| id | select_type | table | partitions | type | possible_keys | key | key_len | ref | rows | filtered | Extra |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-------------+
| 1 | SIMPLE | a8 | NULL | ALL | NULL | NULL | NULL | NULL | 3 | 33.33 | Using where |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-------------+
目前表中只有一条 id=1 的记录，但 type 已为 ALL ，因为只有唯一性索引才能保证表中最多只有一条记录，只有这样 type 才有可能为 const 。
为 id 加普通索引后， type 变为 ref ，改为加唯一或主键索引后， type 便变为 const 了。

 

**2.4.3、eq_ref 使用有唯一性索引查找（主键或唯一性索引）**

使用这种索引查找，mysql知道最多只返回一条符合条件的记录，这种访问方法可以在mysql使用主键或者唯一性索引查找时看到，它会将它们与某个参考值做比较。mysql对于这类访问类型的优化做得非常好，因为它知道无需估计匹配行的范围或在找到匹配行后再继续查找。

对于eq_ref的解释，mysql手册是这样说的:"对于每个来自于前面的表的行组合，从该表中读取一行。这可能是最好的联接类型，除了const类型。它用在一个索引的所有部分被联接使用并且索引是UNIQUE或PRIMARY KEY"。eq_ref可以用于使用=比较带索引的列。看下面的语句

建表及插入数据：
create table a5(id int primary key);
create table a5_info(id int primary key, title char(1));
insert into a5 value(1),(2);
insert into a5_info value(1, 'a'),(2, 'b');
mysql> explain select * from a5 join a5_info using(id);
+----+-------------+---------+------------+-------+---------------+---------+---------+------+------+----------+----------------------------------------------------+
| id | select_type | table | partitions | type | possible_keys | key | key_len | ref | rows | filtered | Extra |
+----+-------------+---------+------------+-------+---------------+---------+---------+------+------+----------+----------------------------------------------------+
| 1 | SIMPLE | a5 | NULL | index | PRIMARY | PRIMARY | 4 | NULL | 2 | 100.00 | Using index |
| 1 | SIMPLE | a5_info | NULL | ALL | PRIMARY | NULL | NULL | NULL | 2 | 50.00 | Using where; Using join buffer (Block Nested Loop) |
+----+-------------+---------+------------+-------+---------------+---------+---------+------+------+----------+----------------------------------------------------+
此时 a5_info 每条记录与 a5 一一对应，通过主键 id 关联起来，所以 a5_info 的 type 为 eq_ref。

删除 a_info 的主键：ALTER TABLE `a5_info` DROP PRIMARY KEY;
现在 a_info 已经没有索引了：

mysql> explain select * from a5 join a5_info using(id);
+----+-------------+---------+------------+--------+---------------+---------+---------+--------------------+------+----------+-------------+
| id | select_type | table | partitions | type | possible_keys | key | key_len | ref | rows | filtered | Extra |
+----+-------------+---------+------------+--------+---------------+---------+---------+--------------------+------+----------+-------------+
| 1 | SIMPLE | a5_info | NULL | ALL | NULL | NULL | NULL | NULL | 2 | 100.00 | NULL |
| 1 | SIMPLE | a5 | NULL | eq_ref | PRIMARY | PRIMARY | 4 | ud_omcs.a5_info.id | 1 | 100.00 | Using index |
+----+-------------+---------+------------+--------+---------------+---------+---------+--------------------+------+----------+-------------+
这次 MySQL 调整了执行顺序，先全表扫描 a5_info 表，再对表 a5 进行 eq_ref 查找，因为 a5 表 id 还是主键。

删除 a 的主键：alter table a5 drop primary key;
现在 a 也没有索引了：

mysql> explain select * from a5 join a5_info using(id);
+----+-------------+---------+------------+------+---------------+------+---------+------+------+----------+----------------------------------------------------+
| id | select_type | table | partitions | type | possible_keys | key | key_len | ref | rows | filtered | Extra |
+----+-------------+---------+------------+------+---------------+------+---------+------+------+----------+----------------------------------------------------+
| 1 | SIMPLE | a5 | NULL | ALL | NULL | NULL | NULL | NULL | 2 | 100.00 | NULL |
| 1 | SIMPLE | a5_info | NULL | ALL | NULL | NULL | NULL | NULL | 2 | 50.00 | Using where; Using join buffer (Block Nested Loop) |
+----+-------------+---------+------------+------+---------------+------+---------+------+------+----------+----------------------------------------------------+

现在两个表都使用全表扫描了。

建表及插入数据：
create table a6(id int primary key);
create table a6_info(id int, title char(1), key(id));
insert into a6 value(1),(2);
insert into a6_info value(1, 'a'),(2, 'b');

现在 a6_info 表 id 列变为普通索引（非唯一性索引）：

mysql> explain select * from a6 join a6_info using(id) where a6.id=1;
+----+-------------+---------+------------+-------+---------------+---------+---------+-------+------+----------+-------------+
| id | select_type | table | partitions | type | possible_keys | key | key_len | ref | rows | filtered | Extra |
+----+-------------+---------+------------+-------+---------------+---------+---------+-------+------+----------+-------------+
| 1 | SIMPLE | a6 | NULL | const | PRIMARY | PRIMARY | 4 | const | 1 | 100.00 | Using index |
| 1 | SIMPLE | a6_info | NULL | ref | id | id | 5 | const | 1 | 100.00 | NULL |
+----+-------------+---------+------------+-------+---------------+---------+---------+-------+------+----------+-------------+

a6_info 表 type 变为 ref 类型了。

所以，唯一性索引才会出现 eq_ref （非唯一性索引会出现 ref ），因为唯一，所以最多只返回一条记录，找到后无需继续查找，因此比 ref 更快。

**2.4.4、 ref 非唯一性索引访问**

这是一种索引访问（有时也叫做索引查找），它返回所有匹配某个单个值的行，然而，它可能会找到多个符合条件的行。因此，它是查找和扫描的混合体，此类索引访问只有当使用非唯一性索引或者唯一性索引的非唯一性前缀时才会发生。把它叫做ref是因为索引要跟某个参考值相比较。这个参考值或者是一个常数，或者是来自多表查询前一个表里的结果值。

ref_or_null是ref之上的一个变体，它意味着mysql必须在初次查找的结果里进行第二次查找以找出null条目。

　　 对于每个来自于前面的表的行组合，所有有匹配索引值的行将从这张表中读取。如果联接只使用键的最左边的前缀，或如果键不是UNIQUE或PRIMARY KEY（换句话说，如果联接不能基于关键字选择单个行的话），则使用ref。如果使用的键仅仅匹配少量行，该联接类型是不错的。

看下面这条语句 explain select * from uchome_space where uchome_space.friendnum = 0，得到结果如下，这条语句能搜出1w条数据

![img](http://img.blog.csdn.net/20131108110428796?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvemh1eGluZWxp/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

示例2

建表：
create table a4(a_id int not null, key(a_id));
insert into a4 values(1),(2),(3),(4),(5),(6),(7),(8),(9),(10);
mysql> explain select * from a4 where a_id=1;
+----+-------------+-------+------------+------+---------------+------+---------+-------+------+----------+-------------+
| id | select_type | table | partitions | type | possible_keys | key | key_len | ref | rows | filtered | Extra |
+----+-------------+-------+------------+------+---------------+------+---------+-------+------+----------+-------------+
| 1 | SIMPLE | a4 | NULL | ref | a_id | a_id | 4 | const | 1 | 100.00 | Using index |
+----+-------------+-------+------------+------+---------------+------+---------+-------+------+----------+-------------+

**2.4.5、 ref_or_null** 

　　该联接类型如同ref，但是添加了MySQL可以专门搜索包含NULL值的行。在解决子查询中经常使用该联接类型的优化。

上面这五种情况都是很理想的索引使用情况。

 

**2.4.6、 index_merge**

　　该联接类型表示使用了索引合并优化方法。在这种情况下，key列包含了使用的索引的清单，key_len包含了使用的索引的最长的关键元素。

**2.4.7、 unique_subquery** 

**2.4.8、 index_subquery**

**2.4.9、 range** 给定范围内的检索，使用一个索引来检查行。

使用between或者在where自己里带有>的查询。

还有in()或者or列表，也会显示为范围扫描，然而这两者其实是不同类型的访问，在性能上也有差异（结论：or的效率为O(n)，而in的效率为O(logn)，如果in和or所在列有索引或者主键的话，or和in没啥差别，执行计划和执行时间都几乎一样。如果in和or所在列没有索引的话in效率高。）。此类扫描的开销跟索引类型的相当。）。

举例说明：
建表：
create table aaa(a_id int not null, key(a_id));
insert into aaa values(1),(2),(3),(4),(5),(6),(7),(8),(9),(10);

mysql> explain select * from aaa where a_id > 1;
+----+-------------+-------+------------+-------+---------------+------+---------+------+------+----------+--------------------------+
| id | select_type | table | partitions | type | possible_keys | key | key_len | ref | rows | filtered | Extra |
+----+-------------+-------+------------+-------+---------------+------+---------+------+------+----------+--------------------------+
| 1 | SIMPLE | aaa | NULL | range | a_id | a_id | 4 | NULL | 9 | 100.00 | Using where; Using index |
+----+-------------+-------+------------+-------+---------------+------+---------+------+------+----------+--------------------------+

IN 比较符也会用 range 表示：
mysql> explain select * from aaa where a_id in (1,3,4);
+----+-------------+-------+------------+-------+---------------+------+---------+------+------+----------+--------------------------+
| id | select_type | table | partitions | type | possible_keys | key | key_len | ref | rows | filtered | Extra |
+----+-------------+-------+------------+-------+---------------+------+---------+------+------+----------+--------------------------+
| 1 | SIMPLE | aaa | NULL | range | a_id | a_id | 4 | NULL | 3 | 100.00 | Using where; Using index |
+----+-------------+-------+------------+-------+---------------+------+---------+------+------+----------+--------------------------+

**2.4.10、 index**     该联接类型与ALL相同都是扫描表，但index只对索引树进行扫描，而ALL是是对数据表文件的扫描。这通常比ALL快，因为索引文件通常比数据文件小。（也就是说虽然all和Index都是读全表，但index是从索引中读取的，而all是从硬盘中读的）主要优点是避免了排序，因为索引是排好序的。

Extra列中看到“Using index”，说明mysql正在使用覆盖索引，只扫描索引的数据。

举例说明：
1、建表：
create table aa(a_id int not null, key(a_id));
insert into aa value(1),(2);
2、查询
mysql> explain select * from aa;
+----+-------------+-------+------------+-------+---------------+------+---------+------+------+----------+-------------+
| id | select_type | table | partitions | type | possible_keys | key | key_len | ref | rows | filtered | Extra |
+----+-------------+-------+------------+-------+---------------+------+---------+------+------+----------+-------------+
| 1 | SIMPLE | aa | NULL | index | NULL | a_id | 4 | NULL | 2 | 100.00 | Using index |
+----+-------------+-------+------------+-------+---------------+------+---------+------+------+----------+-------------+

**2.4.11、  ALL** 
全表扫描，MySQL 从头到尾扫描整张表查找行。

```
mysql> explain select * from a;
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-------+
| id | select_type | table | partitions | type | possible_keys | key | key_len | ref | rows | filtered | Extra |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-------+
| 1 | SIMPLE | a | NULL | ALL | NULL | NULL | NULL | NULL | 4 | 100.00 | NULL |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-------+
```

如果加上 limit 如 select * from a limit 10 MySQL 会扫描 10 行，但扫描方式不会变，还是从头到尾扫描。

例如，在查询里使用了Limit，或者在Extra列中显示“Using distinct/not exists”

### 2.5、possible_keys 列     可能被用到的索引

一个会列出大量可能的索引(例如多于3 个)的QEP 意味着备选索引数量太多了，同时也可能提示存在一个无效的单列索引。

**可以用第2 章详细介绍过的SHOW INDEXES 命令来检查索引是否有效且是否具有合适的基数。《**[SHOW INDEX语法的实际应用](http://www.cnblogs.com/duanxz/p/5106615.html)**》**

为查询确定QEP 的速度也会影响到查询的性能。如果发现有大量的可能的索引，则意味着这些索引没有被使用到。

相关的QEP 列还包括key 列。

示例：

建表：
create table a10 (a_id int primary key, a_age int, key (a_id, a_age));
insert into a10 values(1,2),(2,3);
mysql> select * from a10;
+------+-------+
| a_id | a_age |
+------+-------+
| 1 | 2 |
| 2 | 3 |
+------+-------+
此表有 主键及普通索引 两个索引。
mysql> explain select * from a10 where a_id=2 and a_age=2;
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-----------------------------------------------------+
| id | select_type | table | partitions | type | possible_keys | key | key_len | ref | rows | filtered | Extra |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-----------------------------------------------------+
| 1 | SIMPLE | NULL | NULL | NULL | NULL | NULL | NULL | NULL | NULL | NULL | Impossible WHERE noticed after reading const tables |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-----------------------------------------------------+
1 row in set, 1 warning (0.00 sec)

mysql> explain select * from a10 where a_id=2 and a_age=3;
+----+-------------+-------+------------+-------+---------------+---------+---------+-------+------+----------+-------+
| id | select_type | table | partitions | type | possible_keys | key | key_len | ref | rows | filtered | Extra |
+----+-------------+-------+------------+-------+---------------+---------+---------+-------+------+----------+-------+
| 1 | SIMPLE | a10 | NULL | const | PRIMARY,a_id | PRIMARY | 4 | const | 1 | 100.00 | NULL |
+----+-------------+-------+------------+-------+---------------+---------+---------+-------+------+----------+-------+
1 row in set, 1 warning (0.00 sec)

### 2.6、key 列 

　　key 列指出mysql优化器决定选择使用的索引来优化对该表的访问。一般来说SQL 查询中的每个表都仅使用一个索引。也存在索引合并的少数例外情况，如给定表上用到了两个或者更多索引。查询过程中实际使用的索引。

mysql> explain select * from a10 where a_id=2 and a_age=3;
+----+-------------+-------+------------+-------+---------------+---------+---------+-------+------+----------+-------+
| id | select_type | table | partitions | type | possible_keys | key | key_len | ref | rows | filtered | Extra |
+----+-------------+-------+------------+-------+---------------+---------+---------+-------+------+----------+-------+
| 1 | SIMPLE | a10 | NULL | const | PRIMARY,a_id | PRIMARY | 4 | const | 1 | 100.00 | NULL |
+----+-------------+-------+------------+-------+---------------+---------+---------+-------+------+----------+-------+
1 row in set, 1 warning (0.00 sec)

### 2.7、key_len 列

　　key_len 列定义了mysql在索引里使用的字节数。如果mysql正在使用的只是索引里的某些列，那么就可以用这个值来算出具体是哪些列。

在mysql5.5及以前的版本里，只能使用索引的最左前缀。例如，sakila.film_actor的主键是两个SMALLINT列，并且每个SMALLINT列是两个字节，那么索引中的每项是4个字节。也即说明**key_len通过查找表的定义而被计算出，而不是表中的数据**。

用于SQL 语句的连接条件的键的长度。此列值对于确认索引的有效性以及多列索引中用到的列的数目很重要。索引字段最大可能使用的长度。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
mysql> explain select * from a10 where a_id=2 and a_age=3;
+----+-------------+-------+------------+-------+---------------+---------+---------+-------+------+----------+-------+
| id | select_type | table | partitions | type | possible_keys | key | key_len | ref | rows | filtered | Extra |
+----+-------------+-------+------------+-------+---------------+---------+---------+-------+------+----------+-------+
| 1 | SIMPLE | a10 | NULL | const | PRIMARY,a_id | PRIMARY | 4 | const | 1 | 100.00 | NULL |
+----+-------------+-------+------------+-------+---------------+---------+---------+-------+------+----------+-------+
1 row in set, 1 warning (0.00 sec)
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

a_id 是 int 类型，int 的长度是 4 字节，所以 key_len 为 4。

示例2：

此列的一些示例值如下所示：

```
key_len: 4 // INT NOT NULL
key_len: 5 // INT NULL
key_len: 30 // CHAR(30) NOT NULL
key_len: 32 // VARCHAR(30) NOT NULL
key_len: 92 // VARCHAR(30) NULL CHARSET=utf8
```

从这些示例中可以看出，是否可以为空、可变长度的列以及key_len 列的值只和用在**连接**和**WHERE 条件中**的**索引的列有关**。索引中的其他列会在ORDER BY或者GROUP BY 语句中被用到。下面这个来自于著名的开源博客软件WordPress 的表展示了如何以最佳方式使用带有定义好的表索引的SQL 语句：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
CREATE TABLE `wp_posts` (  
`ID` bigint(20) unsigned NOT NULL AUTO_INCREMENT,  
`post_date` datetime NOT NULL DEFAULT '0000-00-00 00:00:00',  
`post_status` varchar(20) NOT NULL DEFAULT 'publish' ,  
`post_type` varchar(20) NOT NULL DEFAULT 'post',  
PRIMARY KEY (`ID`),  
KEY `type_status_date`(`post_type`,`post_status`,`post_date`,`ID`)  
) DEFAULT CHARSET=utf8;
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

这个表的索引包括post_type、post_status、post_date 以及ID列。下面是一个演示索引列用法的SQL 查询：

```
EXPLAIN SELECT ID, post_title FROM wp_posts WHERE post_type=’post’ AND post_date > ‘2010-06-01’;
```

这个查询的QEP 返回的key_len 是62。这说明只有post_type列上的索引用到了(因为(20×3)+2=62)。尽管查询在WHERE 语句中使用了post_type 和post_date 列，但只有post_type 部分被用到了。其他索引没有被使用的原因是MySQL 只能使用定义索引的最左边部分。为了更好地利用这个索引，可以修改这个查询来调整索引的列。请看下面的示例：

```
mysql> EXPLAIN SELECT ID, post_title
-> FROM wp_posts  
-> WHERE post_type='post'  
-> AND post_status='publish'  
-> AND post_date > '2010-06-01';
```

在SELECT查询的添加一个post_status 列的限制条件后，QEP显示key_len 的值为132，这意味着post_type、post_status、post_date三列(62+62+8，(20×3)+2，(20×3)+2，8)都被用到了。此外，这个索引的主码列ID 的定义是使用MyISAM 存储索引的遗留痕迹。当使用InnoDB 存储引擎时，在非主码索引中包含主码列是多余的，这可以从key_len 的用法看出来。

相关的QEP 列还包括带有Using index 值的Extra 列。

### 2.8、ref 列

　　显示了之前的表在key列记录的索引中查找**值**所用的列或常量。

指出对 key 列所选择的索引的查找方式，常见的值有 const, func, NULL, 具体字段名。当 key 列为 NULL ，即不使用索引时，此值也相应的为 NULL 。

建表及插入数据：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
create table a11(id int primary key, age int);
insert into a11 value(1, 10),(2, 10);
mysql> desc select * from a11 where age=10;
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-------------+
| id | select_type | table | partitions | type | possible_keys | key | key_len | ref | rows | filtered | Extra |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-------------+
| 1 | SIMPLE | a11 | NULL | ALL | NULL | NULL | NULL | NULL | 2 | 50.00 | Using where |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-------------+
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

当 key 列为 NULL ， ref 列也相应为 NULL 。
mysql> desc select * from a11 where id=1;
+----+-------------+-------+------------+-------+---------------+---------+---------+-------+------+----------+-------+
| id | select_type | table | partitions | type | possible_keys | key | key_len | ref | rows | filtered | Extra |
+----+-------------+-------+------------+-------+---------------+---------+---------+-------+------+----------+-------+
| 1 | SIMPLE | a11 | NULL | const | PRIMARY | PRIMARY | 4 | const | 1 | 100.00 | NULL |
+----+-------------+-------+------------+-------+---------------+---------+---------+-------+------+----------+-------+
这次 key 列使用了主键索引，where id=1 中 1 为常量， ref 列的 const 便是指这种常量。

mysql> desc select * from a11 where id in(1,2);
+----+-------------+-------+------------+-------+---------------+---------+---------+------+------+----------+-------------+
| id | select_type | table | partitions | type | possible_keys | key | key_len | ref | rows | filtered | Extra |
+----+-------------+-------+------------+-------+---------------+---------+---------+------+------+----------+-------------+
| 1 | SIMPLE | a11 | NULL | range | PRIMARY | PRIMARY | 4 | NULL | 2 | 100.00 | Using where |
+----+-------------+-------+------------+-------+---------------+---------+---------+------+------+----------+-------------+
不理解 ref 为 NULL 的含意，比如上面这个查询， key 列有使用索引，但 ref 列却为 NULL 。网上搜索及查阅了一下 MySQL 帮助手册都没有找到相关的描述。

再建表及插入数据：
create table a12(id int primary key, a_name int not null);
create table b12(id int primary key, b_name int not null);
insert into a12 value(1, 1),(2, 2),(3, 3);
insert into b12 value(1, 111),(2, 222),(3, 333);

mysql> explain select * from a12 join b12 using(id);
...+-------+--------+...+---------+...+-----------+...
...| table | type |...| key |...| ref |...
...+-------+--------+...+---------+...+-----------+...
...| a | ALL |...| NULL |...| NULL |...
...| b | eq_ref |...| PRIMARY |...| test.a.id |...
...+-------+--------+...+---------+...+-----------+...

这里 test.a.id 即为具体字段，意为根据表 a 的 id 字段的值查找表 b 的主键索引。
mysql> explain select * from a12 join b12 using(id) where b12.id=3;
+----+-------------+-------+------------+-------+---------------+---------+---------+-------+------+----------+-------+
| id | select_type | table | partitions | type | possible_keys | key | key_len | ref | rows | filtered | Extra |
+----+-------------+-------+------------+-------+---------------+---------+---------+-------+------+----------+-------+
| 1 | SIMPLE | a12 | NULL | const | PRIMARY | PRIMARY | 4 | const | 1 | 100.00 | NULL |
| 1 | SIMPLE | b12 | NULL | const | PRIMARY | PRIMARY | 4 | const | 1 | 100.00 | NULL |
+----+-------------+-------+------------+-------+---------------+---------+---------+-------+------+----------+-------+
因为 a join b 的条件为 id 相等，而 b.id=1 ，就是 a.id 也为 1 ，所以 a,b 两个表的 ref 列都为 const 。

ref 为 func 的情况出现在子查询中，暂不明其原理：
mysql> explain select * from a12 where id in (select id from b12 where id in (1,2));
+----+-------------+-------+------------+--------+---------------+---------+---------+----------------+------+----------+--------------------------+
| id | select_type | table | partitions | type | possible_keys | key | key_len | ref | rows | filtered | Extra |
+----+-------------+-------+------------+--------+---------------+---------+---------+----------------+------+----------+--------------------------+
| 1 | SIMPLE | b12 | NULL | index | PRIMARY | PRIMARY | 4 | NULL | 3 | 66.67 | Using where; Using index |
| 1 | SIMPLE | a12 | NULL | eq_ref | PRIMARY | PRIMARY | 4 | ud_omcs.b12.id | 1 | 100.00 | NULL |
+----+-------------+-------+------------+--------+---------------+---------+---------+----------------+------+----------+--------------------------+

### 2.9、rows 列

这一列是mysql估计为了找到所需的行而要读取的行数。这个数字是内嵌循环关联计划里的循环数目，也就是说它不是mysql认为它最终要从表里读取出来的行数，而是mysql为了找到符合查询的每一点上标准的那些行而必须读取的行的平均数。

rows 列提供了试图分析所有存在于累计结果集中的行数目的MySQL 优化器估计值。QEP 很容易描述这个很困难的统计量。

查询中总的读操作数量是基于合并之前行的每一行的rows 值的连续积累而得出的。这是一种嵌套行算法。

 

以连接两个表的QEP 为例。通过id=1 这个条件找到的第一行的rows 值为1，这等于对第一个表做了一次读操作。第二行是通过id=2 找到的，rows 的值为5。这等于有5 次读操作符合当前1 的积累量。参考两个表，读操作的总数目是6。在另一个QEP中，第一rows 的值是5，第二rows 的值是1。这等于第一个表有5 次读操作，对5个积累量中每个都有一个读操作。因此两个表总的读操作的次数是10(5+5)次。

最好的估计值是1，一般来说这种情况发生在当寻找的行在表中可以通过主键或者唯一键找到的时候。

在下面的QEP 中，外面的嵌套循环可以通过id=1 来找到，其估计的物理行数是1。第二个循环处理了10行。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
********************* 1. row ***********************  
id: 1  
select_type: SIMPLE  
table: p  
type: const  
possible_keys: PRIMARY  
key: PRIMARY  
key_len: 4  
ref: const  
rows: 1  
Extra:  
********************* 2. row ***********************  
id: 1  
select_type: SIMPLE  
table: c  
type: ref  
possible_keys: parent_id  
key: parent_id  
key_len: 4  
ref: const  
rows: 10  
Extra:
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

可以使用SHOW STATUS 命令来查看实际的行操作。这个命令可以提供最佳的确认物理行操作的方式。请看下面的示例：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
mysql> SHOW SESSION STATUS LIKE 'Handler_read%';  
+-----------------------+-------+  
| Variable_name         | Value |  
+-----------------------+-------+  
| Handler_read_first    | 0     |  
| Handler_read_key      | 0     |   
| Handler_read_last     | 0     |  
| Handler_read_next     | 0     |  
| Handler_read_prev     | 0     |  
| Handler_read_rnd      | 0     |  
| Handler_read_rnd_next | 11    |  
+-----------------------+-------+  
7 rows in set (0.00 sec)
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

在下一个QEP 中，通过id=1 找到的外层嵌套循环估计有160行。第二个循环估计有1 行。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
********************* 1. row ***********************  
id: 1  
select_type: SIMPLE  
table: p  
type: ALL  
possible_keys: NULL  
key: NULL  
key_len: NULL  
ref: NULL  
rows: 160  
Extra:  

********************* 2. row ***********************  
id: 1  
select type: SIMPLE  
table: c  
type: ref  
possible_keys: PRIMARY,parent_id  
key: parent_id  
key_len: 4  
ref: test.p.parent_id  
rows: 1  
Extra: Using where  
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

通过SHOW STATUS 命令可以查看实际的行操作，该命令表明物理读操作数量大幅增加。请看下面的示例：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
mysql> SHOW SESSION STATUS LIKE 'Handler_read%';  
+--------------------------------------+---------+  
| Variable_name | Value |  
+--------------------------------------+---------+  
| Handler_read_first | 1 |  
| Handler_read_key | 164 |  
| Handler_read_last | 0 |  
| Handler_read_next | 107 |  
| Handler_read_prev | 0 |  
| Handler_read_rnd | 0 |  
| Handler_read_rnd_next | 161 |  
+--------------------------------------+---------+  
相关的QEP 列还包括key列。
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

### 2.10、**filtered** 列

　　在mysql5.1里新加的，在使用explain extended时出现。它显示的是针对表里符合条件的记录数的百分比所做的一个悲观估算值。

　　filtered 列给出了一个百分比的值，这个百分比值和rows 列的值相乘，可以估计出那些将要和QEP 中的前一个表进行连接的行的数目。前一个表就是指id 列的值比当前表的id 小的表。这一列只有在EXPLAIN EXTENDED 语句中才会出现。

### 2.11、Extra 列

　　显示上述信息之外的其它信息，但却很重要。Extra 列可以包含多个值，可以有很多不同的取值，并且这些值还在随着MySQL 新版本的发布而进一步增加。下面给出常用值的列表。你可以从下面的地址找到更全面的值的列表：http://dev.mysql.com/doc/refman/5.5/en/explain-output.html。

**2.11.1 Distinct**     MySQL发现第1个匹配行后，停止为当前的行组合搜索更多的行。一直没见过这个值

**2.11.2** **Not exists**  

**2.11.3** **range checked for each record**

没有找到合适的索引

**2.11.4 using filesort**   

　　若查询所需的排序与使用的索引的排序一致，因为索引是已排序的，因此按索引的顺序读取结果返回，否则，在取得结果后，还需要按查询所需的顺序对结果进行排序，这时就会出现 Using filesort。需要优化

建表及插入数据：
create table a19(a_id int, b_id int);
insert into a19 values(1,1),(1,1),(2,1),(2,2),(3,1);
mysql> explain select * from a19 order by a_id;
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+----------------+
| id | select_type | table | partitions | type | possible_keys | key | key_len | ref | rows | filtered | Extra |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+----------------+
| 1 | SIMPLE | a19 | NULL | ALL | NULL | NULL | NULL | NULL | 5 | 100.00 | Using filesort |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+----------------+
对于没有索引的表，只要 order by 必会出现 Using filesort 。

现在增加索引：create index a_id on a19(a_id);
把表 a 的记录增加到约 100w(1048576) 条， a_id 与 b_id 都是随机生成的数字：

mysql> select * from a order by rand() limit 10;
+-------+--------+
| a_id | b_id |
+-------+--------+
| 61566 | 961297 |
| 33951 | 680542 |
| ..... | ...... |
+-------+--------+
mysql> explain select * from a19 order by rand() limit 10;
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+---------------------------------+
| id | select_type | table | partitions | type | possible_keys | key | key_len | ref | rows | filtered | Extra |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+---------------------------------+
| 1 | SIMPLE | a19 | NULL | ALL | NULL | NULL | NULL | NULL | 5 | 100.00 | Using temporary; Using filesort |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+---------------------------------+
同样是 Using filesort ，type 为 ALL ，全表扫描。听说“取全表数据根据ID排序，走索引一定不如直接查，因为可以减少因为需要索引改变数据访问顺序造成随机IO的概率，数据库放弃索引是应该的”，参考：
http://isky000.com/database/mysql_order_by_implement#comment-2981

当 type 为 rang、 ref 或者 index 的时候才有可能利用索引排序，其它，如 ALL ，都无法通过索引排序，此时若有 order by ，如上例，便会出现 Using filesort 。

现在增加 where 子句：
mysql> explain select * from a19 where a_id=10 order by a_id;
+----+-------------+-------+------------+------+---------------+------+---------+-------+------+----------+-------+
| id | select_type | table | partitions | type | possible_keys | key | key_len | ref | rows | filtered | Extra |
+----+-------------+-------+------------+------+---------------+------+---------+-------+------+----------+-------+
| 1 | SIMPLE | a19 | NULL | ref | a_id | a_id | 5 | const | 1 | 100.00 | NULL |
+----+-------------+-------+------------+------+---------------+------+---------+-------+------+----------+-------+
查询走了索引 a_id ，此时 type 为 ref ，直接按索引顺序返回，没有 Using filesort 。
修改 where 子句：
mysql> explain select * from a19 where a_id>10 and a_id<100 order by a_id;
+----+-------------+-------+------------+-------+---------------+------+---------+------+------+----------+-----------------------+
| id | select_type | table | partitions | type | possible_keys | key | key_len | ref | rows | filtered | Extra |
+----+-------------+-------+------------+-------+---------------+------+---------+------+------+----------+-----------------------+
| 1 | SIMPLE | a19 | NULL | range | a_id | a_id | 5 | NULL | 1 | 100.00 | Using index condition |
+----+-------------+-------+------------+-------+---------------+------+---------+------+------+----------+-----------------------+
同样利用索引排序，没有 Using filesort 。

再修改 where 子句：
mysql> explain select * from a19 where a_id >1 order by a_id;
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-----------------------------+
| id | select_type | table | partitions | type | possible_keys | key | key_len | ref | rows | filtered | Extra |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-----------------------------+
| 1 | SIMPLE | a19 | NULL | ALL | a_id | NULL | NULL | NULL | 5 | 60.00 | Using where; Using filesort |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-----------------------------+

又出现 Using filesort 且 type 变为 ALL 。注意以上例子的 rows 列，此列表示 MySQL 估计查询需要读取的行数，分别为 1048576, 8, 712, 1048576 ，特别注意最后两个数字： 712, 1048576 。

可见，当索引能为查询排除大部份行时（ a_id=10 时约读取 8 行，排除了大部份， a_id>10 and a_id<100 时约读取 712 行，同样排除了大部份）便使用索引，否则，如 a_id>10 时约读取 1048576 ， MySQL 直接改用全表扫描，再 Using filesort 。也就是说， MySQL 会根据表中的信息及查询来决定使用任种方式。

关于 MySQL 读取数据表的方式，可参考（暂缺参考资料），就会明白为什么需读取 1048576 行时，先读索引再读表数据还不如全表扫描了。

对于多字段排序(order by a, b)及带 group by 的查询，可参考 MySQL 帮助手册 7.2.12. MySQL如何优化ORDER BY 。

**2.11.5** **using index** 

此查询使用了覆盖索引(Covering Index)，即通过索引就能返回结果，无需访问表。若没显示"Using index"表示读取了表数据。

建表及插入数据：
create table a13 (id int primary key, age int);
insert into a13 value(1, 10),(2, 10);
mysql> explain select id from a13;
+----+-------------+-------+------------+-------+---------------+---------+---------+------+------+----------+-------------+
| id | select_type | table | partitions | type | possible_keys | key | key_len | ref | rows | filtered | Extra |
+----+-------------+-------+------------+-------+---------------+---------+---------+------+------+----------+-------------+
| 1 | SIMPLE | a13 | NULL | index | NULL | PRIMARY | 4 | NULL | 2 | 100.00 | Using index |
+----+-------------+-------+------------+-------+---------------+---------+---------+------+------+----------+-------------+
因为 id 为主键索引，索引中直接包含了 id 的值，所以无需访问表，直接查找索引就能返回结果。

mysql> explain select age from a13;
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-------+
| id | select_type | table | partitions | type | possible_keys | key | key_len | ref | rows | filtered | Extra |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-------+
| 1 | SIMPLE | a13 | NULL | ALL | NULL | NULL | NULL | NULL | 2 | 100.00 | NULL |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-------+
age 列没有索引，因此没有 Using index ，意即需要访问表。
为 age 列添加索引：
create table a14 (id int primary key, age int);
insert into a14 value(1, 10),(2, 10);
create index age on a14(id, age);
mysql> explain select age from a14;
+----+-------------+-------+------------+-------+---------------+------+---------+------+------+----------+-------------+
| id | select_type | table | partitions | type | possible_keys | key | key_len | ref | rows | filtered | Extra |
+----+-------------+-------+------------+-------+---------------+------+---------+------+------+----------+-------------+
| 1 | SIMPLE | a14 | NULL | index | NULL | age | 9 | NULL | 2 | 100.00 | Using index |
+----+-------------+-------+------------+-------+---------------+------+---------+------+------+----------+-------------+
现在索引 age 中也包含了 age 列的值，因此不用访问表便能返回结果了。

建表：create table a15(id int auto_increment primary key, age int, name char(10));
插入 100w 条数据：insert into a15 value(null, rand()*100000000, 'jack');

 

**2.11.6** **using temporary** **使用到临时表**

　　为了解决查询，MySQL需要创建一个临时表来容纳结果。典型情况如查询包含可以按不同情况列出列的GROUP BY和ORDER BY子句时。

出现using temporary就说明语句需要优化了，举个例子来说

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
EXPLAIN SELECT ads.id FROM ads, city WHERE   city.city_id = 8005   AND ads.status = 'online'   AND city.ads_id=ads.id ORDER BY ads.id desc

id  select_type  table   type    possible_keys   key      key_len  ref                     rows  filtered  Extra                          
------  -----------  ------  ------  --------------  -------  -------  --------------------  ------  --------  -------------------------------
     1  SIMPLE       city    ref     ads_id,city_id  city_id  4        const                   2838    100.00  Using temporary; Using filesort
     1  SIMPLE       ads     eq_ref  PRIMARY         PRIMARY  4        city.ads_id       1    100.00  Using where    
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

这条语句会使用using temporary,而下面这条语句则不会

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
EXPLAIN SELECT ads.id FROM ads, city WHERE   city.city_id = 8005   AND ads.status = 'online'   AND city.ads_id=ads.id ORDER BY city.ads_id desc

id  select_type  table   type    possible_keys   key      key_len  ref                     rows  filtered  Extra                      
------  -----------  ------  ------  --------------  -------  -------  --------------------  ------  --------  ---------------------------
     1  SIMPLE       city    ref     ads_id,city_id  city_id  4        const                   2838    100.00  Using where; Using filesort
     1  SIMPLE       ads    eq_ref  PRIMARY         PRIMARY  4        city.ads_id       1    100.00  Using where    
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

这是为什么呢？他俩之间只是一个order by不同，MySQL 表关联的算法是 Nest Loop Join，是通过驱动表的结果集作为循环基础数据，然后一条一条地通过该结果集中的数据作为过滤条件到下一个表中查询数据，然后合并结果。EXPLAIN 结果中，第一行出现的表就是驱动表（Important!）以上两个查询语句，驱动表都是 city，如上面的执行计划所示！

对驱动表可以直接排序，对非驱动表（的字段排序）需要对循环查询的合并结果（临时表）进行排序（Important!）

因此，order by ads.id desc 时，就要先 using temporary 了！

驱动表的定义

[wwh999](http://blog.csdn.net/wwh999/article/details/643493) 在 2006年总结说，当进行多表连接查询时， [驱动表] 的定义为：
1）指定了联接条件时，满足查询条件的记录行数少的表为[驱动表]；
2）未指定联接条件时，行数少的表为[驱动表]（Important!）。

示例2：

建表及插入数据：
create table a18(a_id int, b_id int);
insert into a18 values(1,1),(1,1),(2,1),(2,2),(3,1);
mysql> explain select distinct a_id from a18;
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-----------------+
| id | select_type | table | partitions | type | possible_keys | key | key_len | ref | rows | filtered | Extra |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-----------------+
| 1 | SIMPLE | a18 | NULL | ALL | NULL | NULL | NULL | NULL | 5 | 100.00 | Using temporary |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-----------------+
MySQL 使用临时表来实现 distinct 操作。



**2.11.7、Using where**
表示 MySQL 服务器从存储引擎收到行后再进行“后过滤”（Post-filter）。所谓“后过滤”，就是先读取整行数据，再检查此行是否符合 where 句的条件，符合就留下，不符合便丢弃。因为检查是在读取行后才进行的，所以称为“后过滤”。

建表及插入数据：
create table a16 (num_a int not null, num_b int not null, key(num_a));
insert into a16 value(1,1),(1,2),(2,1),(2,2);
mysql> explain select * from a16 where num_a=1;
+----+-------------+-------+------------+------+---------------+-------+---------+-------+------+----------+-------+
| id | select_type | table | partitions | type | possible_keys | key | key_len | ref | rows | filtered | Extra |
+----+-------------+-------+------------+------+---------------+-------+---------+-------+------+----------+-------+
| 1 | SIMPLE | a16 | NULL | ref | num_a | num_a | 4 | const | 2 | 100.00 | NULL |
+----+-------------+-------+------------+------+---------------+-------+---------+-------+------+----------+-------+

虽然查询中有 where 子句，但只有 num_a=1 一个条件，且 num_a 列存在索引，通过索引便能确定返回的行，无需进行“后过滤”。
所以，并非带 WHERE 子句就会显示"Using where"的。
mysql> explain select * from a16 where num_a=1 and num_b=1;
+----+-------------+-------+------------+------+---------------+-------+---------+-------+------+----------+-------------+
| id | select_type | table | partitions | type | possible_keys | key | key_len | ref | rows | filtered | Extra |
+----+-------------+-------+------------+------+---------------+-------+---------+-------+------+----------+-------------+
| 1 | SIMPLE | a16 | NULL | ref | num_a | num_a | 4 | const | 2 | 25.00 | Using where |
+----+-------------+-------+------------+------+---------------+-------+---------+-------+------+----------+-------------+

此查询增加了条件 num_b=1 ，此列没有索引，但可以看到查询同样能使用 num_a 索引。 MySQL 先通过索引 num_a 找到 num_a=1 的行，然后读取整行数据，再检查 num_b 是否等于 1 ，执行过程看上去象这样：

num_a索引|num_b 没有索引，属于行数据
+-------+-------+
| num_a | num_b | where 子句(num_b=1)
+-------+-------+
| 1 | 1 | 符合
| 1 | 2 | 不符合
| ... | ... | ...
+-------+-------+

在《高性能 MySQL 》（第二版）P144(pdf.167) 页有更形象的说明图片（图 4-5 MySQL 通过整表扫描查找数据）。

字段是否允许 NULL 对 Using where 的影响：

建表及插入数据：
create table a17 (num_a int null, num_b int null, key(num_a));
insert into a17 value(1,1),(1,2),(2,1),(2,2);

这次 num_a, num_b 字段允许为空。

在上例 num_a not null 时， num_a 索引的长度 key_len 为 4 ，当 num_a null 时， num_a 索引的长度变为了 5 ：

mysql> explain select * from a17 where num_a is null;
+----+-------------+-------+------------+------+---------------+-------+---------+-------+------+----------+-----------------------+
| id | select_type | table | partitions | type | possible_keys | key | key_len | ref | rows | filtered | Extra |
+----+-------------+-------+------------+------+---------------+-------+---------+-------+------+----------+-----------------------+
| 1 | SIMPLE | a17 | NULL | ref | num_a | num_a | 5 | const | 1 | 100.00 | Using index condition |
+----+-------------+-------+------------+------+---------------+-------+---------+-------+------+----------+-----------------------+
1 row in set, 1 warning (0.00 sec)

mysql> explain select * from a17 where num_a=3;
+----+-------------+-------+------------+------+---------------+-------+---------+-------+------+----------+-------+
| id | select_type | table | partitions | type | possible_keys | key | key_len | ref | rows | filtered | Extra |
+----+-------------+-------+------------+------+---------------+-------+---------+-------+------+----------+-------+
| 1 | SIMPLE | a17 | NULL | ref | num_a | num_a | 5 | const | 1 | 100.00 | NULL |
+----+-------------+-------+------------+------+---------------+-------+---------+-------+------+----------+-------+

 

 

**2.11.8、**Using join buffer

 

这个值强调了在获取连接条件时没有使用索引，并且需要连接缓冲区来存储中间结果。如果出现了这个值，那应该注意，根据查询的具体情况可能需要添加索引来改进性能。

 

**2.11.9**. Impossible where

 

这个值强调了where 语句会导致没有符合条件的行。请看下面的示例：mysql> EXPLAIN SELECT * FROM user WHERE 1=2;

 

**2.11.10**. Select tables optimized away

 

这个值意味着仅通过使用索引，优化器可能仅从聚合函数结果中返回一行。

 

**2.11.11、** Distinct

 

这个值意味着MySQL 在找到第一个匹配的行之后就会停止搜索其他行。

 

**2.11.12、** Index merges

 

当MySQL 决定要在一个给定的表上使用超过一个索引的时候，就会出现以下格式中的一个，详细说明使用的索引以及合并的类型。

 

 Using sort_union(…)

 Using union(…)

 Using intersect(…)

# EXPLAIN结果中哪些信息要引起关注

我们使用EXPLAIN解析SQL执行计划时，如果有下面几种情况，就需要特别关注下了：

首先看下 **type** 这列的结果，如果有类型是 ALL 时，表示预计会进行全表扫描（full table scan）。通常全表扫描的代价是比较大的，建议创建适当的索引，通过索引检索避免全表扫描。此外，全索引扫描（full index scan）的代价有时候是比全表扫描还要高的，除非是基于InnoDB表的主键索引扫描。

再来看下 **Extra** 列的结果，如果有出现 **Using temporary** 或者 **Using filesort** 则要多加关注：

**Using temporary**，表示需要创建临时表以满足需求，通常是因为GROUP BY的列没有索引，或者GROUP BY和ORDER BY的列不一样，也需要创建临时表，建议添加适当的索引。

**Using filesort**，表示无法利用索引完成排序，也有可能是因为多表连接时，排序字段不是驱动表中的字段，因此也没办法利用索引完成排序，建议添加适当的索引。

**Using where**，通常是因为全表扫描或全索引扫描时（**type** 列显示为 **ALL** 或 **index**），又加上了WHERE条件，建议添加适当的索引。

暂时想到上面几个，如果有遗漏，以后再补充。