---
title: [Spring] Spring 探秘  之  线程池技术（一）
date: 2019-01-18 21:22:00
categories:
- spring-thread
tags:
- spring
- thread-pool-task-executor
- blockqueue
---

  多线程和高并发场景，Spring 线程池技术 之  ThreadPoolTaskExecutor

---


# [Spring] Spring 探秘  之  线程池技术（一）

> 多线程和高并发场景，Spring 线程池技术 之  ThreadPoolTaskExecutor

> [官方API](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/scheduling/concurrent/ThreadPoolTaskExecutor.html)

> Links

[>>>[JDK]Spring 探秘  之  线程池技术（二）](./2019-01-17-jdk-thread-pool-executor.md)

## 使用场景

    - 并发操作
    - 异步操作

## 引入方式

### Spring 配置

> maven引入spring-context支持

```
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
    </dependency>
```

> 线程池对象配置

```
    <bean id="threadPoolTaskExecutor" class="org.springframework.scheduling.concurrent.ThreadPoolTaskExecutor" destroy-method="destroy">
        <!--核心线程数(线程池维护线程的最少数量) -->
        <property name="corePoolSize" value="4"/>
        <!-- 最大线程数(线程池维护线程的最大数量) -->
        <property name="maxPoolSize" value="4"/>
        <!-- 线程最大空闲时间 -->
        <property name="keepAliveSeconds" value="300"/>
        <!-- 队列大小 >= mainExecutor.maxSize (缓存队列) -->
        <property name="queueCapacity" value="200"/>
        <!-- 线程名称前缀 -->
        <property name="threadNamePrefix" value="xxx_ThreadPool-"/>
        <!-- 配置拒绝策略 -->
        <property name="rejectedExecutionHandler">
            <bean class="java.util.concurrent.ThreadPoolExecutor.AbortPolicy"/>
        </property>
    </bean>
```

> 配置拒绝策略

    - ThreadPoolExecutor.AbortPolicy

        用于被拒绝任务的处理程序，它将抛出RejectedExecutionException。

    - ThreadPoolExecutor.CallerRunsPolicy

        用于被拒绝任务的处理程序，它直接在execute方法的调用线程中运行被拒绝的任务。当抛出RejectedExecutionException异常时，会调用rejectedExecution方法

```java
/**
  * A handler for rejected tasks that runs the rejected task
  * directly in the calling thread of the {@code execute} method,
  * unless the executor has been shut down, in which case the task
  * is discarded.
  */
public static class CallerRunsPolicy implements RejectedExecutionHandler {
    /**
     * Creates a {@code CallerRunsPolicy}.
     */
    public CallerRunsPolicy() { }

    /**
     * Executes task r in the caller's thread, unless the executor
     * has been shut down, in which case the task is discarded.
     *
     * @param r the runnable task requested to be executed
     * @param e the executor attempting to execute this task
     */
    public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
        if (!e.isShutdown()) {
            r.run();
        }
    }
}

```

    - ThreadPoolExecutor.DiscardOldestPolicy
        用于被拒绝任务的处理程序，它放弃最旧的未处理请求，然后重试execute。

    - ThreadPoolExecutor.DiscardPolicy
        用于被拒绝任务的处理程序，默认情况下它将丢弃被拒绝的任务。  

    - 自定义拒绝策略

```java
package com.example.config;

import lombok.extern.slf4j.Slf4j;

import java.util.concurrent.RejectedExecutionException;
import java.util.concurrent.RejectedExecutionHandler;
import java.util.concurrent.ThreadPoolExecutor;

/**
 * <p>
 *  自定义拒绝策略
 * </p>
 *
 * @author xiachaoyang
 * @version V1.0
 * @date 2019年01月17日 11:40
 * @modificationHistory=========================逻辑或功能性重大变更记录
 * @modify By: {修改人} 2019年01月17日
 * @modify reason: {方法名}:{原因}
 * ...
 */
@Slf4j
public class CapAbortPolicy implements RejectedExecutionHandler {
    /**
     * Method that may be invoked by a {@link ThreadPoolExecutor} when
     * {@link ThreadPoolExecutor#execute execute} cannot accept a
     * task.  This may occur when no more threads or queue slots are
     * available because their bounds would be exceeded, or upon
     * shutdown of the Executor.
     *
     * <p>In the absence of other alternatives, the method may throw
     * an unchecked {@link RejectedExecutionException}, which will be
     * propagated to the caller of {@code execute}.
     *
     * @param r        the runnable task requested to be executed
     * @param executor the executor attempting to execute this task
     * @throws RejectedExecutionException if there is no remedy
     */
    @Override
    public void rejectedExecution(Runnable r, ThreadPoolExecutor executor) {
        log.error("{}rejectedExecution Task has exceeded max size, abort it:",r);
    }
}
```

> 线程池对象注入

```java
  @Resource
  private ThreadPoolTaskExecutor threadPoolTaskExecutor;
```

> 利用线程池管理线程(线程资源空闲时自动执行run方法)

```java
  threadPoolTaskExecutor.execute(new Thread("threadName"){

    @Override
    public void run(){
        //todo
    }

  }
```

`Ex.1`

```java

/**
 * 多线程并发处理demo
 * @author xiazhaoyang
 *
 */
public class MultiThreadDemo implements Runnable {

    private MultiThreadProcessService multiThreadProcessService;

    public MultiThreadDemo() {
    }

    public MultiThreadDemo(MultiThreadProcessService multiThreadProcessService) {
        this.multiThreadProcessService = multiThreadProcessService;
    }

    @Override
    public void run() {
        multiThreadProcessService.processSomething();
    }

}

/**
 * 测试用例
 */
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = { MultiThreadConfig.class })
public class MultiThreadTest {

    @Autowired
    private ThreadPoolTaskExecutor threadPoolTaskExecutor;

    @Autowired
    private MultiThreadProcessService multiThreadProcessService;

    @Test
    public void test() {

        int n = 20;
        for (int i = 0; i < n; i++) {
            threadPoolTaskExecutor.execute(new MultiThreadDemo(multiThreadProcessService));
            System.out.println("int i is " + i + ", now threadpool active threads totalnum is " + taskExecutor.getActiveCount());
        }

        try {
            System.in.read();
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }
}
```

## 源码分析

> org.springframework.scheduling.concurrent.ThreadPoolTaskExecutor 初始化过程

- org.springframework.scheduling.concurrent.ThreadPoolTaskExecutor.initializeExecutor

```java

/**
 * Note: This method exposes an {@link ExecutorService} to its base class
 * but stores the actual {@link ThreadPoolExecutor} handle internally.
 * Do not override this method for replacing the executor, rather just for
 * decorating its {@code ExecutorService} handle or storing custom state.
 */
@Override
protected ExecutorService initializeExecutor(
		ThreadFactory threadFactory, RejectedExecutionHandler rejectedExecutionHandler) {

	BlockingQueue<Runnable> queue = createQueue(this.queueCapacity);

	ThreadPoolExecutor executor;
	if (this.taskDecorator != null) {
		executor = new ThreadPoolExecutor(
				this.corePoolSize, this.maxPoolSize, this.keepAliveSeconds, TimeUnit.SECONDS,
				queue, threadFactory, rejectedExecutionHandler) {
			@Override
			public void execute(Runnable command) {
				Runnable decorated = taskDecorator.decorate(command);
				if (decorated != command) {
					decoratedTaskMap.put(decorated, command);
				}
				super.execute(decorated);
			}
		};
	}
	else {
		executor = new ThreadPoolExecutor(
				this.corePoolSize, this.maxPoolSize, this.keepAliveSeconds, TimeUnit.SECONDS,
				queue, threadFactory, rejectedExecutionHandler);

	}

	if (this.allowCoreThreadTimeOut) {
		executor.allowCoreThreadTimeOut(true);
	}

	this.threadPoolExecutor = executor;
	return executor;
}
```

- java.util.concurrent.ThreadPoolExecutor构造函数（之一）

```java

/**
  * Creates a new {@code ThreadPoolExecutor} with the given initial
  * parameters.
  *
  * @param corePoolSize the number of threads to keep in the pool, even
  *        if they are idle, unless {@code allowCoreThreadTimeOut} is set
  * @param maximumPoolSize the maximum number of threads to allow in the
  *        pool
  * @param keepAliveTime when the number of threads is greater than
  *        the core, this is the maximum time that excess idle threads
  *        will wait for new tasks before terminating.
  * @param unit the time unit for the {@code keepAliveTime} argument
  * @param workQueue the queue to use for holding tasks before they are
  *        executed.  This queue will hold only the {@code Runnable}
  *        tasks submitted by the {@code execute} method.
  * @param threadFactory the factory to use when the executor
  *        creates a new thread
  * @param handler the handler to use when execution is blocked
  *        because the thread bounds and queue capacities are reached
  * @throws IllegalArgumentException if one of the following holds:<br>
  *         {@code corePoolSize < 0}<br>
  *         {@code keepAliveTime < 0}<br>
  *         {@code maximumPoolSize <= 0}<br>
  *         {@code maximumPoolSize < corePoolSize}
  * @throws NullPointerException if {@code workQueue}
  *         or {@code threadFactory} or {@code handler} is null
  */
 public ThreadPoolExecutor(int corePoolSize,//线程池维护线程的最少数量
                           //线程池维护线程的最大数量
                           int maximumPoolSize,
                           //线程池维护线程所允许的空闲时间
                           long keepAliveTime,
                           //线程池维护线程所允许的空闲时间的单位,unit可选的参数为java.util.concurrent.TimeUnit中的几个静态属性：NANOSECONDS、MICROSECONDS、MILLISECONDS、SECONDS。
                           TimeUnit unit,
                           //线程池所使用的缓冲队列,常用的是：java.util.concurrent.ArrayBlockingQueue
                           BlockingQueue<Runnable> workQueue,
                           ThreadFactory threadFactory,
                           // 线程池对拒绝任务的处理策略)
                           RejectedExecutionHandler handler{
     if (corePoolSize < 0 ||
         maximumPoolSize <= 0 ||
         maximumPoolSize < corePoolSize ||
         keepAliveTime < 0)
         throw new IllegalArgumentException();
     if (workQueue == null || threadFactory == null || handler == null)
         throw new NullPointerException();
     this.acc = System.getSecurityManager() == null ?
             null :
             AccessController.getContext();
     this.corePoolSize = corePoolSize;
     this.maximumPoolSize = maximumPoolSize;
     this.workQueue = workQueue;
     this.keepAliveTime = unit.toNanos(keepAliveTime);
     this.threadFactory = threadFactory;
     this.handler = handler;
}

```

> 分析

- java.util.concurrent.ThreadPoolExecutor的参数

    int corePoolSize: 线程池维护线程的最小数量.

    int maximumPoolSize: 线程池维护线程的最大数量.

    long keepAliveTime: 空闲线程的存活时间.

    TimeUnit unit: 时间单位,现有纳秒,微秒,毫秒,秒枚举值.

    BlockingQueue<Runnable> workQueue: 持有等待执行的任务队列.

    RejectedExecutionHandler handler: 用来拒绝一个任务的执行，有两种情况会发生这种情况。
        一是在execute方法中若addIfUnderMaximumPoolSize(command)为false，即线程池已经饱和；
        二是在execute方法中, 发现runState!=RUNNING || poolSize == 0,即已经shutdown,就调用ensureQueuedTaskHandled(Runnable command)，在该方法中有可能调用reject。    

- `ThreadPoolTaskExecutor` 有两个execute的重载，但翻看代码可以知道调用的是同一个方法，所以只调用execute就可以了

- 既然`ThreadPoolTaskExecutor`是直接使用`ThreadPoolExecutor`进行处理，所以运算规则肯定一样。`ThreadPoolTaskExecutor`依赖于`ThreadPoolExecutor`的处理流程：

    1）当池子大小小于corePoolSize就新建线程，并处理请求    

    2）当池子大小等于corePoolSize，把请求放入workQueue中，池子里的空闲线程就去从workQueue中取任务并处理

    3）当workQueue放不下新入的任务时，新建线程入池，并处理请求，如果池子大小撑到了maximumPoolSize就用RejectedExecutionHandler来做拒绝处理

    4）另外，当池子的线程数大于corePoolSize的时候，多余的线程会等待keepAliveTime长的时间，如果无请求可处理就自行销毁

    其会优先创建 `corePoolSize` 线程， 当继续增加线程时，先放入`Queue`中，当 `corePoolSize`  和 `Queue` 都满的时候，就增加创建新线程，当线程达到`maxPoolSize`的时候，就会抛出错误 `org.springframework.core.task.TaskRejectedException`

    另外maxPoolSize的设定如果比系统支持的线程数还要大时，会抛出`java.lang.OutOfMemoryError: unable to create new native thread`异常。

**_在ThreadPoolExecutor中表现为_**  

      如果当前运行的线程数小于corePoolSize，那么就创建线程来执行任务（执行时需要获取全局锁）。

      如果运行的线程大于或等于corePoolSize，那么就把task加入BlockQueue。

      如果创建的线程数量大于BlockQueue的最大容量，那么创建新线程来执行该任务。

      如果创建线程导致当前运行的线程数超过maximumPoolSize，就根据饱和策略来拒绝该任务。

## 线程管理`ThreadPoolTaskExecutor`

> 提交任务

    无返回值的任务使用execute(Runnable)

```java
@Override
public void execute(Runnable task) {
  Executor executor = getThreadPoolExecutor();
  try {
    executor.execute(task);
  }
  catch (RejectedExecutionException ex) {
    throw new TaskRejectedException("Executor [" + executor + "] did not accept task: " + task, ex);
  }
}

@Override
public void execute(Runnable task, long startTimeout) {
  execute(task);
}

```

    有返回值的任务使用submit(Runnable)

```java
@Override
public Future<?> submit(Runnable task) {
	ExecutorService executor = getThreadPoolExecutor();
	try {
		return executor.submit(task);
	}
	catch (RejectedExecutionException ex) {
		throw new TaskRejectedException("Executor [" + executor + "] did not accept task: " + task, ex);
	}
}

@Override
public <T> Future<T> submit(Callable<T> task) {
	ExecutorService executor = getThreadPoolExecutor();
	try {
		return executor.submit(task);
	}
	catch (RejectedExecutionException ex) {
		throw new TaskRejectedException("Executor [" + executor + "] did not accept task: " + task, ex);
	}
}

```

> 处理流程

      当一个任务被提交到线程池时，首先查看线程池的核心线程是否都在执行任务，否就选择一条线程执行任务，是就执行第二步。

      查看核心线程池是否已满，不满就创建一条线程执行任务，否则执行第三步。

      查看任务队列是否已满，不满就将任务存储在任务队列中，否则执行第四步。

      查看线程池是否已满，不满就创建一条线程执行任务，否则就按照策略处理无法执行的任务。

> 关闭线程池

调用shutdown或者shutdownNow，两者都不会接受新的任务，而且通过调用要停止线程的interrupt方法来中断线程，有可能线程永远不会被中断，不同之处在于：

    shutdownNow会首先将线程池的状态设置为STOP，然后尝试停止所有线程（有可能导致部分任务没有执行完）然后返回未执行任务的列表。
    shutdown只是将线程池的状态设置为shutdown，然后中断所有没有执行任务的线程，并将剩余的任务执行完。    

> 线程个数配置

-   如果是CPU密集型任务，那么线程池的线程个数应该尽量少一些，一般为CPU的个数+1条线程。
-   如果是IO密集型任务，那么线程池的线程可以放的很大，如2*CPU的个数。
-   对于混合型任务，如果可以拆分的话，通过拆分成CPU密集型和IO密集型两种来提高执行效率；如果不能拆分的的话就可以根据实际情况来调整线程池中线程的个数。

> 线程池状态检控

    taskCount：线程需要执行的任务个数。

    completedTaskCount：线程池在运行过程中已完成的任务数。

    largestPoolSize：线程池曾经创建过的最大线程数量。

    getPoolSize获取当前线程池的线程数量。

    getActiveCount：获取活动的线程的数量

通过继承线程池，重写`beforeExecute`，`afterExecute`和`terminated`方法来在线程执行任务前，线程执行任务结束，和线程终结前获取线程的运行情况，根据具体情况调整线程池的线程数量。

`Ex.2`

```java
@Resource
private ThreadPoolTaskExecutor threadPoolTaskExecutor;
//...
threadPoolTaskExecutor.getThreadPoolExecutor().getLargestPoolSize();
//...
```

## 注意事项

> 以下文配置为例，进行总结

```xml
<bean id="threadPoolTaskExecutor" class="org.springframework.scheduling.concurrent.ThreadPoolTaskExecutor" destroy-method="destroy">
     <!--线程池核心线程数-->
     <property name="corePoolSize" value="8"/>    
     <!--最大线程数-->
     <property name="maxPoolSize" value="16"/>    
     <!--线程池达空闲时间后销毁-->
     <property name="keepAliveSeconds" value="300"/>    
     <!--线程池等待队列-->
     <property name="queueCapacity" value="200" />    
     <!-- 核心线程数也准守 keepAliveSeconds，否则corePoolSize = maxPoolSize时，实际线程池永远保持corePoolSize线程 -->
     <property name="allowCoreThreadTimeOut" value="true"/>
     <!--设置创建的线程为守护线程-->
     <property name="daemon" value="true"/>
     <!--线程前缀-->
     <property name="threadNamePrefix" value="capsule-concurrence-pool-"/>
     <!--设置拒绝策略-->
     <property name="rejectedExecutionHandler">
         <bean class="com.example.config.CapAbortPolicy"/>
     </property>
</bean>
```

1. `corePoolSize`,` maxPoolSize`, `queueCapacity`配置说明

        a. 当我提交10个任务时，线程池只会有8个线程在处理，2个在等待队列。
        b. 只有当第209个任务提交时才会创建第9个线程，即当`maxPoolSize`>`corePoolSize`时，只有当`queueCapacity`队列满了才会创建新的线程；
        c. 当217个任务提交时，会触发`rejectedExecutionHandler`回调；
2. 当`allowCoreThreadTimeOut`未配置时，线程池一开始创建就有8个核心线程，永远不会销毁。建议配置该参数，否则如果安装了软件程序，软件程序有没有使用，那么该软件程序会占用已配置的核心线程。
3. 当daemon参数未配置时，线程池创建的线程都`不是`守护线程。

`Ex.3`

```java
public class DaemonThread extends Thread {

    public DaemonThread(Runnable target, String name) {
        super(null, target, name, 0);
        super.setDaemon(true);
    }

    public DaemonThread(String name) {
        super(null, null, name, 0);
        super.setDaemon(true);
    }
}
//Ex.3.1
threadPoolTaskExecutor.execute(new DaemonThread(() -> doFunc(params), "thread-daemon"));

//Ex.3.2
Thread thread = new Thread(() -> doFunc(params), "thread-daemon");
thread.setDaemon(true);
threadPoolTaskExecutor.execute(thread);
```

从代码上我提交给线程池的是一个守护线程，但是实际去执行的是线程池中的线程，不是你创建的守护线程（执行的线程名字也不是叫`thread-daemon`，而是线程池配置的名称）。
其实与`threadPoolTaskExecutor.execute(() -> doFunc(params));` 这个代码没有任何差别。
如果要让线程池中的线程都是`守护线程`就要`设置daemon`。

```xml
<!--设置创建的线程为守护线程-->
<property name="daemon" value="true"/>
```

---

## REFRENCES

1.  [ThreadPoolTaskExecutor使用详解](https://blog.csdn.net/foreverling/article/details/78073105)
2.  [spring线程池ThreadPoolTaskExecutor与阻塞队列BlockingQueue](http://www.cnblogs.com/lic309/p/4186880.html)
3.  [ThreadPoolExecutor使用介绍](https://blog.csdn.net/wangwenhui11/article/details/6760474)

## 更多

<center>
<img src="https://images.gitee.com/uploads/images/2018/0717/215030_8e782063_912956.png" width="50%" height="50%"/>
</center>

扫码关注或搜索`架构探险之道`获取最新文章，坚持每周一更，坚持技术分享的我和你们一起成长 `^_^ `！
