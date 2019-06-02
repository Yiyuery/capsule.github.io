---
title: Java 多线程 之 线程安全
date: 2018-09-10 16:22:01
categories:
- jdk
tags:
- thread
- lock
---

  Java 多线程 之 线程安全

---

# Java 多线程 之 线程安全

> 多线程并发操作时数据共享如何安全进行？

## 线程安全与共享

> 多线程操作静态变量(非线程安全)

`SynchronizedLockTest:`

```Java
/**
 * <p>
 *  测试类
 * </p>
 *
 * @author xiachaoyang
 * @version V1.0
 * @date 2018年07月20日 18:37
 * @modificationHistory=========================逻辑或功能性重大变更记录
 * @modify By: {修改人} 2018年07月20日
 * @modify reason: {方法名}:{原因}
 * ...
 */
public class SynchronizedLockTest {
  /**
   * 多线程操作静态变量
   * 10^3个运算正常，超出会发生count != loop * n 的情况
   * 多线程操作静态变量需进行加锁
   */
  @Test
  public void multipleThreadTest(){
      startAllThreads(regThreads(new CountService(100),1000));
      //阻塞等待线程打印
      while(!Constants.END);
  }

  /**
   * 启动所有线程
   * @param threads
   */
  private void startAllThreads(Set<Thread> threads) {
      Iterator<Thread> iterator = threads.iterator();
      Thread next;
      while (iterator.hasNext()){
          next = iterator.next();
          next.start();
      }
  }

  /**
   * 注册多线程任务集合
   * 集合中都是交给主线程的 线程任务管理器 进行管理的
   * @param countService
   * @param n
   * @return
   */
  private Set<Thread> regThreads(Runnable countService, int n) {
      int i = 0;
      Set<Thread> threads = new HashSet<Thread>(16);
      while(i++ < n){
          threads.add(new Thread(countService));
      }
      return threads;
  }


}
```

`CountInterface:`

```Java
package com.example.chapter1.thread;

/**
 * <p>
 *
 * </p>
 *
 * @author xiachaoyang
 * @version V1.0
 * @date 2018年09月10日 13:52
 * @modificationHistory=========================逻辑或功能性重大变更记录
 * @modify By: {修改人} 2018年09月10日
 * @modify reason: {方法名}:{原因}
 * ...
 */
public interface CountInterface {

    void count();

    void insertThread(Thread thread) throws InterruptedException;
}


```

`CountService:`

```Java
/*
 * @ProjectName: 编程学习
 * @Copyright:   2018 HangZhou Yiyuery Dev, Ltd. All Right Reserved.
 * @address:     https://yiyuery.club
 * @date:        2018/7/28 18:15
 * @email:       xiazhaoyang@live.com
 * @description: 本内容仅限于编程技术学习使用，转发请注明出处.
 */
package com.example.chapter1.thread;

/**
 * <p>
 *
 * </p>
 *
 * @author xiachaoyang
 * @version V1.0
 * @date 2018年07月20日 18:48
 * @modificationHistory=========================逻辑或功能性重大变更记录
 * @modify By: {修改人} 2018年07月20日
 * @modify reason: {方法名}:{原因}
 * ...
 */
public class CountService implements Runnable, CountInterface {

    private int loop;

    public CountService(int loop) {
        this.loop = loop;
    }

    @Override
    public void run() {
        for (int i = 0; i < loop; i++) {
            count();
        }
        System.out.println(String.format("%s >>> count is %d", Thread.currentThread().getName(), Constants.COUNT));
    }

    @Override
    public void count() {
        Constants.COUNT++;
    }

    @Override
    public void insertThread(Thread thread) throws InterruptedException {
        //Todo nothing
    }
}
```

```
Thread-276 >>> count is 99601
Thread-273 >>> count is 99701
Thread-678 >>> count is 99801
Thread-742 >>> count is 99901

```

  多次运行`multipleThreadTest`打印count不一致。


## 利用Lock和synchronized实现线程安全

> 利用synchronized关键字加锁

```Java
/**
 * 同步方法保护静态变量（synchronized关键字加锁）
 */
@Test
public void multipleThreadSyncTest(){
    startAllThreads(regThreads(new CountSyncService(100),1000));
    //阻塞等待线程打印
    while(!Constants.END);
}

```

`Constants:`

```Java
/*
 * @ProjectName: 编程学习
 * @Copyright:   2018 HangZhou Yiyuery Dev, Ltd. All Right Reserved.
 * @address:     https://yiyuery.club
 * @date:        2018/7/28 18:15
 * @email:       xiazhaoyang@live.com
 * @description: 本内容仅限于编程技术学习使用，转发请注明出处.
 */
package com.example.chapter1.thread;

import com.sun.org.apache.bcel.internal.generic.RETURN;

/**
 * <p>
 *
 * </p>
 *
 * @author xiachaoyang
 * @version V1.0
 * @date 2018年07月20日 18:49
 * @modificationHistory=========================逻辑或功能性重大变更记录
 * @modify By: {修改人} 2018年07月20日
 * @modify reason: {方法名}:{原因}
 * ...
 */
public class Constants {

    public static int COUNT;

    public static boolean END = false;
    /**
     * 同步方法避免多线程操作变量
     */
    public synchronized void syncAdd() {
        COUNT++;
    }

    /**
     * 静态内部类式单例
     */
    private static class SingletonHolder {
        private static final Constants instance = new Constants();
    }

    /**
     * 私有化构造器
     */
    private Constants() {}

    /**
     * 单例获取
     * @return
     */
    public static Constants getInstance() {
        return SingletonHolder.instance;
    }
}

```

`CountSyncService:`

```Java

/*
 * @ProjectName: 编程学习
 * @Copyright:   2018 HangZhou Yiyuery Dev, Ltd. All Right Reserved.
 * @address:     https://yiyuery.club
 * @date:        2018/7/28 18:15
 * @email:       xiazhaoyang@live.com
 * @description: 本内容仅限于编程技术学习使用，转发请注明出处.
 */
package com.example.chapter1.thread;

/**
 * <p>
 *
 * </p>
 *
 * @author xiachaoyang
 * @version V1.0
 * @date 2018年09月10日 14:08
 * @modificationHistory=========================逻辑或功能性重大变更记录
 * @modify By: {修改人} 2018年09月10日
 * @modify reason: {方法名}:{原因}
 * ...
 */
public class CountSyncService implements Runnable, CountInterface{

    private int loop;

    public CountSyncService(int loop) {
        this.loop = loop;
    }


    @Override
    public void count() {
        Constants.getInstance().syncAdd();
    }

    @Override
    public void insertThread(Thread thread) throws InterruptedException {
        //
    }

    @Override
    public void run() {
        for (int i = 0; i < loop; i++) {
            count();
        }
        System.out.println(String.format("%s >>> count is %d", Thread.currentThread().getName(), Constants.COUNT));
    }
}

```

  `Constants中的syncAdd方法`添加了synchronized关键字，如此多线程在操作`Constants.COUNT`时就会先获取锁，没有的话就等待，避免多线程同时操作数据。

> Lock实现线程安全(ReentrantLock)

`核心测试类`
```Java
/**
 * 加锁方法保护静态变量
 */
@Test
public void multipleThreadLockTest(){
    //LockMethodEnum > lock()
    startAllThreads(regThreads(new CountLockService(10,LockMethodEnum.LOCK_METHOD_TYPE_LOCK),10));
    //LockMethodEnum > lockInterruptibly()
    //startPartThreads(regCapThreads(new CountLockService(1, LockMethodEnum.LOCK_METHOD_TYPE_LOCK_INTERRUPTIBLY),5));
    //LockMethodEnum > tryLock()
    //startAllThreads(regThreads(new CountLockService(10,LockMethodEnum.LOCK_METHOD_TYPE_TRY_LOCK),10));
    //阻塞等待线程打印
    while(!Constants.END);
}
/**
 * 构造Thread子类调用countLockService的方法测试 中断等待锁的线程
 * @param countLockService
 * @param loop
 * @return
 */
private Set<Thread> regCapThreads(CountLockService countLockService, int loop) {
    CapThread capThread = new CapThread(countLockService);
    return regThreads(capThread,loop);
}
/**
 * 启动后随机中断一部分线程
 * @param threads
 */
private void startPartThreads(Set<Thread> threads) {
    Iterator<Thread> iterator = threads.iterator();
    Thread next;
    while (iterator.hasNext()){
        next = iterator.next();
        next.start();
    }
    try {
        Thread.sleep(1800);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    iterator =  threads.iterator();
    while (iterator.hasNext()){
        next = iterator.next();
        if(random()){
            next.interrupt();
        }
    }
}

/**构造随机数*/
private boolean random() {
    return Double.valueOf(Math.random() * 3).intValue() % 2 == 0;
}
```

`CapThread:`

```Java
/*
 * @ProjectName: 编程学习
 * @Copyright:   2018 HangZhou Yiyuery Dev, Ltd. All Right Reserved.
 * @address:     https://yiyuery.club
 * @date:        2018/7/28 18:15
 * @email:       xiazhaoyang@live.com
 * @description: 本内容仅限于编程技术学习使用，转发请注明出处.
 */
package com.example.chapter1.thread;

/**
 * <p>
 *
 * </p>
 *
 * @author xiachaoyang
 * @version V1.0
 * @date 2018年09月10日 17:17
 * @modificationHistory=========================逻辑或功能性重大变更记录
 * @modify By: {修改人} 2018年09月10日
 * @modify reason: {方法名}:{原因}
 * ...
 */
public class CapThread extends Thread {

    private CountInterface countInterface;

    public CapThread(CountInterface countInterface){
        this.countInterface = countInterface;
    }

    @Override
    public void run() {
        try {
            countInterface.insertThread(Thread.currentThread());
        } catch (InterruptedException e) {
            System.out.println(Thread.currentThread().getName()+"被中断");
        }
    }
}


```

`LockMethodEnum:`定义枚举来区分Lock的不同加锁方式

```Java
package com.example.chapter1.thread;

/**
 * <p>
 *
 * </p>
 *
 * @author xiachaoyang
 * @version V1.0
 * @date 2018年09月10日 15:10
 * @modificationHistory=========================逻辑或功能性重大变更记录
 * @modify By: {修改人} 2018年09月10日
 * @modify reason: {方法名}:{原因}
 * ...
 */
public enum LockMethodEnum {
    /**lock()*/
    LOCK_METHOD_TYPE_LOCK,
    /**tryLock()*/
    LOCK_METHOD_TYPE_TRY_LOCK,
    /**lockInterruptibly()*/
    LOCK_METHOD_TYPE_LOCK_INTERRUPTIBLY,
    /**tryLock(long time, TimeUnit unit)*/
    LOCK_METHOD_TYPE_LOCK_WAIT;
}
```

```Java
/*
 * @ProjectName: 编程学习
 * @Copyright:   2018 HangZhou Yiyuery Dev, Ltd. All Right Reserved.
 * @address:     https://yiyuery.club
 * @date:        2018/7/28 18:15
 * @email:       xiazhaoyang@live.com
 * @description: 本内容仅限于编程技术学习使用，转发请注明出处.
 */
package com.example.chapter1.thread;

import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

/**
 * <p>
 *
 * </p>
 *
 * @author xiachaoyang
 * @version V1.0
 * @date 2018年09月10日 14:08
 * @modificationHistory=========================逻辑或功能性重大变更记录
 * @modify By: {修改人} 2018年09月10日
 * @modify reason: {方法名}:{原因}
 * ...
 */
public class CountLockService implements Runnable, CountInterface {

    private int loop;

    private Lock lock = new ReentrantLock();

    private LockMethodEnum lockMethodTypeLock;

    public CountLockService(int loop, LockMethodEnum lockMethodTypeLock) {
        this.loop = loop;
        this.lockMethodTypeLock = lockMethodTypeLock;
    }


    @Override
    public void count() {
        Constants.COUNT++;
    }

    /**
     * 中断等待锁的线程
     * @param thread
     * @throws InterruptedException
     */
    @Override
    public void insertThread(Thread thread) throws InterruptedException {
        lock.lockInterruptibly();   //注意，如果需要正确中断等待锁的线程，必须将获取锁放在外面，然后将InterruptedException抛出
        try {
            System.out.println(thread.getName() + "得到了锁");
            long startTime = System.currentTimeMillis();
            for (;;) {
                if (System.currentTimeMillis() - startTime >= 2000){
                    handleCountTask();
                    break;
                }
            }
        } finally {
            System.out.println(Thread.currentThread().getName() + "执行finally");
            lock.unlock();
            System.out.println(thread.getName() + "释放了锁");
        }
        //Thread-2得到了锁
        //Thread-1被中断
        //Thread-5被中断
        //Thread-3被中断
        //Thread-2 >>> get lock
        //Thread-2 >>> count is 1
        //Thread-2执行finally
        //Thread-2释放了锁
        //Thread-4得到了锁
        //Thread-4 >>> get lock
        //Thread-4 >>> count is 2
        //Thread-4执行finally
        //Thread-4释放了锁
    }

    @Override
    public void run() {
        switch (lockMethodTypeLock) {
            case LOCK_METHOD_TYPE_LOCK:
                doLock();
                break;
            case LOCK_METHOD_TYPE_TRY_LOCK:
                doTryLock();
                break;
        }
    }


    private void doTryLock() {
        if (lock.tryLock()) {
            try {
                handleCountTask();
            } catch (Exception e) {
                //
            } finally {
                lock.unlock();
                System.out.println(String.format("%s >>> release lock", Thread.currentThread().getName()));
            }
        } else {
            System.out.println(String.format("%s >>> get lock failed!", Thread.currentThread().getName()));
        }
        //...
        //Thread-23 >>> get lock failed!
        //Thread-59 >>> get lock failed!
        //Thread-70 >>> count is 4000
        //Thread-89 >>> get lock failed!
        //Thread-5 >>> get lock failed!
        //Thread-10 >>> get lock failed!
        //...(others failed)
    }

    private void doLock() {
        lock.lock();
        handleCountTask();
        lock.unlock();
        System.out.println(String.format("%s >>> release lock", Thread.currentThread().getName()));
        //Thread-76 >>> get lock
        //Thread-76 >>> release lock
        //Thread-76 >>> count is 10000

    }

    private void handleCountTask() {
        System.out.println(String.format("%s >>> get lock", Thread.currentThread().getName()));
        for (int i = 0; i < loop; i++) {
            count();
        }
        System.out.println(String.format("%s >>> count is %d", Thread.currentThread().getName(), Constants.COUNT));
    }

}
```

> Lock接口中的方法

- `lock()`方法是平常使用得最多的一个方法，就是用来获取锁。如果锁已被其他线程获取，则进行等待。

- `tryLock()`方法是有返回值的，它表示用来尝试获取锁，如果获取成功，则返回true，如果获取失败（即锁已被其他线程获取），则返回false，也就说这个方法无论如何都会立即返回。在拿不到锁时不会一直在那等待

- `tryLock(long time, TimeUnit unit)`方法和tryLock()方法是类似的，只不过区别在于这个方法在拿不到锁时会等待一定的时间，在时间期限之内如果还拿不到锁，就返回false。如果如果一开始拿到锁或者在等待期间内拿到了锁，则返回true。

- `lockInterruptibly()`方法比较特殊，当通过这个方法去获取锁时，如果线程正在等待获取锁，则这个线程能够响应中断，即中断线程的等待状态。也就使说，当两个线程同时通过lock.lockInterruptibly()想获取某个锁时，假若此时线程A获取到了锁，而线程B只有在等待，那么对线程B调用threadB.interrupt()方法能够中断线程B的等待过程。

> ReadWriteLock

- 普通读写锁方式

```Java
/**
 * 一般获取锁的方式
 */
@Test
public void regularTest(){
    startAllThreads(regThreads(()-> {
        get(Thread.currentThread());
    },2));
    //阻塞等待线程打印
    while(!Constants.END);
    //Thread-1正在进行读操作
    //Thread-1读操作完毕
    //Thread-0正在进行读操作
    //Thread-0正在进行读操作
    //Thread-0正在进行读操作
    //Thread-0读操作完毕
}

/**
 * 同步方法获取当前线程获取锁的操作
 * @param thread
 */
public synchronized void get(Thread thread) {
    long start = System.currentTimeMillis();
    while(System.currentTimeMillis() - start <= 1) {
        System.out.println(thread.getName()+"正在进行读操作");
    }
    System.out.println(thread.getName()+"读操作完毕");
}
```

- ReadWriteLock读写锁

```Java

private ReadWriteLock rwl = new ReentrantReadWriteLock();
/**
 * ReadWriteLock接口定义了两个方法，ReentrantReadWriteLock为具体实现类（读锁和写锁）
 */
@Test
public void readWriteLockTest(){
    startAllThreads(regThreads(()-> {
        get2(Thread.currentThread());
    },10));
    //阻塞等待线程打印
    while(!Constants.END);
    //Thread-6正在进行读操作
    //Thread-6正在进行读操作
    //Thread-6正在进行读操作
    //Thread-6正在进行读操作
    //Thread-6正在进行读操作
    //Thread-6读操作完毕
    //Thread-5正在进行读操作
    //Thread-5读操作完毕
    //Thread-8正在进行读操作
    //Thread-8读操作完毕
    //Thread-3正在进行读操作
    //Thread-3读操作完毕
    //Thread-0正在进行读操作
    //Thread-0读操作完毕
}

private void get2(Thread thread) {
    rwl.readLock().lock();
    try {
        long start = System.currentTimeMillis();
        while(System.currentTimeMillis() - start <= 1) {
            System.out.println(thread.getName()+"正在进行读操作");
        }
        System.out.println(thread.getName()+"读操作完毕");
    } finally {
        rwl.readLock().unlock();
    }
}

```

明显提升读操作的效率，不过要注意的是，如果有一个线程已经占用了读锁，则此时其他线程如果要申请写锁，则申请写锁的线程会一直等待释放读锁。如果有一个线程已经占用了写锁，则此时其他线程如果申请写锁或者读锁，则申请的线程会一直等待释放写锁。

> Lock和synchronized的选择

- Lock是一个接口，而synchronized是Java中的关键字，synchronized是内置的语言实现；

- synchronized在发生异常时，会自动释放线程占有的锁，因此不会导致死锁现象发生；而Lock在发生异常时，如果没有主动通过unLock()去释放锁，则很可能造成死锁现象，因此使用Lock时需要在finally块中释放锁；

- Lock可以让等待锁的线程响应中断，而synchronized却不行，使用synchronized时，等待的线程会一直等待下去，不能够响应中断；

- 通过Lock可以知道有没有成功获取锁，而synchronized却无法办到。

- Lock可以提高多个线程进行读操作的效率。

      采用synchronized关键字来实现同步，如果多个线程都只是进行读操作，所以当一个线程在进行读操作时，其他线程只能等待无法进行读操作。因此就需要一种机制来使得多个线程都只是进行读操作时，线程之间不会发生冲突，通过Lock就可以办到[需要有一种机制可以不让等待的线程一直无期限地等待下去（比如只等待一定的时间或者能够响应中断）]。

- Lock不是Java语言内置的，synchronized是Java语言的关键字，因此是内置特性。Lock是一个类，通过这个类可以实现同步访问；

- 获取锁的线程释放锁只会有两种情况
  - 获取锁的线程执行完了该代码块，然后线程释放对锁的占有；
  - 线程执行发生异常，此时JVM会让线程自动释放锁。

在性能上来说，如果竞争资源不激烈，两者的性能是差不多的，而当竞争资源非常激烈时（即有大量线程同时竞争），此时Lock的性能要远远优于synchronized。所以说，在具体使用时要根据适当情况选择。

## 锁优化

> 参阅`REFERENCES-1`

## REFERENCES

1. [Java 多线程编程 — 锁优化](http://www.importnew.com/27935.html)
2. [Lock和synchronized比较详解](http://www.cnblogs.com/dolphin0520/p/3923167.html)

---

## 更多

> 扫码关注“架构探险之道”，回复`文章名称`获取更多源码和文章资源

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190403222309957.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzI4NjkwNDE3,size_16,color_FFFFFF,t_70)

> 知识星球(扫码加入获取源码和文章资源链接)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190403222322267.jpeg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzI4NjkwNDE3,size_16,color_FFFFFF,t_70)
