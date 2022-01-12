**LockSupport详解**

**1.LockSupport是什么？**

​		LockSupport是用来创建锁和其他同步类的基本线程阻塞原语。

​		LockSupport类使用了一种名为Permit（许可）的概念来做到阻塞和唤醒线程的功能，每个线程都有一个许可（permit），permit只有两个值1和零，默认是0.

​		可以把许可看成是一种(0,1)信号量(Semaphore)，但与Semaphore不同的是，许可的**累加上限是1**.

**2.核心函数分析**

```java
//1.park函数
	阻塞线程
//2.unpark函数
	释放线程的许可，即激活调用park后阻塞的线程。这个函数不是安全的，调用这个函数时要确保线程依旧存活。（使用park/unpark函数比wait/notify/await/signal好，因为它对线程的阻塞与唤醒顺序性变化不会报错。）
```

**3.常见问题**

```java
//1.为什么unpark两次再park两次还是会阻塞？
	因为unpark发放许可证，许可证的最大值是1.
```





