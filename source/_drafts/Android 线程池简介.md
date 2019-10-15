---
title: Android 线程池简介
permalink: android-thread-pool
comments: true
date: 2018-04-22 10:52:06
updated: 2018-04-22 10:52:06
tags:
  - 线程池
categories:
  - Android 技术文章
---

`Android` 中有多种实现多线程的方式，其中最重要的是使用 `Thread` 或 `Runnable`。但是，使用 `Thread` 或 `Runnable` 实现多线程的方式，无法有效的在任务结束之后将结果返回。为了解决这一问题，`Java` 额外提供了 `Callable` 和 `FutureTask` 两种创建线程的方式，允许我们在线程执行完成之后得到返回的结果。

另外，基于 `Thread`、`Runnable`、`Callable` 和 `FutureTask` 的封装，`Android` 系统中还提供了如下多线程的实现方式：

* `AsyncTask`：基于线程池和 `Handler` 的封装，通常用于执行需要更新 `UI` 的任务
* `HandlerThread`：基于 `Thread` 和 `Handler` 的封装，接收来自 `Handler` 的消息，其内部只有一个线程在执行任务
* `IntentService`：基于 `Service` 和 `HandlerThread` 的封装，线程的优先级更高

本文将围绕线程池的使用展开讨论。
<!--more-->

## ThreadPoolExecutor

### ThreadPoolExecutor 类型的线程池

`ThreadPoolExecutor` 是 `Java` 中线程池的最终实现，`J.U.C` 包下提供的多种不同的线程池实现是基于 `ThreadPoolExecutor` 的：

* `FixedThreadPool`：通过 `Executors.newFixedThreadPool(int nThreads)` 创建的线程池大小为 `nThreads` 的且全部为核心线程的线程池：

```java
public static ExecutorService newFixedThreadPool(int nThreads) {
    return new ThreadPoolExecutor(nThreads, nThreads, 0L, TimeUnit.MILLISECONDS, new LinkedBlockingQueue<Runnable>());
}
```

其中，`nThreads` 为线程池的核心线程数（最大线程数）。

上述过程会创建一个核心线程数和最大线程数为 `nThreads`（这也就意味着 `FixedThreadPool` 中的所有线程都是核心线程）。如果当前所有线程都处于活动状态，那么新提交的任务将会被添加到 `LinkedBlockingQueue` 队列（无限制大小）中进行等待。而且由于其超时时间（`keepAliveTime`）为0，则意味着该线程池没有超时机制，所有的线程（全部为核心线程）都不会在空闲时被回收。

`FixedThreadPool` 适用于紧急的、需要快速响应而且任务不能被回收的业务场景。

* `SingleThreadExecutor`：通过 `Executors.newSingleThreadExecutor()` 创建的线程池大小（全部为核心线程）为1的线程池：

```java
public static ExecutorService newSingleThreadExecutor() {
    return new FinalizableDelegatedExecutorService(new ThreadPoolExecutor(1, 1, 0L, TimeUnit.MILLISECONDS, new LinkedBlockingQueue<Runnable>()));
}
```

`AbstractExecutorService` 和 `ThreadPoolExecutor` 在抽象层级上位于同一高度。从具体实现来说，`AbstractExecutorService` 接收一个 `ThreadPoolExecutor` 对象作为参数，再具体实现上，与直接使用下列代码并没有本质的区别：

```java
public static ExecutorService newSingleThreadExecutor() {
    return new ThreadPoolExecutor(1, 1, 0L, TimeUnit.MILLISECONDS, new LinkedBlockingQueue<Runnable>());
}
```

从实现的角度来说，`SingleThreadExecutor` 和 `FixedThreadPool` 除了核心线程的数量区别外，并没有本质的区别。这也就是说，使用 `Executors.newFixedThreadPool(int nThreads)` 创建线程池时，如果 `nThreads = 1`，那么将会创建一个 `SingleThreadExecutor` 的线程池。

`SingleThreadExecutor` 适用于非紧急的但是任务不能被回收的业务场景。

* `CachedThreadPool`：通过 `Executors.newCachedThreadPool()` 创建的核心线程数为0，线程总数为 `Integer.MAX_VALUE` 的线程池：

```java
public static ExecutorService newCachedThreadPool() {
    return new ThreadPoolExecutor(0, Integer.MAX_VALUE, 60L, TimeUnit.SECONDS, new SynchronousQueue<Runnable>());
}
```

`CachedThreadPool` 是一个核心线程数为0，最大线程数为 `Integer.MAX_VALUE`，超时时间为60秒的线程池。当有新的任务进入时，如果没有空闲的线程，那么将会直接创建新的线程来执行任务（并不会进入任务队列）。而且，在线程空闲超过60秒之后，线程将会被回收。

`CachedThreadPool` 适用于执行大量低耗时的任务。

* `ScheduledThreadPool`：通过 `Executors.newScheduledThreadPool(int corePoolSize)` 创建的核心线程数为 `corePoolSize` 的执行周期性任务的线程池：

```java
public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize) {
    return new ScheduledThreadPoolExecutor(corePoolSize);
}
```

`ScheduledThreadPoolExecutor` 是实现了 `ScheduledExecutorService` 接口的 `ThreadPoolExecutor` 子类，其在拥有线程池的属性之外，还增加了周期性执行任务的功能。上述代码中，`ScheduledThreadPoolExecutor` 构造方法的具体实现为：

```java
public ScheduledThreadPoolExecutor(int corePoolSize) {
    super(corePoolSize, Integer.MAX_VALUE, DEFAULT_KEEPALIVE_MILLIS, MILLISECONDS, new DelayedWorkQueue());
}
```

上述过程会创建一个核心线程数为1，线程总数为 `Integer.MAX_VALUE` 的线程池，其超时时间为10毫秒，这也就意味着，任务执行完毕，线程会在极短的时间内（10毫秒）被回收。

`ScheduledThreadPool` 适用于执行定时任务和周期性执行的任务。

* `SingleThreadScheduledExecutor`：通过 `Executors.newSingleThreadScheduledExecutor()` 创建的核心线程数为1的执行周期性任务的线程池：

```java
public static ScheduledExecutorService newSingleThreadScheduledExecutor() {
    return new DelegatedScheduledExecutorService(new ScheduledThreadPoolExecutor(1));
}
```

与 `ScheduledThreadPool` 一样的，`SingleThreadScheduledExecutor` 实际上是创建了一个核心线程数为1的 `SingleThreadScheduledExecutor` 对象。

### ThreadPoolExecutor 构造方法

从上文中我们可以看出，各种类型的线程池实际上是对 `ThreadPoolExecutor` 初始化参数的不同调用：

```java
public ThreadPoolExecutor(int corePoolSize, int maximumPoolSize, long keepAliveTime, TimeUnit unit, BlockingQueue<Runnable> workQueue, ThreadFactory threadFactory, RejectedExecutionHandler handler) {
    if (corePoolSize < 0 || maximumPoolSize <= 0 || maximumPoolSize < corePoolSize || keepAliveTime < 0)
        throw new IllegalArgumentException();
    if (workQueue == null || threadFactory == null || handler == null)
        throw new NullPointerException();
    this.corePoolSize = corePoolSize;
    this.maximumPoolSize = maximumPoolSize;
    this.workQueue = workQueue;
    this.keepAliveTime = unit.toNanos(keepAliveTime);
    this.threadFactory = threadFactory;
    this.handler = handler;
}
```

其中：

* `corePoolSize`：线程池中核心线程的数量。线程池创建之后，并不会立刻创建所有的核心线程（调用 `ThreadPoolExecutor` 的 `prestartAllCoreThreads` 或 `prestartCoreThread` 方法会立刻创建核心线程），而是有新的任务进入之后，如果线程数小于 `corePoolSize`，那么就创建一个新的线程，否则按照线程池的策略进行处理。

通常情况，核心线程并不会被回收，但是如果创建线程池时调用 `allowCoreThreadTimeOut(true)`，那么核心线程也会被超时回收。

* `maximumPoolSize`：线程池中最大的线程数量。在某些类型的线程池策略中（比如 `FixedThreadPool`），如果当前线程数量已经超过 `maximumPoolSize` 的值，按照特定的策略，新的任务将会放入任务队列中等待。

当有新的任务进入时，如果当前 `maximumPoolSize` 小于 `corePoolSize`，那么即便是其他非核心线程处于空闲状态，也会创建一个新的核心线程来执行任务。如果有大于 `corePoolSize` 但小于 `maximumPoolSize` 的线程正在执行，那么只有当任务队列达到上限时才会创建新的线程。但是线程总数不会超过 `maximumPoolSize`。

* `keepAliveTime` 和 `unit`：`keepAliveTime` 是线程在执行完任务之后空闲的时间，`unit` 是单位。如果超过 `keepAliveTime` 空闲时间，那么线程会被回收。

通常情况下，`keepAliveTime` 仅适用于存在超过 `corePoolSize` 数量的线程的情况。这也就是说，核心线程并不会被回收。但是如果 `keepAliveTime` 值不为0，那么如果调用 `allowCoreThreadTimeOut(true)` 方法也会造成核心线程的回收。

* `workQueue`：任务队列 `BlockingQueue` 用于存放进入线程池的任务。队列的实际容量与线程池的大小关联：

  * 如果当前线程池的任务数量小于核心线程池的数量，执行器会优先创建一个线程而不是从线程队列中获取一个空闲线程
  * 如果当前线程池的任务数量大于核心线程池的数量，如果此时线程队列中有空闲线程，那么执行器会读取一个空闲线程用于执行任务，否则会创建一个线程执行任务，直到线程总数超过 `maximumPoolSize`，新添加的任务将会触发线程池的饱和策略

`Java` 中提供了如下四种任务队列：

* `ArrayBlockingQueue`：基于数组的有界阻塞队列，按照 `FIFO` 排序任务
* `LinkedBlockingQuene`：基于链表结构的阻塞队列，按照 `FIFO` 排序任务，吞吐量通常要高于 `ArrayBlockingQueue`
* `SynchronousQuene`：一个不存储元素的阻塞队列，每个插入操作必须等到另一个线程调用移除操作，否则插入操作一直处于阻塞状态，吞吐量通常要高于 `LinkedBlockingQuene`
* `PriorityBlockingQuene`：具有优先级的无界阻塞队列

* `handler`：线程池的饱和策略。如果线程池的任务队列已经饱和，而且没有空闲的线程用于处理任务，如果继续向该线程池提交任务，必须采取一种策略处理该任务。线程池提供了四种策略：

  * `AbortPolicy`：默认策略，抛出 `RejectedExecutionException` 运行时异常
  * `CallerRunsPolicy`：提供了一个简单的反馈控制机制，可以减慢提交新任务的速度
  * `DiscardPolicy`：直接丢弃新提交的任务
  * `DiscardOldestPolicy`：如果执行器没有关闭，队列头的任务将会被丢弃，然后执行器重新尝试执行任务，如果失败，则重复这一过程


## ForkJoinPool

`ForkJoinPool` 是对 `Fork/Join` 并行框架的线程池封装。`ForkJoinPool` 的核心思想是将大的任务拆分成多个小的任务（即 `Fork`），然后将多个小任务处理汇总到一个结果上面（即 `Join`）。与 `ThreadPoolExecutor` 一样的，`ForkJoinPool` 提供基本的线程池功能。而且 `ForkJoinPool` 引入了`任务窃取`机制，在多 `CPU` 的设备上有更好的性能表现。

> 关于 `Fork/Join` 框架，可以参考 [The Java™ Tutorials Fork/Join](https://docs.oracle.com/javase/tutorial/essential/concurrency/forkjoin.html)。

通过 `Executors.newWorkStealingPool(int parallelism)` 可以创建一个基于 `ForkJoinPool` 线程池的 `ExecutorService` 对象。其中，`parallelism` 表示并行级别，通常情况下使用 `CPU` 的核心数即可。
