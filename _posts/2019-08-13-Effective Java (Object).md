---
layout: post
title: Effective Java (Object)
tags: Java
---

## 创建和销毁对象

### 考虑静态工厂方法代替构造器

- 静态工厂有**名称**
- **不必**每次调用**创建新对象**
- 可以返回原**返回类型的子类对象**
- 返回对象的类根据参数值变化
- 返回对象的类在静态工厂方法中不存在，用其实现的接口作为返回值 

```
Map<String, String> map = new HashMap<String, String>;
public static <K, V> HashMap<K, V> newInstance(){
	return new HashMap<K, V>();
}
Map<String, String> map = HashMap.newInstance();
```

- **无公有或者受保护**的构造器，**不能被子类化**
- API文档未标出，查明如何实例化类较难

惯用名称：

- valueOf/of 同样值，转换类型用
- getInstance 根据参数返回实例，或者单例模式
- newInstance 每个实例与其他所有实例不同

### 多个必要和可选域时考虑构建器

- 重叠构造器**可读性差**
	- 必要参数
	- 必要参数 + 一个可选参数
	- 必要参数 + 两个可选参数 
	- ……
- JavaBeans模式**安全性差**
	- 无参构造器
	- JavaBeans可能不一致状态，不能成为不可变类（线程安全需要考虑）
- 构造器模式
	- Builder类为外部类的静态成员类，Builder可以访问外部类的private带Builder参数的构造方法，用this为参数将Builder中的值赋予外部类private final修饰的属性 

```
A a = new A.Builder(1,1).optional1(1).optional2(1).build();

public class A {
	private final int req1;
	private final int req2;
	private final int opt1;
	private final int opt2;

	pubilc static class Builder {
		private final int req1;
		private final int req2;
		private int opt1 = 0;
		private int opt2 = 0;

		public Builder(int req1, int req2){
			this.req1 = req1;
			this.req2 = req2;
		}
		public Builder optional1(int val){
			opt1 = val;
			return this;
		}
		public Builder optional2(int val){
			opt2 = val;
			return this;
		}

		public A build(){
			return new A(this);
		}
		private A(Builder builer){
			req1 = builder.req1;
			req2 = builder.req2;
			opt1 = builder.opt1;
			opt2 = builder.opt2;
		}
	}
}
```

### 单例模式

```
public enum Elvis{
	INSTANCE;
	public void leaveTheBuilding(){ ... }
}
```

### 私有构造器防止实例化

- 抽象类的子类可以被实例化
- 私有构造器的防止实例化
- 私有构造器的类无法子类化

### 依赖注入

- 通过构造器，静态工厂，构建器注入
- 注入的对象资源不可变

### 不建立不必要对象

- 对象成本高。比如判断一个人出生是否在1946-1964年，1946和1964年的对象只用创建一次，可以放在static块中，仅仅在类初始化时创建一次实例；String.matches方法会自动创建Pattern对象
- 优先使用基本类型而非装箱基本类型。
- 适配器针对给定后备对象，不需要创建多个适配器实例

### 对象引用造成内存泄漏

- Stack类自己管理内存，存储池包含了elements数组的元素
- 缓存
- 监听器和其他回调

### try-with-resources优先于try-finally

```
try{
    br.readLine();
}
finally{
    br.close();
}
```
close报出的异常抹除了readLine的异常信息，Java7引入AutoCloseable接口
```
try(InputStream in = new InputStream(src);
    OutputStream out = new OutputStream(src)){
    in.read();
    out.write();
}
```
自动关闭，保留第一个异常；查看全部异常在Java7接口Throwable中getSuppressed方法