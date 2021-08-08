### @Transactional(rollbackFor=Exception.class)的使用

**1.先来看看异常的分类：**

![img](https://img-blog.csdn.net/20171027131229769)

error是一定会回滚的；

Exception是异常，他又分为运行时异常RuntimeException和非运行时异常；

![img](https://img-blog.csdn.net/20171027133607797?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvTWludDY=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

可查的异常（checked exceptions）:Exception下除了RuntimeException外的异常

不可查的异常（unchecked exceptions）:RuntimeException及其子类和错误（Error）



​	  如果不对运行时异常进行处理，那么出现运行时异常之后，要么是线程中止，要么是主程序终止。 
如果不想终止，则必须捕获所有的运行时异常，决定不让这个处理线程退出。队列里面出现异常数据了，正常的处理应该是把异常数据舍弃，然后记录日志。不应该由于异常数据而影响下面对正常数据的处理。



​      非运行时异常是RuntimeException以外的异常，类型上都属于Exception类及其子类。如IOException、SQLException等以及用户自定义的Exception异常。对于这种异常，JAVA编译器强制要求我们必需对出现的这些异常进行catch并处理，否则程序就不能编译通过。所以，面对这种异常不管我们是否愿意，只能自己去写一大堆catch块去处理可能的异常。

**2.@Transactional 的写法**

```java
	@Transactional`注解既可以写在方法上也可以写在类上。写在类上则这个类的所有public方法都将会有事务。这样显然会使一些不需要事务的方法产生事务，所以更好的做法是在想要添加事务的方法上标注`@Transactional`。
```

​	@Transactional如果只这样写，

**Spring框架的事务基础架构代码将默认地 只 在抛出运行时和unchecked exceptions时才标识事务回滚( 即抛出**`RuntimeException` **或其子类例的实例或者ERROR时**)。**但是从事务方法中抛出的****Checked exceptions将** **不 被标识进行事务回滚。**

**注意**： 如果异常被try｛｝catch｛｝了，事务就不回滚了，如果想让事务回滚必须再往外抛try｛｝catch｛throw Exception｝。（只是在try抛出异常，catch之后未抛出，这种业务方法也就等于脱离了spring事务的管理，因为没有任何异常会从业务方法中抛出，全被捕获并“吞掉”，导致spring异常抛出触发事务回滚策略失效，因而不会触发回滚。）



**3.玩转Spring —— 消失的事务**

**一个没有加@Transactional注解的方法，去调用一个加了@Transactional的方法，会不会产生事务？**

文字苍白，还是用代码说话。

先写一个@Transactional的方法（**本文的所有代码，可到Github上下载**）：

```java
@Transactional
public void deleteAllAndAddOneTransactional(Customer customer) {
 customerRepository.deleteAll();
 if ("Yang".equals(customer.getFirstName())) {
 throw new RuntimeException();
 }
 customerRepository.save(customer);
}
```

方法内先去执行deleteAll()，删除表中全部数据；然后执行save()保存数据。

这两个方法中间，会判断传进来的firstName是不是等于“Yang”，是，则抛异常，用于模拟两个数据库操作之间可能发生的异常场景。

如果没有加@Transactional注解，那么这两个操作就不在一个事务里面，不具有原子性。如果deleteAll之后抛异常，那么就会导致只删除不新增。

而加了@Transactional之后，这两个动作在一个事务里头，具有原子性，要么全部成功，要么全部失败。如果deleteAll之后抛异常，则事务回滚，恢复原先被删除的数据。

测试一下，启动Spring Boot工程，首先调用findAll接口，看看数据库中都有哪些数据：

![img](https://pic4.zhimg.com/80/v2-aab046be31e137c4f32dfd8254909e7b_720w.jpg)



接着调用deleteAndSave接口，故意传入firstName=”Yang”，果然返回失败：

![img](https://pic3.zhimg.com/80/v2-0e851443752b34037874cdadccc8d9e6_720w.jpg)



然后，在回过头去调用findAll接口，看看数据是不是还在：

![img](https://pic4.zhimg.com/80/v2-aab046be31e137c4f32dfd8254909e7b_720w.jpg)



数据都在，说明产生事务了。

上面都没啥，都跟符合我们的直觉。

问题来了，如果我的接口是去调用一个没有加@Transactional的方法，然后这个方法再去调用加了@Transactional的方法呢？

```java
public void deleteAllAndAddOne(Customer customer) {
 System.out.println("go into deleteAllAndAddOne");
 deleteAllAndAddOneTransactional(customer);
 }


 @Transactional
 public void deleteAllAndAddOneTransactional(Customer customer) {
 customerRepository.deleteAll();
 if ("Yang".equals(customer.getFirstName())) {
 throw new RuntimeException();
 }
 customerRepository.save(customer);
 }
```

直觉告诉我，会的。

重新编译，启动，调用新的接口，继续故意让它抛异常：

![img](https://pic1.zhimg.com/80/v2-a9a6df3e5e8581d17add47997c542f90_720w.jpg)



然后再去findAll，看看数据还在不在：

![img](https://pic3.zhimg.com/80/v2-5dd1f5dd8712a3c4375a645f5b05d0ca_720w.jpg)



**WTF! 空空如也！数据都没了！**

**看来我又一次被直觉欺骗了。**

还是得老老实实看代码，弄懂原理。

看了一晚上代码，恍然大悟。咱们先画个图解释一下，再来看看代码。

## 图解@Transactional

首先，我们得先弄懂@Transactional的原理。

为什么第一种情况，也就是直接调用@Transactional方法，会产生事务？

**其实Spring的@Transactional，跟Spring AOP一样，都是利用了动态代理。**

我们写了一个类，里面写了一个加了@Transactional注解的方法，这原本平淡无奇，什么用也没有，就像这样：

![img](https://pic1.zhimg.com/80/v2-e7543500ba039c79126ef97edd1c4eb8_720w.jpg)



关键在于，Spring在检查到@Transactional注解之后，给这个对象生成了一个代理对象proxy：

![img](https://pic4.zhimg.com/80/v2-d4123abbe4f094b6e279bdd62cc0bfa7_720w.jpg)



代理对象的methodB，会先开启事务（beginTransaction），然后再去执行原先对象target的methodB，如果抛异常，则回滚（rollBack），如果一切顺利，则提交（commit）。

而最后注入Spring容器的，也正是这个带有事务逻辑的代理对象。所以我们调用methodB时会产生事务。

现在，我们写了一个新方法，methodA，里头去调用methodB：

![img](https://pic2.zhimg.com/80/v2-bc6ab354334ec7e95842fdba4f09ce29_720w.jpg)



从上面的分析，可以知道，我们最终拿到的，是代理对象。

那么代理对象的methodA是长什么样子的呢？长这样：

![img](https://pic3.zhimg.com/80/v2-82e4bc379cd122beef50aa0a08bd5606_720w.jpg)



由于methodA没有加@Transactional注解，所以代理对象里面，直接就是target.methodA()，直接调用了原来对象的methodA。

这下就很清晰了，代理对象的methodA，去调用原来对象的methodA，原来对象的methodA，再去调用原来对象的methodB，而原来对象的methodB，是不具有事务的。事务只存在于代理对象的methodB. 所以整个方法也就没有事务了。

## 看看代码

最后再来看看代码。

只需要在deleteAllAndAddOneTransactional方法内，打一个断点，一切了然。

分别调用两个接口，比较调用堆栈：

![img](https://pic3.zhimg.com/80/v2-ea7e51ac41a02dcf76362f4513649cd6_720w.jpg)



明显可以看出，直接调用@Transactional方法，堆栈更长，而且会经过一个叫TransactionInterceptor的拦截器。

跟着堆栈往上走，会发现关键就在于这个if-else的逻辑，CglibAopProxy：

![img](https://pic3.zhimg.com/80/v2-69b53c4822092926a5758ae1d1f1bbc2_720w.jpg)



CglibAopProxy会去检查要调用的方法，有没有AOP调用链：

- 没有，则走if里面的逻辑，直接调用target对象的方法，也就是上面间接调用@Transactional方法的情形；
- 有，则走else逻辑，也就是直接调用@Transactional方法的情形。

当然，如果deleteAllAndAddOne方法被别的切面拦截，那么调用链chain也不会为空，也会走if逻辑，这时候是否会有事务呢？思考题。





总结：如果抛出的异常在本方法未被捕获，则继续往上层方法抛，如果上层方法有try{}catch{}，则被捕获；如果上层没有try{}catch{},则继续往外抛，如此反复。

