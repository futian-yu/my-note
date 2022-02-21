**juc常见问题**

![1645260576941](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1645260576941.png)









二、常见问题

```java
//1.AtomicLong等Atomic类的底层原理是什么？

//2.知道LongAdder对象吗？
对于count++操作，使用如下类实现：AtomicInteger count = new AtomicInteger();count.addAndGet(1); 
如果是JDK8，推荐使用LongAdder对象，比AtomicLong性能更好(减少了乐观锁的重试次数)。

//3.什么是重量级锁？重量级指的是什么？
synchronized在1.7之前是重量级锁，1.8之后已经越来越轻量化了。重量级不是指它的代码量多效率低，而是指依赖的多少。synchronized1.7之前严重依赖操作系统内核很多命令。

//4.知道synchronized的锁膨胀吗？
1）偏向锁
2）轻量级锁（CAS+自旋）
3）重量级锁

//5.比较jdk1.7和jdk1.8中synchronized锁的区别？








```

