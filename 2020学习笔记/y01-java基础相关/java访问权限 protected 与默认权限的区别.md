# java protected 与默认权限的区别

![img](https://csdnimg.cn/release/phoenix/template/new_img/reprint.png)

[BOY](https://me.csdn.net/jiang_bing) 2012-07-20 11:28:56 ![img](https://csdnimg.cn/release/phoenix/template/new_img/articleReadEyes.png) 7123 ![img](https://csdnimg.cn/release/phoenix/template/new_img/tobarCollect.png) 收藏 1

分类专栏： [Java基础](https://blog.csdn.net/jiang_bing/category_910347.html) 文章标签： [java](https://www.csdn.net/gather_24/NtTaIg5sMzYyLWJsb2cO0O0O.html)



| 作用域            | 当前类 | 同package | 子孙类 | 其他package |
| ----------------- | ------ | --------- | ------ | ----------- |
| public            | √      | √         | √      | √           |
| protected         | √      | √         | √      | ×           |
| friendly(default) | √      | √         | ×      | ×           |
| private           | √      | ×         | ×      | ×           |

\1. 子类可以通过继承获得不同包父类的protected权限成员变量和成员方法，在子类中可以直接访问

\2. 在子类中可以通过子类的对象访问父类的protected成员变量和方法

\3. 在子类中反而不能通过父类的对象访问父类的protected成员变量和方法



friendly 就是默认访问权限（成员变量前面不加public protected 和 private）
重点看protected和fiendly两种权限的区别：对于protected成员变量，子孙类在任何地方都能访问（包内或者包外），但是对于friendly或者说默认成员变量，其实是不存在子孙类访问权限的概念的，就是说如果子孙类在包内，则可以访问，子孙类在包外则不可以访问。



protected在其子类中可以访问，无论是子类内部还是子类的实例，无论它们是在哪个包中，

但如果子类与父类不在同一个包中，在子类中用父类的实例去访问的话不可以





==========注意===========

​		类中的私有方法在其他类直接是访问不到的，dto中的私有变量是通过public方法而获取到的。

一般写成私有方法的话，只是打算自己类内调用。其他地方如果要调用类的私有方法，只能通过反射。