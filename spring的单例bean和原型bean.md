**spring的单例bean和原型bean**

一、前置知识了解.

1.在Spring框架中由Spring创建和管理的对象的对象称之为Bean对象。

2.spring在启动的时候，默认会创建所有的Bean对象，但是有的bean对象如果长时间是用不到的，那启动的时候就创建出来就会占用资源，所以这时候spring提供了懒加载机制，即启动时不创建懒加载的Bean，在你需要用的时候自己去手动创建。懒加载可以在类上加注解@Lazy。

3.spring最基本的两种作用作用域:单例(singleton)和多例(protoType)两种.



二、spring的单例bean和原型bean

1.spring默认创建的就是单例bean. 例如在类上加注解@Controller、@Service 注入的bean都是单例bean.

2.创建出来的单例Bean放在这样一个Map中，所以每次创建bean的时候，首先在singletonObjects这个Map中根据beanName找对应的单例bean，如果找到直接返回；如果没找到，调用singletonFactory工厂中的creatBean方法创建一个新的bean.（这里的createBean是抽象方法，具体实现交给了各种scope的实现类负责）

```java
 /** Cache of singleton objects: bean name --> bean instance */
    private final Map<String, Object> singletonObjects = new ConcurrentHashMap<String, Object>(256);
```



3.单例bean的优势:

- 节省资源，节省创建实例所消耗的资源.
- 减少jvm垃圾回收，创建实例少，回收也就少了.

- 可以从缓存中快速获取bean.



