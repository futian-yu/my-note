**写在前面**

HashMap作为常用的数据结构之一，应用场景和使用范围都是非常广泛的。操作集合不外乎就是存取数据。下面我就介绍几种常用的遍历Map的方式。

**第一种**

使用keySet遍历HashMap

![img](https://pics0.baidu.com/feed/b812c8fcc3cec3fd3ece77715d7e4d398694273b.png?token=31a26adb54d5fc0984c3f9c2e003ee35)

**注意：**

**使用此方法遍历HashMap时效率是最低的。因为该方法的底层其实是遍历了2次。第一次遍历将Map集合中所有的key拿到，放到一个Set集合中。第二次遍历是通过key获取对应的value。因此当数据量特别大是不考虑使用该方法。**

**第二种**

使用entrySet遍历HashMap

![img](https://pics3.baidu.com/feed/a2cc7cd98d1001e9fbadb7e532f8e2ea56e79740.png?token=16606c1a9a611d1b20ab864634238c6a)

**注意：**

使用该方法遍历HashMap只是遍历了一次就把key和value都放到Entry中了。省去一次遍历的操作，当数据量大时就会明显比使用keySet遍历效率更高。

**第三种**

其实还有更高效的方法，当使用的是JDK8时，可以采用Map.forEach方法

![img](https://pics0.baidu.com/feed/8c1001e93901213fd24a940adf11afd72d2e95ec.png?token=ac05f0eb8a84237090be91e437fa7598)

**注意：**

forEach方法是JDK8中新增的方法。该方法不是线程安全的方法，在多线程并发进行读写操作时会产生ConcurrentModificationException（并发修改异常）。因此使用时需要考虑实际需求。



map的使用场景：可以先取出list<dto>对象，然后对list对象进行list转map操作，取出想要的字段为键值对类型；

