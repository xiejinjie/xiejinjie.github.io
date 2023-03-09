---
layout: post
title: "java知识-线程池ThreadPoolExecutor学习"
date: 2023-03-04 10:10:00 +0800
categories: tech
---

## 简介
并发编程过程中经常会用到线程池，他帮我们管理线程，减少了频繁创建线程的开销。通常会使用Executors创建线程池，Executors提供了这三种常用的线程池，Executors.newCachedThreadPool()、Executors.newFixedThreadPool(int)、Executors.newSingleThreadExecutor()。这三种线程池对应的都是**ThreadPoolExecutor**这个类，只是使用了不同的构造参数，这里讲解一下这些参数，从而加深对线程池的理解。

先看一下ThreadPoolExecutor全部的构造参数，如下所示：
```
public ThreadPoolExecutor(  int corePoolSize,
                            int maximumPoolSize,
                            long keepAliveTime,
                            TimeUnit unit,
                            BlockingQueue<Runnable> workQueue,
                            ThreadFactory threadFactory,
                            RejectedExecutionHandler handler)
```

## 核心线程数和最大线程数（corePoolSize，maximumPoolSize）
线程池会根据corePoolSize和maximumPoolSize动态调整线程池的大小。当提交一个新的任务时，会根据当前工作的线程的数量和核任务队列的数量进行不同处理：

1. 线程数<核心线程数；则创建新的线程处理任务。注意，线程没有任务执行也会创建新线程。
2. 核心线程数<=工作线程数<最大线程数，且任务队列未满；则将任务添加至任务队列。
3. 核心线程数<=工作线程数<最大线程数，且任务队列已满；则创建新的线程处理任务。
4. 工作线程数=最大线程数,且任务队列已满；拒绝任务。

将核心线程数和最大线程数设置为相同的值，就得到了固定容量的线程池，等同于Executors.newFixedThreadPool(int)。将最大线程数设置为Integer.MAX_VALUE，线程池可以容纳任意数量的并发任务。
核心线程默认只会在新任务提交时初始化，想要提前初始化，可以使用prestartCoreThread()或prestartAllCoreThreads()启动一个或多个核心线程。

下面是一个的示例程序，创建了一个核心线程为1，最大线程为2，任务队列为1的线程池，依次提交了4个任务，每个任务直接休眠10s。

```
public class ThreadPoolTest {

    public static void main(String[] args) {
        ThreadPoolExecutor threadPool = new ThreadPoolExecutor(
                1, 2,
                10, TimeUnit.SECONDS,
                new LinkedBlockingQueue<>(1));
        printFields(threadPool, 1);
        // 线程数<核心线程数；创建线程
        threadPool.execute(task());
        printFields(threadPool, 2);
        // 核心线程数<=工作线程数<最大线程数，任务队列未满；任务添加队列
        threadPool.execute(task());
        printFields(threadPool, 3);
        // 核心线程数<=工作线程数<最大线程数，任务队列已满；创建线程
        threadPool.execute(task());
        printFields(threadPool, 4);
        // 工作线程数=最大线程数,任务队列已满；拒绝任务
        threadPool.execute(task());
        printFields(threadPool, 5);
    }

    private static void printFields(ExecutorService threadPool, int num) {
        try {
            Class<ThreadPoolExecutor> threadPoolClass = ThreadPoolExecutor.class;
            Field workers = threadPoolClass.getDeclaredField("workers");
            Field workQueue = threadPoolClass.getDeclaredField("workQueue");
            Field corePoolSize = threadPoolClass.getDeclaredField("corePoolSize");
            Field maximumPoolSize = threadPoolClass.getDeclaredField("maximumPoolSize");
            workers.setAccessible(true);
            workQueue.setAccessible(true);
            corePoolSize.setAccessible(true);
            maximumPoolSize.setAccessible(true);

            System.out.printf("%s worker=%s,workQueue=%s,corePoolSize=%s,maximumPoolSize=%s %n", num,
                    ((HashSet)workers.get(threadPool)).size(),
                    ((BlockingQueue)workQueue.get(threadPool)).size(),
                    corePoolSize.getInt(threadPool), maximumPoolSize.getInt(threadPool));
        } catch (NoSuchFieldException | IllegalAccessException e) {
            throw new RuntimeException(e);
        }
    }

    private static Runnable task() {
        return () -> {
            try {
                Thread.sleep(10000);
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
        };
    }

}
```
![](https://raw.githubusercontent.com/xiejinjie/xiejinjie.github.io/gh-pages/assets/img/20230309193008.png)

## 存活时间（keepAliveTime,unit）
当线程池内的线程数量超过了核心线程数，那么超过的线程的空闲时间超过设置的线程存活时间，就会被终止。这是为了减少线程池在不使用时资源的占用。同时使用Long.MAX_VALUE、TimeUnit.NANOSECONDS这两个参数，可以让空闲的线程不被终止。使用allowCoreThreadTimeOut(boolean) 方法，可以让核心线程也会被空闲超时终止。

## 任务队列（workQueue）
用来存储提交的任务，任务队列会影响线程池的容量，提交任务时：
1. 当线程数少于核心线程数，会优先创建新的线程
2. 当工作线程大于等于核心线程数，会优先加入队列
3. 当队列已满，就会创建新的线程，线程数量达到最大线程数据，就会拒绝任务。

一般有以下三种使用队列的策略：
1. 直接传递：比如使用SynchronousQueue，队列没有容量，会将任务直接交给线程。当处理的任务集有内部依赖时，这样可以避免锁定。
2. 无界队列：比如使用LinkedBlockingQueue，不指定容量，会导致工作线程数量不会超过核心线程。当每个任务都是独立的，这可能会很合适。
3. 有界队列：比如使用ArrayBlockingQueue，可能更不容易的调试和控制。可以通过调整最大线程数和队列大小获得不同的使用效果。

可以使用getQueue()方法来获取任务队列，用来进行监控和调试。

## 线程工厂类（threadFactory）
用来创建新的线程，默认使用Executors.defaultThreadFactory()，创建的线程都是非守护线程，属于同一个ThreadGroup，优先级为NORM_PRIORITY。

## 拒绝策略（handler）
当线程池终止，或者工作线程和工作队列都已满的时候，提交任务就会被拒绝。会调用线程池设置的RejectedExecutionHandler的rejectedExecution(Runnable, ThreadPoolExecutor)方法。有几种预设的处理器可以使用：
1. ThreadPoolExecutor.AbortPolicy：处理器会抛出RejectedExecutionException异常。
2. ThreadPoolExecutor.CallerRunsPolicy：使用提交任务线程直接执行任务。可以减缓提交任务速度。
3. ThreadPoolExecutor.DiscardPolicy：任务会直接丢弃。
4. ThreadPoolExecutor.DiscardOldestPolicy：任务队列中第一个任务会被丢弃，当前任务重试。

## 钩子函数
ThreadPoolExecutor提供了几个钩子函数，可以进行重写来满足特殊的业务。
- beforeExecute(Thread, Runnable) 每个任务提交前执行。
- afterExecute(Runnable, Throwable) 每个任务提交后执行。
- terminated() 所有的线程终止后执行。

Java doc中提供了一个使用钩子函数实现线程池暂停的样例，如下：

```
class PausableThreadPoolExecutor extends ThreadPoolExecutor {
    private boolean isPaused;
    private ReentrantLock pauseLock = new ReentrantLock();
    private Condition unpaused = pauseLock.newCondition();

    public PausableThreadPoolExecutor(...) { super(...); }

    protected void beforeExecute(Thread t, Runnable r) {
        super.beforeExecute(t, r);
        pauseLock.lock();
        try {
        while (isPaused) unpaused.await();
        } catch (InterruptedException ie) {
        t.interrupt();
        } finally {
        pauseLock.unlock();
        }
    }

    public void pause() {
        pauseLock.lock();
        try {
        isPaused = true;
        } finally {
        pauseLock.unlock();
        }
    }

    public void resume() {
        pauseLock.lock();
        try {
        isPaused = false;
        unpaused.signalAll();
        } finally {
        pauseLock.unlock();
        }
    }
}
```

## 拓展
工作中遇到一个场景：有超大量的计算任务需要异步执行，不限制任务队列容量会导致内存耗尽。限制任务队列容量后，需要在任务队列满时，后续提交任务的线程进行等待。最后参考了这个问题的答案： [ThreadPoolExecutor Block When its Queue Is Full? - Stackoverflow](https://stackoverflow.com/questions/4521983/java-executorservice-that-blocks-on-submission-after-a-certain-queue-size/4522411#4522411)。
继承LinkedBlockingQueue，并将offer重写为带有阻塞的put方法。

```
public class LimitedQueue<E> extends LinkedBlockingQueue<E> {
    public LimitedQueue(int maxSize){
        super(maxSize);
    }

    @Override
    public boolean offer(E e) {
        // turn offer() and add() into a blocking calls (unless interrupted)
        try {
            put(e);
            return true;
        } catch(InterruptedException ie) {
            Thread.currentThread().interrupt();
        }
        return false;
    }

}
```

## 参考资料
[ThreadPoolExecutor - Javadoc](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/ThreadPoolExecutor.html)
[ThreadPoolExecutor Block When its Queue Is Full? - Stackoverflow](https://stackoverflow.com/questions/4521983/java-executorservice-that-blocks-on-submission-after-a-certain-queue-size/4522411#4522411)
