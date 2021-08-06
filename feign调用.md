**feign调用**

​		feign调用可以调用同一个erueka注册中心的服务api,也可以调用注册中心之外的api，如下。

​		1.调用同一个注册中心的api;

​		2.调用注册中心之外的其他api;

​		以下方法中，name指定的是注册中心所对应的服务的名称，url则是指定具体的ip/域名；当name和url同时存在时，url优先级更高。

```java
@FeignClient(name = "bss-service", url = "${bss-web-service-url}", fallback = BssServiceFeignImpl.class)
public interface BssServiceFeign {

    /**
     * 调用服务中心，发送过期license失效提醒
     */
    @RequestMapping(value = "/sc/system/expire/innerMsg", method = RequestMethod.POST)
    void expireInnerMsg();

    /**
     * 调用报表中心，统计昨日在线设备
     */
    @GetMapping(value = "/sc/service/job/dailyLicense")
    void subscribeDurationByUsername();

}

```

​		注意：别忘了在启动类上加上@EnableFeignClients注解，否则会报错。