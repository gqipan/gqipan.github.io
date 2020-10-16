---
layout: post
title:  "2.行为型模式-策略模式"
categories: "Design-Patterns"
tags: "Strategy-Design-Patterns"
---


* content
{:toc}


行为型模式-策略模式
---

#### 模式动机
* 完成一项任务，往往可以有多种不同的方式，每一种方式称为一个策略，我们可以根据环境或者条件的不同选择不同的策略来完成该项任务。
* 在软件开发中也常常遇到类似的情况，实现某一个功能有多个途径，此时可以使用一种设计模式来使得系统可以灵活地选择解决途径，也能够方便地增加新的解决途径。
* 在软件系统中，有许多算法可以实现某一功能，如查找、排序等，一种常用的方法是硬编码(Hard Coding)在一个类中，如需要提供多种查找算法，可以将这些算法写到一个类中，在该类中提供多个方法，每一个方法对应一个具体的查找算法；当然也可以将这些查找算法封装在一个统一的方法中，通过if…else…等条件判断语句来进行选择。这两种实现方法我们都可以称之为硬编码，如果需要增加一种新的查找算法，需要修改封装算法类的源代码；更换查找算法，也需要修改客户端调用代码。在这个算法类中封装了大量查找算法，该类代码将较复杂，维护较为困难。
* 除了提供专门的查找算法类之外，还可以在客户端程序中直接包含算法代码，这种做法更不可取，将导致客户端程序庞大而且难以维护，如果存在大量可供选择的算法时问题将变得更加严重。
* 为了解决这些问题，可以定义一些**独立的类来封装不同的算法**，**每一个类封装一个具体的算法**，在这里，**每一个封装算法的类**我们都可以称之为**策略(Strategy)**，为了保证这些策略的一致性，一般会用一个抽象的策略类来做算法的定义，而具体每种算法则对应于一个具体策略类。

#### 模式定义：
* **策略模式(Strategy Pattern)**：定义一系列算法，将**每一个算法封装起来**，并让**它们可以相互替换**。**策略模式让算法独立于使用它的客户而变化**，也称为政策模式(Policy)。
* 策略模式是一种**对象行为型模式**。

#### 模式结构

* Duck 类： 鸭子抽象类
	* ModelDuck： 模型鸭
* QueakBehavior:  抽取的鸭子叫声的策略接口
	* Squeak： 策略的具体实现（吱吱叫）
	* Quack：策略的具体实现（呱呱叫）
	* MuteQueak: 策略的具体实现（不会叫）
* FlyBehavior：抽取的飞行的策略接口
	* FlyNoWay: 策略的具体实现（不会飞）
	* FlyWithWings: 策略的具体实现(飞)

* ![](http://ww1.sinaimg.cn/large/afac410dgy1fg9l4e60nyj20yg0cvmzo.jpg)

#### 代码

```java
public abstract class Duck {

    private QuackBehavior quackBehavior;
    private FlyBehavior flyBehavior;
    public void setFlyBehavior(FlyBehavior flyBehavior) {
        this.flyBehavior = flyBehavior;
    }
    public void setQuackBehavior(QuackBehavior quackBehavior) {
        this.quackBehavior = quackBehavior;
    }
    public void preFormFly(){
        flyBehavior.fly();
    }
    public void preFormQuack(){
        quackBehavior.quack();
    }
    public abstract void display();
    public void swim(){
        System.out.println("All Ducks float, event decoys");
    }
}
```

```java
   @Test
   public void testMiniDuckSimulator(){
       Duck duck = new ModelDuck(new FlyWithWings(), new MuteQuack());

       duck.preFormFly();
       duck.preFormQuack();

       duck.setFlyBehavior(new FlyNoWay());
       duck.setQuackBehavior(new Squeak());

       duck.preFormFly();
       duck.preFormQuack();

   }
```

#### 涉及的设计原则

##### 设计原则一：

* 找出应用中可能需要变化之处，把他们独立出来，不要和那些不需要变化的代码混在一起

##### 设计原则二：

* 针对接口编程，而不是针对实现编程

##### 设计原则三：

* 多用组合，少用继承
* 有一个(Has - a) 比 （Is - a）更好

#### 优点

策略模式的优点

* 策略模式提供了对“开闭原则”的完美支持，用户可以在不修改原有系统的基础上选择算法或行为，也可以灵活地增加新的算法或行为。
* 策略模式提供了管理相关的算法族的办法。
* 策略模式提供了可以替换继承关系的办法。
* 使用策略模式可以避免使用多重条件转移语句。


#### 缺点

 策略模式的缺点
* 客户端必须知道所有的策略类，并自行决定使用哪一个策略类。
* 策略模式将造成产生很多策略类，可以通过使用享元模式在一定程度上减少对象的数量。

#### 适用环境

在以下情况下可以使用策略模式：

* 如果在一个系统里面有许多类，它们之间的区别仅在于它们的行为，那么使用策略模式可以动态地让一个对象在许多行为中选择一种行为。
* 一个系统需要动态地在几种算法中选择一种。
* 如果一个对象有很多的行为，如果不用恰当的模式，这些行为就只好使用多重的条件选择语句来实现。
* 不希望客户端知道复杂的、与算法相关的数据结构，在具体策略类中封装算法和相关的数据结构，提高算法的保密性与安全性。
