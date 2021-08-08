**maven学习**



![image-20200903161511071](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20200903161511071.png)

仓库分三类：本地仓库，远程仓库【私服】，中央仓库；

​		先扫描本地仓库，本地仓库有的话，就不去远程仓库下；没有去远程仓库，公司远程仓库【私服】（内网即可）再没有

这时候才去中央仓库下载(需要外网)；



mvn compile: 把src/main下面的代码进行编译，并放在target目录下；



**mvn clean install:**

​	mvn clean :  删除项目编译好的target包；

​	mvn install: 帮我们编译并生成了target包，并且在target目录下打了一个war包，并把这个打的war包安装到了我们的本地仓库；



**mvn 生命周期**：

![image-20200903164158592](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20200903164158592.png)



任何一个jar包基本组成的最少属性：

​	![image-20200903172315698](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20200903172315698.png)



