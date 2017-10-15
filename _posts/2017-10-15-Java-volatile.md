---
layout: post
title:  "Java volatile关键字"
categories: Java Java并发编程
tags: 多线程 并发编程 Java
---

* content
{:toc}



### volatile 概念

* volatile 关键字的主要作用是使**变量**在**多个线程之间**可见性。
  * 这里的“可见性”是指当一条线程修改了这个变量值，新值对于其他线程线程来说是可以立即得知的。而普通变量不能做到这一点，普通变量的值在线程之间的传递均需通过主内存来完成：
    * 线程A修改一个普通变量->回写到主内存->线程B在A回写完成之后，从主内存中读取->新变量值才会对线程B可见
  * volatile变量在各个线程的工作内存中不存在**一致性问题**(在各个线程的工作内存中，volatile 变量也可以存在不一致的情况，但是由于每次使用之前都需要刷新，执行引擎开不到不一致的情况，因此可以认为不存在一致性的问题)。
* volatile 第二个语义是**禁止指令重排优化**，普通的变量仅仅会保证在该方法的**执行过程中所有依赖赋值结果的地方都能获取到正确的结果**，而不能保证**变量赋值操作的顺序与程序代码中的执行顺序一致**。（在JDK1.5之前，是不能完全避免重排序导致的问题）

#### Java内存模型

* 理解volatile 关键字，我们首先需要了解 Java 内存模型(Java Memory Model):

  * 主内存与工作内存：

    * Java内存模型规定了所有的变量都存储在主内存中。

    * 每条线程都有自己的工作内存，线程的工作内存中保存了被该线程使用到的变量的**主内存的副本拷贝**

    * 线程对变量的所以操作（读取、赋值等）必须在工作内存中进行，而不能直接读写主内存中的变量

      * > 按照Java虚拟机规范，volatile变量依然有工作内存的拷贝，但是由于它特殊的操作顺序性规定，所以看起来如同在直接在主内存中读写访问一般，所以对于volatile也不存在例外

    * 不同的线程直接也无法直接访问对方工作内存中的变量，线程之间的变量值传递均需要通过主内存来完成。

  ![](http://ww1.sinaimg.cn/large/afac410dgy1fiq8gmeogcj20mn08tgo1.jpg)






* 加锁机制既可以确保可见性又可以确保原子性； 而volatile变量只能确保可见性，并不能保证原子性：
    * 虽然volatile在工作内存中不存在一致性问题，**但是Java里面的运算并非原子操作**。
* **当且仅当满足以下所有条件**，才应该使用volatile变量：
    * 对变量的写入操作不依赖变量的当前值，或者你能确保只有单个线程更新变量的值
    * 该变量不会与其他状态变量一起纳入不变性条件中
    * 在访问该变量时不需要加锁


#### Java 内存模型对 volatile 变量定义的特殊规则

* T表示一个线程， V 和 W 便是两个 volatile 型变量， 那么在进行 read、load、use、assign、store 和 write 操作时需要满足以下规则：
  * 只有当线程T对变量V执行的前一个动作是load的时候，线程T才能对变量V执行use动作；并且，只有当线程T对变量V执行的后一个动作是use的时候，线程T才能对变量V执行 load动作。线程T对变量V的 use 动作可以认为是和线程T对变量V的 load、read 动作相关联，必须连续一起出现（**这条规则要求在工作内存中，每次使用V前都必须先从主内存刷新最新的值，用于保证能看见其他线程对变量V所做的修改后的值**）
    * 因为read 和 load 是不可单独出现，然后又保证了，user之前必须是load， load 之后是必须是user，所以user 和 load、read动作相关联。保证 volatile 变量必须从主内存刷新最新的值
  * 只有当线程T对变量V执行的前一个动作是assign的时候，线程T才能对变量V执行store动作；并且，只有当线程T对变量V执行的后一个动作是store的时候，线程T才能对变量V执行assign动作。线程T对变量V的assign动作可以认为是和线程T对变量V的store、write动作相关联，必须连续一起出现（**这条规则要求在工作内存中，每次修改V后都必须立刻同步回主内存中，用于保证其他线程可以看到自己对变量V所做的修改**）。
    * 同理 store 和 write 也是不可单独出现，规则中又保证了执行 assign 之后必须是store， 执行store之前必须是 assign ，所以assign 和 store 、write 动作相关联，保证了volatile变量修改之后必须立刻同步到主内存
  * 假定动作A是线程T对变量V实施的use或assign动作，假定动作F是和动作A相关联的load或store动作，假定动作P是和动作F相应的对变量V的read或write动作；类似的，假定动作B是线程T对变量W实施的use或assign动作，假定动作G是和动作B相关联的load或store动作，假定动作Q是和动作G相应的对变量W的read或write动作。如果A先于B，那么P先于Q（**这条规则要求volatile修饰的变量不会被指令重排序优化，保证代码的执行顺序与程序的顺序相同**）。



```java
/**
 * 测试volatile 关键字的可见性
 */
public class VolatileVisibility implements Runnable{

    private volatile boolean isRunning = true;
    public void setRunning(boolean running) {
        isRunning = running;
    }
    @Override
    public void run() {
        System.out.println("进入了run方法......");
        while (isRunning){
            // 不能在这使用System.out.println方法, 否则isRunning对线程可见
//            System.out.println("等待isRunning 被设置为false!");
        }
        System.out.println("退出了run方法......");
    }
    public static void main(String[] args) throws InterruptedException {
        VolatileVisibility visibility = new VolatileVisibility();
        Thread thread= new Thread(visibility);
        thread.start();
        Thread.sleep(1000);// 让主线程睡1秒
        visibility.setRunning(false);
        System.out.println("isRunning 已经被设置成了false");
    }
}
```

```java
/**
 * volatile不能保证原子性
 */
public class VolatileNotAtomic {

    private static volatile int count;
    private static void increase(){
        count++;
    }
    private static final int THREAD_COUNT = 20;
    public static void main(String[] args) {
        Thread[] threads = new Thread[THREAD_COUNT];
        for (Thread thread : threads) {
            thread = new Thread(new Runnable() {
                @Override
                public void run() {
                    for (int i = 0; i < 10000; i++) {
                        increase();
                    }
                }
            });
            thread.start();
        }
        while (Thread.activeCount() > 1){
            Thread.yield();
        }
        System.out.println(count);//输出的结果并不等于200000
    }
}
```



