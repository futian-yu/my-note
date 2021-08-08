# **idea将单个文件打包为可执行jar文件**

​	　idea作为一个java开发的便利IDE工具，个人是比较喜欢的，今天来探索个小功能：  导出单个类文件为jar包！

　　偶有这种需求，就是某个类文件独立存在，但是需要将其导出为jar，供别人临时使用，或者一些必要的场合，如： 编写一些特殊的agent使用。

　　不想为某个单个文件写一个项目，就想把代码加载在某个项目的角落里，怎样将该单个类文件导出为jar包呢？

[返回顶部](https://www.cnblogs.com/yougewe/p/9651156.html#_labelTop)

### **1. 写好功能工具类，如：**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public class Hello {

    public static void main(String[] args) throws Exception {
        Hello hello = new Hello();
        hello.sayHello("word. bingo!");
    }

    public void sayHello(String word) {
        System.out.println("hello, " + word);
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

[返回顶部](https://www.cnblogs.com/yougewe/p/9651156.html#_labelTop)

### **2. 点击idea中的 File -> Project Structure... -> Artifacts -> 添加+ -> JAR -> Empty**

 ![img](https://img2018.cnblogs.com/blog/830731/201809/830731-20180915155413428-1778022517.png)

填写好jar name， 添加好对应的资源文件。先创建好与包名对应的文件目录结构（目录结构不一致可能导致后续使用jar文件时报class not found exception），图解如下：

![img](https://img2018.cnblogs.com/blog/830731/201809/830731-20180915155855851-1733073162.png)

最后，加载编写出的单个类文件（编译后的 .class 文件，一般在 target 目录下），如下图打开添加file, 找到文件。

![img](https://img2018.cnblogs.com/blog/830731/201809/830731-20180915160106121-1212739839.png)

加载后，文件如下，设置好jar文件的输出目录，点击ok关闭对话框：

![img](https://img2018.cnblogs.com/blog/830731/201809/830731-20180915160409759-1694824528.png)

 

[返回顶部](https://www.cnblogs.com/yougewe/p/9651156.html#_labelTop)

### **3. 编写清单文件 MANIFEST.MF，如有必要，再将打开 Project Structure...**

点击 Create new ManiFest， 选择位置，然后创建一个默认的 MANIFEST.MF。 然后关闭对话框，进入自行编辑。

![img](https://img2018.cnblogs.com/blog/830731/201809/830731-20180915160749365-686744083.png)

一些基础参数可以直接在上面填写：

![img](https://img2018.cnblogs.com/blog/830731/201809/830731-20180915160926410-971786849.png)

MANIFEST.MF格式如下：

```
Manifest-Version: 1.0
Premain-Class: com.youge.api.Hello
```

 

[返回顶部](https://www.cnblogs.com/yougewe/p/9651156.html#_labelTop)

### **4. 导出jar文件，先运行 build（将java文件编译到class中，从而例jar文件可更新）, 再导出：**

![img](https://img2018.cnblogs.com/blog/830731/201809/830731-20180915161317127-1920538544.png)

导出，点击build后完成导出：

![img](https://img2018.cnblogs.com/blog/830731/201809/830731-20180915161443219-1318152373.png)

 

如此，到之前设置的目录下，就可以找到导出的jar文件了。

**测试运行jar文件：**

```
java -jar hello.jar
```