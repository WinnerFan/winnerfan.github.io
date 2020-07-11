---
layout: post
title: Refactoring
tags: Java
---
## 重新组织函数
1. 提炼函数，小函数复用大
2. 简单的函数应该去除
3. 临时变量替换为函数本身
4. 查询替代临时变量
5. 复杂表达式放入final临时变量
6. 临时变量应该独立，避免`double tmp = 2*(x+y); tmp=x*y;`
7. 对参数不要赋值，对象可以set但不要修改引用
8. 函数修改为类
```
Class Account{ int gamma(int x){...}}
Class Gamma{
    private final Account _account;
    private int x
    Gamma(Account source, int xArg){
        _account=source;
        x=xArg;
    }
}
```

## 对象之间搬移特性
1. 根据相关性搬移函数、字段
2. 提炼类和内联化。Person；Persion，TelephoneNumber互转
3. 隐藏与添加委托。调用Person和Department类；调用Person，Person再调用Department

## 重新组织数据
1. public改为private，getter和setter
2. 对象取代数据
3. 单双向关联
4. 字面常量取代魔法数，例如3.14
5. 集合封装，提供add，remove和get只读副本（Collections.unmodifiableSet()）
6. 类，子类或State/Strategy取代类型码，运行期检验能力


## 简化条件表达式
1. 复杂的条件提炼为独立函数
2. break和return代替flag
3. 卫语句代替嵌套`if(){} if(){} ...`
4. 多态取代switch
5. 引入Null对象，继承自基类

## 简化函数调用

1. 函数名
2. 查询与修改函数分离，例如入侵系统的**判断入侵者和发送警报**分开
3. 明确函数与带参函数，例如加1，加2函数应**合并**为加x；根据参数值set应**分开**为明确函数
4. **传参使用对象**，而不要用对象get的值；或**不传参**，调用static方法
5. 工厂函数代替构造函数，派生子类过程中以工厂函数取代类型码
```
class Employee {
    static Employee create(String name) {
        return (Employee) Class.forName(name).newInstance();
    }
}
Employee eng = Employee.create("Engineer");
```
6. 异常替代错误码

## 处理继承
1. 字段、函数上下移，构造函数上移
2. 提炼子类、超类、接口
3. 模板方法模式
4. 委托取代继承，只需要超类中的一部分，子类中新建字段保存超类，调用超类方法；继承取代委托
5. 继承体系承担两个责任，例如交易处理子类，由被当作数据显示的父类。修改为两个继承，通过委托使其中一个调用另一个