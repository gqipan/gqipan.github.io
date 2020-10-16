---
layout: post
title:  "多线程之间的通信"
categories: Java并发编程
tags: 多线程 并发编程 Java
---

* content
{:toc}

### 多线程之间的通信

* 线程通信概念：线程是操作系统中独立的个体，但这些个体如果不经过特殊处理就不能成为一个整体，线程间的通信就成为整体的必用方式之一。当线程存在通信指挥，系统间的交互性会更强大，在提高CPU利用率的同时还会使开发人员对线程任务在处理过程中进行有效的把控与监督。
* 使用wait/notify 方法实现线程间的通信。（注意这两个方法都是Object类的方法，换句话说Java为所有的对象都提供了这两个方法）
  * **wait 和 notify 必须配合synchronized 关键字使用。**
  * **wait 方法释放锁，notify 方法不释放锁**



#### 使用轮询的方式通信

```java
public static void main(String[] args) {
  final ListAddTest1 list = new ListAddTest1();
  Thread thread_1 = new Thread(new Runnable() {
    @Override
    public void run() {
      for (int i = 0; i < 10; i++) {
        try {
          list.add();
          System.out.println("当前线程：" + Thread.currentThread().getName() + " 添加一个元素！");
          Thread.sleep(500);
        } catch (InterruptedException e) {
          e.printStackTrace();
        }
      }
    }
  }, "Thread-1");

  Thread thread_2 = new Thread(new Runnable() {
    @Override
    public void run() {
      while (true) {
        if (list.size() == 5) {
          System.out.println("当前线程收到通知：" + Thread.currentThread().getName() + " list size = 5 线程停止..");
          throw new RuntimeException();
        }
      }
    }
  }, "Thread-2");

  thread_1.start();
  thread_2.start();
}
```



#### 使用notify 和 wait 方法通信

* 如上述的代码，thread-2 线程一种处于一种轮询的方式，我们可以修改Java中多线程通信的方式来改造， 使用notify 和 wait. 
  * 注意必须配合 synchronized 关键字来使用，否则会报 java.lang.IllegalMonitorStateException 非法监控异常
  * 必须先启动 wait 方法的线程，因为notify 方法不会释放锁，先启动notify 方法的线程，执行到 notify 方法的时候，并没有线程在 wait.



```java
public static void main(String[] args) {
  final ListAddUseNotifyAndWait list = new ListAddUseNotifyAndWait();
  final Object lock = new Object();
  Thread thread_1 = new Thread(new Runnable() {
    @Override
    public void run() {
      //                synchronized (lock) {
      try {
        for (int i = 0; i < 10; i++) {
          list.add();
          System.out.println("当前线程：" + Thread.currentThread().getName() + " 添加一个元素！");
          Thread.sleep(500);
          if (list.size() == 5) {
            System.out.println("已经发出通知！");
            lock.notify();// notify 方法不会释放锁
          }
        }
      } catch (InterruptedException e) {
        e.printStackTrace();
      }
      //                }
    }
  }, "Thread-1");

  Thread thread_2 = new Thread(new Runnable() {
    @Override
    public void run() {
      try {
        //                    synchronized (lock) {
        if (list.size() != 5) {
          System.out.println("Thread-2进入...");
          lock.wait();// wait 方法会释放锁
        }
        System.out.println("当前线程收到通知：" + Thread.currentThread().getName() + " list size = 5 线程停止..");
        throw new RuntimeException();
        //                    }
      } catch (InterruptedException e) {
        e.printStackTrace();
      }
    }
  }, "Thread-2");

  // 需要先启动 thread_2 , 因为如果先启动 thread_1， 那么 notify 方法不会释放锁，
  // t1线程完全执行完才会释放锁, 当t1到notify 方法的时候，并没有 其他线程在 wait
  thread_2.start();
  thread_1.start();
}
```



#### 使用CountDownLatch 闭锁来进行通信

* 在使用notify 和 wait 来进行通信的时候还有一个问题，在程序中我们必须等 thread-1 执行完成，才会释放锁来通知 thread-2，这会造成**不实时的问题**。如果需要程序在 list.size 为5 的时候，就立马执行 thread-2 中的线程的话，我们就可以使用 CountDownLatch 的方式来进行通信



```java
final CountDownLatch countDownLatch = new CountDownLatch(1);
Thread thread_1 = new Thread(new Runnable() {
  @Override
  public void run() {
    try {
      for (int i = 0; i < 10; i++) {
        list.add();
        System.out.println("当前线程：" + Thread.currentThread().getName() + " 添加一个元素！");
        Thread.sleep(500);
        if (list.size() == 5) {
          System.out.println("已经发出通知！");
          countDownLatch.countDown();
        }
      }
    } catch (InterruptedException e) {
      e.printStackTrace();
    }
  }
}, "Thread-1");

Thread thread_2 = new Thread(new Runnable() {
  @Override
  public void run() {
    try {
      if (list.size() != 5) {
        System.out.println("Thread-2进入...");
        countDownLatch.await();
      }
      System.out.println("当前线程收到通知：" + Thread.currentThread().getName() + " list size = 5 线程停止..");
      throw new RuntimeException();
    } catch (InterruptedException e) {
      e.printStackTrace();
    }
  }
}, "Thread-2");

thread_1.start();
thread_2.start();
```







