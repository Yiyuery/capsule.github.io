---
title: [JDK] Spring 探秘  之  线程池技术（二）
date: 2019-01-17 21:22:00
categories:
- jdk
tags:
- jdk
- thread-pool-executor
- blockqueue
---

  ThreadPoolExecutor 线程池配置 和 阻塞队列BlockingQueue

---

# [JDK] Spring 探秘  之  线程池技术（二）

> ThreadPoolExecutor 线程池配置 和 阻塞队列BlockingQueue

## 创建和配置

> ExecutorService 执行器服务,它使用可能的几个池线程之一执行每个提交的任务，通常使用`Executors`工厂方法配置

> 线程池可以`解决两个不同问题`：由于减少了每个任务调用的开销，它们通常可以在执行大量异步任务时提供增强的性能，并且还可以提供绑定和管理资源（包括执行集合任务时使用的线程）的方法。每个ThreadPoolExecutor 还维护着一些基本的统计数据，如完成的任务数。

> 为了便于`跨大量上下文使用`，此类提供了很多可调整的参数和扩展挂钩。但是，强烈建议程序员使用较为方便的 Executors 工厂方法 Executors.newCachedThreadPool()（无界线程池，可以进行自动线程回收）、Executors.newFixedThreadPool(int)（固定大小线程池）和 Executors.newSingleThreadExecutor()（单个后台线程），它们均为大多数使用场景预定义了设置。否则，在手动配置和调整此类时，使用以下指导：

- 核心和最大池大小

ThreadPoolExecutor 将根据 `corePoolSize`（参见 getCorePoolSize()）和 `maximumPoolSize`（参见getMaximumPoolSize()）设置的边界自动调整池大小。当新任务在方法 execute(java.lang.Runnable) 中提交时，如果运行的线程少于 `corePoolSize`，则创建新线程来处理请求，即使其他辅助线程是空闲的。如果运行的线程多于`corePoolSize` 而少于 maximumPoolSize，则仅当队列满时才创建新线程。如果设置的 `corePoolSize` 和 `maximumPoolSize`相同，则创建了固定大小的线程池。如果将 `maximumPoolSize` 设置为基本的无界值（如 Integer.MAX_VALUE），则允许池适应任意数量的并发任务。在大多数情况下，核心和最大池大小仅基于构造来设置，不过也可以使用setCorePoolSize(int) 和 setMaximumPoolSize(int) 进行动态更改。

- 按需构造

默认情况下，即使核心线程最初只是在新任务需要时才创建和启动的，也可以使用方法 prestartCoreThread()或 prestartAllCoreThreads() 对其进行动态重写。

- 创建新线程

使用 `ThreadFactory` 创建新线程。如果没有另外说明，则在同一个 `ThreadGroup` 中一律使用Executors.defaultThreadFactory() 创建线程，并且这些线程具有相同的 NORM_PRIORITY 优先级和非守护进程状态。通过提供不同的 `ThreadFactory`，可以改变线程的名称、线程组、优先级、守护进程状态，等等。如果从 newThread 返回 null 时 ThreadFactory 未能创建线程，则执行程序将继续运行，但不能执行任何任务。

- 保持活动时间

如果池中当前有多于 `corePoolSize` 的线程，则这些多出的线程在空闲时间超过 `keepAliveTime` 时将会终止（参见getKeepAliveTime(java.util.concurrent.TimeUnit)）。这提供了当池处于非活动状态时减少资源消耗的方法。如果池后来变得更为活动，则可以创建新的线程。也可以使用方法 setKeepAliveTime(long, java.util.concurrent.TimeUnit) 动态地更改此参数。使用 Long.MAX_VALUE TimeUnit.NANOSECONDS 的值在关闭前有效地从以前的终止状态禁用空闲线程。

- 排队
  - 所有 BlockingQueue 都可用于传输和保持提交的任务。可以使用此队列与池大小进行交互：    
      - 如果运行的线程少于 `corePoolSize`，则 Executor 始终首选添加新的线程，而不进行排队。
      - 如果运行的线程等于或多于 `corePoolSize`，则 Executor 始终首选将请求加入队列，而不添加新的线程。
      - 如果无法将请求加入队列，则创建新的线程，除非创建此线程超出 `maximumPoolSize`，在这种情况下，任务将被拒绝。
  - 排队有三种通用策略
      - 直接提交: 工作队列的默认选项是 SynchronousQueue，它将任务直接提交给线程而不保持它们。在此，如果不存在可用于立即运行任务的线程，则试图把任务加入队列将失败，因此会构造一个新的线程。此策略可以避免在处理可能具有内部依赖性的请求集合时出现锁定。直接提交通常要求无界 maximumPoolSizes 以避免拒绝新提交的任务。当命令以超过队列所能处理的平均数连续到达时，此策略允许无界线程具有增长的可能性。
      - 无界队列: 使用无界队列（例如，不具有预定义容量的 LinkedBlockingQueue）将导致在所有 corePoolSize 线程都忙的情况下将新任务加入队列。这样，创建的线程就不会超过 corePoolSize。（因此，maximumPoolSize 的值也就无效了。）当每个任务完全独立于其他任务，即任务执行互不影响时，适合于使用无界队列；例如，在 Web 页服务器中。这种排队可用于处理瞬态突发请求，当命令以超过队列所能处理的平均数连续到达时，此策略允许无界线程具有增长的可能性。
      - 有界队列: 当使用有限的 maximumPoolSizes 时，有界队列（如 ArrayBlockingQueue）有助于防止资源耗尽，但是可能较难调整和控制。队列大小和最大池大小可能需要相互折衷：使用大型队列和小型池可以最大限度地降低CPU 使用率、操作系统资源和上下文切换开销，但是可能导致人工降低吞吐量。如果任务频繁阻塞（例如，如果它们是 I/O 边界），则系统可能为超过您许可的更多线程安排时间。使用小型队列通常要求较大的池大小，CPU 使用率较高，但是可能遇到不可接受的调度开销，这样也会降低吞吐量。
  - 被拒绝的任务
      当 Executor 已经关闭，并且 Executor 将有限边界用于最大线程和工作队列容量，且已经饱和时，在方法execute(java.lang.Runnable) 中提交的新任务将被拒绝。在以上两种情况下，execute 方法都将调用其RejectedExecutionHandler 的 RejectedExecutionHandler.rejectedExecution(java.lang.Runnable, java.util.concurrent.ThreadPoolExecutor) 方法。下面提供了四种预定义的处理程序策略：
      - 在默认的 ThreadPoolExecutor.AbortPolicy 中，处理程序遭到拒绝将抛出运行时 RejectedExecutionException。
      - 在 ThreadPoolExecutor.CallerRunsPolicy 中，线程调用运行该任务的 execute 本身。此策略提供简单的反馈控制机制，能够减缓新任务的提交速度。
      - 在 ThreadPoolExecutor.DiscardPolicy 中，不能执行的任务将被删除。
      - 在 ThreadPoolExecutor.DiscardOldestPolicy 中，如果执行程序尚未关闭，则位于工作队列头部的任务将被删除，然后重试执行程序（如果再次失败，则重复此过程）。
      - 定义和使用其他种类的 RejectedExecutionHandler 类也是可能的，但这样做需要非常小心，尤其是当策略仅用于特定容量或排队策略时。
- 挂钩方法
  - 此类提供 protected 可重写的 beforeExecute(java.lang.Thread, java.lang.Runnable) 和 afterExecute(java.lang.Runnable, java.lang.Throwable) 方法，这两种方法分别在执行每个任务之前和之后调用。它们可用于操纵执行环境；例如，重新初始化ThreadLocal、搜集统计信息或添加日志条目。此外，还可以重写方法 terminated() 来执行 Executor 完全终止后需要完成的所有特殊处理。
  - 如果挂钩或回调方法抛出异常，则内部辅助线程将依次失败并突然终止。
- 队列维护
  - 方法 getQueue() 允许出于监控和调试目的而访问工作队列。强烈反对出于其他任何目的而使用此方法。remove(java.lang.Runnable) 和 purge() 这两种方法可用于在取消大量已排队任务时帮助进行存储回收。

`Ex.3`

`ThreadPoolTask` & `doThreadTest2`

```java
  package com.example.concurrence.thread;

  import java.io.Serializable;

  /**
   * <p>
   *
   * </p>
   *
   * @author xiachaoyang
   * @version V1.0
   * @date 2019年01月17日 14:09
   * @modificationHistory=========================逻辑或功能性重大变更记录
   * @modify By: {修改人} 2019年01月17日
   * @modify reason: {方法名}:{原因}
   * ...
   */
  public class ThreadPoolTask  implements Runnable, Serializable {

      private Object attachData;

      public ThreadPoolTask(Object tasks) {
          this.attachData = tasks;
      }

      @Override
      public void run() {
          System.out.println("开始执行任务：" + attachData);
          attachData = null;
      }

      public Object getTask() {
          return this.attachData;
      }
  }


  //ConcurrenceServiceImpl

  @Override
  public void doThreadTest2() {
     // 构造一个线程池
     ThreadPoolExecutor threadPool = new ThreadPoolExecutor(2, 4, 3,
             TimeUnit.SECONDS, new ArrayBlockingQueue<Runnable>(3),
             new ThreadPoolExecutor.DiscardOldestPolicy());

     int produceTaskSleepTime = 2;
     int produceTaskMaxNumber = 10;
     for (int i = 1; i <= produceTaskMaxNumber; i++) {
         try {
             String task = "task@ " + i;
             System.out.println("创建任务并提交到线程池中：" + task);
             threadPool.execute(new ThreadPoolTask(task));
             Thread.sleep(produceTaskSleepTime);
         } catch (Exception e) {
             e.printStackTrace();
         }
     }
     //创建任务并提交到线程池中：task@ 1
     //开始执行任务：task@ 1
     //创建任务并提交到线程池中：task@ 2
     //开始执行任务：task@ 2
     //创建任务并提交到线程池中：task@ 3
     //开始执行任务：task@ 3
     //创建任务并提交到线程池中：task@ 4
     //...
     //创建任务并提交到线程池中：task@ 9
     //开始执行任务：task@ 9
     //创建任务并提交到线程池中：task@ 10
     //开始执行任务：task@ 10
  }
```

## 线程配置和管理

> ThreadPoolExecutor

`ThreadPoolExcutor`为一些`Executor`提供了基本的实现，这些Executor是由Executors中的工厂 `newCahceThreadPool`、`newFixedThreadPool`和`newScheduledThreadExecutor`返回的。 `ThreadPoolExecutor`是一个灵活的健壮的池实现，允许各种各样的用户定制。

- 线程的创建与销毁

    - 核心池大小、最大池大小和存活时间共同管理着线程的创建与销毁。
    - 核心池的大小是目标的大小；线程池的实现试图维护池的大小；即使没有任务执行，池的大小也等于核心池的大小，并直到工作队列充满前，池都不会创建更多的线程。如果当前池的大小超过了核心池的大小，线程池就会终止它。
    - 最大池的大小是可同时活动的线程数的上限。
    - 如果一个线程已经闲置的时间超过了存活时间，它将成为一个被回收的候选者。
    - `newFixedThreadPool`工厂为请求的池设置了核心池的大小和最大池的大小，而且池永远不会超时。
    - `newCacheThreadPool`工厂将最大池的大小设置为`Integer.MAX_VALUE`，核心池的大小设置为0，超时设置为一分钟。这样创建了无限扩大的线程池，会在需求量减少的情况下减少线程数量

- 管理

  - `ThreadPoolExecutor`允许你提供一个`BlockingQueue`来持有等待执行的任务。任务排队有3种基本方法：无限队列、有限队列和同步移交。
  - `newFixedThreadPool`和`newSingleThreadExectuor`默认使用的是一个无限的`LinkedBlockingQueue`。如果所有的工作者线程都处于忙碌状态，任务会在队列中等候。如果任务持续快速到达，超过了它们被执行的速度，队列也会无限制地增加。稳妥的策略是使用有限队列，比如`ArrayBlockingQueue`或有限的`LinkedBlockingQueue`以及`PriorityBlockingQueue`。
  - 对于庞大或无限的池，可以使用`SynchronousQueue`，完全绕开队列，直接将任务由生产者交给工作者线程。
  - 可以使用`PriorityBlockingQueue`通过优先级安排任务


## 阻塞队列BlockingQueue

- SynchronousQueue `UML`

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190117165151369.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzI4NjkwNDE3,size_16,color_FFFFFF,t_70)

- LinkedBlockingQueue `UML`

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190117165203937.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzI4NjkwNDE3,size_16,color_FFFFFF,t_70)

在`ThreadPoolTaskExecutor`源码中我们看到了`BlockingQueue<Runnable> queue = createQueue(this.queueCapacity);`这样一句话用来得到一个队列，这个队列是用来存放任务的。当线程池中有空闲线程时就回去任务队列中拿任务并处理。BlockingQueue是一个阻塞并线程安全的一个队列

　　多线程环境中，通过队列可以很容易实现数据共享，比如经典的“生产者”和“消费者”模型中，通过队列可以很便利地实现两者之间的数据共享。假设我们有若干 生产者线程，另外又有若干个消费者线程。如果生产者线程需要把准备好的数据共享给消费者线程，利用队列的方式来传递数据，就可以很方便地解决他们之间的数 据共享问题。但如果生产者和消费者在某个时间段内，万一发生数据处理速度不匹配的情况呢？理想情况下，如果生产者产出数据的速度大于消费者消费的速度，并 且当生产出来的数据累积到一定程度的时候，那么生产者必须暂停等待一下（阻塞生产者线程），以便等待消费者线程把累积的数据处理完毕，反之亦然。

> BlockingQueue的核心方法：

- 放入数据：
  - `offer(anObject)`:表示如果可能的话,将anObject加到BlockingQueue里,即如果BlockingQueue可以容纳,则返回true,否则返回false.（本方法不阻塞当前执行方法的线程）
  - `offer(E o, long timeout, TimeUnit unit)`,可以设定等待的时间，如果在指定的时间内，还不能往队列中加入BlockingQueue，则返回失败。
  - `put(anObject)`:把anObject加到BlockingQueue里,如果BlockQueue没有空间,则调用此方法的线程被阻断直到BlockingQueue里面有空间再继续

- 获取数据：
  - `poll(long timeout, TimeUnit unit)`：从BlockingQueue取出一个队首的对象，如果在指定时间内,队列一旦有数据可取，则立即返回队列中的数据。否则知道时间超时还没有数据可取，返回失败。
  - `take()`:取走BlockingQueue里排在首位的对象,若BlockingQueue为空,阻断进入等待状态直到BlockingQueue有新的数据被加入;
  - `drainTo(...)`:一次性从`BlockingQueue`获取所有可用的数据对象(还可以指定获取数据的个数),通过该方法，可以提升获取数据效率；不需要多次分批加锁或释放锁。


  > org.springframework.scheduling.concurrent.ThreadPoolTaskExecutor

  ```java
  /**
   * Create the BlockingQueue to use for the ThreadPoolExecutor.
   * <p>A LinkedBlockingQueue instance will be created for a positive
   * capacity value; a SynchronousQueue else.
   * @param queueCapacity the specified queue capacity
   * @return the BlockingQueue instance
   * @see java.util.concurrent.LinkedBlockingQueue
   * @see java.util.concurrent.SynchronousQueue
   */
  protected BlockingQueue<Runnable> createQueue(int queueCapacity) {
    if (queueCapacity > 0) {
      return new LinkedBlockingQueue<>(queueCapacity);
    }
    else {
      return new SynchronousQueue<>();
    }
  }
  ```

ThreadPoolTaskExecutor的代码可以发现，其主要是使用`BlockingQueue`的一种实现`LinkedBlockingQueue`进行实现。

> LinkedBlockingQueue

`LinkedBlockingQueue`是一个基于链表的阻塞队列，其内部也维持着一个数据缓冲队列（该队列由一个链表构成），当生产者往队列中放入一个数据时，队列会从生产者手中获取数据，并缓存在队列内部，而生产者立 即返回；只有当队列缓冲区达到最大值缓存容量时（`LinkedBlockingQueue`可以通过构造函数指定该值），才会阻塞生产者队列，直到消费者从 队列中消费掉一份数据，生产者线程会被唤醒，反之对于消费者这端的处理也基于同样的原理。而`LinkedBlockingQueue`之所以能够高效的处理 并发数据，还因为其对于生产者端和消费者端分别采用了独立的锁来控制数据同步，这也意味着在高并发的情况下生产者和消费者可以并行地操作队列中的数据，以此来提高整个队列的并发性能。
- 生成`LinkedBlockingQueue`时有一个大小限制，其默认为`Integer.MAX_VALUE`.
- 另外`LinkedBlockingQueue`不接受`null`值，当添加`null`的时候，会直接抛出`NullPointerException`：


```java
//java.util.concurrent.LinkedBlockingQueue

/**
 * Inserts the specified element at the tail of this queue, waiting if
 * necessary for space to become available.
 *
 * @throws InterruptedException {@inheritDoc}
 * @throws NullPointerException {@inheritDoc}
 */
public void put(E e) throws InterruptedException {
    if (e == null) throw new NullPointerException();
    // Note: convention in all put/take/etc is to preset local var
    // holding count negative to indicate failure unless set.
    int c = -1;
    Node<E> node = new Node<E>(e);
    final ReentrantLock putLock = this.putLock;
    final AtomicInteger count = this.count;
    putLock.lockInterruptibly();
    try {
        /*
         * Note that count is used in wait guard even though it is
         * not protected by lock. This works because count can
         * only decrease at this point (all other puts are shut
         * out by lock), and we (or some other waiting put) are
         * signalled if it ever changes from capacity. Similarly
         * for all other uses of count in other wait guards.
         */
        while (count.get() == capacity) {
            notFull.await();
        }
        enqueue(node);
        c = count.getAndIncrement();
        if (c + 1 < capacity)
            notFull.signal();
    } finally {
        putLock.unlock();
    }
    if (c == 0)
        signalNotEmpty();
}
```

> 队列的优点

1. 解耦

在项目启动之初来预测将来项目会碰到什么需求，是极其困难的。消息队列在处理过程中间插入了一个隐含的、基于数据的接口层，两边的处理过程都要实现这一接口。这允许你独立的扩展或修改两边的处理过程，只要确保它们遵守同样的接口约束。

2. 冗余

有时在处理数据的时候处理过程会失败。除非数据被持久化，否则将永远丢失。消息队列把数据进行持久化直到它们已经被完全处理，通过这一方式规避了数据丢失风险。在被许多消息队列所采用的"插入-获取-删除"范式中，在把一个消息从队列中删除之前，需要你的处理过程明确的指出该消息已经被处理完毕，确保你的数据被安全的保存直到你使用完毕。

3. 扩展性

因为消息队列解耦了你的处理过程，所以增大消息入队和处理的频率是很容易的；只要另外增加处理过程即可。不需要改变代码、不需要调节参数。扩展就像调大电力按钮一样简单。

4. 灵活性 & 峰值处理能力

在访问量剧增的情况下，你的应用仍然需要继续发挥作用，但是这样的突发流量并不常见；如果为 以能处理这类峰值访问为标准来投入资源随时待命无疑是巨大的浪费。使用消息队列能够使关键组件顶住增长的访问压力，而不是因为超出负荷的请求而完全崩溃。

5. 可恢复性

当体系的一部分组件失效，不会影响到整个系统。消息队列降低了进程间的耦合度，所以即使一个处理消息的进程挂掉，加入队列中的消息仍然可以在系统恢复后被处理。而这种允许重试或者延后处理请求的能力通常是造就一个略感不便的用户和一个沮丧透顶的用户之间的区别。

6. 送达保证

消息队列提供的冗余机制保证了消息能被实际的处理，只要一个进程读取了该队列即可。在此基础上，IronMQ提供了一个"只送达一次"保证。无论有多少进 程在从队列中领取数据，每一个消息只能被处理一次。这之所以成为可能，是因为获取一个消息只是"预定"了这个消息，暂时把它移出了队列。除非客户端明确的 表示已经处理完了这个消息，否则这个消息会被放回队列中去，在一段可配置的时间之后可再次被处理。

7. 排序保证

在许多情况下，数据处理的顺序都很重要。消息队列本来就是排序的，并且能保证数据会按照特定的顺序来处理。

8. 缓冲

在任何重要的系统中，都会有需要不同的处理时间的元素。例如,加载一张图片比应用过滤器花费更少的时间。消息队列通过一个缓冲层来帮助任务最高效率的执行--写入队列的处理会尽可能的快速，而不受从队列读的预备处理的约束。该缓冲有助于控制和优化数据流经过系统的速度。

9. 理解数据流

在一个分布式系统里，要得到一个关于用户操作会用多长时间及其原因的总体印象，是个巨大的挑战。消息系列通过消息被处理的频率，来方便的辅助确定那些表现不佳的处理过程或领域，这些地方的数据流都不够优化。

10. 异步通信

很多时候，你不想也不需要立即处理消息。消息队列提供了异步处理机制，允许你把一个消息放入队列，但并不立即处理它。你想向队列中放入多少消息就放多少，然后在你乐意的时候再去处理它们。

---

## REFRENCES

1.  [ThreadPoolTaskExecutor使用详解](https://blog.csdn.net/foreverling/article/details/78073105)
2.  [spring线程池ThreadPoolTaskExecutor与阻塞队列BlockingQueue](http://www.cnblogs.com/lic309/p/4186880.html)
3.  [ThreadPoolExecutor使用介绍](https://blog.csdn.net/wangwenhui11/article/details/6760474)
---

## 更多

<center>
<img src="https://images.gitee.com/uploads/images/2018/0717/215030_8e782063_912956.png" width="50%" height="50%"/>
</center>

扫码关注或搜索`架构探险之道`获取最新文章，不积跬步无以至千里，坚持每周一更，坚持技术分享`^_^ `！
