**Spring AOP动态代理**

1.为什么要使用代理？

- 增强被代理对象的能力.

2.代理的类型：

 - 静态代理:静态代理要求代理类和委托类都实现同一个接口，代理类里面实现的方法可以增强委托类的能力.（只需传入委托类,通过重载的方法增强额外能力,再执行委托类原本的能力即可）.

 - 动态代理

    - JDK动态代理: **JDK动态代理要求必须有接口**,委托类实现了该接口。然后代理过程中会生成一个匿名类作为代理对象，并且这个代理对象是由代理生成工具动态生成的(Proxy.newProxyInstance)。

      ```java
      //上面的代码实现，可以修改成下面这种方式，同样能实现功能。
      // Subject.java
      public interface Subject {
          
          public void sayHello();
          
      }
      //类 RealSubject.java
      public class RealSubject implements Subject {
      
          @Override
          public void sayHello() {
              System.out.println("hello！");
          }
          
      }
      //类 InvocationHandlerImpl.java
      public class InvocationHandlerImpl implements InvocationHandler {
      
          private Object object;
      
          public InvocationHandlerImpl(Object object) {
              this.object = object;
          }
      
          public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
              System.out.println("Before say hello...");
              Object returnValue = method.invoke(subject, args);
              System.out.println("After say hello...");
              return returnValue;
          }
      }
      //方法 main
          public static void main(String[] args) {
              Subject realSubject = new RealSubject();
              InvocationHandler handler = new InvocationHandlerImpl(realSubject);
              ClassLoader loader = realSubject.getClass().getClassLoader();
              Class[] interfaces = realSubject.getClass().getInterfaces();
              Subject subject = (Subject) Proxy.newProxyInstance(loader, interfaces, handler);
              subject.sayHello();
          }
      ```

      

    - cglib动态代理：如果一个类没有实现接口，它也需要代理，那么可以用cglib动态代理实现。cglib动态代理的原理是生成一个目标类的子类，这个子类就是增强过的代理对象。注意这里是通过修改委托类的class文件的字节码并生成子类来生成代理对象的（注：不管委托类是否实现接口，都可以使用cglib动态代理）

```java
//委托类 RealSubject.java
public class RealSubject {
    public void sayHello() {
        System.out.println("hello！");
    }
}
//代理类 MethodInterceptorImpl.java
public class MethodInterceptorImpl implements MethodInterceptor {
    @Override
    public Object intercept(Object obj, Method method, Object[] args, MethodProxy methodProxy) throws Throwable {
        System.out.println("Before say hello...");
        Object returnValue= methodProxy.invokeSuper(obj, args);
        System.out.println("After say hello...");
        return returnValue;
    }

}
//测试方法 main
    public static void main(String[] args) {
        RealSubject target = new RealSubject();
        RealSubject proxy = (RealSubject) Enhancer.create(target.getClass(),
                new MethodInterceptorImpl());
        proxy.sayHello();
    }
```















