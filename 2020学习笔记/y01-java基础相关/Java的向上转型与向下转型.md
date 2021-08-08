# Java的向上转型与向下转型

java 转型问题其实并不复杂，只要记住一句话：父类引用指向子类对象。

什么叫父类引用指向子类对象，且听我慢慢道来.

从2个名词开始说起：向上转型(upcasting) 、向下转型(downcasting).

举个例子：有2个类，Father是父类，Son类继承自Father。

Father f1 = new Son(); // 这就叫 upcasting （向上转型)

// 现在f1引用指向一个Son对象

Son s1 = (Son)f1; // 这就叫 downcasting (向下转型)

// 现在f1还是指向Son对象

第2个例子：

Father f2 = new Father();

Son s2 = (Son)f2; // 出错，子类引用不能指向父类对象

你或许会问，第1个例子中：Son s1 = (Son)f1;问什么 是正确的呢。

很简单因为f1指向一个子类对象，Father f1 = new Son(); 子类s1引用当然可以指向子类对象了。

而f2 被传给了一个Father对象，Father f2 = new Father（）；子类s1引用不能指向父类对象。

**总结：**

**1。父类引用指向子类对象，而子类引用不能指向父类对象。**

**2。把子类对象直接赋给父类引用叫upcasting向上转型，向上转型不用强制转换。**

**如：Father f1 = new Son();**

**3。把指向子类对象的父类引用赋给子类引用叫向下转型(downcasting)，要强制转换。**

**如：f1 就是一个指向子类对象的父类引用。把f1赋给子类引用s1即 Son s1 = (Son)f1；**

**其中f1前面的(Son)必须加上，进行强制转换。**

一、向上转型。



通俗地讲即是将子类对象转为父类对象。此处父类对象可以是接口。

1，向上转型中的方法调用。

看下面代码：



```java
[java] view plain copy
package com.wensefu.others;  
public class Animal {  

public void eat(){  
       System.out.println("animal eatting...");  
   }  
}  
class Bird extends Animal{  

public void eat(){  
       System.out.println("bird eatting...");  
   }  

public void fly(){  

       System.out.println("bird flying...");  
   }  
}  
class Main{  

public static void main(String[] args) {  

       Animal b=new Bird(); //向上转型 
       b.eat();   
//! error: b.fly(); b虽指向子类对象，但此时丢失fly()方法 
       dosleep(new Male());  
       dosleep(new Female());  
   }  

public static void dosleep(Human h) {  
       h.sleep();  
   }  
}  
```







```java
[java] view plain copy
package com.wensefu.others;  
public class Human {  
public void sleep() {  
       System.out.println("Human sleep..");  
   }  
}  
class Male extends Human {  
@Override 
public void sleep() {  
       System.out.println("Male sleep..");  
   }  
}  
class Female extends Human {  
@Override 
public void sleep() {  
       System.out.println("Female sleep..");  
   }  
}  
```





注意这里的向上转型：
Animal b=new Bird(); //向上转型
b.eat();

此处将调用子类的eat()方法。原因：b实际指向的是Bird子类，故调用时会调用子类本身的方法。

需要注意的是向上转型时b会遗失除与父类对象共有的其他方法。如本例中的fly方法不再为b所有。

2，向上转型的好处。

看上面的代码，

public static void dosleep(Human h) {
h.sleep();
}

这里以父类为参数，调有时用子类作为参数，就是利用了向上转型。这样使代码变得简洁。不然的话，
如果dosleep以子类对象为参数，则有多少个子类就需要写多少个函数。这也体现了JAVA的抽象编程思想。

二、向下转型。

与向上转型相反，即是把父类对象转为子类对象。

看下面代码：



```java
[java] view plain copy
package com.wensefu.other1;  
public class Girl {  
public void smile(){  
       System.out.println("girl smile()...");  
   }  
}  
class MMGirl extends Girl{  

@Override 
public void smile() {  

       System.out.println("MMirl smile sounds sweet...");  
   }  
public void c(){  
       System.out.println("MMirl c()...");  
   }  
}  
class Main{  

public static void main(String[] args) {  

       Girl g1=new MMGirl(); //向上转型 
       g1.smile();  

       MMGirl mmg=(MMGirl)g1; //向下转型,编译和运行皆不会出错 
       mmg.smile();  
       mmg.c();  


       Girl g2=new Girl();  
//      MMGirl mmg1=(MMGirl)g2; //不安全的向下转型,编译无错但会运行会出错 
//      mmg1.smile(); 
//      mmg1.c(); 
/*output: 
* CGirl smile sounds sweet... 
* CGirl smile sounds sweet... 
* CGirl c()... 
* Exception in thread "main" java.lang.ClassCastException: com.wensefu.other1.Girl 
* at com.wensefu.other1.Main.main(Girl.java:36) 
*/ 
if(g2 instanceof MMGirl){  
           MMGirl mmg1=(MMGirl)g2;   
           mmg1.smile();  
           mmg1.c();  
       }  

   }  
}  
```





Girl g1=new MMGirl(); //向上转型
g1.smile();
MMGirl mmg=(MMGirl)g1; //向下转型,编译和运行皆不会出错

这里的向下转型是安全的。因为g1指向的是子类对象。

而
Girl g2=new Girl();
MMGirl mmg1=(MMGirl)g2; //不安全的向下转型,编译无错但会运行会出错

运行出错：

Exception in thread "main" java.lang.ClassCastException: com.wensefu.other1.Girl
at com.wensefu.other1.Main.main(Girl.java:36)
如代码所示，可以通过instanceof来防止出现异常。