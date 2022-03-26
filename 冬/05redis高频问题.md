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
	先更新数据库，再删除缓存。（三个线程，1更新mysql为10,删除缓存；线程2读取了10，但是还没更新到redis，线程3又更新了mysql为6并删除缓存，这时候线程2又冒出来执行了更新redis为10的操作。总结：更新比查询快导致的）===》增加了延迟删除策略，让更新数据库的线程睡几百毫秒，这样又大大降低了概率。
	另外三种为什么不行？	
	先更新数据库，再更新缓存。(不行，多线程下有风险，两个线程更新缓存的操作不知道谁先谁后)；
	先缓存，再数据库。不行，不好回滚且浪费计算资源；
	先删缓存，再更新数据库。不行一个删一个读后再更新可能造成不一致。
	
	单机版数据库采用上面第一种就可以了，但是我们mysql采用的是主从模式，读写分离。可能缓存已经删了，在主库binlog同步数据到从库的过程中断开了，从库还是旧的数据，这时候再请求
	会发现缓存为空，读取从库中的数据到缓存中，造成了mysql和缓存不一致的情况。
	我们采用的解决方式是：用一个订阅binlog的服务系统（阿里的中间件canal），订阅到的binlog是最新的，订阅到后直接把Redis的缓存给更新覆盖掉。

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
	0）字符串（底层sds）
	https://www.cnblogs.com/lonely-wolf/p/14261486.html	
	1）Redis链表
	  双端链表、无环链表，即双端无环。
	  带链表长度计数器：通过len属性获取链表长度的时间复杂度为O(1).
      多态：链表节点使用void*指针来保存节点值，可以保存各种不同类型的值。
    2）字典（简单理解为hash）  
	  结构类似于hashMap，扩容条件不一样，BGSAVE操作且负载因子>=1的时候，进行一个扩容2倍操作。rehash操作也不同，它是一个渐进式的rehash：key/value容量大的时候，分多次进行rehash。这期间，字典的删除查找更新操作可能会在两个哈希表上进行，第一个哈希表没有找到，就去第二个哈希表上查找。但是增加操作一定是在新的哈希表操作。
	3）跳表（可以认为是ZSET）
	  跳表是一种有序的数据结构，它通过在每个节点中维持多个指向其他节点的指针，从而达到快速访问节点的目的。	(跳表不是很清楚)
    4）整数集合（intset不是redis5大数据类型之一，只是底层设计的一种结构）
      结构体包含三部分：length,encoding,contents[]保存编码类型对应的数据的
    5）压缩列表
      原理:将数据按照一定规则编码在一块连续的内存，目的是为了节省内存。压缩列表可以包含多个节点，每个节点可以保存一个字节数组或一个整数值。string[]中最大元素20byte，其他都是1，那么每个都是20，压缩列表则省了。
	
	https://www.bilibili.com/video/BV1V44y117dK?p=4

//25.说说redis事务相关命令？
	https://blog.csdn.net/meism5/article/details/104258034

//26.redis事务支持隔离性吗？(ACID)
	https://www.51cto.com/article/686117.html

//27.redis事务保证原子性吗？支持回滚吗？
	同上
	https://www.51cto.com/article/686117.html

//28.redis 集群模式的工作原理能说一下么？在集群模式下，redis 的 key 是如何寻址的？分布式寻址都有哪些算法？了解一致性 hash 算法吗？
	https://www.bilibili.com/video/BV1Pi4y157L1?p=3
	https://blog.csdn.net/xiaoyaGrace/article/details/104961243

//29.为什么要做redis分区？
	往redis cluster槽的分区的概念上回答，同上。

//30.redis分布式锁用了吗？讲讲你们redis的分布式锁实现方式？
	基于redission的分布式锁。

//31.如何解决redis的并发竞争key问题？
	分布式锁解决。

//32.分布式redis是前期做还是后期规模上来了做好？为什么？
	https://www.iamshuaidi.com/2608.html

//33.redis和memcached比较一下？
	https://support.huaweicloud.com/productdesc-dcs/RedisAndMemcachedChoose.html

//34.redis常见性能问题和解决方案？
	https://blog.csdn.net/qq1332479771/article/details/88424591

//35.一个字符串类型的值能存储最大容量是多少?
	https://blog.csdn.net/meism5/article/details/104245204

//36.假如Redis里面有1亿个key，其中有10w个key是以某个固定的已知的前缀开头的，如果将它们全部找出来？
	https://www.cnblogs.com/programb/p/13020782.html
	https://blog.csdn.net/qq_41144667/article/details/105297002
	
	为什么scan无阻塞？为什么scan可能会有重复的值？
	https://www.bilibili.com/video/BV1QZ4y137nS?spm_id_from=333.337.search-card.all.click

//37.Redis集群架构数据倾斜如何解决？(大key、热key)
	https://segmentfault.com/a/1190000017387491

//38.Redis如何解决分布式集群脑裂问题？(哨兵模式，选举过程，会有脑裂问题吗)。
	https://blog.csdn.net/LO_YUN/article/details/97131426
	
//39.raft算法知道吗？优缺点？
	哨兵领头节点选举用的是raft算法，具体细节不清楚。	

//40.redis 集群，为什么是 16384个slot？
	https://www.modb.pro/db/110934

//41.redis 字符串实现，sds 和 c 区别，空间预分配
	https://www.cnblogs.com/lonely-wolf/p/14261486.html	

//42.redis 有序集合怎么实现的？跳表是什么？
	ZSet、跳表。

//43.往跳表添加一个元素的过程，添加和获取元素，获取分数的时间复杂度？
	暂跳。

//44.为什么不用红黑树？红黑树有什么特点？左旋右旋操作?
	https://z.itpub.net/article/detail/14191D414D97A8CCC558B09FF5DB0BA1

//45.io 模型了解么？多路复用，selete，poll，epoll，epoll 的结构，怎么注册事件，et 和 lt 模式
	io多路复用。

//46.如何利用Redis处理热点数据?
	热key问题

//47.分布式下redis如何保证线程安全？
	分布式锁。

//48.redis持久化的方式及区别？
	aof、rdb

//49.列举下你所知道的常用的Redis客户端并发模型?
	redis锁、写的时候在redis的set操作的时候加sychronized锁。

//50.聊下分布式缓存，一致性hash?
	重复，集群

//51.如何解决缓存单机热点问题？
	即热key、大key问题。

//52.Redis 基本数据结构与实战场景?
	1）string
	2）list
	3）hash
	4）set	Set可以快速查找元素是否存在，用于记录一些不能重复的数据。记录注册用户、抽奖
	5）zset	热搜榜、排行榜。

//53.什么是io多路复用？解释一下？
https://blog.csdn.net/hollis_chuang/article/details/116310861
```

