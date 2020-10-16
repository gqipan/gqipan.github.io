---
layout: post
title:  "同步容器类和并发容器类"
categories: Java并发编程
tags: 多线程 并发编程 Java
---

### 同步容器类

* 同步容器类Vector 和 Hashtable ，以及一些由 Collections.synchronizedXxx 等工厂方法创建的。其底层的机制无非是**用传统的synchronized 关键字对每一个公用的方法都进行同步**，使得每次只能有一个线程访问容器的状态。这很明显不满足我们今天互联网时代的高并发的需求，在保证线程安全的同事，也必须要有足够好的性能。

* **同步类容器都是线程安全的**，但**在某些场景下需要加锁来保护复合操作**。复合操作包括：迭代（反复访问元素，直到遍历完容器中所有元素）、跳转（根据指定顺序找到当前元素的下一个元素）以及条件运算。这些复合操作在多线程并发地修改容器时，可能会表现出意外的行为，最经典的便是：`ConcurrentModificationException`， 原因是容器迭代的过程中，被并发的修改了内容，这是由于早期迭代设计的时候并没有考虑并发修改的问题。

* 同步容器将所有对容器的状态的访问都串行话，以实现他们的线程安全性。这种方法的代价是严重降低并发性，当多个线程竞争容器锁时，吞吐量将严重降低。

  ```java
  //-----------复合操作时，并不能保证线程安全性--------------------
  final Vector<String> tickets = new Vector<>();
  for(int i = 1; i<= 1000; i++){
    tickets.add("火车票"+i);
  }

  // 在迭代的过程中，被后面的线程并发的修改了迭代器中的内容，就会抛出 ConcurrentModificationException
  for (Iterator iterator = tickets.iterator(); iterator.hasNext(); ) {
    String string = (String) iterator.next();
    tickets.remove(20);
  }

  for(int i = 1; i <=10; i ++){
    new Thread("线程"+i){
      public void run(){
        while(true){
          if(tickets.isEmpty()) break;
          System.out.println(Thread.currentThread().getName() + "---" + tickets.remove(0));
        }
      }
    }.start();
  }
  ```



### 并发容器类

* JDK1.5 提供了多种并发容器类来改进同步容器的性能。ConcurrentHashMap, 用来代替同步且基于散列的Map（HashTable），而且在ConcurrentHashMap 中，添加了一些常见复合操作的支持（如：“若没有则添加，替换以及有条件删除等”）。以及 CopyOnWriteArrayList ，用于遍历操作为主要操作的情况下代替同步的List(Voctor)。
* JDK1.5 还增加了两种新的容器类型： Queue 和 BlockingQueue， Queue 用来临时保存一组等待处理的元素。
  * 高性能队列 ConcurrentLinkedQueue，这是一个传统的先进先出的队列。
  * 带优先级的 PriorityQueue, 这是一个**非并发**的**优先队列**
  * BlockingQueue 扩张了 Queue， 增加拉可阻塞的插入和获取元素等操作。如果队列为空，那么获取元素的操作将一直阻塞，知道队列中出现一个可用的元素。 如果队列已满（**对于有界队列来说**），那么插入元素的操作将一直阻塞，知道队列中出现可用的空间。适用于**生产者-消费者**设计模式。
  * 其他实现Queue的类：
    * LinkedBlockingQueue    （无界 Integer.MAX_VALUE）
    * ArrayBlockingQueue    （有界队列）
    * PriorityBlockingQueue （无界队列，数组自动增长）
    * SynchronousQueue   (无内存队列)
    * DelayQueue         （无界队列）
* Java6 引入了 ConcurrentSkipListMap、ConcurrentSkipListSet， 分别作为同步的 SortedMap 和 SortedSet 的并发替代品。例如用（synchronizedMap 包装的TreeMap 或 TreeSet）

#### ConcurrentMap

* ConcurrentMap 接口下有两个重要的实现：
  * ConcurrentHashMap
  * ConcurrentSkipListMap （支持并发排序功能，弥补 ConcurrentHashMap）
* ConcurrentHashMap 内部使用段（Segment）来表示这些不同的部分，每个段其实就是一个小的HashTable，它们有自己的锁。只要多个修改操作发生在不同的段上，它们就可以并发进行。把一个整体分成了16个段（Segment）。也就是最高支持16个线程的并发修改操作。这也是在多线程场景时**减小锁的粒度**从而**降低锁竞争**的一种方案，并且代码中大多共享变量使用volatile关键字声明，目的是第一时间获取修改的内容。性能非常好。

#### Copy-On-Write容器

* Copy-On-Write 简称 COW，是一种用于程序设计中的优化策略。
* JDK里的COW容器有两种： CopyOnWriteArrayList 和 CopyOnWriteArraySet， COW容器非常有用，可以在非常多的并发场景中使用到
* **什么是 CopyOnWrite 容器**：
  * CopyOnWrite 容器即 写时复制 的容器。通俗的理解是当我们**往一个容器添加元素的时候，不直接往当前容器添加，而是先将当前容器进行Copy，复制出一个新的容器**。这样做的好处是我们可以对CopyOnWrite 容器进行并发的读，而不需要加锁，因为当前容器不会添加任何元素。所以CopyOnWrite容器也是一种读写分离的思想，读和写在不同的容器中进行。
  * 当新容器完成写操作之后，会把原来容器的引用指向新的容器上。
 







