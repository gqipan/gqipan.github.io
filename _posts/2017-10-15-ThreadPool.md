---
layout: post
title:  "线程池ThreadPool"
categories: Java Java并发编程
tags: 多线程 并发编程 Java
---

* content
{:toc}


### 线程池ThreadPool

#### 线程池带来的好处

* 第一：降低资源消耗。通过重复利用已创建的线程降低线程创建和销毁造成的消耗。
* 第二：提高响应速度。当任务到达时，任务可以不需要等到线程创建就能立即执行。
* 第三：提高线程的可管理性。线程是稀缺资源，如果无限制地创建，不仅会消耗系统资源，还会降低系统的稳定性，使用线程池可以进行统一分配、调优和监控。但是，要做到合理利用线程池，必须对其实现原理了如指掌。

#### 线程池的实现原理

* **当向线程池提交一个任务之后，线程池是如何处理这个任务的呢？**

  * 1）线程池判断核心线程池里的线程是否都在执行任务。如果不是，则创建一个新的工作线程来执行任务。如果核心线程池里的线程都在执行任务，则进入下个流程。

  * 2）线程池判断工作队列是否已经满。如果工作队列没有满，，则将新提交的任务存储在  这个工作队列里。如果工作队列满了，则进入下个流程。

  * 3）线程池判断线程池的线程是否都处于工作状态。如果没有，则创建一个新的工作线程来执行任务。如果已经满了，则交给饱和策略来处理这个任务

  ![线程池的主要处理流程](http://upload-images.jianshu.io/upload_images/2006966-6be3a945d64dbc82.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* **ThreadPoolExecutor 执行 execute() 方法时的流程如下：**

  * 1）如果当前运行的线程少于 `corePoolSize`，则创建新线程来执行任务（注意，执行这一步骤需要获取全局锁）。

  * 2）如果运行的线程等于或多于 `corePoolSize`，则将任务加入` BlockingQueue`（队列已满），则创建新的线程来处理任务（注意，执行这一步骤需要获取全局锁）。

  * 4）如果创建新线程将使当前运行的线程超出 `maximumPoolSize`，任务将被拒绝，并调用 `RejectedExecutionHandler.rejectedExecution()` 方法

    ![ThreadPoolExecutor执行图](http://upload-images.jianshu.io/upload_images/2006966-a4af509d6ef19811.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



#### 线程池的使用

#####  线程池的创建

* **我们可以通过 ThreadPoolExecutor 来创建一个线程池：**

  * ```java
    new ThreadPoolExecutor(int corePoolSize,
                                  int maximumPoolSize,
                                  long keepAliveTime,
                                  TimeUnit unit,
                                  BlockingQueue<Runnable> workQueue,
                                  ThreadFactory threadFactory,
                                  RejectedExecutionHandler handler);
    ```

  

* 创建一个线程池时需要输入几个参数，如下：

  * 1）`corePoolSize`（线程池的基本大小）：当提交一个任务到线程池时，线程池会创建一个线程来执行任务，**即使其他空闲的基本线程能够执行新任务也会创建线程**，**等到需要执行的任务数大于线程池基本大小时就不再创建**。如果调用了线程池的 **prestartAllCoreThreads()** 方法，**线程池会提前创建并启动所有基本线程**。

  * 2）`runnableTaskQueue`（任务队列）：**用于保存等待执行的任务的阻塞队列**。可以选择以下几个阻塞队列
    *  `ArrayBlockingQueue` ：是一个基于数组结构的有界阻塞队列，此队列按 FIFO（先进先出）原则对元素进行排序。
    *  `LinkedBlockingQueue` ：一个基于链表结构的阻塞队列，此队列按  FIFO 排序元素，吞吐量通常要高于 ArrayBlockingQueue。静态工厂方法 Executors.newFixedThreadPool() 使用了这个队列
    *  `SynchronousQueue` ：一个不存储元素的阻塞队列。每个插入操作必须等到另一个 线程调用移除操作，否则插入操作一直处于阻塞状态，吞吐量通常要高于 `LinkedBlockingQueue`，静态工厂方法 `Executors.newCachedThreadPool` 使用了这个队列。
    *  `PriorityBlockingQueue`：一个具有优先级的无限阻塞队列
  * 3）`maximumPoolSize`（线程池最大数量）：线程池允许创建的最大线程数。**如果队列满了**，**并且已创建的线程数小于最大线程数**，则**线程池会再创建新的线程执行任务**。值得注意的是，如果使用了**无界的任务队列这个参数就没什么效果(无界队列不会满)**。
  * 4）`ThreadFactory `：用于设置创建线程的工厂，可以通过线程工厂给每个创建出来的线程设置更有意义的名字。使用开源框架 guava 提供的 ThreadFactoryBuilder 可以快速给线程池里的线程设置有意义的名字，代码如下:
    * `new ThreadFactoryBuilder().setNameFormat("XX-task-%d").build()`;
  * 5）`RejectedExecutionHandler`（饱和策略）：当**队列和线程池都满了**，说明**线程池处于饱和状态**，那么必须采取一种策略处理提交的新任务。这个策略默认情况下是 `AbortPolicy`，表示无法处理新任务时抛出异常。在 JDK 1.5 中 Java 线程池框架提供了以下 4 种策略。
    * `AbortPolicy`：直接抛出异常
    * ` CallerRunsPolicy`：只用调用者所在线程来运行任务。
    * ` DiscardOldestPolicy`：丢弃队列里最近的一个任务，并执行当前任务。
    * `DiscardPolicy`：不处理，丢弃掉。
    * 当然，也可以根据应用场景需要来实现 RejectedExecutionHandler 接口**自定义策略**。如记录日志或持久化存储不能处理的任务。
  * 6） `keepAliveTime`（线程活动保持时间）：线程池的工作线程空闲后，保持存活的时间。所以，如果任务很多，并且每个任务执行的时间比较短，可以调大时间，提高线程的利用率。
  * 7） `TimeUnit`（线程活动保持时间的单位）：可选的单位有天（DAYS）、小时（HOURS）、分钟（MINUTES）、毫秒（MILLISECONDS）、微秒（MICROSECONDS，千分之一毫秒）和纳秒（NANOSECONDS，千分之一微秒）。

##### 向线程池提交任务

* 可以使用两个方法向线程池提交任务，分别为 execute() 和 submit() 方法。
  *  execute() 方法输入的任务是一个 Runnable 类的实例。`void execute(Runnable command)`
  *  submit() 方法输入任务可以是 Runnable 也可以是 Callable 的示例：
      * `Future<?> submit(Runnable task);`，返回的future 调用 get 方法时，返回的结果 null
      * `<T> Future<T> submit(Runnable task, T result);`，返回的future 调用 get 方法时，返回的结果是 result
      * `<T> Future<T> submit(Callable<T> task);`,  返回的 future 调用get方法时， 是 Callable 实例中 call 方法返回的结果


##### 关闭线程

* 可以通过调用线程池的 shutdown 或 shutdownNow 方法来关闭线程池。它们的原理是遍历线程池中的工作线程，然后逐个调用线程的 interrupt 方法来中断线程，所以**无法响应中断的任务可能永远无法终止**。
* 但是它们存在一定的区别，**shutdownNow 首先将线程池的状态设置成 STOP，然后尝试停止所有的正在执行或暂停任务的线程，并返回等待执行任务的列表**，
* 而 **shutdown 只是将线程池的状态设置SHUTDOWN 状态，然后中断所有没有正在执行任务的线程**



#### 合理地配置线程池

* 分析的角度：
  * **任务的性质**：CPU 密集型任务、IO 密集型任务和混合型任务。
  * **任务的优先级**：高、中和低。
* **性质不同的任务可以用不同规模的线程池分开处理**
  * CPU 密集型任务应配置尽可能小的线程，如配置 Ncpu+1 个线程的线程池。
  * 由于 IO 密集型任务线程并不是一直在执行任务，则应配置尽可能多的线程，如 2*Ncpu。
  * 混合型的任务，如果可以拆分，将其拆分成一个 CPU 密集型任务和一个 IO 密集型任务，只要这两个任务执行的时间相差不是太大，那么分解后执行的吞吐量将高于串行执行的吞吐量。如果这两个任务执行时间相差太大，则没必要进行分解。
  * 可以通过 Runtime.getRuntime().availableProcessors() 方法获得当前设备的 CPU 个数。
  * 优先级不同的任务可以使用优先级队列 PriorityBlockingQueue 来处理。它可以让优先级
    高的任务先执行。
  * 执行时间不同的任务可以交给不同规模的线程池来处理，或者可以使用优先级队列，让执行时间短的任务先执行。
  * 依赖数据库连接池的任务，因为线程提交 SQL 后需要等待数据库返回结果，等待的时间越长，则 CPU 空闲时间就越长，那么线程数应该设置得越大，这样才能更好地利用 CPU。
* **建议使用有界队列。有界队列能增加系统的稳定性和预警能力，可以根据需要设大一点儿，比如几千**。





> 参考 《Java并发编程艺术》、《Java并发编程实战》