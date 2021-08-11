volatile关键字

​	volatile只能保证可见性与禁止重排序，但是volatile并不能保证线程安全，保证线程安全正确的做法是加锁或者使用Atomic类。

