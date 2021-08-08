**invoke方法：调用包装在当前Method对象中的方法.	**

例：

```java
 例：
//动态构造InvokeTest类的实例
Class<?> classType = InvokeTest.class;
Object invokeTest = classType.newInstance();
//动态构造InvokeTest类的add(int num1, int num2)方法，标记为addMethod的Method对象
Method addMethod = classType.getMethod("add", new Class[]{int.class, int.class});
//动态构造的Method对象invoke委托动态构造的InvokeTest对象，执行对应形参的add方法
Object result = addMethod.invoke(invokeTest, new Object[]{1, 2});
//测试输出
System.out.println((Integer)result);
————————————————
原文链接：https://blog.csdn.net/qq_25439417/article/details/84258677
```
