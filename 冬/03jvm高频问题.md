jvm高频问题

1.jvm一个类是如何加载的？

按需加载，需要用到的时候再加载。

![1644662189854](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1644662189854.png)

![1644662323414](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1644662323414.png)

常量赋正式值。

![1644662383755](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1644662383755.png)



![1644663132206](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1644663132206.png)

![1644663667397](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1644663667397.png)

应用程序类加载器还加载用户类路径（ClassPath）下的所有的类库。

//自定义加载器可以自己写，想加载哪里就加载哪里，自己写路径。



![1644664915971](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1644664915971.png)

![1644665053057](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1644665053057.png)

![1644665215589](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1644665215589.png)

![1644665398749](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1644665398749.png)

为了安全性，jdk不允许你自己设置的包名和java核心类库包名相同。

![1644666376896](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1644666376896.png)

![1644666942931](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1644666942931.png)



![1644669266108](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1644669266108.png)

![1644669569147](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1644669569147.png)

![1644669593225](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1644669593225.png)

![1644669638358](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1644669638358.png)

![1644670029709](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1644670029709.png)

![1644670165863](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1644670165863.png)

![1644674731743](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1644674731743.png)



![1644674979792](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1644674979792.png)

![1644675230199](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1644675230199.png)

![1644675395715](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1644675395715.png)

![1644675418864](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1644675418864.png)

![1644675791886](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1644675791886.png)

![1644676274858](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1644676274858.png)

![1644676591338](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1644676591338.png)

![1644676776037](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1644676776037.png)

![1644676853027](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1644676853027.png)

![1644676866334](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1644676866334.png)

![1644677281019](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1644677281019.png)

![1644677864032](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1644677864032.png)

![1644677882066](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1644677882066.png)



![1644677950419](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1644677950419.png)

![1644678167140](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1644678167140.png)

![1644680116634](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1644680116634.png)

![1644680173337](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1644680173337.png)

![1644680196102](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1644680196102.png)

==========>什么是老年代空间分配担保机制？

![1644680402406](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1644680402406.png)

![1644680490962](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1644680490962.png)

![1644680535262](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1644680535262.png)

![1644680545234](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1644680545234.png)

![1644680592789](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1644680592789.png)

![1644680673160](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1644680673160.png)

![1644681047481](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1644681047481.png)

![1644681065017](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1644681065017.png)

![1644681246252](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1644681246252.png)

![1644681313804](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1644681313804.png)

========》jvm为什么要将堆分成年轻代和老年代？

因为年轻代朝生夕死，老年代存活时间比较长。各自回收频率和适合的算法不同。

![1644681609413](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1644681609413.png)

![1644681735934](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1644681735934.png)

![1644681977540](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1644681977540.png)

![1644682036914](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1644682036914.png)

![1644682045780](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1644682045780.png)

![1644682066531](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1644682066531.png)

![1644682082336](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1644682082336.png)

![1644682149005](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1644682149005.png)

=========》复制算法

![1644682328105](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1644682328105.png)

![1644682389422](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1644682389422.png)

![1644682427668](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1644682427668.png)

![1644682641870](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1644682641870.png)

![1644682654328](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1644682654328.png)

![1644682685056](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1644682685056.png)

![1644682718050](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1644682718050.png)

![1644737003764](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1644737003764.png)

![1644737024900](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1644737024900.png)

![1644737125608](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1644737125608.png)

![1644737138610](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1644737138610.png)

![1644738843244](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1644738843244.png)

![1644738975419](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1644738975419.png)

![1644741430421](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1644741430421.png)







二、问题记录

```java
//1.继承时父子类的初始化顺序是怎样的？


//2.JVM有哪些类加载器？

//3.JVM不同的类加载器分别加载哪些文件？（ms必问）

//4.JVM三层类加载器之间的关系是继承吗？
	不存在继承关系。Bootstrap ClassLoader是c++实现的，另外两个是java实现的。包括自定义类加载器，继承的是java.lang.ClassLoader.
//5.你了解JVM类加载的双亲委派模型吗？优缺点？

//6.JDK为什么要设计双亲委派模型，有什么好处？


//7.可以打破JVM双亲委派模型吗？如何打破JVM双亲委派模型？
可以。想要打破这种模型，那么就自定义一个类加载器，重写其中的loadClass方法，使其不进行双亲委派即可。(java.lang.ClassLoader的loadClass方法里通过递归实现了双亲委派)。        
        
//8.如何自定义自己的类加载器？
1）继承ClassLoader
2）覆盖findClass(String name)方法或loadClass()方法。（覆盖findClass(String name)方法不会打破双亲委派模型--更好，因为切入点更小；覆盖loadClass()方法会打破）

//9.ClassLoader中的LoadClass()、findClass()、defineClass()的区别？

//10.加载一个类采用Class.forName()和ClassLoader有什么区别？
Class.forName()加载类的时候会完成类的初始化，ClassLoader去加载类的时候不会完成类的初始化，只到链接阶段。

//11.你了解Tomcat的类加载机制吗？
Tomcat自定义了WebAppClassLoader类加载器，打破了双亲委派的机制，即如果收到类的加载请求，首先会自己尝试去加载，如果找不到再交给父加载器去加载，目的就是为了优先加载Web应用自己定义的类，我们知道ClassLoader默认的loadClass方法是以双亲委派模型进行加载的，那么Tomcat打破了这个规则，重写了loadClass方法，我们可以看到WebAppClassLoader类中重写了loadClass方法。

//12.为什么Tomcat要破坏双亲委派模型？

//13.有没有听说过热加载和热部署，如何自己实现一个热加载？

//14.请介绍一下JVM内存结构划分？

//15.JVM哪些区域是线程私有的，哪些区域是线程共享的？

//16.JVM运行时数据区-程序计数器的特点及作用？

//17.JVM运行时数据区-虚拟机栈的特点及作用？

//18.JVM运行时数据区-本地方法栈的特点及作用？

//19.JVM运行时数据区-Java堆的特点及作用？

//20.JVM中对象如何在堆内存分配？

//21.JVM什么情况下会发生堆内存溢出？

//22.JVM如何判断对象是否可以回收？

//23.说一下JVM堆内存分代模型？
JVM堆内存的分代模型：年轻代，老年代。(1/3,8:1:1;2/3)	

//24.请介绍一下JVM堆中新生代的垃圾回收过程？
        
//25.JVM对象动态年龄判断是怎么回事？        

//26.什么是老年代空间分配担保机制？

//27.什么情况下对象会进入老年代？
        
//28.JVM运行时数据区-元空间(方法区/永久代)的特点及作用？
        
//29.JVM本机直接内存的特点及作用？
        
//30.堆为什么要分成新生代和老年代？
        
//31.JVM堆的年轻代为什么要有两个Survivor区?
        
//32.Eden区与Survivor区的空间大小比值为什么默认是8：1：1？
        
//33.请介绍一下JVM中的垃圾回收算法？
1)标记清除

2)复制算法
        
3)标记-整理
        
4)分代收集        
        
//33.请介绍一下JVM垃圾收集器Concurrent Mark Sweep?
(jdk8默认用的是Parallel Scavenge加Parallel Old，jdk9默认就用G1了)

//34.什么是内存泄漏？什么是内存溢出？什么场景产生的？怎么解决的？


//35.你们线上环境的JVM都设置多大？        

//36.JVM堆溢出后，其他线程是否还可以继续工作？
可能可以继续工作，也可能不能继续工作。如果是全局变量导致的内存溢出，也可能不能继续工作(方法内需要堆空间，但是已经满了)，或者响应很慢。        

//37.Java中的对象都是在堆中分配吗？说明为什么！（逃逸分析）
https://segmentfault.com/a/1190000038262877

//38.cms 收集器过程？ 

//39.G1收集器原理？怎么实现可预测停顿的？region的大小，结构。

//40.说说synchronized锁升级过程，轻量锁可以变成偏向锁么？偏向锁可以变成无锁么？自旋锁，对象头结构，锁状态变化过程？

//41.说一下JVM的线程模型？这些区域都分别是干啥用的？java线程模型和jvm线程模型注意区分

//42.Java GC机制？GC Roots有哪些？

//43.方法区和直接内存什么时候会OOM？

//44.JVM收集器G1的内存模型和CMS的内存模型有什么不同？

//45.JVM调优用过吗？

```

































