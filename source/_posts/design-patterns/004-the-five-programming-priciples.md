---
title: 面向对象设计的SOLID原则
date: 2017-06-19 20:32:03
tags:
categories: "设计模式"
---

S.O.L.I.D是面向对象设计和编程(OOD&OOP)中几个重要编码原则(Programming Priciple)的首字母缩写。

* **SRP** - The Single Responsibility Principle（单一责任原则）
* **OCP** -	The Open Closed Principle（开放封闭原则）
* **LSP** -	The Liskov Substitution Principle（里氏替换原则）
* **DIP** -	The Dependency Inversion Principle（依赖倒置原则）
* **ISP** -	The Interface Segregation Principle（接口分离原则）

### 单一责任原则：

当需要修改某个类的时候原因有且只有一个（THERE SHOULD NEVER BE MORE THAN ONE REASON FOR A CLASS TO CHANGE）。换句话说就是让一个类只做一种类型责任，当这个类需要承当其他类型的责任的时候，就需要分解这个类。

例如，Modem可以链接dial(拨号连接)，hangup(挂断拨号)和send(发送数据)，recv(接收数据).

```java
interface Modem {
      public void dial(String pno);
      public void hangup();
      public void send(char c);
      public char recv();
}
```

Modem类有两个职责,链接管理和数据通信,应该将它们分离.一个类中可以有多个方法,但是一个类只干一件事:

![](/images/categories/design-patterns/004/01.png)

<!--more-->


### 开放封闭原则

软件实体应该是可扩展，而不可修改的。也就是说，对扩展是开放的，而对修改是封闭的。这个原则是诸多面向对象编程原则中最抽象、最难理解的一个。

* 抽象原则

把系统的所有可能的行为抽象成一个底层,由于可从抽象层导出多个具体类来改变系统行为,因此对于可变部分系统设计对扩展是开放的.

* 可变性封装原则

对系统所有可能发生变化的部分进行评估和分类,每个可变的因素都单独进行封装.

比如，计算器的例子,原来客户端只有加法运算,现在需要添加新的功能减法,此时如果你去修改原来类就会违背"开放-封闭原则",而且当添加很多新的运算时,这样的代码很难维护.

此时你需要重构程序,增加一个抽象类的运算类,通过继承或多态等隔离具体加法、减法与client耦合,面对需求变化,对程序的改动是通过增加新代码进行的,而不是更改现有的代码,这就是开闭原则的精髓.

![](/images/categories/design-patterns/004/02.png)

### 里氏替换原则

当一个子类的实例应该能够替换任何其超类的实例时，它们之间才具有is-A关系。

比如，动物的例子,动物具有吃、喝、跑、叫等行为,如果需要其他动物也有类似行为,由于它们都继承于动物,只需要更改实例化的地方,程序其他不许改变.如下图所示：

![](/images/categories/design-patterns/004/03.png)

### 依赖倒置原则

1. 高层模块不应该依赖于低层模块，二者都应该依赖于抽象

2. 抽象不应该依赖于细节，细节应该依赖于抽象

下图表示司机驾驶奔驰车的类图.

![](/images/categories/design-patterns/004/04.png)

代码如下所示,司机通过调用奔驰车run方法开动汽车,通过client场景描述eastmount开奔驰车

```java
//司机类
public class Driver {
	//司机主要职责驾驶汽车
	public void drive(Benz benz) {
		benz.run();
	}
}
//奔驰类
public class Benz {
	public void run() {
		System.out.printIn("奔驰汽车跑");
	}
}
//场景类
public class Client {
	public static void main(String[] args) {
		//eastmount开奔驰车
		Driver eastmount = new Driver();
		Benz benz = new Benz();
		eastmount.drive(benz);
	}
}
```

现在司机eastmount不仅要开奔驰车,还要开宝马车,又该怎么实现呢?自定义宝马汽车类BMW,但是不能开,因为eastmount没有开宝马车的方法,出现的问题就是：

司机类和奔驰车类之间是一个紧耦合关系,这导致了系统的可维护性大大降低,增加新的车类如宝马BWM汽车就需要修改司机类drive()方法,这不是稳定性而是易变的.被依赖者的变化竟然让依赖者来承担修改的代价,所以可以通过依赖倒置原则实现.

如下图所示,通过建立两个接口IDriver和ICar,分别定义了司机的职能就是驾驶汽车,通过drive()方法实现;汽车的职能就是运行,通过run()方法实现.

![](/images/categories/design-patterns/004/05.png)

代码如下,接口只是一个抽象化的概念,是对一类事物的抽象描述,具体的实现代码由相应的类来完成,所以还需要定义Driver实现类;同时在IDriver接口中通过传入ICar接口实现抽象之间的依赖关系,Driver实现类也传入了ICar接口,具体开的是哪种车需要在高层模型中声明,具体开发方法在BMW和Benz类中定义.

```java
//司机接口 驾驶汽车 抽象
public interface IDriver {
	public void drive(ICar car);
}
//司机类 具体实现
public class Driver implements IDriver{
	//司机的主要职责就是驾驶汽车
	public void drive(ICar car){
		car.run();
	}
}
//汽车接口
public interface ICar {
	public void run();
}
//奔驰车类
public class Benz implements ICar{
	public void run(){
		System.out.println("奔驰汽车跑");
	}
}
//宝马车类
public class BMW implements ICar{
	public void run(){
		System.out.println("宝马汽车跑");
	}
}
//实现eastmount开宝马车
public class Client {
	public static void main(String[] args) {
		IDriver eastmount = new Driver();
		ICar bmw = new BMW();
		eastmount.drive(bmw);
	}
}
```

由于"抽象不应该依赖细节",所以我们认为抽象(ICar接口)不依赖BMW和Benz两个实现类(细节),在高层次的模块中应用都是抽象,Client就属于高层次业务逻辑,它对于低层次模块的依赖都是建立在抽象上,eastmount都是以IDriver接口类型进行操作,屏蔽了细节对抽象的影响,现在开不同类型的车,只需要修改业务场景类即可实现:

```java
ICar benz = new Benz();
eastmount.drive(benz);
```

依赖倒置其实可以说是面向对象是设计的标志,用哪种语言来编写程序不重要,如果编写时考虑的都是如何针对抽象编程而不是针对细节编程,即程序中所有的依赖关系都是终止与抽象类或接口,那就是面向对象的设计,反之则是过程化的设计.

### 接口分离原则

不能强迫用户去依赖那些他们不使用的接口。换句话说，使用多个专门的接口比使用单一的总接口总要好。

如下图所示,类A依赖接口I中方法1,方法2,方法3,类B是对类A依赖的实现,类C依赖于接口I中的方法方法1,方法4,方法5,类D是对类C依赖的实现.对于类B和类D中都存在用不到的方法,但是由于实现了接口I,所以需要实现这些不用的方法.

![](/images/categories/design-patterns/004/06.png)

可以发现接口过于臃肿,只要接口中出现的方法,不管对依赖于它的类有没有用,实现类中都必须去实现这些方法,如类B中方法4和方法5,类D中方法2和方法3,显然这样的设计不适用.

设计如下修改为符合接口隔离原则,必须对接口I进行拆分,拆分为三个接口I1,I2,I3.此时实现类B和类D中方法都是需要使用的方法,这就体现了建立单一接口而不是庞大接口的好处.

![](/images/categories/design-patterns/004/07.png)

接口隔离原则即尽量细化接口,接口中的方法尽量少,为各个类建立专用的接口,而不是试图去建立一个庞大而臃肿的接口提供所有依赖于他的类去调用.

说到这里,很多人会觉的接口隔离原则跟之前的单一职责原则很相似,其实不然。

其一,单一职责原则原注重的是职责,而接口隔离原则注重对接口依赖的隔离.

其二,单一职责原则主要是约束类,其次才是接口和方法,它针对的是程序中的实现和细节;而接口隔离原则主要约束接口,主要针对抽象,针对程序整体框架的构建.
