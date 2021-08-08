**服务器tomcat启动的几种方式，catalina.out实时日志查看**

 

***\*1\**\**、在\**\**Linux\**\**系统下，重启\**\**Tomcat\**\**使用命令操作的！\****



 ***\*方法一：\****

首先，进入Tomcat下的bin目录

```java
cd /usr/local/tomcat/bin
```

使用Tomcat关闭命令

```java
./shutdown.sh
```

查看Tomcat是否以关闭

```java
ps-ef|grep java
```

如果显示以下相似信息，说明Tomcat还没有关闭

```
root   7010
   1 0 Apr19 ?    00:30:13 /usr/local/java/bin/java
-Djava.util.logging.config.file=/usr/local/tomcat/conf/logging.properties
 -Djava.awt.headless=true
-Dfile.encoding=UTF-8-server -Xms1024m -Xmx1024m -XX:NewSize=256m
-XX:MaxNewSize=256m-XX:PermSize=256m -XX:MaxPermSize=256m
-XX:+DisableExplicitGC
-Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager
-Djava.endorsed.dirs=/usr/local/tomcat/endorsed-classpath
/usr/local/tomcat/bin/bootstrap.jar-Dcatalina.base=/usr/local/tomcat
-Dcatalina.home=/usr/local/tomcat
-Djava.io.tmpdir=/usr/local/tomcat/temp
org.apache.catalina.startup.Bootstrapstart
```

*如果你想直接干掉Tomcat，你可以使用kill命令，直接杀死Tomcat进程

```java
 kill -9 7010
```

然后继续查看Tomcat是否关闭

```java
 ps -ef|grep java
```

如果出现以下信息，则表示Tomcat已经关闭

```
root   7010  1 0 Apr19 ?    00:30:30 [java]
```

最后，启动Tomcat

```java
 ./startup.sh
```



*****\*方法二：**

\****不知道你linux下的[目录结构](https://www.baidu.com/s?wd=目录结构&tn=44039180_cpr&fenlei=mv6quAkxTZn0IZRqIHckPjm4nH00T1d9mWuWnWFhm1bsPWcznA7W0ZwV5Hcvrjm3rH6sPfKWUMw85HfYnjn4nH6sgvPsT6K1TL0qnfK1TL0z5HD0IgF_5y9YIZ0lQzqlpA-bmyt8mh7GuZR8mvqVQL7dugPYpyq8Q1csrHDYPHRkPs)是什么样子的。以下我常用的操作步骤，希望能给你启发

```
— cd /tomcat7/logs/   
— tail -f catalina.out   （catalina.out 是控制台日志文件）
```

***\*Tomcat\**\**启动关闭常见命令：\****


Linux下tomcat服务的启动、关闭与错误跟踪，使用PuTTy远程连接到服务器以后，通常通过以下几种方式启动关闭tomcat服务：
切换到tomcat主目录下的bin目录（cd usr/local/tomcat/bin）

1，启动tomcat服务
方式一：直接启动 ./startup.sh
方式二：作为服务启动 nohup ./startup.sh &
方式三：控制台动态输出方式启动 ./catalina.sh run 动态地显示tomcat后台的控制台输出信息,Ctrl+C后退出并关闭服务
解释：
通过方式一、方式三启动的tomcat有个弊端，当客户端连接断开的时候，tomcat服务也会立即停止，通过方式二可以作为linux服务一直运行
通过方式一、方式二方式启动的tomcat，其日志会写到相应的日志文件中，而不能动态地查看tomcat控制台的输出信息与错误情况，通过方式三可以以控制台模式启动tomcat服务，
直接看到程序运行时后台的控制台输出信息，不必每次都要很麻烦的打开catalina.out日志文件进行查看，这样便于跟踪查阅后台输出信息。tomcat控制台信息包括log4j和System.out.println()等输出的信息。

2，关闭tomcat服务
./shutdown.sh

***\*2\**\**、启动\****：一般是执行sh tomcat/bin/startup.sh 停止：一般是执行sh tomcat/bin/shutdown.sh脚本命令 查看：执行ps -ef|grep tomcat 输出如下 *** 5144 。。。等等.Bootstrap start 说明tomcat已经正常启动， 5144 就为进程号 pid = 5144



```java
杀死：kill -9 5144
```

***\*------------------------linux\**\**下实时查看\**\**tomcat\**\**运行日志\**\**-------------------------\****

1、先切换到：

```
cd tomcat/logs
```

2、查看实时日志：

```
tail -f catalina.out
```

3、这样运行时就可以实时查看运行日志了

```
Ctrl+c 是退出tail命令。
```

======================

**linux相关基础知识：**（需要找个时间，系统的练一下，然后做一波笔记。）

查看某个机器ip是否通:   ping ip/域名  

​		如：ping 127.0.0.1

查看机器下的某个端口号是否能够访问:  telnet ip/域名 : 端口号    

​		如：telnet 127.0.0.1 1521

查看某个端口号是否被占用：

​	netstat -anp|grep 8080



=====================恒大运维

项目常用linux命令：

1）项目部署：

​	mvn package -P sit （成功后会在项目的target目录下生成一个war包）： 将项目打个war包，之后部署在服务器的tomcat/webapps下

​	ps -ef|grep tomcat ： 查看包含tomcat关键字的路径及相关信息；

​	war就放到Tomcat应用发布的目录下 然后重启就行了，可以先把之前的war包做个备份

​			如：mv hscs.war hscs-20200927.war

​					将打包好的war包传输进去；

​	**重启tomcat：**

​				cd /usr/local/tomcat/bin，

​		bin目录下使用命令：

​				./shutdown.sh；

​		查看tomcat是否关闭：

​				ps -ef|grep java；

​		启动tomcat：

​				./startup.sh 



生产发版 : mvn package -P prod  job服务器tomcat；mvn package -P prod_web     web服务器；



**重启redis服务器：**

​		（默认启动服务：./redis-server   redis.conf），但是当这两个文件没有放在同一个目录下时，需要指定其具体目录，如下：

​		./bin/redis-server ./etc/redis.conf（在redis目录下执行）

**遇见tomcat关闭不了的情况：**

​	./shutdown.sh：关闭报错，连接失败；（这个问题是因为关闭tomcat的时候太急了，tomcat还没有启动完成就关闭，会导致这个错误）；

​	首先我们需要关闭tomcat,因为8005端口没有启动，也关闭不了；即为8005端口未运行，使用命令netstat -ant 发现 没有找到8005端口

​	最直接的办法是杀死进程：

​		ps -ef|grep tomcat    查看端口号；

![image-20201026190348128](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20201026190348128.png)

	kill -9 7588
	kill -9 8983
	kill -9 10335
	kill -9 10423    //进程被杀死，tomcat成功被关闭，但是这样做并没有实际作用；



==================windows下关闭端口（先找进程号，杀进程）

​	1.查看端口是否被占用：netstat -aon|findstr "8080" 

 	2. -F 强制关闭： taskkill -F -PID 8080;  //关闭8080端口 





//  基础要练，新特性也要练；(尽量多敲.慢慢就熟悉了,尽量不要复制,多理解)