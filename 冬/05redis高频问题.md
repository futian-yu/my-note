**redis高频问题**

![1644746190837](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1644746190837.png)



![1644746410735](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1644746410735.png)

![1644746527459](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1644746527459.png)

![1644748475758](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1644748475758.png)

![1644749927696](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1644749927696.png)

![1644749971685](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1644749971685.png)

redis的key都是string的类型，value有五种。

![1644751832637](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1644751832637.png)

![1644761296544](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1644761296544.png)

![1644761473350](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1644761473350.png)

![1644761618424](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1644761618424.png)

![1644761937348](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1644761937348.png)

![1644762107574](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1644762107574.png)

![1644762327510](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1644762327510.png)

![1644762580852](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1644762580852.png)

![1644762790456](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1644762790456.png)



二、常见问题

```java
//1.缓存穿透、缓存击穿、缓存雪崩的原因和对应的解决方案？
	https://cloud.tencent.com/developer/article/1824584

//2.请说说redis缓存双写不一致问题及解决方案？
	我们这边用的是先写数据库，再删除缓存。设置了延迟更新。  三线程更读更有问题。
    数据库更新成功，缓存删除失败解决方案？	  记住，任何问题都可以使用第三方中间件解决。

//3.redis为什么单线程性能还非常得高？
1）几乎所有的操作都是纯内存操作。
2）核心是基于非阻塞的IO多路复用机制。（具体查一下IO多路复用的意思：一个线程监视多个文件）
3）单线程反而避免了多线程的频繁上下文切换带来的性能问题。

//4.简述redis事务如何实现？
	事务开始(MULTI)、命令入队（除MULTI、EXEC、WATCH、DISCARD）、事务执行。
	https://www.zhihu.com/zvideo/1374806450712268800

//5.redis主从复制的核心原理？
	https://blog.csdn.net/zcczero/article/details/117354382

//6.redis哨兵模式的核心原理？
	https://www.cnblogs.com/gunduzi/p/13160448.html

//7.redis集群模式的核心原理？
	解决这几个问题：redis 集群模式的工作原理能说一下么？在集群模式下，redis 的 key 是如何寻址的？分布式寻址都有哪些算法？了解一致性 hash 算法吗？
	https://segmentfault.com/a/1190000022717518
	
//8.reids核心数据结构 - String的动态扩容机制？
	https://blog.csdn.net/gfyann/article/details/103137686

//9.redis如何实现持久化？
	RDB:redis默认的持久化方式。每隔一定时间，将内存中的数据以快照的方式保存到硬盘中，对应产生的文件为dump.rdb。优点：性能最大化，子进程完成写操作，主进程继续处理。缺点：宕机可能会丢失某一个时间段内的数据。
	aof：记录命令。always的方式。
	
//10.redis内存淘汰策略LRU/LFU算法？
	暂跳。
	
//11.Redis是如何处理过期数据的？
	https://segmentfault.com/a/1190000037475073

//12.Redis多节点数据同步复制原理？
	https://blog.csdn.net/zcczero/article/details/117354382
	
//13.Redis如何解决分布式集群脑裂问题？
	https://blog.csdn.net/LO_YUN/article/details/97131426

//14.常见问题汇总：https://juejin.cn/post/6844904127055527950

//15.说几个你用到的redis的应用场景？
	做缓存（token、分布式全局唯一id主键自增，热点数据）、做持久化（下拉框）。

//16.说说redis的持久化机制？各自的优缺点？
	如第9个问题。

//17.redis持久化数据和缓存怎么做扩容？
	redis集群，基于slot槽的概念做扩容。均分16384个槽。
	
//18.随便说几个redis的过期键删除策略。
	https://www.cnblogs.com/ysocean/p/12422635.html

//19.redis key的过期时间和永久有效分别怎么设置？
 	expire和persist
 
//20.expire可以设置key的过期时间，那么对过期的数据怎么处理呢？
	https://www.cnblogs.com/linqiaobao/p/13378831.html

//21.mysql里有2000w数据，redis中只有20w的数据，如何保证redis中的数据都是热点数据？
	https://zhuanlan.zhihu.com/p/260833627
	
//22.redis的内存用完了会发生什么？
	内存淘汰策略。

//23.redis如何做内存优化？
	https://www.jianshu.com/p/16648f0a1abc

//24.说说redis的内存模型？

//25.说说redis事务相关命令？

//26.redis事务支持隔离性吗？

//27.redis事务保证原子性吗？支持回滚吗？

//28.redis 集群模式的工作原理能说一下么？在集群模式下，redis 的 key 是如何寻址的？分布式寻址都有哪些算法？了解一致性 hash 算法吗？

//29.为什么要做redis分区？

//30.redis分布式锁用了吗？讲讲你们redis的分布式锁实现方式？

//31.如何解决redis的并发竞争key问题？

//32.分布式redis是前期做还是后期规模上来了做好？为什么？

//33.redis和memcached比较一下？

//34.redis常见性能问题和解决方案？

//35.一个字符串类型的值能存储最大容量是多少?

//36.假如Redis里面有1亿个key，其中有10w个key是以某个固定的已知的前缀开头的，如果将它们全部找出来？

//37.Redis集群架构数据倾斜如何解决？(大key、热key)

//38.Redis如何解决分布式集群脑裂问题？



```

