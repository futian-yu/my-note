#### **List集合中的一些方法；**

​		list.clear()将list集合中的所有对象都释放了，而且集合也都空了，所以我们没必要多次创建list 集合而只需要调用一下 clear() 方法就可以了。

**1.List去重方式及效率对比**

对List去重并保证添加顺序主要有三种方式：

方式一，利用HashSet不能添加重复数据的特性 由于HashSet不能保证添加顺序，所以只能作为判断条件：

```java
private static void removeDuplicate(List<String> list) {
    HashSet<String> set = new HashSet<String>(list.size());
    List<String> result = new ArrayList<String>(list.size());
    for (String str : list) {
        if (set.add(str)) {
            result.add(str);
        }
    }
    list.clear();
    list.addAll(result);
}
```

方式二，利用LinkedHashSet不能添加重复数据并能保证添加顺序的特性 ：

```java
private static void removeDuplicate(List<String> list) {
    LinkedHashSet<String> set = new LinkedHashSet<String>(list.size());
    set.addAll(list);
    list.clear();
    list.addAll(set);
}
```

方式三，利用List的contains方法循环遍历：

```java
private static void removeDuplicate(List<String> list) {
    List<String> result = new ArrayList<String>(list.size());
    for (String str : list) {
        if (!result.contains(str)) {
            result.add(str);
        }
    }
    list.clear();
    list.addAll(result);
}
```

准备测试程序：

private static void main(String[] args) {
    final List<String> list = new ArrayList<String>();
    for (int i = 0; i < 1000; i++) {
        list.add("haha-" + i);
    }

    long time = System.currentTimeMillis();
    for (int i = 0; i < 10000; i++) {
        removeDuplicate(list);
    }
    long time1 = System.currentTimeMillis();
    System.out.println("time1:"+(time1-time));
    
    for (int i = 0; i < 10000; i++) {
        removeDuplicate2(list);
    }
    long time2 = System.currentTimeMillis();
    System.out.println("time2:"+(time2-time1));
    
    for (int i = 0; i < 10000; i++) {
        removeDuplicate3(list);
    }
    long time3 = System.currentTimeMillis();
    System.out.println("time3:"+(time3-time2));

}

结果为：

time1:329
time2:292
time3:17315
————————————————
原文链接：https://blog.csdn.net/u012156163/article/details/78338574



**2.List的并集 交集 差集 去重复并集**

```java
public static void main(String[] args) {
    List<String> list1 = new ArrayList<String>();
    list1.add("A");
    list1.add("B");
    list1.add("C");

List<String> list2 = new ArrayList<String>();
list2.add("C");
list2.add("B");
list2.add("D");
// 并集
list1.addAll(list2);
// 去重复并集
list2.removeAll(list1);
list1.addAll(list2);
// 交集
list1.retainAll(list2);
// 差集
list1.removeAll(list2);
}
```


**Java8中的 lambda表达式处理集合：**

```java
package com.ymdd.galaxy.appmanage.core.appauth.service;

import java.util.ArrayList;
import java.util.List;
import static java.util.stream.Collectors.toList;

public class Test {

public static void main(String[] args) {
    List<String> list1 = new ArrayList<String>();
    list1.add("1");
	list1.add("2");
	list1.add("3");
	list1.add("5");
	list1.add("6");

​    List<String> list2 = new ArrayList<String>();
​    list2.add("2");
​	list2.add("3");
​	list2.add("7");
​	list2.add("8");

​    // 交集
​    List<String> intersection = list1.stream().filter(item -> list2.contains(item)).collect(toList());
​    System.out.println("---交集 intersection---");
​    intersection.parallelStream().forEach(System.out :: println);

​    // 差集 (list1 - list2)
​    List<String> reduce1 = list1.stream().filter(item -> !list2.contains(item)).collect(toList());
​    System.out.println("---差集 reduce1 (list1 - list2)---");
​    reduce1.parallelStream().forEach(System.out :: println);

​    // 差集 (list2 - list1)
​    List<String> reduce2 = list2.stream().filter(item -> !list1.contains(item)).collect(toList());
​    System.out.println("---差集 reduce2 (list2 - list1)---");
​    reduce2.parallelStream().forEach(System.out :: println);

​    // 并集
​    List<String> listAll = list1.parallelStream().collect(toList());
​    List<String> listAll2 = list2.parallelStream().collect(toList());
​    listAll.addAll(listAll2);
​    System.out.println("---并集 listAll---");
​    listAll.parallelStream().forEachOrdered(System.out :: println);

​    // 去重并集
​    List<String> listAllDistinct = listAll.stream().distinct().collect(toList());
​    System.out.println("---得到去重并集 listAllDistinct---");
​    listAllDistinct.parallelStream().forEachOrdered(System.out :: println);

​    System.out.println("---原来的List1---");
​    list1.parallelStream().forEachOrdered(System.out :: println);
​    System.out.println("---原来的List2---");
​    list2.parallelStream().forEachOrdered(System.out :: println);

}

}
//集合取交差并集，操作的都是同一类型的list


//提取出list对象中的一个属性
List<String> stIdList1 = stuList.stream()
				.map(Student::getId)
				.collect(Collectors.toList());
/* stuList是Stident对象的实例，list对象的id属性，返回一个list；....map(Student::getId).distinct()...表示提取并去重；*/
```


原文链接：https://blog.csdn.net/weixin_38256991/article/details/81672235

顺便复习一下知识点：
**==和!=比较的是对象的引用**
**要想比较对象的内容，需要使用equals方法。**
**基本类型的值比较，直接使用的是==和!=。**

java8使用lamada表达式的一些操作：

（1）可以根据list对象里面的某个属性来分组

（2）List转Map

（3）过滤Filter，从集合中过滤出来符合条件的元素：

（4）求和；将集合中的数据按照某个属性求和；

（5）查找流中的最大 最小值

（6）去重，可以根据某个字段去重

