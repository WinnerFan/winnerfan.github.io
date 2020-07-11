---
layout: post
title: Agile Software Development
tags: Programming
---

## 原则

### 单一职责(SRP)

一个类而言，应该仅有一个引起它变化的原因。

- 错误示例：Rectangle类具有两个职责，一个是计算面积area()和一个是绘制打屏draw()
- 模糊实例：调制解调器接口第一个连接管理，第二个数据通信；取决于是否会重新编译，改变的原因为职责，**总是同时变化则不必分离**
- 正确实例：业务规则(多变)和持久化(稳定)不应混合

### 开放-封闭原则(OCP)

软件应该是可以扩展的，但是不可以修改。

- 扩展开放(需求改变)，更改封闭(不必更改源代码、二进制代码、DDL、.jar)。
- **封闭建立在抽象之上**
```
Shape s = shape; // Circle Rectangle
shape.draw();
// 顺序抽象体，抽象体定义了一个抽象接口，通过抽象接口可以表示任何可能的排序策略。一个排序策略代表两个对象可以推导出哪一个先绘制，返回true或false。数据驱动，使用表格，使得Shape派生类间互不知晓。
```
- **拒绝不成熟的抽象**，频繁变动再抽象

### 里氏替换原则(LSP)

任何基类出现的地方，子类一定可以出现。
- 错误示例：利用if else显式确定对象类型，运行时类别辨别(RTTI)
- 错误示例：Square继承Rectangle类重写方法后，Square编写违反了Rectangle的不变性，添加了Rectangle的束缚。g函数中Square对象行为方式和Rectangle**对象行为方式**不同，**IS-A关系是就行为方式而言的**
```
void g(Rectangle r){
	r.setWidth(5);
	r.setHight(4);
	assert(r.Area() == 20);
}
```
- **提取公共部分**替换继承
- 错误示例：子类退化函数，子类抛出父类未添加的异常

### 依赖倒置原则(DIP)

高层模块**不**应依赖底层模块，两者依赖抽象；抽象**不**应依赖细节，细节应依赖抽象

- 错误示例：未使用抽象的层次Console Service Dao，Console对于其下直到Dao改动敏感
- 正确示例：**高层使用接口，低层实现接口**，接口所有权倒置
- 依赖于抽象：变量不应该持有具体类引用，任何类不应该从具体类派生，任何方法不应覆盖基类已经实现方法

### 接口隔离原则(ISP)

接口不应过胖，而应该抽象基类聚合接口

- 错误示例：门增加定时报警，Door接口实现Timer接口，此时无需定时接口需要退化函数，违反LSP
- 使用委托分离接口，创建对象实现Timer接口，将对象赋给已经实现Door接口的TimedDoor
- 多重实现接口，TimedDoor实现Door和Timer接口，**优先**

## 设计模式

### 命令模式(Command)
```
interface Command { void do(); void undo(); }
public class Command1 implements Command {
    void do {System.out.println("open");}
    void undo {System.out.println("close");}
}
public class Command2 implements Command {
    void do {System.out.println("close");}
    void undo {System.out.println("open");}
}
// 传入Open或者Close实例
public class Deal{
    public static void deal(Command c) {
        c.do();
    }
}
```

### 模板方法模式(Template Method)和策略模式(Strategy)
```
public abstract class Template {
    public abstract void open();
    public abstract void close();
    public void doSomething() {System.out.println("do");}
    public void run() {
        open();
        doSomething();
        close();
    }
}
public class Deal extends Template {...}
```
继承，派生类不可避免的和基类绑定。**做不到**其他类想替换掉的doSomething，修改基类已经实现的方法，违反DIP。

```
interface FlyBehavior {
    void fly();
}
public class FlyWithWings implements FlyBehavior {...}
public class FlyNoWay implements FlyBehavior {...}
public class Animal {
    private FlyBehavior flyBehavior;
    setFlyBehavior(FlyBehavior flyBehavior) {...}
    performFly { flyBehavior.fly(); }
}
```
- 目标不同：策略模式不同算法做同一件事情；命令模式不同命令做不同事情
- 主体不同：策略模式主体是具体应用对象，实现程序的行为模板，例如排序器中的排序算法；命令模式主体是请求发送者和接收者，例如遥控器和空调

### 外观模式(Facade)和中介者模式(Mediator)
- 简化一群类的接口，整个系统提供一个接待员即可。
- 用一个中介对象封装一系列对象的交互，如Entity，POJO。

### 单例模式(Singleton)和单态模式(Monostate)
- 构造函数声明为private限制客户端程序对类的直接new创建实例化，并使用static（类属）的方式来保证类的对象单一。跨平台，适合用于任何类，可以创建子类单例；不能继承，不透明（使用者知道是个单例）
- 将它的构造函数声明为public，而将类中所有的字段声明为static（类属）；不限制个数，单状态只有一个。可以派生，透明性（和普通的类没区别）；不可跨平台，不能创建子类单态

### 空对象模式(Null Object)
```
Employee e = DB.getEmployee("Bob");
if(e != null && e.isTimeToPay(today)) {
    e.pay;
}
```
Employee作为接口，两个实现NullEmployee和EmployImp。前者方法中“什么也不做”，例如isTimeToPay返回false
```
Employee e = DB.getEmployee("Bob");
if(e.isTimeToPay(today)) {
    e.pay;
}
```
### 观察者模式(Observer)
```
public interface Subject {
    public void registerObserver(Observer o);
    public void removeObserver(Observer o);
    public notifyObserver();
}
public interface Observer {
    public update(double temp);
}
public class WeatherData implements Subject {
    private ArrayList observers;
    private double temperature;
    public void registerObserver(Observer o) {
        observers.add(o);
    }
    ...
    public notifyObserver() {
        for(int i=0; i<observers.size(); i++) {
            Observer o = (Observer) observers.get(i);
            o.update(temperature)
        }
    }
    public void setData(double temperature) {
        this.temperature = temperature;
        notifyObserver();
    }
}
public class CurrentConditionDisplay implements Observer {
    private Subject weatherData;
    private double temperature;
    public CurrentConditionDisplay(Subject weatherData){
        this.weatherData = weatherData;
        weatherData.registerObserver(this);
    }
    public void update(double temperature) {
        this.temperature = temperature;
        display();
    }
    ...
}
```
setData调用notifyObserver，调用已注册的观察者的update，传递参数并展示。

### 适配器模式(Adapter)
```
public interface Duck {
    public void quack();
}
public interface Turkey {
    public void gobble();
}
public class TurkeyAdapter implements Duck {
    Turkey turkey;
    public TurkeyAdapter (Turkey turkey) {
        this.turkey = turkey;
    }
    public void quack() {
        turkey.gobble();
    }
}
public class Test {
    public static void test(Duck duck) {
        duck.quack();
    }
}
```

### 装饰者模式
```
public abstract class Beverage {
    String description = "Unknown";
    public String getDescription() {
        return description;
    }
    public abstract double cost();
}
public abstract class CondimentDecorator {
    public abstract String getDescription();
}

public class Espresso extends Beverage() {
    public Espresso() {
        description ="Espresso";
    }
    public dobule cost() {
        return 1.99;
    }
}
public class Mocha extends CondimentDecorator {
    Beverage beverage;
    public Mocha (Beverage beverage) {
        this.beverage = beverage;
    }
    public String getDescription() {
        return beverage.getDescription() + ", Mocha";
    }
    public double cost() {
        return beverage.cost() + 0.20;
    }
}
```
### 状态模式
```
public class GumballMachine {
    State soldOutState;
    State soldState;
    State hasQuarterState;
    State noQuarterState;
    State state = soldOutState;
    int count = 0;
    public GumballMachine (int numberGumballs) {
        soldOutState = new SoldOutState(this);
        soldState = new SoldState(this);
        ...
        count = numberGumballs;
        if(numberGumballs > 0) {
            state = noQuarterState;
        }
    }
    public void insertQuarter() {
        state.insertQuarter();
    }
    ...
}
// 具有状态转换的每个动作
public interface State {
    public void insertQuarter();
    ...
}
// 状态转变在具体状态类中
public class NoQuarterState implements State {
    GumballMachine gumballMachine;
    public NoQuarterState(GumballMachine gumballMachine) {
        this.gumballMachine = gumballMachine;
    }
    public void insertQuarter() {
        System.out.println("You insert a quarter.");
        gumballMachine.setState(gumballMachine.getHasQuarterState);
    }
    ...
}
```

### 代理模式
为另一个对象提供一个替身或占位符以控制这个对象的访问。例如Icon接口，ImageProxy和ImageIcon实现，分别是代理和实体类。代理中创建实体类的引用，针对实体类是否为null给出**Proxy的处理或调用实体类的方法**

### 工厂模式
### 简单工厂模式
```
public interface Product{...}
public class ProductA implements Product{...}
public class ProductB implements Product{...}
public class Factory{
    public static Product create(String str){...}
}
```
不符合开放封闭原则，添加产品C，需要修改原来类Factory代码
### 工厂方法模式
```
public interface Factory{createProduct ...}
public class FactoryA implements Factory{new ProductA ...}
public class FactoryB implements Factory{new ProductB ...}
```
符合开放封闭原则，但是多个产品族，工厂过多
### 抽象工程模式
```
public interface Gift{...}
public class GiftA implements Gift{...}
public class GiftB implements Gift{...}
public interface Factory{createProduct;createGift;...}
public class FactoryA implements Factory{new ProductA;new GiftA;...}
public class FactoryB implements Factory{new ProductB;new GiftB;...}
```