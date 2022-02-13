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
//1.缓存雪崩、缓存穿透、缓存击穿的原因和对应的解决方案？

//2.请说说redis缓存双写不一致问题及解决方案？

//3.redis为什么单线程性能还非常得高？
1）几乎所有的操作都是纯内存操作。
2）核心是基于非阻塞的IO多路复用机制。
3）单线程反而避免了多线程的频繁上下文切换带来的性能问题。

//4.简述redis事务如何实现？

//5.redis主从复制的核心原理？

//6.redis哨兵模式的核心原理？

//7.redis集群模式的核心原理？





```

