# Java函数式编程

在很长的一段时间里，Java一直是面向对象的语言，一切皆对象，如果想要调用一个函数，函数必须属于一个类或对象，然后在使用类或对象进行调用。但是在其它的编程语言中，如js，c++，我们可以直接写一个函数，然后在需要的时候进行调用，即可以说是面向对象编程，也可以说是函数式编程。

从功能上来看，面向对象编程没什么不好的地方，但是从开发的角度来看，面向对象编程会多写很多可能是重复的代码行。比如创建一个Runnable的匿名类的时候：

```
        Runnable runnable = new Runnable() {
            @Override
            public void run() {
                System.out.println("do something...");
            }
        };
复制代码
```

这一段代码中真正有用的只有run方法中的内容，剩余的部分都是属于Java编程语言的结构部分，没什么用，但是要写。

幸运的是Java8开始，引入了函数式编程接口与Lambda表达式，帮助我们`写更少更优雅的代码`

```
// 一行即可
Runnable runnable = () -> System.out.println("do something...");
复制代码
```

### 函数式接口-Functional interfaces

一个接口类`只有一个`抽象的方法叫做函数式接口(Functional Interface)，在JDK中大部分函数式接口都会标记上`@FunctionalInterface`注解，并不是所有的函数式接口都要写`@FunctionalInterface`注解，只是用来方便我们区分哪些是函数式接口的，当然如果标记了这个注解，内部有多个抽象方法的时候，会报编译错误。

```
Error:(3, 1) java: 意外的 @FunctionalInterface 注释
 xxx 不是函数接口
 在 接口 xxx 中找到多个非覆盖抽象方法
复制代码
```

当一个类是函数式接口的时候，我们可以直接使用Lambda表达式来实例化它，而不用写很多模板式代码。

写法：

```
// 类名称  变量 = (参数) -> (函数体)
// 如
FunctionTest test = () -> { };
复制代码
```

#### 内置函数式接口

JDK中已经内置了一些标准的函数式接口，位于`java.util.function`包下，满足我们大多数情况下的需求。包下的接口都比较通用，如果我们想要写新的函数式接口，可以首先看这个包下是不是已经提供了。

那么这个包提供了那些接口呢？

- 最常见的四种，Function，Consumer，Supplier，Predicate。标准输入输出，优雅代码所推荐的写法

  - Function：即一个入参一个出参的场景。
  - Consumer：一个入参，但是没有出参
  - Supplier：无入参，一个出参
  - Predicate：可以看做是特殊的Function，一个入参，出参为bool类型。

- 两个入参的函数式接口，即两个入参，一个出参。

  ```
  BiFunction<T, U, R>
  ```

  , 最典型的使用案例即Java的Map方法。

  - BiFunction： 两个入参，一个泛型出参
  - ToDoubleBiFunction,ToIntBiFunction,ToLongBiFunction。两个入参，返回原始类型的出参

```
        HashMap<String, Object> map = new HashMap<>();
        map.put("a",1);
        map.put("b",2);
        map.replaceAll((o, o2) -> o + o2.toString());
        System.out.println(map.values())
复制代码
```

- 一元函数式接口， 可以看做是特殊的Function接口，入参与出参有相同的类型。UnaryOperator，相同类型的转换。
- 原始类型Function，因为Java的原始类型或者叫做主类型，int，short，double之类的，不能作为泛型参数。官方对这些原始类型提供了特殊的函数式接口
  - 以int类型：IntFunction, IntComsumer,IntSupplieer, IntPredicate, IntToDoubleFcuntion, IntToLongFunction, IntUnaryOperator, ToIntFunction

官方：

[docs.oracle.com/javase/8/do…](https://docs.oracle.com/javase/8/docs/api/java/util/function/package-summary.html)

### Lambda表达式

如果理解的了函数式接口，在一定程度上也是理解了Lambda表达式，Lambda表达式相当于对函数式接口的使用，我们不能只去写接口，但不去使用。

尽管Java支持了函数式接口，但是Java的函数仍然要寄托与类之中，不能单独存在，但是因为函数式接口只有一个抽象的函数，那么我们很明确的知道要使用拿一个方法。所以编译器可以自动帮我们推导要用哪个方法，不用在写多余的模板式代码：

```
Runnable runnable = () -> System.out.println("do something...");
复制代码
```

- 首先Runnable是一个函数式的接口，所以我们可以使用Lambda表达式来实例化它
- 因为run方法没有参数，所以lambda表达式：**(argument) -> {body}**，参数为空`()`
- 关于函数体部分，如果我们只有一行代码可以省略掉{}。

除了函数调用的Lambda表达式，还有一些方法的引用操作，使用`::`，引用方法或者构造器而没有真正的实例化对象，可以探索下

```
String::toUpperCase
System.out::println
"abc"::length
ArrayList::new
int[]::new
```