# [mysql limit查询（分页查询）探究](https://www.cnblogs.com/JMLiu/p/8384076.html)

# **MySQL的Limit子句**

 

```
LIMIT offset,length
```

 

 

Limit子句可以被用于强制 SELECT 语句返回指定的记录数。Limit接受一个或两个数字参数。参数必须是一个整数常量。如果给定两个参数，第一个参数指定第一个返回记录行的偏移量，第二个参数指定返回记录行的最大数目。

 

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
//初始记录行的偏移量是 0(而不是 1)：
　　mysql> SELECT * FROM table LIMIT 5,10; //检索记录行6-15

　　//为了检索从某一个偏移量到记录集的结束所有的记录行，可以指定第二个参数为 -1：
　　mysql> SELECT * FROM table LIMIT 95,-1; // 检索记录行 96-last

　　//如果只给定一个参数，它表示返回最大的记录行数目。换句话说，LIMIT n 等价于 LIMIT 0,n：
　　mysql> SELECT * FROM table LIMIT 5;     //检索前 5 个记录行
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

**Limit的效率有时候很高**

　　常说的Limit的执行效率高，是对于一种特定条件下来说的：即数据库的数量很大，但是只需要查询一部分数据的情况。
　　高效率的原理是：避免全表扫描，提高查询效率。

　　比如：每个用户的email是唯一的，如果用户使用email作为用户名登陆的话，就需要查询出email对应的一条记录。
　　SELECT * FROM t_user WHERE email=?;
　　上面的语句实现了查询email对应的一条用户信息，但是由于email这一列没有加索引，会导致全表扫描，效率会很低。
　　SELECT * FROM t_user WHERE email=? LIMIT 1;
　　加上LIMIT 1，只要找到了对应的一条记录，就不会继续向下扫描了，效率会大大提高。

 

# **Limit的效率有时候比较低**

 

　　在一种情况下，使用limit效率低，那就是：只使用limit来查询语句，并且偏移量特别大的情况

 

　　做以下实验：
      语句1：
         　　select * from table limit 150000,1000;
　　语句2:
         　　select * from table while id>=150000 limit 1000;
　　语句1为0.2077秒；语句2为0.0063秒
　　两条语句的时间比是：语句1/语句2＝32.968

　　比较以上的数据时，我们可以发现采用where...limit....性能基本稳定，受偏移量和行数的影响不大，而单纯采用limit的话，受偏移量的影响很大，当偏移量大到一定后性能开始大幅下降。不过在数据量不大的情况下，两者的区别不大。

 

　　所以应当先使用where等查询语句，配合limit使用，效率才高

 

　　注意，在sql语句中，limt关键字是最后才被处理的，是对查询好的结果进行分页。以下条件的处理顺序一般是：where->group by->having-order by->limit

# 原因探究：

当Limit和order by连用时，mysql只会产生offset+length的排序集，对于剩下的数据，mysql是不会再去进行排序的。mysql对limit做了许多类似优化，当offset很大的时候，这些优化要操作的信息就非常多，优化就会失效，所有性能就下降了。

具体的优化官方文档上有，总结了一下几点：

> - If you select only a few rows with `LIMIT`, MySQL uses indexes in some cases when normally it would prefer to do a full table scan.
>
> - If you combine `LIMIT *row_count*` with `ORDER BY`, MySQL stops sorting as soon as it has found the first *row_count* rows of the sorted result, rather than sorting the entire result. If ordering is done by using an index, this is very fast. If a filesort must be done, all rows that match the query without the `LIMIT` clause are selected, and most or all of them are sorted, before the first *row_count* are found. After the initial rows have been found, MySQL does not sort any remainder of the result set.
>
>   One manifestation of this behavior is that an `ORDER BY` query with and without `LIMIT` may return rows in different order, as described later in this section.
>
> - If you combine `LIMIT *row_count*` with `DISTINCT`, MySQL stops as soon as it finds *row_count* unique rows.
>
> - In some cases, a `GROUP BY` can be resolved by reading the index in order (or doing a sort on the index), then calculating summaries until the index value changes. In this case, `LIMIT *row_count*` does not calculate any unnecessary `GROUP BY` values.
>
> - As soon as MySQL has sent the required number of rows to the client, it aborts the query unless you are using `SQL_CALC_FOUND_ROWS`. In that case, the number of rows can be retrieved with `SELECT FOUND_ROWS()`. See [Section 12.14, “Information Functions”](https://dev.mysql.com/doc/refman/5.7/en/information-functions.html).
>
> - `LIMIT 0` quickly returns an empty set. This can be useful for checking the validity of a query. It can also be employed to obtain the types of the result columns within applications that use a MySQL API that makes result set metadata available. With the [**mysql**](https://dev.mysql.com/doc/refman/5.7/en/mysql.html) client program, you can use the [`--column-type-info`](https://dev.mysql.com/doc/refman/5.7/en/mysql-command-options.html#option_mysql_column-type-info) option to display result column types.
>
> - If the server uses temporary tables to resolve a query, it uses the `LIMIT *row_count*` clause to calculate how much space is required.
>
> - If an index is not used for `ORDER BY` but a `LIMIT` clause is also present, the optimizer may be able to avoid using a merge file and sort the rows in memory using an in-memory `filesort` operation. For details, see [The In-Memory filesort Algorithm](https://dev.mysql.com/doc/refman/5.7/en/order-by-optimization.html#order-by-filesort-in-memory).

可以看到，无论是和order by 子句还是和group by子句联合使用，mysql都对limit操作的查询实行了懒惰策略，指要查询的结果达到了length，就不再据需往下操作了。但是由于offset的存在，使得mysql不得不使操作的数据行数达到offset+length，所以性能就下降了。

# 解决方案：

在使用limit之前，先对数据进行一定的处理，比如先用where语句，减少数据的总量之后再分页。或者order by子句用上索引。总之，思路就是优化limit操作对象的检索速度。