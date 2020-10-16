---
layout: post
title:  "阻塞队列 BlockingQueue"
categories: Java并发编程
tags: 多线程 并发编程 Java
---


* content
{:toc}


### 阻塞队列 BlockingQueue


#### BlockingQueue用法

* BlockingQueue  通常用于一个线程生产对象，而另外一个线程消费这些对象的场景。下图是
  对这个原理的阐述：

  ![](http://ww1.sinaimg.cn/large/afac410dgy1fj9xow085nj20hu07awf4.jpg)

* 一个线程将会持续生产新对象并将其插入到队列之中，直到队列达到它所能容纳的临界点。
  也就是说，它是有限的。如果该阻塞队列到达了其临界点，负责生产的线程将会在往里边插
  入新对象时发生阻塞。它会一直处于阻塞之中，直到负责消费的线程从队列中拿走一个对象。 
  负责消费的线程将会一直从该阻塞队列中拿出对象。如果消费线程尝试去从一个空的队列中
  提取对象的话，这个消费线程将会处于阻塞之中，直到一个生产线程把一个对象丢进队列。

#### BlockingQueue 的方法

* BlockingQueue  具有  4  组不同的方法用于插入、移除以及对队列中的元素进行检查。如果请求的操作不能得到立即执行的话，每个方法的表现也不同。这些方法如下： 

| /    | 抛异常       | 特定值      | 阻塞     | 超时                          |
| ---- | --------- | -------- | ------ | --------------------------- |
| 插入   | add(o)    | offer(o) | put(o) | offer(o, timeout, TimeUnit) |
| 移出   | remove()  | poll()   | take() | poll(timeout, timeunit)     |
| 检查   | element() | peek()   |        |                             |



* 四组不同的行为方式解释： 

1.  抛异常：如果试图的操作无法立即执行，抛一个异常。 
2.  特定值：如果试图的操作无法立即执行，返回一个特定的值(常常是  true / false)。 
3.  阻塞：如果试图的操作无法立即执行，该方法调用将会发生阻塞，直到能够执行。 
4.  超时：如果试图的操作无法立即执行，该方法调用将会发生阻塞，直到能够执行，但等待时间不会超过给定值。返回一个特定值以告知该操作是否成功(典型的是  true / false)。 

* **无法向一个  BlockingQueue  中插入  null。如果你试图插入  null，BlockingQueue  将会抛出一个  NullPointerException。** 

#### 方法详解（只分析 ArrayBlockingQueue 这一种实现，其他的类似）

* **插入数据方法：add(o)、offer(o)、put(o)、offer(o, timeout, TimeUnit)** 

  * `void put(E e) throws InterruptedException;` ：队列的容量已满时，线程会一直阻塞

    * ```java
      public void put(E e) throws InterruptedException {
        checkNotNull(e);
        final ReentrantLock lock = this.lock;
        lock.lockInterruptibly();
        try {
          while (count == items.length) // 当队列的容量已满时，线程会一直阻塞着
            notFull.await();
          insert(e);     // 会在插入方法中调用 notEmpty.signal(); 唤醒阻塞的take方法线程
        } finally {
          lock.unlock();
        }
      }
      ```

  * `boolean offer(E e);`  试着往队列中插入值，不管队列满没满，都会返回结果

    * ```java
      public boolean offer(E e) {
        checkNotNull(e);
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
          if (count == items.length) // 当队列已满时，直接返回false
            return false;
          else {
            insert(e);
            return true;
          }
        } finally {
          lock.unlock();
        }
      }
      ```

  * `boolean add(E e);`  add方法实际上是`AbstractQueue` 中的方法，里面的实现是先调用上述的 offer 方法，当 offer 方法返回为 true 是，直接返回true， 如果是返回 false 是，直接抛出异常  `IllegalStateException Queue full `

    * ```java
      public boolean add(E e) {
        if (offer(e)) // 直接调用 offer 方法
          return true;
        else // 如果 offer 返回为 false， 那么会抛出  Queue full 异常
          throw new IllegalStateException("Queue full");
      }
      ```

  * `boolean offer(E e, long timeout, TimeUnit unit) throws InterruptedException;`  跟 `offer(o)`方法相似，会先判断 队列是否已满，如果队列已满，会循环等待传入的时间，当超过设置的时间后，还是无空闲位置。会直接返回一个false。

    * ```java
      public boolean offer(E e, long timeout, TimeUnit unit) throws InterruptedException {
        checkNotNull(e);
        long nanos = unit.toNanos(timeout);
        final ReentrantLock lock = this.lock;
        lock.lockInterruptibly();
        try {
          while (count == items.length) { // 轮询判断队列是否已满
            if (nanos <= 0)  // 如果队列满了，判断设置的等待时间是否超时了，超时了直接返回false
              return false;
            nanos = notFull.awaitNanos(nanos); // 没超时，调用计算时间方法， 进入下一个循环中继续判断
          }
          insert(e);  // 在轮询等待的时间内，有空闲就立刻插入数据
          return true;
        } finally {
          lock.unlock();
        }
      }
      ```

* **移出数据的方法：remove()、 remove(o)、poll()、take()、 poll(timeout, TimeUnit)** 

  * `E poll();`  获取队列头的数据，并删除该数据（将该位置数据置为null）。如果队列已经空了，那么直接返回 null

    * ```java
      public E poll() {
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
          return (count == 0) ? null : extract(); // 判断队列是否为空，为空直接返回null
        } finally {
          lock.unlock();
        }
      }

      // 如果队列不为空，调用extract() 方法， 这个方法调用必须获取锁
      /**
       * Extracts element at current take position, advances, and signals.
       * Call only when holding lock.
       */
      private E extract() {
        final Object[] items = this.items;
        E x = this.<E>cast(items[takeIndex]);  // 获取当前位置的元素（队列头）
        items[takeIndex] = null;              // 然后把该位置置为null
        takeIndex = inc(takeIndex);           //  把索引向前推进
        --count;                              // 元素容量计数也减少
        notFull.signal();                     // 通知其他线程，可以进行put操作了
        return x;                             // 返回结果
      }
      ```

  * `E poll(long timeout, TimeUnit unit) throws InterruptedException;` 判断队列是否为空，如果为空等待设置的时间，等待时间超过设置的时间。直接返回null。

    * ```java
      public E poll(long timeout, TimeUnit unit) throws InterruptedException {
        long nanos = unit.toNanos(timeout);
        final ReentrantLock lock = this.lock;
        lock.lockInterruptibly();
        try {
          while (count == 0) {   // 判断队列是否为空
            if (nanos <= 0)       // 如果为空，判断是否等待时间已耗尽
              return null;        // 耗尽，直接返回null 
            nanos = notEmpty.awaitNanos(nanos); // 没耗尽，调用计算时间方法进入下一个循环
          }
          return extract();  // 队列不为空，直接调用获取数据的方法
        } finally {
          lock.unlock();
        }
      }
      ```

  * `E take() throws InterruptedException;`   take 方法，如果队列为空，会进入阻塞状态

    * ```java
      public E take() throws InterruptedException {
        final ReentrantLock lock = this.lock;
        lock.lockInterruptibly();
        try {
          while (count == 0)   //当队列中没有元素时，会进入阻塞状态等待唤醒
            notEmpty.await();  
          return extract();
        } finally {
          lock.unlock();
        }
      }
      ```

  * `E remove();`   实际是调用 poll() 方法获取队列头对象并删除，如果返回的值不为null，那么久直接返回。如果为null 就抛出 `NoSuchElementException` 

    * ```java
      public E remove() {
        E x = poll();    // 移出队列头对象
        if (x != null)
          return x;
        else
          throw new NoSuchElementException();  // 如果返回的值是null 直接抛出异常信息
      }
      ```

  *  `boolean remove(Object o);`  该方法不管有没有获取到队列头对象，都会返回一个结果值。不会抛出异常信息。

    * ```java
      public boolean remove(Object o) {
        if (o == null) return false;
        final Object[] items = this.items;
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
          for (int i = takeIndex, k = count; k > 0; i = inc(i), k--) {  //遍历队列
            if (o.equals(items[i])) {	// 判断对象是否相等
              removeAt(i);
              return true;
            }
          }
          return false;     // 如果没有相等的对象返回false
        } finally {
          lock.unlock();
        }
      }


      void removeAt(int i) {
        final Object[] items = this.items;
        // if removing front item, just advance
        if (i == takeIndex) {   // 如果需要移出的对象就是当前的队列头，直接置为null且索引后移
          items[takeIndex] = null;
          takeIndex = inc(takeIndex);
        } else {
          // slide over all others up through putIndex.
          for (;;) {  
            int nexti = inc(i);        // 需要移出元素索引的后一个为 nexti
            if (nexti != putIndex) {  // 当下一个索引不等于下一个添加元素的位置, 首次移出时ArrayBlockingQueue 的 putIndex 为0， 队列头。移除后为移出的索引位置
              items[i] = items[nexti];  // 后一个元素覆盖前一个元素
              i = nexti;               // 继续向后推进
            } else {
              items[i] = null;   // 把最后一个元素置为null
              putIndex = i;      // 最新添加元素的位置 putIndex 为 i
              break;
            }
          }
        }
        --count;
        notFull.signal();           
      }
      ```

* **检查数据的方法： element()、peek()**

  * `E peek();`   返回队列头的元素，但是不删除。如果队列为空返回null

    * ```java
      public E peek() {
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
          return (count == 0) ? null : itemAt(takeIndex);  // 返回队列头位置元素
        } finally {
          lock.unlock();
        }
      }

      final E itemAt(int i) {
        return this.<E>cast(items[i]);   // 直接返回队列头元素，不做删除操作
      }
      ```

  * `E element();`   返回队列头元素，如果为 null 抛出 `NoSuchElementException` 异常,  调用的实际上是  `peek()` 方法。

    * ```java
      public E element() {
        E x = peek();
        if (x != null)
          return x;
        else
          throw new NoSuchElementException();
      }
      ```

* 其他方法： contains(o)、remainingCapacity()、drainTo(Collection<? super E> c)、drainTo(Collection<? super E> c, int maxElements);

  * `public boolean contains(Object o);`  使用迭代器遍历元素判断是否包含Object，返回true or false

  * `int remainingCapacity();`   返回剩余容量大小，如果是ArrayBlockingQueue 那么就是数组容量的大小减去已经存在元素的个数值。

  * `int drainTo(Collection<? super E> c);`   把队列全部转换成 Collection 类，并且清空队列自身。

  * `int drainTo(Collection<? super E> c, int maxElements);`  从队列头部开始，转换并清空 maxElements 个元素成  Collection 类

    ​





> 待后续的其他队列实现类粗略讲解