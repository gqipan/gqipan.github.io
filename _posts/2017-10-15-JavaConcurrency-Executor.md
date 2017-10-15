---
layout: post
title:  "Executor 框架"
categories: Java Java并发编程
tags: 多线程 并发编程 Java
---

* content
{:toc}


## Executor 框架

### Executor 框架结构

* Executor 框架主要由 3 大部分组成如下。
  * **任务**。包括被执行任务需要实现的接口：**Runnable 接口**或 **Callable 接口**。
  * **任务的执行**。包括任务执行机制的**核心接口 Executor**，以及继承自 Executor 的  **ExecutorService** 接 口。Executor 框 架 有 两 个 关 键 类 实 现 了 **ExecutorService** 接 口（**ThreadPoolExecutor** 和 **ScheduledThreadPoolExecutor**）。
  * **异步计算的结果**。包括**接口 Future** 和**实现 Future 接口的 FutureTask 类**。
  * ![Executor 框架的使用示意图](http://ww1.sinaimg.cn/large/afac410dgy1fjoxj49t19j20iw0bjgma.jpg)

###  Executor 框架的成员

* Executor 框架的主要成员：ThreadPoolExecutor、ScheduledThreadPoolExecutor、Future 接口、Runnable 接口、Callable 接口和  Executors。

  * （1）ThreadPoolExecutor：

    * ThreadPoolExecutor 通常使用工厂类 Executors 来创建。Executors 可以创建 3 种类型的ThreadPoolExecutor：SingleThreadExecutor、FixedThreadPool 和 CachedThreadPool。

    * `FixedThreadPool `：固定数量的线程池，Executors 的 newFixedThreadPool 方法提供两个API

      * ```java
        public static ExecutorService newFixedThreadPool(int nThreads) {
          return new ThreadPoolExecutor(nThreads, nThreads,
                                        0L, TimeUnit.MILLISECONDS,
                                        new LinkedBlockingQueue<Runnable>());
        }
        public static ExecutorService newFixedThreadPool(int nThreads, ThreadFactory threadFactory) {
          return new ThreadPoolExecutor(nThreads, nThreads,
                                        0L, TimeUnit.MILLISECONDS,
                                        new LinkedBlockingQueue<Runnable>(),
                                        threadFactory);
        }
        ```

      * FixedThreadPool 适用于为了满足资源管理的需求，而需要限制当前线程数量的应用场景，它适用于负载比较重的服务器

      * FixedThreadPool  的 corePoolSize 和 maximumPoolSize 都被设置为创建是的指定参数 nThreads

        * 当线程池中的线程数大于 corePoolSize 时， keepAliveTime 为多余的空闲线程等待新任务的最长时间，超过时间多余的空闲线程将被终止。这里的 keepAliveTime 为0L，意味着多余的空闲线程会被立即终止
        * 如果线程数少于 corePoolSize ，则创建新的线程来执行任务（不管有没有空闲的线程，都会创建）
        * 线程池完成预热之后（当前运行的线程等于corePoolSize ）将任务加入 LinkedBlockingQueue

      * **FixedThreadPool  使用无界队列 LinkedBlockingQueue （队列容量为Integer.MAX_VALUE）对线程池带来的影响**。

        * 1）线程池线程数量到达 corePoolSize 后，新任务将在无界队列中等待，因此**线程池的数量不会超过corePoolSize** 

        * 2）由1），使用无界队列时 maximumPoolSize  将是无效参数

        * 3）由于1）、2），使用无界队列时 keepAliveTime 将是一个无效参数。(**无界队列不会满，将也不会出现已经创建的线程小于最大线程数，所以也不会有空闲线程** )

        * 4）由于使用无界队列，所以运行中的 FixedThreadPool  （未执行两个 shutdown 方法）不会拒接任务（不会调用RejectedExecutionHandler.rejectedExecution 方法）
      ​

    * `SingleThreadExecutor`：单个线程的线程池，Executors 的 newSingleThreadExecutor 方法提供了两个API

      * ```java
        public static ExecutorService newSingleThreadExecutor() {
          return new FinalizableDelegatedExecutorService
            (new ThreadPoolExecutor(1, 1,
                                    0L, TimeUnit.MILLISECONDS,
                                    new LinkedBlockingQueue<Runnable>()));
        }
        public static ExecutorService newSingleThreadExecutor(ThreadFactory threadFactory) {
          return new FinalizableDelegatedExecutorService
            (new ThreadPoolExecutor(1, 1,
                                    0L, TimeUnit.MILLISECONDS,
                                    new LinkedBlockingQueue<Runnable>(),
                                    threadFactory));
        }
        ```

      * SingleThreadExecutor 适用于需要保证顺序地执行各个任务；并且在任意时间点，不会有多个线程是活动的应用场景

      * SingleThreadExecutor 的 corePoolSize 和 maximumPoolSize 都被设置为1，其余参数都与FixedThreadPool  相同，也是使用了无界队列LinkedBlockingQueue作为线程池的工作队列。带来的影响跟 FixedThreadPool  相同

        ​

    * `CachedThreadPool`：大小无界的线程池，Executors 的 newCachedThreadPool 方法提供了两个API

      * ```java
        public static ExecutorService newCachedThreadPool() {
          return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                        60L, TimeUnit.SECONDS,
                                        new SynchronousQueue<Runnable>());
        }
        public static ExecutorService newCachedThreadPool(ThreadFactory threadFactory) {
          return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                        60L, TimeUnit.SECONDS,
                                        new SynchronousQueue<Runnable>(),
                                        threadFactory);
        }
        ```

      * CachedThreadPool 是大小无界的线程池，适用于执行很多短期的异步任务的小程序，或者是负载较轻的服务器

      * CachedThreadPool 的 corePoolSize  为0，但是 maximumPoolSize 被设置为Integer.MAX_VALUE ，所以可以说是 无界的，这里把 keepAliveTime 设置为 60L 意味着线程池中的空闲线程最长等待时间为60秒，超过将被终止

      * **CachedThreadPool 使用的是没有容量的 SynchronousQueue 队列作为线程池的工作队列，但maximumPoolSize 是无界的。这意味着，如果主线程提交任务的速度高于 maximumPoolSize  中的线程处理任务的速度时，CachedThreadPool将会不断的创建新的线程，极端的情况下会因为创建过多的线程而耗尽CPU 和 内存的资源**

        ​

  * （2）ScheduledThreadPoolExecutor （**定时任务**）：

    * ![ScheduledThreadPoolExecutor任务传递图](http://ww1.sinaimg.cn/large/afac410dgy1fjozmc88evj20mi0g2758.jpg)

    * ScheduledThreadPoolExecutor 通常使用 Executors  来创建。 可以创建两种类型的 ScheduledThreadPoolExecutor， 如下： 的 newScheduledThreadPool() 方法

      * ScheduledThreadPoolExecutor 包含若干线程的 ScheduledThreadPoolExecutor 
      * SingleThreadScheduledExecutor 只包含一个线程的 ScheduledThreadPoolExecutor 

    * 创建固定个数线程的 ScheduledThreadPoolExecutor  API：

      * ```java
        public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize){
          return new ScheduledThreadPoolExecutor(corePoolSize);
        }
        public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize, 						ThreadFactory threadFactory) {
          return new ScheduledThreadPoolExecutor(corePoolSize, threadFactory);
        }
        // --------------------
        public ScheduledThreadPoolExecutor(int corePoolSize) {
          super(corePoolSize, Integer.MAX_VALUE, 0, TimeUnit.NANOSECONDS,
                new DelayedWorkQueue());
        }
        public ScheduledThreadPoolExecutor(int corePoolSize,
                                               ThreadFactory threadFactory) {
          super(corePoolSize, Integer.MAX_VALUE, 0, TimeUnit.NANOSECONDS,
                new DelayedWorkQueue(), threadFactory);
        }
        ```

      * ScheduledThreadPoolExecutor  适用于需要**多个后台线程执行周期任务**，同时为了满足资源管理的需求而需要限制后台线程的数量的应用场景。内部实际上是用延迟队列 DelayedWorkQueue 来实现

    * 创建单个线程的 SingleThreadScheduledExecutor 的 API：

      * ```java
        public static ScheduledExecutorService newSingleThreadScheduledExecutor() {
          return new DelegatedScheduledExecutorService
            (new ScheduledThreadPoolExecutor(1));
        }
        public static ScheduledExecutorService newSingleThreadScheduledExecutor(ThreadFactory threadFactory) {
          return new DelegatedScheduledExecutorService
            (new ScheduledThreadPoolExecutor(1, threadFactory));
        }
        ```

      * 内部实际上还是用 new ScheduledThreadPoolExecutor 的发那个是实现，但是包装了一层委托 DelegatedScheduledExecutorService 

      * 适用于需要**单个后台线程执行周期任务**，同时需要保证顺序地执行各个任务的应用场景

    