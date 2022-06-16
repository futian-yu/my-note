### @Configuration注解

1.构造函数的参数、@bean的参数是如何来的？

​	初始化容器，@Configuration对象在创建的的时候，也会将容器中已经有的类自动给构造函数。@Bean注解想容器注入对象的时候，会自动将容器中已经有的对象传入到@Bean注解的方法参数中。（@Configuration相当于初始化一个容器，@bean相当于自动注入一个bean。）



2.类中各注解的初始化顺序？

Construct构造器------>@Autowire------>@PostConstruct

