**面试题知识点总结**

**1.stringBuilder和StringBuffer和StringJoiner?**

​		stringBuilder是线程不安全的，StringBuffer是线程安全的，其内部方法几乎全部一样，只是StringBuffer里面的方法都加了synchrnoized修饰方法，所以效率比StringBuilder低。

​		StringJoiner是java8新出的一个类，用于构造分隔符分隔的字符序列，并可选择性地从自己所需的前缀开始并以自己所需后缀结束（根据不同的构造方法实现）。		

```
StringJoiner sj = new StringJoiner(":", "[", "]");
sj.add("George").add("Sally").add("Fred");
String desiredString = sj.toString();
//结果 : 代码输出为[George:Sally:Fred]
```



**2.session、cookie和token的区别.**

​		1.http是一个无状态协议

​			什么是无状态呢？就是说这一次请求和上一次请求是没有任何关系的，互不认识的，没有关联的。这种无状态的的好处是快速。坏处是假如我们想要把`www.zhihu.com/login.html`和`www.zhihu.com/index.html`关联起来，必须使用某些手段和工具		

​		2.cookie和session

​			由于http的无状态性，为了使某个域名下的所有网页能够共享某些数据，session和cookie出现了。客户端访问服务器的流程如下

- 首先，客户端会发送一个http请求到服务器端。
- 服务器端接受客户端请求后，建立一个session，并发送一个http响应到客户端，这个响应头，其中就包含Set-Cookie头部。该头部包含了sessionId。Set-Cookie格式如下，具体请看[Cookie详解](http://bubkoo.com/2014/04/21/http-cookies-explained/)
  `Set-Cookie: value[; expires=date][; domain=domain][; path=path][; secure]`
- 在客户端发起的第二次请求，假如服务器给了set-Cookie，浏览器会自动在请求头中添加cookie（此时第二次请求，请求头中比第一次多了个cookie）
- 服务器接收请求，分解cookie，验证信息，核对成功后返回response给客户端

![请求流程](https://segmentfault.com/img/bVbmYbQ?w=400&h=200)

### 注意

- cookie只是实现session的其中一种方案。虽然是最常用的，但并不是唯一的方法。禁用cookie后还有其他方法存储，比如放在url中
- 现在大多都是Session + Cookie，但是只用session不用cookie，或是只用cookie，不用session在理论上都可以保持会话状态。可是实际中因为多种原因，一般不会单独使用
- 用session只需要在客户端保存一个id，实际上大量数据都是保存在服务端。如果全部用cookie，数据量大的时候客户端是没有那么多空间的。
- 如果只用cookie不用session，那么账户信息全部保存在客户端，一旦被劫持，全部信息都会泄露。并且客户端数据量变大，网络传输的数据量也会变大

### 小结

**简而言之, session 有如用户信息档案表, 里面包含了用户的认证信息和登录状态等信息. 而 cookie 就是用户通行证**（相当于第一次请求拿到了档案信息表，第二次把这个档案办理了一个用户通行证）

## token

token 也称作令牌，由uid+time+sign[+固定参数]
token 的认证方式类似于**临时的证书签名**, 并且是一种服务端无状态的认证方式, 非常适合于 REST API 的场景. 所谓无状态就是服务端并不会保存身份认证相关的数据。

### 组成

- uid: 用户唯一身份标识
- time: 当前时间的时间戳
- sign: 签名, 使用 hash/encrypt 压缩成定长的十六进制字符串，以防止第三方恶意拼接
- 固定参数(可选): 将一些常用的固定参数加入到 token 中是为了避免重复查库

### 存放

token在客户端一般存放于localStorage，cookie，或sessionStorage中。在服务器一般存于数据库中

### token认证流程

token 的认证流程与cookie很相似

- 用户登录，成功后服务器返回Token给客户端。
- 客户端收到数据后保存在客户端
- 客户端再次访问服务器，将token放入headers中
- 服务器端采用filter过滤器校验。校验成功则返回请求数据，校验失败则返回错误码

## token可以抵抗csrf，cookie+session不行

假如用户正在登陆银行网页，同时登陆了攻击者的网页，并且银行网页未对csrf攻击进行防护。攻击者就可以在网页放一个表单，该表单提交src为`http://www.bank.com/api/transfer`，body为`count=1000&to=Tom`。倘若是session+cookie，用户打开网页的时候就已经转给Tom1000元了.因为form 发起的 POST 请求并不受到浏览器同源策略的限制，因此可以任意地使用其他域的 Cookie 向其他域发送 POST 请求，形成 CSRF 攻击。在post请求的瞬间，cookie会被浏览器自动添加到请求头中。但token不同，token是开发者为了防范csrf而特别设计的令牌，浏览器不会自动添加到headers里，攻击者也无法访问用户的token，所以提交的表单无法通过服务器过滤，也就无法形成攻击。

## 分布式情况下的session和token

我们已经知道session时有状态的，一般存于服务器内存或硬盘中，当服务器采用分布式或集群时，session就会面对负载均衡问题。

- 负载均衡多服务器的情况，不好确认当前用户是否登录，因为多服务器不共享session。这个问题也可以将session存在一个服务器中来解决，但是就不能完全达到负载均衡的效果。当今的几种[解决session负载均衡](http://blog.51cto.com/zhibeiwang/1965018)的方法。

而token是无状态的，token字符串里就保存了所有的用户信息

- 客户端登陆传递信息给服务端，服务端收到后把用户信息加密（token）传给客户端，客户端将token存放于localStroage等容器中。客户端每次访问都传递token，服务端解密token，就知道这个用户是谁了。通过cpu加解密，服务端就不需要存储session占用存储空间，就很好的解决负载均衡多服务器的问题了。这个方法叫做[JWT(Json Web Token)](https://huanqiang.wang/2017/12/28/JWT 介绍/)

## 总结

- session存储于服务器，可以理解为一个状态列表，拥有一个唯一识别符号sessionId，通常存放于cookie中。服务器收到cookie后解析出sessionId，再去session列表中查找，才能找到相应session。依赖cookie
- cookie类似一个令牌，装有sessionId，存储在客户端，浏览器通常会自动添加。
- token也类似一个令牌，无状态，用户信息都被加密到token中，服务器收到token后解密就可知道是哪个用户。需要开发者手动添加。
- jwt只是一个跨域认证的方案



**3.接口里面可以实现方法吗？可以创建成员变量吗？接口和抽象类的区别。**

​		接口里面可以实现方法：

​		java8开始，可以在接口里面添加默认方法和静态方法。默认方法用default修饰，只能用在接口中，静态方法用static修饰。并且静态方法和默认方法都可以有多个。举例： java.util.Map

```java
public interface Map<K,V> {
    ...
    /**
    * 接口默认方法
    */
    default boolean remove(Object key, Object value) {
        Object curValue = get(key);
        if (!Objects.equals(curValue, value) ||
            (curValue == null && !containsKey(key))) {
            return false;
        }
        remove(key);
        return true;
    }
    
    ...
    
    /**
    * 接口静态方法
    */
    public static <K extends Comparable<? super K>, V> Comparator<Map.Entry<K,V>> comparingByKey() {
        return (Comparator<Map.Entry<K, V>> & Serializable)
            (c1, c2) -> c1.getKey().compareTo(c2.getKey());
    }
    
    ...
    
}   
————————————————
版权声明：本文为CSDN博主「南伯基尼」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/weixin_44203158/article/details/109340113
```

​		**（1）为什么要有接口默认方法？**

​				举一个很现实的例子：我们的接口老早就写好了，后面因为各种业务问题，避免不了要修改接口。

在 Java 8 之前，比如要在一个接口中添加一个抽象方法，那所有的接口实现类都要去实现这个方法，不然就会编译错误，而某些实现类根本就不需要实现这个方法也被迫要写一个空实现，改动会非常大。

所以，接口默认方法就是为了解决这个问题，只要在一个接口添加了一个默认方法，所有的实现类就自动继承，不需要改动任何实现类，也不会影响业务，爽歪歪。

另外，接口默认方法可以被接口实现类重写。

​		（2）为什么要有接口静态方法？

​				接口静态方法和默认方法类似，只是接口静态方法不可以被接口实现类重写。

​				接口静态方法只可以直接通过静态方法所在的 接口名.静态方法名 来调用。

​		（3）接口默认方法多继承冲突问题

​				只需重写该冲突的方法即可。在方法里面还能直接调用指定父接口的默认方法。

**接口可以创建成员变量：**

​		接口可以创建成员变量,只是其成员变量默认是public static final的，静态且不可变,比如说我们常量类里面就经常这样用。

**接口和抽象类的区别：**

​		简单来说，接口是公开的，里面不能有私有的方法或者变量，是用于让别人使用的，而抽象类是可以有私有方法或私有变量的。另外，实现接口一定要实现接口里的所有方法，而实现抽象类可以有选择地重写需要用到的地方。一般应用里，最顶级的是接口，然后是抽象类实现接口，最后再到具体实现类。还有，接口可以实现多重继承，而一个类只能继承一个超类，但可以通过继承多个接口实现多重继承。



**4.有哪些常见异常？异常和error有什么区别？**

​		![1617958459123](C:\Users\Yuft\AppData\Roaming\Typora\typora-user-images\1617958459123.png)

​		java中所有的类都直接或者间接地继承自Object类。

​		从继承关系可知：Throwable是所有异常体系的根，它继承自Object。Throwable有两个体系：Error和Exception，Error表示严重的错误，程序一般对此无能为力，例如：

```java
OutOfMemoryError：内存耗尽
NoClassDefFoundError：无法加载某个Class
StackOverflowError：栈溢出
```

​		而Exception则是运行时的错误，他可以被捕获并处理。某些异常是应用程序逻辑处理的一部分，应该捕获并处理。例如：

```java
NumberFormatException：数值类型的格式错误
FileNotFoundException：未找到文件
SocketException：读取网络失败
```

​	还有一些异常是程序逻辑编写不对造成的，应该修复程序本身。例如：

```
NullPointerException：对某个null的对象调用方法或字段
IndexOutOfBoundsException：数组索引越界
```

​	Exception又分为两大类：

```
RuntimeException以及它的子类；
非RuntimeException（包括IOException、ReflectiveOperationException等等），即checked Exception
```

​	Java规定：

- 必须捕获的异常，包括`Exception`及其子类，但不包括`RuntimeException`及其子类，这种类型的异常称为Checked Exception。
- 不需要捕获的异常，包括`Error`及其子类，`RuntimeException`及其子类。

**总结：**

Java使用异常来表示错误，并通过`try ... catch`捕获异常；

Java的异常是`class`，并且从`Throwable`继承；

`		 Error`是无需捕获的严重错误，`Exception`是应该捕获的可处理的错误；

`	  RuntimeException`无需强制捕获，非`RuntimeException`（Checked Exception）需强制捕获，或者用`throws`声明；

不推荐捕获了异常但不进行任何处理。

​		

**5.创建线程池的正确姿势。**

​		我们先看java手册上说的：

​		![1618912779922](C:\Users\Yuft\AppData\Roaming\Typora\typora-user-images\1618912779922.png)

参数说明：

​	![1618912794129](C:\Users\Yuft\AppData\Roaming\Typora\typora-user-images\1618912794129.png)

类图结构：

![1618912813095](C:\Users\Yuft\AppData\Roaming\Typora\typora-user-images\1618912813095.png)

​		Executors的创建线程池的方法，创建出来的线程池都实现了ExecutorService接口。常用方法有以下几个：

（1）newFixedThreadPool(int thread) : 创建固定数目线程的线程池。

（2）newCachedThreadPool()：创建一个可缓存的线程池，调用execute将重用以前构造的线程（如果线程可用）。如果没有可用的线程，则创建一个新的线程并添加至线程池中。终止并从缓存中移除那些已有60秒钟未被使用的线程。

（3）newSingleThreadExecutor() 创建一个单线程化的Executor。

（4）newScheduledThreadPool(int corePoolSize) 创建一个支持定时及周期性的任务执行的线程池，多数情况下可以用来代替timer类。

**执行原理：**

​		线程池执行会根据corePoolSize和maximumPoolSize自动地调整线程池的大小。

​		当在execute（Runnable）方法中提交新任务并且少于corePoolSize正在运行时，即使其他线程处于空闲状态，也会新建一个线程。如果   corePoolSize < 目前线程 < maximumPoolSize，则看队列是否已满，未满则加入队列等待，已满则创建一个新线程。

- 通过设置corePoolSize和maximumPoolSize相同，您可以创建一个固定大小的线程池。
  - 通过将maximumPoolSize设置为基本上无界的值，例如Integer.MAX_VALUE，您可以允许池容纳任意数量的并发任务。

![1618912875440](C:\Users\Yuft\AppData\Roaming\Typora\typora-user-images\1618912875440.png)

**使用Executor可以创建四种类型的线程池：**（Executors工厂类一共可以创建四种类型的线程池，通过Executors.newXXX即可创建。下面就分别都介绍一下。）

​	1. FixedThreadPool(固定大小的线程池)

```java
public static ExecutorService newFixedThreadPool(int nThreads){
    return new ThreadPoolExecutor(nThreads,nThreads,0L,TimeUnit.MILLISECONDS,new LinkedBlockingQueue<Runnable>());
}
```

```
* 它是一种固定大小的线程池；
* corePoolSize和maximunPoolSize都为用户设定的线程数量nThreads；
```

- keepAliveTime为0，意味着一旦有多余的空闲线程，就会被立即停止掉；但这里keepAliveTime无效；
- 阻塞队列采用了LinkedBlockingQueue，它是一个无界队列；
- 由于阻塞队列是一个无界队列，因此永远不可能拒绝任务；
- 由于采用了无界队列，实际线程数量将永远维持在nThreads，因此maximumPoolSize和keepAliveTime将无效。



​	2. CachedThreadPool(它是一个可以无限扩大的线程池)

```java
public static ExecutorService newCachedThreadPool(){
    return new ThreadPoolExecutor(0,Integer.MAX_VALUE,60L,TimeUnit.MILLISECONDS,new SynchronousQueue<Runnable>());
}
```

- 它是一个可以无限扩大的线程池；
- 它比较适合处理执行时间比较小的任务；
- corePoolSize为0，maximumPoolSize为无限大，意味着线程数量可以无限大；
- keepAliveTime为60S，意味着线程空闲时间超过60S就会被杀死；
- 采用SynchronousQueue装等待的任务，这个阻塞队列没有存储空间，这意味着只要有请求到来，就必须要找到一条工作线程处理他，如果当前没有空闲的线程，那么就会再创建一条新的线程。



3. SingleThreadExecutor(它只创建一条线程)

```java
public static ExecutorService newSingleThreadExecutor(){
    return new ThreadPoolExecutor(1,1,0L,TimeUnit.MILLISECONDS,new LinkedBlockingQueue<Runnable>());
}
```



4.ScheduledThreadPool(它用来处理延时任务或定时任务)



# Executors存在什么问题

我们先来一个简单的例子，模拟一下使用Executors导致OOM的情况。

```java
/**
 * @author Hollis
 */
public class ExecutorsDemo {
    private static ExecutorService executor = Executors.newFixedThreadPool(15);
    public static void main(String[] args) {
        for (int i = 0; i < Integer.MAX_VALUE; i++) {
            executor.execute(new SubThread());
        }
    }
}

class SubThread implements Runnable {
    @Override
    public void run() {
        try {
            Thread.sleep(10000);
        } catch (InterruptedException e) {
            //do nothing
        }
    }
}
```

通过指定JVM参数：-Xmx8m -Xms8m 运行以上代码，会抛出OOM:

```java
Exception in thread "main" java.lang.OutOfMemoryError: GC overhead limit exceeded
    at java.util.concurrent.LinkedBlockingQueue.offer(LinkedBlockingQueue.java:416)
    at java.util.concurrent.ThreadPoolExecutor.execute(ThreadPoolExecutor.java:1371)
    at com.hollis.ExecutorsDemo.main(ExecutorsDemo.java:16)
以上代码指出，ExecutorsDemo.java的第16行，就是代码中的executor.execute(new SubThread());。
```

## Executors为什么存在缺陷

通过上面的例子，我们知道了Executors创建的线程池存在OOM的风险，那么到底是什么原因导致的呢？我们需要深入Executors的源码来分析一下。

其实，在上面的报错信息中，我们是可以看出蛛丝马迹的，在以上的代码中其实已经说了，真正的导致OOM的其实是LinkedBlockingQueue.offer方法。

```javascript
Exception in thread "main" java.lang.OutOfMemoryError: GC overhead limit exceeded
    at java.util.concurrent.LinkedBlockingQueue.offer(LinkedBlockingQueue.java:416)
    at java.util.concurrent.ThreadPoolExecutor.execute(ThreadPoolExecutor.java:1371)
    at com.hollis.ExecutorsDemo.main(ExecutorsDemo.java:16)
```

如果读者翻看代码的话，也可以发现，其实底层确实是通过LinkedBlockingQueue实现的：

```javascript
public static ExecutorService newFixedThreadPool(int nThreads) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>());
```

如果读者对Java中的阻塞队列有所了解的话，看到这里或许就能够明白原因了。

Java中的BlockingQueue主要有两种实现，分别是ArrayBlockingQueue 和 LinkedBlockingQueue。

ArrayBlockingQueue是一个用数组实现的有界阻塞队列，必须设置容量。

LinkedBlockingQueue是一个用链表实现的有界阻塞队列，容量可以选择进行设置，不设置的话，将是一个无边界的阻塞队列，最大长度为Integer.MAX_VALUE。

# 创建线程池的正确姿势

避免使用Executors创建线程池，主要是避免使用其中的默认实现，那么我们可以自己直接调用ThreadPoolExecutor的构造函数来自己创建线程池。在创建的同时，给BlockQueue指定容量就可以了。

```java
private static ExecutorService executor = new ThreadPoolExecutor(10, 10,
        60L, TimeUnit.SECONDS,
        new ArrayBlockingQueue(10));
```



****

**6.mysql有哪几种事务隔离级别，默认的是哪种？sql优化**

## 事务隔离级别(图文详解)

##### 什么是事务?

​		事务是逻辑上的一组操作，要么都执行，要么都不执行。

##### 事物的特性(ACID)

​		![事物的特性](https://camo.githubusercontent.com/429c7cf4d94faa9ee216b110bf3258bf2a346987/68747470733a2f2f6d792d626c6f672d746f2d7573652e6f73732d636e2d6265696a696e672e616c6979756e63732e636f6d2f323031392d362f2545342542412538422545352538412541312545372538392542392545362538302541372e706e67)

1. 原子性： 事务是最小的执行单位，不允许分割。事务的原子性确保动作要么全部完成，要么完全不起作用；
2. 一致性： 执行事务前后，数据保持一致，多个事务对同一个数据读取的结果是相同的；
3. 隔离性： 并发访问数据库时，一个用户的事务不被其他事务所干扰，各并发事务之间数据库是独立的；
4. 持久性： 一个事务被提交之后。它对数据库中数据的改变是持久的，即使数据库发生故障也不应该对其有任何影响。

### 并发事务带来的问题

在典型的应用程序中，多个事务并发运行，经常会操作相同的数据来完成各自的任务（多个用户对统一数据进行操作）。并发虽然是必须的，但可能会导致以下的问题。

- 脏读（Dirty read）: 当一个事务正在访问数据并且对数据进行了修改，而这种修改还没有提交到数据库中，这时另外一个事务也访问了这个数据，然后使用了这个数据。因为这个数据是还没有提交的数据，那么另外一个事务读到的这个数据是“脏数据”，依据“脏数据”所做的操作可能是不正确的。
- 丢失修改（Lost to modify）: 指在一个事务读取一个数据时，另外一个事务也访问了该数据，那么在第一个事务中修改了这个数据后，第二个事务也修改了这个数据。这样第一个事务内的修改结果就被丢失，因此称为丢失修改。 例如：事务1读取某表中的数据A=20，事务2也读取A=20，事务1修改A=A-1，事务2也修改A=A-1，最终结果A=19，事务1的修改被丢失。
- 不可重复读（Unrepeatableread）: 指在一个事务内多次读同一数据。在这个事务还没有结束时，另一个事务也访问该数据。那么，在第一个事务中的两次读数据之间，由于第二个事务的修改导致第一个事务两次读取的数据可能不太一样。这就发生了在一个事务内两次读到的数据是不一样的情况，因此称为不可重复读。
- 幻读（Phantom read）: 幻读与不可重复读类似。它发生在一个事务（T1）读取了几行数据，接着另一个并发事务（T2）插入了一些数据时。在随后的查询中，第一个事务（T1）就会发现多了一些原本不存在的记录，就好像发生了幻觉一样，所以称为幻读。

不可重复度和幻读区别：

不可重复读的重点是修改，幻读的重点在于新增或者删除。

例1（同样的条件, 你读取过的数据, 再次读取出来发现值不一样了 ）：事务1中的A先生读取自己的工资为 1000的操作还没完成，事务2中的B先生就修改了A的工资为2000，导 致A再读自己的工资时工资变为 2000；这就是不可重复读。

例2（同样的条件, 第1次和第2次读出来的记录数不一样 ）：假某工资单表中工资大于3000的有4人，事务1读取了所有工资大于3000的人，共查到4条记录，这时事务2 又插入了一条工资大于3000的记录，事务1再次读取时查到的记录就变为了5条，这样就导致了幻读。

### 事务隔离级别

SQL 标准定义了四个隔离级别：

- READ-UNCOMMITTED(读取未提交)： 最低的隔离级别，允许读取尚未提交的数据变更，可能会导致脏读、幻读或不可重复读。
- READ-COMMITTED(读取已提交)： 允许读取并发事务已经提交的数据，可以阻止脏读，但是幻读或不可重复读仍有可能发生。
- REPEATABLE-READ(可重复读)： 对同一字段的多次读取结果都是一致的，除非数据是被本身事务自己所修改，可以阻止脏读和不可重复读，但幻读仍有可能发生。
- SERIALIZABLE(可串行化)： 最高的隔离级别，完全服从ACID的隔离级别。所有的事务依次逐个执行，这样事务之间就完全不可能产生干扰，也就是说，该级别可以防止脏读、不可重复读以及幻读。

------

| 隔离级别         | 脏读 | 不可重复读 | 幻影读 |
| ---------------- | ---- | ---------- | ------ |
| READ-UNCOMMITTED | √    | √          | √      |
| READ-COMMITTED   | ×    | √          | √      |
| REPEATABLE-READ  | ×    | ×          | √      |
| SERIALIZABLE     | ×    | ×          | ×      |

MySQL InnoDB 存储引擎的默认支持的隔离级别是 REPEATABLE-READ（可重读）。我们可以通过`SELECT @@tx_isolation;`命令来查看

```
mysql> SELECT @@tx_isolation;
+-----------------+
| @@tx_isolation  |
+-----------------+
| REPEATABLE-READ |
+-----------------+
```

这里需要注意的是：与 SQL 标准不同的地方在于InnoDB 存储引擎在 **REPEATABLE-READ（可重读）事务隔离级别下使用的是Next-Key Lock 锁算法，因此可以避免幻读的产生，这与其他数据库系统(如 SQL Server)是不同的。所以说InnoDB 存储引擎的默认支持的隔离级别是 REPEATABLE-READ（可重读） 已经可以完全保证事务的隔离性要求，即达到了 SQL标准的SERIALIZABLE(可串行化)**隔离级别。

因为隔离级别越低，事务请求的锁越少，所以大部分数据库系统的隔离级别都是READ-COMMITTED(读取提交内容):，但是你要知道的是InnoDB 存储引擎默认使用 **REPEATABLE-READ（可重读）**并不会有任何性能损失。

InnoDB 存储引擎在 分布式事务 的情况下一般会用到**SERIALIZABLE(可串行化)**隔离级别。

### 实际情况演示

在下面我会使用 2 个命令行mysql ，模拟多线程（多事务）对同一份数据的脏读问题。

MySQL 命令行的默认配置中事务都是自动提交的，即执行SQL语句后就会马上执行 COMMIT 操作。如果要显式地开启一个事务需要使用命令：`START TARNSACTION`。

我们可以通过下面的命令来设置隔离级别。

```
SET [SESSION|GLOBAL] TRANSACTION ISOLATION LEVEL [READ UNCOMMITTED|READ COMMITTED|REPEATABLE READ|SERIALIZABLE]
```

我们再来看一下我们在下面实际操作中使用到的一些并发控制语句:

- `START TARNSACTION` |`BEGIN`：显式地开启一个事务。
- `COMMIT`：提交事务，使得对数据库做的所有修改成为永久性。
- `ROLLBACK`：回滚会结束用户的事务，并撤销正在进行的所有未提交的修改。

#### 脏读(读未提交)

[![img](https://camo.githubusercontent.com/423e9159fb3b438e50d1bd2d1bb6ecb28efb43c0/68747470733a2f2f6d792d626c6f672d746f2d7573652e6f73732d636e2d6265696a696e672e616c6979756e63732e636f6d2f323031392d33312d31e8848fe8afbb28e8afbbe69caae68f90e4baa429e5ae9ee4be8b2e6a7067)](https://camo.githubusercontent.com/423e9159fb3b438e50d1bd2d1bb6ecb28efb43c0/68747470733a2f2f6d792d626c6f672d746f2d7573652e6f73732d636e2d6265696a696e672e616c6979756e63732e636f6d2f323031392d33312d31e8848fe8afbb28e8afbbe69caae68f90e4baa429e5ae9ee4be8b2e6a7067)

#### 避免脏读(读已提交)

[![img](https://camo.githubusercontent.com/7bc1e2826a119d239e6d5a2efce12a4a0d4efe0a/68747470733a2f2f6d792d626c6f672d746f2d7573652e6f73732d636e2d6265696a696e672e616c6979756e63732e636f6d2f323031392d33312d32e8afbbe5b7b2e68f90e4baa4e5ae9ee4be8b2e6a7067)](https://camo.githubusercontent.com/7bc1e2826a119d239e6d5a2efce12a4a0d4efe0a/68747470733a2f2f6d792d626c6f672d746f2d7573652e6f73732d636e2d6265696a696e672e616c6979756e63732e636f6d2f323031392d33312d32e8afbbe5b7b2e68f90e4baa4e5ae9ee4be8b2e6a7067)

#### 不可重复读

还是刚才上面的读已提交的图，虽然避免了读未提交，但是却出现了，一个事务还没有结束，就发生了 不可重复读问题。

[![img](https://camo.githubusercontent.com/96b5be61ff302e4499a47a6c3a84ba70f1546a71/68747470733a2f2f6d792d626c6f672d746f2d7573652e6f73732d636e2d6265696a696e672e616c6979756e63732e636f6d2f323031392d33322d31e4b88de58fafe9878de5a48de8afbbe5ae9ee4be8b2e6a7067)](https://camo.githubusercontent.com/96b5be61ff302e4499a47a6c3a84ba70f1546a71/68747470733a2f2f6d792d626c6f672d746f2d7573652e6f73732d636e2d6265696a696e672e616c6979756e63732e636f6d2f323031392d33322d31e4b88de58fafe9878de5a48de8afbbe5ae9ee4be8b2e6a7067)

#### 可重复读

[![img](https://camo.githubusercontent.com/8731e42dac0807620c48c4a112bd32e4af89591b/68747470733a2f2f6d792d626c6f672d746f2d7573652e6f73732d636e2d6265696a696e672e616c6979756e63732e636f6d2f323031392d33332d32e58fafe9878de5a48de8afbb2e6a7067)](https://camo.githubusercontent.com/8731e42dac0807620c48c4a112bd32e4af89591b/68747470733a2f2f6d792d626c6f672d746f2d7573652e6f73732d636e2d6265696a696e672e616c6979756e63732e636f6d2f323031392d33332d32e58fafe9878de5a48de8afbb2e6a7067)

#### 防止幻读(可重复读)

[![img](https://camo.githubusercontent.com/d9984535df6e3a15ad442295b6fb437f0086ba18/68747470733a2f2f6d792d626c6f672d746f2d7573652e6f73732d636e2d6265696a696e672e616c6979756e63732e636f6d2f323031392d3333e998b2e6ada2e5b9bbe8afbb28e4bdbfe794a8e58fafe9878de5a48de8afbb292e6a7067)](https://camo.githubusercontent.com/d9984535df6e3a15ad442295b6fb437f0086ba18/68747470733a2f2f6d792d626c6f672d746f2d7573652e6f73732d636e2d6265696a696e672e616c6979756e63732e636f6d2f323031392d3333e998b2e6ada2e5b9bbe8afbb28e4bdbfe794a8e58fafe9878de5a48de8afbb292e6a7067)

一个事务对数据库进行操作，这种操作的范围是数据库的全部行，然后第二个事务也在对这个数据库操作，这种操作可以是插入一行记录或删除一行记录，那么第一个是事务就会觉得自己出现了幻觉，怎么还有没有处理的记录呢? 或者 怎么多处理了一行记录呢?

幻读和不可重复读有些相似之处 ，但是不可重复读的重点是修改，幻读的重点在于新增或者删除。

### Next-Key锁

事务的隔离级别中虽然只定义了读数据的要求，实际上这也可以说是写数据的要求。上文的“读”，实际是讲的快照读；而这里说的“写”就是当前读了。

为了解决当前读中的幻读问题，MySQL事务使用了Next-Key锁。

行锁防止别的事务修改或删除，GAP锁防止别的事务新增，行锁和GAP锁结合形成的的Next-Key锁共同解决了RR级别在写数据时的幻读问题。

参考文章：https://tech.meituan.com/2014/08/20/innodb-lock.html

sql优化：https://database.51cto.com/art/202008/623584.htm





​	==============》	（// 30min(9:21 - 9:51)）

**7.hashMap链表和红黑树（什么时候红黑树会退化为链表？）     其中链表的作用？	hashMap计算hash值的具体实现。（后续再总结）**

​		//30min

**8.volicate关键字（后面再看与总结）**

​		//20min

**9.Nio和...     有哪几种io?（后面再看与总结）**

​		//30min

**10.synchrnoized相关知识（后面再看与总结）**

​		//15min

**11.redis相关面试题（后面再看与总结）**