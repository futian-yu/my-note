**LockSupport**

**1.LockSupport是什么？**

​		用于创建锁和其他同步类的基本线程阻塞原语的一个类，可以认为是线程等待唤醒机制(wait/notify)的加强版.

**2.LockSupport类的重要方法.**

​		LockSupport中的park()和unpark()的作用分别是阻塞线程和解除阻塞线程.。