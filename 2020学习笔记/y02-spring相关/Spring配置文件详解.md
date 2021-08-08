**Spring项目实践（三）--- Spring配置文件详解**

```xml
//这些参数在不同阶段之间又住住需要改变,单独写到.properties文件中
<context:property-placeholder location="classpath:jdbc.properties"/>



```








​		不同于我们讲的pom.xml以及web.xml，这两个文件的名称是固定的，不可更改的，这里的设计采用的是约定优于配置的原则。

​	而Spring的配置文件的名称是可以更改的，实际上我们在《Spring项目实践（二）---web.xml文件详解 》中已经给Spring的配置文件命名过了：

```xml
<context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>
        /WEB-INF/spring-*.xml
    </param-value>
</context-param>
```

所有在WEB-INF目录下的，spring-*格式的xml，都是我们的配置文件。
这里为什么我们要使用一个匹配符呢？

其实我们也可以使用唯一的一个文件，但是这样的话，所有的配置都堆在一起，不容易查找和更改。

通常我们根据不同的业务或者功能，会区分出来几个不同的配置文件，例如：

spring-common.xml:用来存放通用的配置

spring-mybatis.xml：用来存放数据库相关的配置

spring-shiro.xml：用来存放shiro相关的配置

跟之前的文章一样，我们还是一步步的来说明：



一.头部beans部分

pom.xml使用project作为root标签，web.xml使用web-app作为root标签，而spring的配置文件使用beans作为root标签。

```xml
<beans xmlns="http://www.springframework.org/schema/beans" 
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  //前两句是基本的命名空间定义
	xmlns:p="http://www.springframework.org/schema/p" 
	xmlns:tx="http://www.springframework.org/schema/tx"  //启动声明事务的命名空间定义
	xmlns:context="http://www.springframework.org/schema/context" //context用来启用自动扫描或者注解配置
	xsi:schemaLocation="http://www.springframework.org/schema/beans  
 http://www.springframework.org/schema/beans/spring-beans-3.1.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-3.1.xsd http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-3.1.xsd">
```

二.自动扫描 or 注解装配
这部分有点意思。

```xml
context:annotation-config> 注解装配

<context:component-scan> 自动扫描
```

先说结论吧：自动扫描包括了注解装配，也就是说，如果你使用了<context:component-scan>，那么就无需再配置<context:annotation-config>。
如果你非要两个都配置，也没有关系，系统会忽略注解装配，直接认为你使用了自动扫描
那么这两者的作用究竟是什么呢？
举例来说明：我们有类A，B和C。其中A使用了B和C的实例，我们采用依赖注入的方式来给A注入BC的实例。
如果没有注解装配和自动扫描的话，我们的代码应该是这样的：

~~~java
public Class A{
	private B b;
	private C c;
	

```
A(){
	printf("create A");
}
void setB(B b){
	printf("create B");
	this.b = b;
}

void setC(C c){
	printf("set C instance");
	this.c = c;
}
```

}

public Class B{
	B(){
		printf("create B");
	}
}

public Class C{
	C(){
		printf("create C");
	}
}
~~~

我们的配置文件是这样的：

```xml
<bean id="bBean" class="com.xxx.B"/>
<bean id="cBean" class="com.xxx.C"/>
<bean id="aBean" class="com.xxx.A">
	<property name="b" ref="bBean">
	<property name="c" ref="cBean">
</bean>
```

当我们装载这三个类的时候，会打印出：
create B
create C
create A
set B instance
set C instance
而如果使用注解装配，我们会这么写：

~~~java
package com.xxx;
public Class A{
	private B b;
	private C c;
	

```
A(){
	printf("create A");
}
@Autowired
void setB(B b){
	printf("set B instance");
	this.b = b;
}

@Autowired
void setC(C c){
	printf("set C instance")
	this.c = c;
}
```

}

package com.xxx;
public Class B{
	B(){
		printf("create B");
	}
}

package com.xxx;
public Class C{
	C(){
		printf("create C");
	}
}
~~~

可以看到，我们就加了两个注解@Autowired
这样的话，配置文件里就不用写property参数了
<bean id="bBean" class="com.xxx.B"/>
<bean id="cBean" class="com.xxx.C"/>
<bean id="aBean" class="com.xxx.A"/>

但是这样运行一下，你会发现，打印出来的是：
create B
create C
create A


也就是说实际上没有执行两个set方法的bean注入。
这是为啥呢？
因为我们需要配置一下注解装配，这样才可以使用注解：
<context:annotation-config />
<bean id="bBean" class="com.xxx.B"/>
<bean id="cBean" class="com.xxx.C"/>
<bean id="aBean" class="com.xxx.A"/>

现在执行一下，打印是正确的。
可以看到，使用注解装配，我们就不需要在配置文件里面指定bean的参数了


但是现在，我觉得在配置文件里面写bean也很麻烦，这个时候就轮到自动扫描登场了。

~~~java
package com.xxx;
@Component
public Class A{
	private B b;
	private C c;
	

```
A(){
	printf("create A");
}
@Autowired
void setB(B b){
	printf("set B instance");
	this.b = b;
}

@Autowired
void setC(C c){
	printf("set C instance")
	this.c = c;
}
```

}

package com.xxx;
@Component
public Class B{
	B(){
		printf("create B");
	}
}

package com.xxx;
@Component
public Class C{
	C(){
		printf("create C");
	}
}
~~~

可以看到，我们又加了一种注解@Component，这个注解会告诉spring，带有这些注解的类，需要被声明成bean
配置文件这样写：
<context:component-scan base-package="com.xxx"/>

这样spring就会去扫描com.xxx包下面的所有类，找到带有@Component注解的，并把它声明成一个bean。
更具体的，可以看这篇博客：https://www.cnblogs.com/leiOOlei/p/3713989.html

另外补充一个进阶知识：

```xml
<context:component-scan base-package="com.xxx">
	<context:exclude-filter type="annotation"
		expression="org.springframework.stereotype.Controller" />
</context:component-scan>
```

可以看到，我们添加了一个参数叫做context:exclude-filter，type是annotation。这里的意思是，不去扫描所有的Controller的实现类（Controller是spring的一个接口）
为什么不去扫描Controller的实现类呢？



是因为上面的配置文件，是spring的。


而对Controller的扫描，实际上是springmvc的任务，所以我们通常会在springmvc的配置文件里面加上：

<context:component-scan base-package="com.xxx">
		<context:include-filter type="annotation"
			expression="org.springframework.stereotype.Controller" />
	</context:component-scan>
注意，这里不是exclude，而是include。


三.bean部分
是不是使用了自动扫描，就不需要在配置文件里面写bean了？


当然不是，因为自动扫描只能扫描你自己的包和类，对于第三方的，尤其是spring自带的，我们还是需要配置一下的。
下面列出来几个经常用到的bean


1.PropertyPlaceholderConfigurer 

这个bean的作用是：允许我们使用属性配置文件.properties
关于.properties文件，其实我们在讲pom.xml的时候就遇到过了，你可以使用.properties文件，来定义一些键值对，针对不同的开发环境，配置不同的参数。
在pom中，我们使用.properties来定义开发环境和生产环境不同的编译参数。
在spring中，我们经常使用.properties来定义开发环境和生产环境不同的数据库参数。
比如mybatis的配置：

```xml
 <bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
        <property name="driverClassName" value="${jdbc.driver}"/>
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
        <property name="maxWait" value="100"/>
        <property name="maxActive" value="300"/>
        <property name="maxIdle" value="100"/>
        <property name="defaultAutoCommit" value="true"/>
        <property name="removeAbandonedTimeout" value="60"/>
        <property name="logAbandoned" value="false"/>
        <property name="testOnBorrow" value="true"/>
        <property name="testWhileIdle" value="true"/>
        <property name="validationQuery" value="select 1"/>
        <property name="poolPreparedStatements" value="true"/>
        <property name="timeBetweenEvictionRunsMillis" value="3000"/>
        <property name="minEvictableIdleTimeMillis" value="3000"/>
    </bean>
```

对应的.properties文件：

```properties
profile.jdbc.driver=com.mysql.jdbc.Driver
profile.jdbc.url=jdbc:mysql://localhost:3306/xxx?characterEncoding=utf-8&autoReconnect=true
profile.jdbc.username=root
profile.jdbc.password=root
```

那么spring如何扫描并找到这些.properties文件的呢？就是使用了我们上面说的PropertyPlaceholderConfigurer。
在配置文件里面，是这样来配置的：
	

```xml
<bean id="propertyConfigurer"
		class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
		<property name="order" value="1" />
		<property name="ignoreUnresolvablePlaceholders" value="true" />
		<property name="locations">
			<list>
				<value>classpath:common.properties</value>
				<value>classpath:jdbc.properties</value>
			</list>
		</property>
	</bean>
```

关注点是classpath：common.properties和classpath：jdbc.properties
这里classpath是你的编译路径，具体到文件夹，默认是target/WEB-INF/classes
但是这不意味着你要把common.properties和jdbc.properties两个文件放到classpath路径下，实际上你也无法放过去，因为classpath是编译时生成的一个文件夹。
所以我们通常把.properties文件放到resource文件夹下面，resource文件夹下面的文件不会参与编译，但是编译器会把它下面的文件移到classpath下。
PS：除了使用PropertyPlaceholderConfigurer，我们还可以使用PropertiesFactoryBean来管理.properties文件（其实这两个类都继承了PropertiesLoaderSupport类，不过使用上有所不同）。
更详细的可以看这个：
http://blog.csdn.net/caomiao2006/article/details/51813564

2.配置JDBC
这个其实代码我们上面已经说过了，同理还需要配置mybatis等等
————————————————
原文链接：https://blog.csdn.net/l00149133/article/details/78970770