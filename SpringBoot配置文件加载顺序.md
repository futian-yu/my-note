**SpringBoot配置文件加载顺序**



首先注意：一个可运行的springBoot项目只能有一个application.properties文件，若要配置多环境，如:

```properties
application-dev.properties：开发环境
application-test.properties：测试环境
application-pro.properties：生产环境
```

需要在application.properties中添加：

spring.profiles.active = dev/prod



spring boot 启动会扫描以下位置的application.properties或者application.yml文件作为Spring boot的默认配置文件

```properties
–file:./config/
–file:./
–classpath:/config/
–classpath:/
```

即如下图所示：

![](./images/8.jpg)

以上是按照优先级从高到低(1-4)的顺序，所有位置的文件都会被加载，高优先级配置内容会覆盖低优先级配置内容。

SpringBoot会从这四个位置全部加载主配置文件，如果高优先级中配置文件属性与低优先级配置文件不冲突的属性，则会共同存在—互补配置。

我们也可以通过配置spring.config.location来改变默认配置。

java -jar spring-boot-02-config-02-0.0.1-SNAPSHOT.jar --spring.config.location=D:/application.properties
1
项目打包好以后，我们可以使用命令行参数的形式，启动项目的时候来指定配置文件的新位置。指定配置文件和默认加载的这些配置文件共同起作用形成互补配置。
————————————————

也可**@Configuration注解类上的@PropertySource**配置加载顺序，具体可自行Google。

