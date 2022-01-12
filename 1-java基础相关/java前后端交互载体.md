**java前后端交互**

​	前端和后端交互需要载体，这个载体以我现在了解有两种，**一个是json字符串，另一个就是实现serializable持久化对象。**

1.为什么springBoot项目实体类没有序列化也能和前端交互?

​	使用springMvc返回java对象数据时会自动转为json格式传给前端，这是因为SpringMvc的@RestController和@ResponseBody注解使用时返回的java对象数据会自动转化为json格式。

2.对象序列化和反序列化是什么意思？

​	对象序列化是指一个将对象状态转换为二进制字节流的过程，可以将其保存到磁盘或者通过网络发送到任意平台。将二进制字节流转换为对象的过程称为反序列化。(实现java.io.Serializable接口启用可序列化)	

