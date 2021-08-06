===========================java8流的使用（主要针对list的操作）

//参考资料：https://www.cnblogs.com/Fndroid/p/6087380.html

理论知识：

**Streams**

Streams的思想很简单，就是遍历。

一个流的生命周期分为三个阶段：

1. **生成**
2. **操作、变换（可以多次）**
3. **消耗（只有一次）**

**--生成**

生成Stream对象

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```java
1 // 1. 对象
2 Stream stream = Stream.of("a", "b", "c");
3 // 2. 数组
4 String [] strArray = new String[] {"a", "b", "c"};
5 stream = Stream.of(strArray);
6 stream = Arrays.stream(strArray);
7 // 3. 集合
8 List<String> list = Arrays.asList(strArray);
9 stream = list.stream();
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

生成DoubleSteam、IntSteram或LongStream对象（这是目前支持的三个数值类型Stream对象）

```java
1 IntStream.of(new int[]{1, 2, 3}); // 根据数组生成
2 IntStream.range(1, 3); // 按照范围生成，不包括3
3 IntStream.rangeClosed(1, 3); // 按照范围生成，包括3
```

***--变换***

一个流可以经过多次的变换，变换的结果仍然是一个流。

常见的变换：**map** (mapToInt, flatMap 等)、 **filter**、 **distinct**、 **sorted**、 peek、 limit、 skip、 parallel、 sequential、 unordered

***--消耗***

一个流对应一个消耗操作。

常见的消耗操作：**forEach**、 forEachOrdered、 **toArray**、 **reduce**、 **collect**、 min、 max、 count、 **anyMatch**、 **allMatch**、 noneMatch、 findFirst、 findAny、 iterator

**例子：**

定义一个学生类：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public class Student {
 2     public enum Sax{
 3         FEMALE, MALE
 4     }
 5 
 6     private String name;
 7     private int age;
 8     private Sax sax;
 9     private int height;
10 
11     public Student(String name, int age, Sax sax, int height) {
12         this.name = name;
13         this.age = age;
14         this.height = height;
15         this.sax = sax;
16     }
17 
18     public String getName() {
19         return name;
20     }
21 
22     public int getAge() {
23         return age;
24     }
25 
26     public Sax getSax() {
27         return sax;
28     }
29 
30     public int getHeight() {
31         return height;
32     }
33 
34     @Override
35     public String toString() {
36         return "Student{" +
37                 "name='" + name + '\'' +
38                 ", age=" + age +
39                 ", sax=" + sax +
40                 ", height=" + height +
41                 '}';
42     }
43 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

在main方法中创建示例数据：

```
1 List<Student> students = Arrays.asList(
2         new Student("Fndroid", 22, Student.Sax.MALE, 180),
3         new Student("Jack", 20, Student.Sax.MALE, 170),
4         new Student("Liliy", 18, Student.Sax.FEMALE, 160)
5 );
```

**常用stream示例：**

1.**过滤**得到性别为MALE的学生

```java
1 students.stream() // 打开流
2         .filter(student -> student.getSax() == Student.Sax.MALE) // 进行过滤
3         .forEach(System.out::println); // 输出
```

2.求出所有学生的**平均**年龄(提取对象中的年龄属性，再进行计算)：

```java
1 OptionalDouble averageAge = students.stream()
2         .mapToInt(Student::getAge) // 将对象映射为整型
3         .average(); // 根据整形数据求平均值
4 System.out.println("所有学生的平均年龄为：" + averageAge.orElse(0));//orElse方法获取其值，如果值为null，则取默认值0
```

3.输出每个学生**姓名的大写形式**（.map提取对象某一属性，且不做其他操作）：

```java
1 List<String> names = students.stream()
2         .map(Student::getName) // 将Student对象映射为String（姓名）
3         .map(String::toUpperCase) // 将姓名转为小写
4         .collect(Collectors.toList()); // 生成列表
5 System.out.println("所有学生姓名的大写为：" + names);
```

4.按照年龄从小到大**排序**：

```java
1 List<Student> sortedStudents = students.stream()
2         .sorted((o1, o2) -> o1.getAge() - o2.getAge()) // 按照年龄排序
3         .collect(Collectors.toList()); // 生成列表
4 System.out.println("按年龄排序后列表为：" + sortedStudents);
```

5.判断**是否存在**名为Fndroid的学生（此处也可以通过过滤得到对象，然后判断对象是否为空判定是否存在）：

```java
boolean isContain = students.stream()
        .anyMatch(student -> student.getName().equals("Fndroid")); // 查询任意匹配项是否存在
System.out.println("是否包含姓名为Fndroid的学生：" + isContain);
```

6.将所有学生**按照性别分组**：

```java
1 Map<Student.Sax, List<Student>> groupBySax = students.stream()
2         .collect(Collectors.groupingBy(Student::getSax)); // 根据性别进行分组
3 System.out.println(groupBySax.get(Student.Sax.FEMALE));
```

7.求出每个学生身高比例：

```java
1 double sumHeight = students.stream().mapToInt(Student::getHeight).sum(); // 求出身高总和
2 DecimalFormat formator = new DecimalFormat("##.00"); // 保留两位小数
3 List<String> percentages = students.stream()
4         .mapToInt(Student::getHeight) // 将Student对象映射为身高整型值
5         .mapToDouble(value -> value / sumHeight * 100) // 求出比例
6         .mapToObj(per -> formator.format(per) + "%") // 组装为字符串
7         .collect(Collectors.toList()); 
8 System.out.println("所有学生身高比例：" + percentages);
```

//8.list与list之间的交叉并集，请自行google.



















