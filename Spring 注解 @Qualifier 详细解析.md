# Spring 注解 @Qualifier 详细解析

原文：https://juejin.cn/post/6844904035342893063

**1.@Autowire注解**

​		spring使用@Autowire进行依赖注入很方便，但是有些场景下仅仅靠这个注解spring还是不知道要注入哪个bean。默认情况下，`@Autowired` 按类型装配 **Spring Bean**。 如果容器中有多个相同类型的 **bean**，则框架将抛出 `NoUniqueBeanDefinitionException`， 以提示有多个满足条件的 **bean** 进行自动装配。例如：

```java
  @Component("fooFormatter")
    public class FooFormatter implements Formatter {
        public String format() {
            return "foo";
        }
    }

    @Component("barFormatter")
    public class BarFormatter implements Formatter {
        public String format() {
            return "bar";
        }
    }

    @Component
    public class FooService {
        @Autowired
        private Formatter formatter;
        
        //todo 
    }
```

​         如果我们尝试将 `FooService` 加载到我们的上下文中，**Spring** 框架将抛出 `NoUniqueBeanDefinitionException`。这是因为 **Spring** 不知道要注入哪个 **bean**。为了避免这个问题，有几种解决方案。那么我们本文要讲解的 `@Qualifier` 注解就是其中之一。

​         通过使用 `@Qualifier` 注解，我们可以消除需要注入哪个 **bean** 的问题。让我们重新回顾一下前面的例子，看看我们如何通过包含 `@Qualifier` 注释来指出我们想要使用哪个 **bean** 来解决问题：

```
    @Component
    public class FooService {
        @Autowired
        @Qualifier("fooFormatter")
        private Formatter formatter;
        
        //todo 
    }
```






















