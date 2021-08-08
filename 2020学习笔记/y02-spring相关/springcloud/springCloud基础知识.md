**springCloud**

![image-20200917145339582](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20200917145339582.png)



支付微服务集群设置：不直接访问ip端口号，支需关注Eurera中暴露出来的微服务名称(微服务中有几个服务集群，并不关心，端口也并不关心，默认是轮询算法，轮询访问各个集群端口)；

使用@LoadBalanced注解赋予RestTemplate负载均衡的能力；

Ribbon和Eureka整合之后,Cobsumer可以直接调用服务而不用关心地址和端口号，且该服务还有负载均衡功能了(轮询算法)。



1.Eureka相当于一个病人与专家中间的门诊挂号,通过这个门诊挂号可以清楚的知道专家是否空闲,前面还有几个人,如何安排门诊专家等. 其实如果病人少,就算没有这个门诊,也可以一对一看病,怕的就是病人太多,量变引起质变,导致全部混乱.(Eureka:服务注册,服务治理.)

​			服务注册:将服务信息注册进注册中心

​			服务发现:从注册中心上获取服务信息

​			实质:存key服务名, 取value调用地址;

![image-20201116114034058](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20201116114034058.png)

​	Eureka的自我保护机制: 一句话: 某时刻某一个微服务不可用了,Eureka不会立刻清理,依旧会对该微服务的信息进行保存.(属于CAP里面的AP).



2.我本地的机器在eureka上看得到,说明我本地的服务注册进了erreka的注册中心.(springcloud是在本地服务上pom引进了Eureka依赖,然后再yml文件中配置了eureka的注册地址等,然后在主启动类上添加了@EnableEureka)

![image-20201115214749211](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20201115214749211.png)

支付服务注册进zookeeper(springCloud整合zookeeper).



**Consul**:也是一个服务注册中心.(分布式的服务发现和配置管理系统)

- Consistency(强一致性)： 数据一致更新，所有数据变动都是同步的
- Availability(可用性)： 好的响应性能
- Partition tolerance(分区容错性)： 可靠性

三个注册中心异同点：

![image-20200916214822885](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20200916214822885.png)



**Ribbon:** 负载均衡+RestTemplate调用;（其实Eurera默认包含了Ribbon，可以从pom的依赖关系中看出）

RestTemplate:

​		getForObject方法/getForEntity方法；

​		postForObject/postForEntity方法；返回json串



负载均衡默认算法：rest接口第几次请求数 % 服务器集群总数量 = 实际调用服务器位置下标，每次服务重启后rest接口计数从1开始；

​	

**OpenFegin:**  服务调用，服务接口绑定器，用起来比Ribbon简单些;



​	**Hystrix断路器**：Hystrix是一个用于处理分布式系统的延迟和容错的开源库，在分布式系统里，许多依赖不可避免地会调用失败，比如超时、异常等，Hustrix能够保证在一个依赖出问题的情况下，不会导致整体服务失败，避免级联故障，以提高分布式系统的弹性（高可用）。

​	“断路器本身是一种开关装置”，当某个服务单元故障后，他可以像调用方返回一个备选响应，而不是长时间的等待或者抛出调用方无法处理的异常。

​		**三大作用：**

​			**服务降级：**fallback   触发时点：

​					程序运行异常；

​					 超时；

​					 服务熔断触发服务降级；

​					 线程池/信号量打满也会导致服务降级；

​			**服务熔断：**break

​					类比保险丝达到最大服务访问后，直接拒绝访问，拉闸限电，然后调用服务降级的方法；

​			**服务限流：**flowlimit

​					秒杀高并发等操作，严禁一窝蜂的过来拥挤；

fallback:运行时异常、超时、宕机



服务降级：...

服务熔断：当检测到该节点微服务调用响应正常后，熔断后可以自恢复调用链路。

​			多次错误，慢慢发现正确率搞了，闸道才会打开，链路才会合并；

监控：7色一圈一线



**Zuul和GateWay**：Zuul已被淘汰,Gateway是spring社区自行研发的新一代网关

​	springCloud Gateway使用的Webflux中的reactor-netty响应式编程组件，底层使用了Netty通讯框架。

gateway作用：反向代理、鉴权、流量控制、熔断、日志监控；	

![image-20200918175957722](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20200918175957722.png)



三大核心概念：Route(路由)、Predicate(断言)、Filter（过滤）

可以不暴露真实的端口，统一暴露gateway的端口

![image-20200921120501108](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20200921120501108.png)

gateway有两种配置方式：1.yml；   2.配置类

yml配置中的属性：Predicate:断言；

Filter：如：全局日志记录、统一网关权鉴	



**SpringCloud Config**为微服务架构中的微服务提供集中化的外部配置支持，配置服务器为各个不同微服务应用的所有环境提供了一个中心化的外部配置。

总线bus：![image-20200921155813693](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20200921155813693.png)



市面上常用的四种消息中间件：Ribbit Mq,rocket Mq,kafka...

**SpringCloud Stream消息驱动**：屏蔽底层消息中间件的差异，降低切换成本，统一消息的编程模型。

Binder:  input对应生产者，output对应消费者。	

![image-20200921165522486](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20200921165522486.png)

Sleuth[sluθ]:链路监控展现。包含了zipkin(和Sleuth一样);



**Cloud Alibaba**

Netflix宣布2018/12/12整个进入维护模式(Maintenance Mode) 



Nacos：服务注册中心、服务配置中心、总线（Nacos = Eureka + Config + Bus）;



​     Nacos集群和持久化配置(重要):默认Nacos使用嵌入式数据库实现数据的存储。所以，如果启动多个默认配置下的Nacos节点，数据存储时存在一致性问题的。为了解决这个问题，Nacos采用了集中式存储的方式来支持集群化部署，目前只支持Mysql的存储。





Sentinel:P111

​	**Sentinel:**分布式系统的流量防卫兵。

​			流控规则：限流

​					关联： b达到阈值，a限流；

​					预热：

​					排队等待：

**Seata**

​		Seata是一款开源的分布式事务解决方案，致力于在微服务架构下提供

高性能和简单易用的分布式事务