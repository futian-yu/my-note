# Java 反射由浅入深 | 进阶必备

本博文主要记录我学习 Java 反射（reflect）的一点心得，在了解反射之前，你应该先了解 Java 中的 Class 类，如果你不是很了解，可以先简单了解下。

# 一、Java 反射机制

参考了许多博文，总结了以下个人观点，若有不妥还望指正：

> Java 反射机制在程序**运行时**，对于任意一个类，都能够知道这个类的所有属性和方法；对于任意一个对象，都能够调用它的任意一个方法和属性。这种 **动态的获取信息** 以及 **动态调用对象的方法** 的功能称为 **java 的反射机制**。

> 反射机制很重要的一点就是“运行时”，其使得我们可以在程序运行时加载、探索以及使用编译期间完全未知的 `.class` 文件。换句话说，Java 程序可以加载一个运行时才得知名称的 `.class` 文件，然后获悉其完整构造，并生成其对象实体、或对其 fields（变量）设值、或调用其 methods（方法）。

不知道上面的理论你能否明白，反正刚接触反射时我一脸懵比，后来写了几个例子之后：哦~~原来是这个意思！

若暂时不明白理论没关系，先往下看例子，之后再回来看相信你就能明白了。

# 二、使用反射获取类的信息

为使得测试结果更加明显，我首先定义了一个 `FatherClass` 类（默认继承自 `Object` 类），然后定义一个继承自 `FatherClass` 类的 `SonClass` 类，如下所示。可以看到测试类中变量以及方法的访问权限不是很规范，是为了更明显得查看测试结果而故意设置的，实际项目中不提倡这么写。

**FatherClass.java**

```
public class FatherClass {
    public String mFatherName;
    public int mFatherAge;
    
    public void printFatherMsg(){}
}
复制代码
```

**SonClass.java**

```
public class SonClass extends FatherClass{

    private String mSonName;
    protected int mSonAge;
    public String mSonBirthday;

    public void printSonMsg(){
        System.out.println("Son Msg - name : "
                + mSonName + "; age : " + mSonAge);
    }

    private void setSonName(String name){
        mSonName = name;
    }

    private void setSonAge(int age){
        mSonAge = age;
    }

    private int getSonAge(){
        return mSonAge;
    }

    private String getSonName(){
        return mSonName;
    }
}
复制代码
```

## 1. 获取类的所有变量信息

```
/**
 * 通过反射获取类的所有变量
 */
private static void printFields(){
    //1.获取并输出类的名称
    Class mClass = SonClass.class;
    System.out.println("类的名称：" + mClass.getName());
    
    //2.1 获取所有 public 访问权限的变量
    // 包括本类声明的和从父类继承的
    Field[] fields = mClass.getFields();
    
    //2.2 获取所有本类声明的变量（不问访问权限）
    //Field[] fields = mClass.getDeclaredFields();
    
    //3. 遍历变量并输出变量信息
    for (Field field :
            fields) {
        //获取访问权限并输出
        int modifiers = field.getModifiers();
        System.out.print(Modifier.toString(modifiers) + " ");
        //输出变量的类型及变量名
        System.out.println(field.getType().getName()
		         + " " + field.getName());
    }
}
复制代码
```

以上代码注释很详细，就不再解释了。需要注意的是注释中 2.1 的 `getFields()` 与 2.2的 `getDeclaredFields()` 之间的区别，下面分别看一下两种情况下的输出。看之前强调一下： `SonClass` extends `FatherClass` extends `Object` ：

- 调用 `getFields()` 方法，输出 `SonClass` 类以及其所继承的父类( 包括 `FatherClass` 和 `Object` ) 的 `public` 方法。注：`Object` 类中没有成员变量，所以没有输出。

  ```
  类的名称：obj.SonClass
  public java.lang.String mSonBirthday
  public java.lang.String mFatherName
  public int mFatherAge
  复制代码
  ```

- 调用 `getDeclaredFields()` ， 输出 `SonClass` 类的所有成员变量，不问访问权限。

  ```
  类的名称：obj.SonClass
  private java.lang.String mSonName
  protected int mSonAge
  public java.lang.String mSonBirthday
  复制代码
  ```

## 2. 获取类的所有方法信息

```
/**
 * 通过反射获取类的所有方法
 */
private static void printMethods(){
    //1.获取并输出类的名称
    Class mClass = SonClass.class;
    System.out.println("类的名称：" + mClass.getName());
    
    //2.1 获取所有 public 访问权限的方法
    //包括自己声明和从父类继承的
    Method[] mMethods = mClass.getMethods();
    
    //2.2 获取所有本类的的方法（不问访问权限）
    //Method[] mMethods = mClass.getDeclaredMethods();
    
    //3.遍历所有方法
    for (Method method :
            mMethods) {
        //获取并输出方法的访问权限（Modifiers：修饰符）
        int modifiers = method.getModifiers();
        System.out.print(Modifier.toString(modifiers) + " ");
        //获取并输出方法的返回值类型
        Class returnType = method.getReturnType();
        System.out.print(returnType.getName() + " "
                + method.getName() + "( ");
        //获取并输出方法的所有参数
        Parameter[] parameters = method.getParameters();
        for (Parameter parameter:
             parameters) {
            System.out.print(parameter.getType().getName()
		            + " " + parameter.getName() + ",");
        }
        //获取并输出方法抛出的异常
        Class[] exceptionTypes = method.getExceptionTypes();
        if (exceptionTypes.length == 0){
            System.out.println(" )");
        }
        else {
            for (Class c : exceptionTypes) {
                System.out.println(" ) throws "
                        + c.getName());
            }
        }
    }
}
复制代码
```

同获取变量信息一样，需要注意注释中 2.1 与 2.2 的区别，下面看一下打印输出：

- 调用 `getMethods()` 方法 获取 `SonClass` 类所有 `public` 访问权限的方法，包括从父类继承的。打印信息中，`printSonMsg()` 方法来自 `SonClass` 类， `printFatherMsg()` 来自 `FatherClass` 类，其余方法来自 Object 类。

  ```
  类的名称：obj.SonClass
  public void printSonMsg(  )
  public void printFatherMsg(  )
  public final void wait(  ) throws java.lang.InterruptedException
  public final void wait( long arg0,int arg1, ) throws java.lang.InterruptedException
  public final native void wait( long arg0, ) throws java.lang.InterruptedException
  public boolean equals( java.lang.Object arg0, )
  public java.lang.String toString(  )
  public native int hashCode(  )
  public final native java.lang.Class getClass(  )
  public final native void notify(  )
  public final native void notifyAll(  )
  复制代码
  ```

- 调用 `getDeclaredMethods()` 方法

  打印信息中，输出的都是 `SonClass` 类的方法，不问访问权限。

  ```
  类的名称：obj.SonClass
  private int getSonAge(  )
  private void setSonAge( int arg0, )
  public void printSonMsg(  )
  private void setSonName( java.lang.String arg0, )
  private java.lang.String getSonName(  )
  复制代码
  ```

# 三、访问或操作类的私有变量和方法

在上面，我们成功获取了类的变量和方法信息，验证了在运行时 **动态的获取信息** 的观点。那么，仅仅是获取信息吗？我们接着往后看。

都知道，对象是无法访问或操作类的私有变量和方法的，但是，通过反射，我们就可以做到。没错，反射可以做到！下面，让我们一起探讨如何利用反射访问 **类对象的私有方法** 以及修改 **私有变量或常量**。

老规矩，先上测试类。

注：

1. 请注意看测试类中变量和方法的修饰符（访问权限）；
2. 测试类仅供测试，不提倡实际开发时这么写 : )

**TestClass.java**

```
public class TestClass {

    private String MSG = "Original";

    private void privateMethod(String head , int tail){
        System.out.print(head + tail);
    }

    public String getMsg(){
        return MSG;
    }
}
复制代码
```

## 3.1 访问私有方法（*）

以访问 `TestClass` 类中的私有方法 `privateMethod(...)` 为例，方法加参数是为了考虑最全的情况，很贴心有木有？先贴代码，看注释，最后我会重点解释部分代码。

```
/**
 * 访问对象的私有方法
 * 为简洁代码，在方法上抛出总的异常，实际开发别这样
 */
private static void getPrivateMethod() throws Exception{
    //1. 获取 Class 类实例
    TestClass testClass = new TestClass();
    Class mClass = testClass.getClass();
    
    //2. 获取私有方法
    //第一个参数为要获取的私有方法的名称
    //第二个为要获取方法的参数的类型，参数为 Class...，没有参数就是null
    //方法参数也可这么写 ：new Class[]{String.class , int.class}
    Method privateMethod =
            mClass.getDeclaredMethod("privateMethod", String.class, int.class);
            
    //3. 开始操作方法
    if (privateMethod != null) {
        //获取私有方法的访问权
        //只是获取访问权，并不是修改实际权限
        privateMethod.setAccessible(true);
        
        //使用 invoke 反射调用私有方法
        //privateMethod 是获取到的私有方法
        //testClass 要操作的对象
        //后面两个参数传实参
        privateMethod.invoke(testClass, "Java Reflect ", 666);
    }
}
复制代码
```

需要注意的是，第3步中的 `setAccessible(true)` 方法，是获取私有方法的访问权限，如果不加会报异常 **IllegalAccessException**，因为当前方法访问权限是“private”的，如下：

```
java.lang.IllegalAccessException: Class MainClass can not access a member of class obj.TestClass with modifiers "private"
复制代码
```

正常运行后，打印如下，**调用私有方法**成功：

```
Java Reflect 666
复制代码
```

## 3.2 修改私有变量

以修改 `TestClass` 类中的私有变量 `MSG` 为例，其初始值为 "Original" ，我们要修改为 "Modified"。老规矩，先上代码看注释。

```
/**
 * 修改对象私有变量的值
 * 为简洁代码，在方法上抛出总的异常
 */
private static void modifyPrivateFiled() throws Exception {
    //1. 获取 Class 类实例
    TestClass testClass = new TestClass();
    Class mClass = testClass.getClass();
    
    //2. 获取私有变量
    Field privateField = mClass.getDeclaredField("MSG");
    
    //3. 操作私有变量
    if (privateField != null) {
        //获取私有变量的访问权
        privateField.setAccessible(true);
        
        //修改私有变量，并输出以测试
        System.out.println("Before Modify：MSG = " + testClass.getMsg());
        
        //调用 set(object , value) 修改变量的值
        //privateField 是获取到的私有变量
        //testClass 要操作的对象
        //"Modified" 为要修改成的值
        privateField.set(testClass, "Modified");
        System.out.println("After Modify：MSG = " + testClass.getMsg());
    }
}
复制代码
```

此处代码和访问私有方法的逻辑差不多，就不再赘述，从输出信息看出 **修改私有变量** 成功：

```
Before Modify：MSG = Original
After Modify：MSG = Modified
复制代码
```

## 3.3 修改私有常量

在 3.2 中，我们介绍了如何修改私有 **变量**，现在来说说如何修改私有 **常量**，

### 01. 真的能修改吗？

常量是指使用 `final` 修饰符修饰的成员属性，与变量的区别就在于有无 `final` 关键字修饰。在说之前，先补充一个知识点。

Java 虚拟机（JVM）在编译 `.java` 文件得到 `.class` 文件时，会优化我们的代码以提升效率。其中一个优化就是：JVM 在编译阶段会把引用常量的代码替换成具体的常量值，如下所示（部分代码）。

编译前的 `.java` 文件：

```
//注意是 String  类型的值
private final String FINAL_VALUE = "hello";

if(FINAL_VALUE.equals("world")){
    //do something
}
复制代码
```

编译后得到的 `.class` 文件（当然，编译后是没有注释的）：

```
private final String FINAL_VALUE = "hello";
//替换为"hello"
if("hello".equals("world")){
    //do something
}
复制代码
```

但是，并不是所有常量都会优化。经测试对于 `int` 、`long` 、`boolean` 以及 `String` 这些基本类型 JVM 会优化，而对于 `Integer` 、`Long` 、`Boolean` 这种包装类型，或者其他诸如 `Date` 、`Object` 类型则不会被优化。

总结来说：**对于基本类型的静态常量，JVM 在编译阶段会把引用此常量的代码替换成具体的常量值**。

这么说来，在实际开发中，如果我们想修改某个类的常量值，恰好那个常量是基本类型的，岂不是无能为力了？反正我个人认为除非修改源码，否则真没办法！

这里所谓的无能为力是指：**我们在程序运行时刻依然可以使用反射修改常量的值（后面会代码验证），但是 JVM 在编译阶段得到的 .class 文件已经将常量优化为具体的值，在运行阶段就直接使用具体的值了，所以即使修改了常量的值也已经毫无意义了**。

下面我们验证这一点，在测试类 `TestClass` 类中添加如下代码：

```
//String 会被 JVM 优化
private final String FINAL_VALUE = "FINAL";

public String getFinalValue(){
    //剧透，会被优化为: return "FINAL" ,拭目以待吧
    return FINAL_VALUE;
}
复制代码
```

接下来，是修改常量的值，先上代码，请仔细看注释：

```
/**
 * 修改对象私有常量的值
 * 为简洁代码，在方法上抛出总的异常，实际开发别这样
 */
private static void modifyFinalFiled() throws Exception {
    //1. 获取 Class 类实例
    TestClass testClass = new TestClass();
    Class mClass = testClass.getClass();
    
    //2. 获取私有常量
    Field finalField = mClass.getDeclaredField("FINAL_VALUE");
    
    //3. 修改常量的值
    if (finalField != null) {
    
        //获取私有常量的访问权
        finalField.setAccessible(true);
        
        //调用 finalField 的 getter 方法
        //输出 FINAL_VALUE 修改前的值
        System.out.println("Before Modify：FINAL_VALUE = "
                + finalField.get(testClass));
        
        //修改私有常量
        finalField.set(testClass, "Modified");
        
        //调用 finalField 的 getter 方法
        //输出 FINAL_VALUE 修改后的值
        System.out.println("After Modify：FINAL_VALUE = "
                + finalField.get(testClass));
        
        //使用对象调用类的 getter 方法
        //获取值并输出
        System.out.println("Actually ：FINAL_VALUE = "
                + testClass.getFinalValue());
    }
}
复制代码
```

上面的代码不解释了，注释巨详细有木有！特别注意一下第3步的注释，然后来看看输出，已经迫不及待了，擦亮双眼：

```
Before Modify：FINAL_VALUE = FINAL
After Modify：FINAL_VALUE = Modified
Actually ：FINAL_VALUE = FINAL
复制代码
```

结果出来了:

第一句打印修改前 `FINAL_VALUE` 的值，没有异议；

第二句打印修改后常量的值，说明`FINAL_VALUE`确实通过反射修改了；

第三句打印通过 `getFinalValue()` 方法获取的 `FINAL_VALUE` 的值，但还是初始值，导致修改无效！

这结果你觉得可信吗？什么，你还不信？问我怎么知道 JVM 编译后会优化代码？那要不这样吧，一起来看看 `TestClass.java` 文件编译后得到的 `TestClass.class` 文件。为避免说代码是我自己手写的，我决定不粘贴代码，直接截图：



![TestClass.class 文件](https://user-gold-cdn.xitu.io/2017/8/12/2bdcae6d3cde4638c23a0cfe34f85aa6?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



看到了吧，有图有真相，`getFinalValue()` 方法直接 `return "FINAL"`！同时也说明了，**程序运行时是根据编译后的 .class 来执行的**。

顺便提一下，如果你有时间，可以换几个数据类型试试，正如上面说的，有些数据类型是不会优化的。你可以修改数据类型后，根据我的思路试试，看输出觉得不靠谱就直接看 `.classs` 文件，一眼就能看出来哪些数据类型优化了 ，哪些没有优化。下面说下一个知识点。

### 02. 想办法也要修改！

不能修改，这你能忍？别着急，不知你发现没，刚才的常量都是在声明时就直接赋值了。你可能会疑惑，常量不都是在声明时赋值吗？不赋值不报错？当然不是啦。

**方法一**

事实上，Java 允许我们声明常量时不赋值，但必须在构造函数中赋值。你可能会问我为什么要说这个，这就解释：

我们修改一下 `TestClass` 类，在声明常量时不赋值，然后添加构造函数并为其赋值，大概看一下修改后的代码（部分代码 ）：

```
public class TestClass {

    //......
    private final String FINAL_VALUE;
    
    //构造函数内为常量赋值 
    public TestClass(){
        this.FINAL_VALUE = "FINAL";
    }
    //......
}
复制代码
```

现在，我们再调用上面贴出的修改常量的方法，发现输出是这样的：

```
Before Modify：FINAL_VALUE = FINAL
After Modify：FINAL_VALUE = Modified
Actually ：FINAL_VALUE = Modified
复制代码
```

纳尼，最后一句输出修改后的值了？对，修改成功了！想知道为啥，还得看编译后的 `TestClass.class` 文件的贴图，图中有标注。



![TestClass.class 文件](https://user-gold-cdn.xitu.io/2017/8/12/1bc016abdefa8f0822dde437fdd76323?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



解释一下：我们将赋值放在构造函数中，构造函数是我们运行时 new 对象才会调用的，所以就不会像之前直接为常量赋值那样，在编译阶段将 `getFinalValue()` 方法优化为返回常量值，而是指向 `FINAL_VALUE` ，这样我们在运行阶段通过反射修改敞亮的值就有意义啦。但是，看得出来，程序还是有优化的，将构造函数中的赋值语句优化了。再想想那句 **程序运行时是根据编译后的 .class 来执行的** ，相信你一定明白为什么这么输出了！

**方法二**

请你务必将上面捋清楚了再往下看。接下来再说一种改法，不使用构造函数，也可以成功修改常量的值，但原理上都一样。去掉构造函数，将声明常量的语句改为使用三目表达式赋值：

```
private final String FINAL_VALUE
        = null == null ? "FINAL" : null;
复制代码
```

其实，上述代码等价于直接为 `FINAL_VALUE` 赋值 "FINAL"，但是他就是可以！至于为什么，你这么想：`null == null ? "FINAL" : null` 是在运行时刻计算的，在编译时刻不会计算，也就不会被优化，所以你懂得。

总结来说，不管使用构造函数还是三目表达式，根本上都是**避免在编译时刻被优化**，这样我们通过反射修改常量之后才有意义！好了，这一小部分到此结束！

**最后的强调**：

必须提醒你的是，无论**直接为常量赋值** 、 **通过构造函数为常量赋值** 还是 **使用三目运算符**，实际上我们都能通过反射成功修改常量的值。而我在上面说的修改"成功"与否是指：**我们在程序运行阶段通过反射肯定能修改常量值，但是实际执行优化后的 .class 文件时，修改的后值真的起到作用了吗？换句话说，就是编译时是否将常量替换为具体的值了？如果替换了，再怎么修改常量的值都不会影响最终的结果了，不是吗？**。

其实，你可以直接这么想：**反射肯定能修改常量的值，但修改后的值是否有意义**？

### 03. 到底能不能改？

到底能不能改？也就是说反射修改后到底有没有意义？

如果你上面看明白了，答案就简单了。俗话说“一千句话不如一张图”，下面允许我用不太规范的流程图直接表达答案哈。

注：图中"没法修改"可以理解为"能修改值但没有意义"；"可以修改"是指"能修改值且有意义"。



![判断能不能改](https://user-gold-cdn.xitu.io/2017/8/12/7a376b8628ac1343bba0715b211a531b?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)