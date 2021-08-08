**Spring事务嵌套引发的血案**   -- Transaction rolled back because it has been marked as rollback-only

​	先看问题：存在一个A方法(带有事物REQUIRED)，B方法也存在事物REQUIRED，A方法调用B方法，担心B方法中会报错，就进行了try(){}处理，也没有往上层去抛，这个时候执行结果是什么。

​	重现问题：A方法调用了B方法，A和B都存在事物，并且在A调用的时候把B进行了try{}catch{}操作

![image-20200922103347806](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20200922103347806.png)

![image-20200922103455866](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20200922103455866.png)

​		在executeSqlList方法准备提交的时候，transaction已经被excuteInsertSql方法设置为rollback-only了，但是executeSqlList方法中将excuteInsertSql的报错将给抓住消化了，没有继续向外抛出，所以executeSqlList结束的时候，transaction会执commit操作，所以就报错了。看看处理回滚的源码：

![image-20200922105838774](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20200922105838774.png)

![image-20200922105848843](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20200922105848843.png)

 简单分析：excuteInsertSql()有事务，然后处理的时候有这么一句：

![image.png](https://user-gold-cdn.xitu.io/2019/1/31/168a1701acc860a3?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

这个时候把参数unexpectedRollback置为false了，所以当executeSqlList()事务需要提交的时候

![image.png](https://user-gold-cdn.xitu.io/2019/1/31/168a1701e4092f5e?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

所以，就之前抛出异常了。

**分析:**当A方法的事物(REQUIRED),B方法的事物(REQUIRED),A调用B方法，在spring中，spring将会把这些事务合二为一。当整个方法中每个子方法没报错时，整个方法执行完才提交事务，

如果某个子方法有异常，spring将该事务标志为rollback only。如果这个子方法没有将异常往上整个方法抛出或整个方法未往上抛出，那么该异常就不会触发事务进行回滚，事务就会在整个方法执行完后就会提交，这时就会造成Transaction rolled back because it has been marked as rollback-only的异常。

​		总结：此时程序运行，如果excuteInsertSql方法报错，抛出异常，被上层方法executeSqlList捕获,但是捕获后未继续往外抛异常，简单处理后相当于吞掉了该异常，程序继续向下执行，可以执行完毕。但是由于spring会整合事务，此时事务被标记为rollback only,但程序执行完毕，故抛出异常。



**如何解决**这个问题：

​	1.将这个excuteInsertSql()中的错误try(){}后继续往上层去抛，交给上层去处理掉，不能内部消化

​	2.将excuteInsertSql()中的事物改为(REQUIRES_NEW)

程序执行正常。数据库插入了一条数据，excuteInsertSql()方法事务正常执行,excuteInsertSql()方法中进行 了回滚（因为单开了新的事务）。



##### 附：事务传播方式

------

@see org.springframework.transaction.annotation.Propagation

| 事务传播方式              | 说明                                                         |
| :------------------------ | :----------------------------------------------------------- |
| PROPAGATION_REQUIRED      | 如果当前没有事务，就新建一个事务，如果已经存在一个事务中，加入到这个事务中。这是默认的传播方式 |
| PROPAGATION_SUPPORTS      | 支持当前事务，如果当前没有事务，就以非事务方式执行           |
|                           |                                                              |
| PROPAGATION_MANDATORY     | 使用当前的事务，如果当前没有事务，就抛出异常                 |
| PROPAGATION_REQUIRES_NEW  | 新建事务，如果当前存在事务，把当前事务挂起                   |
| PROPAGATION_NOT_SUPPORTED | 以非事务方式执行操作，如果当前存在事务，就把当前事务挂起     |
| PROPAGATION_NEVER         | 以非事务方式执行，如果当前存在事务，则抛出异常               |
| PROPAGATION_SUPPORTS      | 支持当前事务，如果当前没有事务，就以非事务方式执行。         |
| PROPAGATION_NESTED        | 如果当前存在事务，则在嵌套事务内执行。如果当前没有事务，则执行与PROPAGATION_REQUIRED类似的操作。 |



