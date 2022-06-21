**maven学习**

仓库分三类：本地仓库，远程仓库【私服】，中央仓库；

​		先扫描本地仓库，本地仓库有的话，就不去远程仓库下；没有去远程仓库，公司远程仓库【私服】（内网即可）再没有

这时候才去中央仓库下载(需要外网)；



mvn compile: 把src/main下面的代码进行编译，并放在target目录下；



**mvn clean install:**

​	mvn clean :  删除项目编译好的target包；

​	mvn install: 帮我们编译并生成了target包，并且在target目录下打了一个war包，并把这个打的war包安装到了我们的本地仓库；

- mvn package 和mvn install(mvn package:完成了项目编译、单元测试、打包功能，但是没有把打好的可执行jar包部署到本地maven仓库和远程私服仓库;mvn install比mvn package多了一步部署到本地maven仓库的动作;mvn deploy又比mvn install 多了一步将打好的包部署到远程私服仓库的动作.



================================   maven常见问题  ===================================

1.多模块下的maven打包出错问题.

​	最近项目上使用的是idea ide的多模块话，需要模块之间的依赖，比如说系统管理模块依赖授权模块进行认证和授权，而认证授权模块需要依赖系统管理模块进行，然后，我就开始相互依赖，然后出现这样的问题：

“Could not resolve dependencies for project”，让我百思不得其解，最后网络搜了搜，最后给的解决方案是：

```java
我也碰到这个问题，需要把parent工程，也就是package是pom的那个工程先install一下，因为模块与模块之间可能存在一些循环依赖相互依赖的关系.
```

2.maven的依赖有时可能有下载失败或者不全的情况.

​		maven的依赖有时可能有下载失败或者不全的情况，这个时候可以找到仓库对应的位置删掉该包，重新下载依赖.



3、到idea-》Other Setting中设置maven配置才会对其他项目都生效，Setting中配置maven只对当前有效。



4、多模块下的maven互相引用

​	多模块时maven也可相互引用，注意是否打成jar包可以到对应的目录下应该看得到。package有jar、war、pom三种。多模块时，被引用的子模块第一次可能需要先mvn install一下到本地仓库，以供父模块引用得到。





