---
layout: post
title:  "线程安全性"
categories: Java Java并发编程
tags: 多线程 并发编程 Java
---

* content
{:toc}


### 什么是线程安全性

* 当多个线程访问某个类时，这个类始终都能表现出正确的行为，那么久称这个类是线程安全的。

>&emsp;&emsp;当多个线程访问某个类时，不管运行时环境采用何种调度方式或者这些线程将如何交替执行，并且在主调代码中不需要任何额外的同步或协同，这个类都能表现出正确的行为，那么就称这个类是线程安全的。

* 无状态对象一定是线程安全的。

### 原子性

>&emsp;&emsp;假定有两个操作A 和 B，如果从执行A的线程来看，当另一个线程执行B时，要么将B全部执行完，要么完全不执行B，那么A 和 B对彼此来说都是原子的，原子操作是指：**对于访问同一个状态的所以操作（包括该操作本身来说），这个操作是一个以原子方式执行的操作**

* 在并发编程中，这种由于不恰当的执行时序而出现不正确的结果是一种非常严重的情况，它有一个正式的名字：**竞态条件（RaceCondition）**

* **竞态条件的本质**： 基于一种可能失效的观察结果来做出判断或者执行某个计算


### 内置锁(synchronized)

#### synchronized锁可重入

>* **如果某个线程视图获取一个已经由它自己持有的锁，那么这个请求就会成功**
>* 重入锁的一种实现方法是，为每个锁关联一个获取计数值和一个所有者线程。当计数值为0时，这个锁就被认为是没有被任何线程持有。当线程请求一个未被持有的锁时，JVM将记下锁的持有者，并且将获取的计数值置为1。如果同一个线程再次获取这个锁，计数值将递增，而当线程退出同步代码块时，计数值会相应地递减。当计数值为0时，这个锁将被释放。

* 关键字 synchronized 拥有**锁重入的功能**，也就是在使用 synchronized 时，当一个线程得到一个对象锁后， 再次请求该对象时是可以再次得到该对象的锁。 

    ```java
    //======继承上的锁重入, 如果内置锁不是可重入的，那么这段断码将发生死锁====//
    public class Widget{
        public synchronized void doSomething(){
            ...
        }
    }

    public class LoggingWidget extends Widget{
        public synchronized void doSomething(){
            Systeme.out.println(toString() + ": calling doSomething")
            super.doSomething();
        }
    }

    //===========  synchronized的重入:  同步方法之间的锁重入, 调用方法链 ========//
    public synchronized void method1(){
    	System.out.println("method1..");
    	method2();
    }
    public synchronized void method2(){
    	System.out.println("method2..");
    	method3();
    }
    public synchronized void method3(){
    	System.out.println("method3..");
    }

    ```


* **出现异常，锁自动释放**：
    * 说明：对于web应用程序，异常释放锁的情况，如果不及时处理，可能对你的应用程序业务逻辑产生严重的错误。 例如： 你现在执行一个队列任务，很多对象都去在等待第一个对象正确执行完毕再去释放锁。当时第一个对象由于异常的出现，导致业务逻辑没有正常的执行完毕，就释放了锁，那么可想而知后续的对象执行的都是错误的逻辑。所以这一点一定要引起注意，在编写代码的时候，一定要考虑周全。

#### **synchronized 代码块**

* 使用 synchronized 声明方法在某些情况下是有弊端的， 比如 A线程调用同步方法执行一个很长时间的任务，那么 B线程就必须等到比较长的时间才能执行，这样的情况下可以使用 synchronized 代码块去优化代码执行时间， 也就是通常所说的**减小锁的粒度**
* synchronized 可以使用**任意的Object ** 进行加锁，用法比较灵活
* 另外需要特别注意的一个问题，就是**不要使用String 常量加锁，会出现死循环问题**。当t1线程进入代码块之后，一直占用当前的资源。  因为常量池的概念，t2现在到这一步并不会获取到一个新的锁。所以t1一直处于循环状态。


```java
// 使用String 常量出现死循环问题
//分别使用new String("abc")和"abc"
synchronized ("abc") {  //这里是一个String类型的常量锁
  try {
    while(true){
      System.out.println("当前线程 : "  + Thread.currentThread().getName() + "开始");
      Thread.sleep(1000);
      System.out.println("当前线程 : "  + Thread.currentThread().getName() + "结束");
    }
  } catch (InterruptedException e) {
    e.printStackTrace();
  }
}
```

#### 锁对象改变问题

* 当使用一个对象进行加锁的时候，要注意对象本身发生改变的时候，那么持有的锁就不同。
* 如果对象本身不发生改变，那么依然是同步的，即使是对象的属性发生了改变。

#### 死锁问题

* 当t1 持有锁 lock-1,  t2 持有锁 lock-2，然后t1 在临界资源中需要lock-2， t2也需要lock-1就会出现死锁问题

```java
//-------------- t1线程 -----------
synchronized (lock1) {
  try {
    System.out.println("当前线程 : "  + Thread.currentThread().getName() + " 进入lock1执行");
    Thread.sleep(2000);
  } catch (InterruptedException e) {
    e.printStackTrace();
  }
  synchronized (lock2) { //需要lock2的资源
    System.out.println("当前线程 : "  + Thread.currentThread().getName() + " 进入lock2执行");
  }
}
//-------------- t2线程 ---------
synchronized (lock2) {
  try {
    System.out.println("当前线程 : "  + Thread.currentThread().getName() + " 进入lock2执行");
    Thread.sleep(2000);
  } catch (InterruptedException e) {
    e.printStackTrace();
  }
  synchronized (lock1) {//需要lock1的资源
    System.out.println("当前线程 : "  + Thread.currentThread().getName() + " 进入lock1执行");
  }
}

```



### 脏读问题

* 对于同一个变量的读和写操作，为了保证业务的原子性，需要使用完整的 synchronized， 就是在读和写上都加上synchronized, 避免脏读现象.

    ```java
    public class SynchronizedInteger{
        
        private int value;
        
        public synchronized int get(){
            return value;
        }
        
        public synchronized void set(int value){
            this.value = value;
        }
    }
    ```

* **非原子的64位操作**
>   * &emsp;非 volatile 类型的64位数值变量（double 和 long)。Java 内存模型要求，变量的读取操作和写入操作都必须是原子操作，但对于非volatile类型的double 和 long 变量， JVM 允许将64位的读操作和写操作分解成两个32位操作。 当读取一个非 volatile 类型的long变量时, 如果对该变量的读操作和写操作在不同的线程中执行，那么**很可能会读取到某个值高32位和另一个值得低32位**。因此，即使不考虑失效数据的问题，在多线程程序中使用共享且可变的long和double等类型的变量也是不安全的，除非用关键字volatile来声明他们，或者用锁保护起来。



