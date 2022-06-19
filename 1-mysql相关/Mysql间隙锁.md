### Mysql间隙锁

#### 1、什么是间隙锁？

​		所谓的next key lock就是一个行锁（record lock）+范围锁（gap lock），比如某一个辅助索引（比如上面的class_id），如果它有1,3,5这几个值，那么当我们使用next key lock的锁住class_id=1的时候，实际上锁住了（-无穷，1]，或者锁住class_id=3的时候，实际上锁住的是（1,3]，也就是一个**左开右闭的区间**。如果此时别的事务要在这个区间内插入数据，就会被阻塞住。这个锁一直到事务提交才会释放。因此，即使出现了上面图片里面这种情况，也可以保证前后两次去读的内容一致，因为对这个辅助索引上的锁是：“next key lock”，他会锁住一个区间。

 	  但是注意，对于可重复读默认使用的就是next key lock，但是对于“唯一索引”，**比如主键的索引，next key lock会降级成行锁**，而不会锁住一个区间。因此，如果上面的事务1的update使用的是主键，事务2也使用主键进行插入，那么实际上事务2根本不会被阻塞，可以立即插入并返回。而对于非唯一索引，next key lock则不会降级。

​	  间隙锁有时候和表锁一样，可能是个巨大的灾难，因为当锁住的行数非常多时，一是影响范围大速度慢，二是可能会产生死锁。



#### 2、什么时候产生间隙锁，间隙锁的范围？

​		例a、）同一个事务中先delete后insert的情况下，当我们通过一个参数去删除一条记录的时候， 如果参数在数据库中存在， 那么这个时候产生的是普通行锁， 锁住这个记录， 然后删除， 然后释放锁。如果这条记录不存在，问题就来了， 数据库会扫描索引，发现这个记录不存在， 这个时候的delete语句获取到的就是一个间隙锁，然后数据库会向左扫描扫到第一个比给定参数小的值， 向右扫描扫描到第一个比给定参数大的值（左开右闭）， 然后以此为界，构建一个区间， 锁住整个区间内的数据， 一个特别容易出现死锁的间隙锁诞生了。

```java
举个例子：
表task_queue
Id           taskId
1              2
3              9
10            20
40            41

开启一个会话： session 1
sql> set autocommit=0;
   ##
取消自动提交

sql> delete from task_queue where taskId = 20;
sql> insert into task_queue values(20, 20);

在开启一个会话： session 2
sql> set autocommit=0;
   ##
取消自动提交

sql> delete from task_queue where taskId = 25;
sql> insert into task_queue values(30, 25);

在没有并发，或是极少并发的情况下， 这样会可能会正常执行，在Mysql中， 事务最终都是穿行执行， 但是在高并发的情况下， 执行的顺序就极有可能发生改变， 变成下面这个样子：
sql> delete from task_queue where taskId = 20;
sql> delete from task_queue where taskId = 25;
sql> insert into task_queue values(20, 20);
sql> insert into task_queue values(30, 25);

这个时候最后一条语句：insert into task_queue values(30, 25); 执行时就会爆出死锁错误。因为删除taskId = 20这条记录的时候，20 --  41 都被锁住了， 他们都取得了这一个数据段的共享锁， 所以在获取这个数据段的排它锁时出现死锁。
这种问题的解决办法：前面说了， 通过修改innodb_locaks_unsafe_for_binlog参数来取消间隙锁从而达到避免这种情况的死锁的方式尚待商量， 那就只有修改代码逻辑， 存在才删除，尽量不去删除不存在的记录。
```

​		例b、）Select * from  emp where empid > 100 for update; 这条sql属于当前读。会锁住（100，+无穷大）

但是我们常用的普通查询：select * from emp where empid>100属于快照读，并不会产生间隙锁。



#### 3、唯一索引会使间隙锁降为行锁。

​	尽量使用唯一索引，唯一索引会使间隙锁降为行锁。



#### 参考文档

[MVCC能否解决幻读](https://www.cnblogs.com/xuwc/p/13873293.html)

[mysql间隙锁](https://developer.aliyun.com/article/283419)

