*Spring Aop**

**1.Spring Aop五种通知详解**

**1)前置通知**Before advice：在连接点前面执行，前置通知不会影响连接点的执行，除非此处抛出异常。

**2)正常返回通知**After returning advice：在连接点正常执行完成后执行，如果连接点抛出异常，则不会执行。

**3)异常返回通知**After throwing advice：在连接点抛出异常后执行。

**4)后置通知**After (finally) advice：在连接点执行完成后执行，不管是正常执行完成，还是抛出异常，都会执行返回通知中的内容。

**5)环绕通知**Around advice：环绕通知围绕在连接点前后，比如一个方法调用的前后。这是最强大的通知类型，能在方法调用前后自定义一些操作。环绕通知还需要负责决定是继续处理join point(调用ProceedingJoinPoint的proceed方法)还是中断执行。



**2.五种通知的执行顺序**

```java
// 在方法执行正常时：
1. 环绕通知`@Around` 
2. 前置通知`@Before` 
3. 方法执行
4. 环绕通知`@Around` 
5. 后置通知`@After` 
6. 正常返回通知`@AfterReturning` 

// 在方法执行抛出异常时：
这个时候顺序会根据环绕通知的不同而发生变化：
- 环绕通知捕获异常并不抛出： 
  1. 环绕通知`@Around` 
  2. 前置通知`@Before` 
  3. 方法执行
  4. 环绕通知`@Around` 
  5. 后置通知`@After` 
  6. 正常返回通知`@AfterReturning` 
- 环绕通知捕获异常并抛出： 
  1. 环绕通知`@Around` 
  2. 前置通知`@Before` 
  3. 方法执行
  4. 环绕通知`@Around` 
  5. 后置通知`@After` 
  6. 异常返回通知`@AfterThrowing`

```

