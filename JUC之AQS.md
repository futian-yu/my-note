**JUC之AQS**

**1.AQS(AbstractQueuedSynchronizer)是什么？**

​		AQS是抽象的队列同步器，基石类的框架。它是用来构建锁或者其他同步器组件的重量级基础框架及整个JUC体系的基石。通过内置的FIFO队列来完成资源获取线程的排队工作，并通过一个int类型变量表示持有锁的状态。AQS实现了对同步状态的管理，以及对阻塞线程进行排队，等待通知等一下底层的实现处理。

**2.AQS底层**

​		AQS = state + CLH队列(头、尾、前、后四个节点)

​		在AQS有一个静态内部类Node，其中有这样一些属性：

```java
volatile int waitStatus //节点状态 
volatile Node prev //当前节点/线程的前驱节点 
volatile Node next; //当前节点/线程的后继节点 
volatile Thread thread;//加入同步队列的线程引用 Node nextWaiter;//等待队列中的下一个节点
```

​		节点的状态有以下这些：

```java
int CANCELLED =  1//节点从同步队列中取消 
int SIGNAL    = -1//后继节点的线程处于等待状态，如果当前节点释放同步状态会通知后继节点，使得后继节点的线程能够运行； 
int CONDITION = -2//当前节点进入等待队列中 
int PROPAGATE = -3//表示下一次共享式同步状态获取将会无条件传播下去 
int INITIAL = 0;//初始状态
```

