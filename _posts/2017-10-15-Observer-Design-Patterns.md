---
layout: post
title:  "3.行为型模式-观察者模式"
categories: "Design-Patterns Behavioral-Design-Patterns"
tags: "Observer-Design-Patterns"
---

* content
{:toc}


行为模式-观察者模式
---


#### 1. 模式动机

* 建立一种对象与对象之间的依赖关系，一个对象发生改变时将自动通知其他对象，其他对象将相应做出反应。
* 在此，发生改变的对象称为观察目标，而被通知的对象称为观察者，一个观察目标可以对应多个观察者，而且这些观察者之间没有相互联系，可以根据需要增加和删除观察者，使得系统更易于扩展，这就是观察者模式的模式动机。


#### 2. 模式定义

* 观察者模式(Observer Pattern)：
    * 定义对象间的一种一对多依赖关系，使得每当一个对象状态发生改变时，其相关依赖对象皆得到通知并被自动更新。
    * 观察者模式又叫做**发布-订阅（Publish/Subscribe）模式**、**模型-视图（Model/View）模式**、**源-监听器（Source/Listener）模式**或从**属者（Dependents）模式**。

#### 模式结构

* Subject: 被观察的目标
    * WeatherData: 气象数据（目标实现类）
* Observer: 观察者
    * CurrentConditionDisplay: 观察者具体实现

* 模式无关：DisplayElent

![自定义模式结构](http://ww1.sinaimg.cn/large/afac410dgy1fgh57ug8ahj20q80eijs3.jpg)


* 使用继承 Observable 方式实现目标类
* 实现 Observer 接口来作为观察者

![使用Util工具类的方式](http://ww1.sinaimg.cn/large/afac410dgy1fgh5e0ke8pj20n90jjt9u.jpg)


#### 代码

```java
/**
 * Created by Pan on 2017/6/10.
 * 观察者模式中主题对象
 */
public interface Subject {

	/**
	 * 注册观察者对象
	 * @param observer
	 */
	void registerObserver(Observer observer);

	/**
	 * 注销观察者对象
	 * @param observer
	 */
	void removeObserver(Observer observer);

	/**
	 * 主题发生改变
	 * 通知所有观察者对象
	 */
	void notifyObserver();
}
```

```java
/**
 * Created by Pan on 2017/6/10.
 * 被观察的对象需要实现主题接口
 */
public class WeatherData implements Subject {

	/**
	 * 这个主题中注册的观察者集合
	 */
	private List<Observer> observers;
	private float temperature;
	private float humidity;
	private float pressure;

	public WeatherData(){
		observers = new ArrayList<>();
	}

	@Override
	public void registerObserver(Observer observer) {
		if (observer != null){
			observers.add(observer);
		}
	}
	
	@Override
	public void removeObserver(Observer observer) {
		if (observer == null){
			return;
		}
		observers.removeIf(t -> t == observer);
	}

	@Override
	public void notifyObserver() {
		observers.forEach(observer -> observer.update(temperature, humidity, pressure));
	}

	/**
	 * 从气象局中得到的更新值：
	 * 		得到更新值之后，主动通知各个注册的观察者
	 */
	public void measurementsChanged(){
		notifyObserver();
	}

	/**
	 * 用来测试气象站的类
	 * @param temperature
	 * @param humidity
	 * @param pressure
	 */
	public void setMeasurements(float temperature, float humidity, float pressure){
		this.temperature = temperature;
		this.humidity = humidity;
		this.pressure = pressure;
		measurementsChanged();
	}
}

```

```java
/**
 * Created by Pan on 2017/6/10.
 * 观察者对象的抽象方法：
 * 		所有观察者对象必须实现下面的方法
 */
public interface Observer {

	void update(float temp, float humidity, float pressure);
}
```

```java
/**
 * Created by Pan on 2017/6/10.
 * 具体的观察者，需要继承观察者接口
 */
public class CurrentConditionDisplay implements Observer, DisplayElement {

	private float temperature;
	private float humidity;
	private Subject weatherData;

	/**
	 * 表示需要针对哪一个主题进行观察:
	 * 并且注册到对应的主题中去
	 *
	 * @param weatherData
	 */
	public CurrentConditionDisplay(Subject weatherData) {
		this.weatherData = weatherData;
		weatherData.registerObserver(this);
	}

	@Override
	public void display() {
		System.out.println("Current Conditions: " + temperature + "F degrees and " + humidity + "% humidity");
	}

	@Override
	public void update(float temp, float humidity, float pressure) {
		this.temperature = temp;
		this.humidity = humidity;
		display();
	}
}
```

#### 涉及的设计原则

* 为了交互对象之间的松耦合设计而努力
* 找出程序中变化的方面，然后将其和固定不变的方面相分离
* 针对接口编程，不针对实现编程
* 多用组合，少用继承

#### 模式分析

* 观察者模式描述了如何建立对象与对象之间的依赖关系，如何构造满足这种需求的系统。
* 这一模式中的关键对象是观察目标和观察者，一个目标可以有任意数目的与之相依赖的观察者，一旦目标的状态发生改变，所有的观察者都将得到通知。
* 作为对这个通知的响应，每个观察者都将即时更新自己的状态，以与目标状态同步，这种交互也称为发布-订阅(publishsubscribe)。目标是通知的发布者，它发出通知时并不需要知道谁是它的观察者，可以有任意数目的观察者订阅它并接收通知

#### 优点

* 观察者模式的优点

    * 观察者模式可以实现表示层和数据逻辑层的分离，并定义了稳定的消息更新传递机制，抽象了更新接口，使得可以有各种各样不同的表示层作为具体观察者角色。
    * 观察者模式在观察目标和观察者之间建立一个抽象的耦合。
    * 观察者模式支持广播通信。
    * 观察者模式符合“开闭原则”的要求

#### 缺点

* 观察者模式的缺点

    * 如果一个观察目标对象有很多直接和间接的观察者的话，将所有的观察者都通知到会花费很多时间。
    * 如果在观察者和观察目标之间有循环依赖的话，观察目标会触发它们之间进行循环调用，可能导致系统崩溃。
    * 观察者模式没有相应的机制让观察者知道所观察的目标对象是怎么发生变化的，而仅仅只是知道观察目标发生了变化。

#### 适用环境

* 在以下情况下可以使用观察者模式：

    * 一个抽象模型有两个方面，其中一个方面依赖于另一个方面。将这些方面封装在独立的对象中使它们可以各自独立地改变和复用。
    * 一个对象的改变将导致其他一个或多个对象也发生改变，而不知道具体有多少对象将发生改变，可以降低对象之间的耦合度。
    * 一个对象必须通知其他对象，而并不知道这些对象是谁。
    * 需要在系统中创建一个触发链，A对象的行为将影响B对象，B对象的行为将影响C对象……，可以使用观察者模式创建一种链式触发机制。










