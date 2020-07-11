---
layout: post
title: Java Concurrency
tags: Java
---
## 线程基础
### 常见线程安全
- **无状态对象**一定线程安全。即不包含任何域，也不包含任何对其他类中域的引用。线程栈上局部变量不影响线程安全
- **不可变对象**一定线程安全
- **线程封闭（对象封闭在一个线程中）**一定线程安全
- 委托给**线程安全类**

### 常见线程不安全
- 竞态条件常见类型为**先检查后执行**，使用先检查后执行的常见情况为**延迟初始化**
    1. 安全的延迟初始化
    >public synchronized static Resource getInstance(){ if(resource==null) resource==new Resource)(); return resource; }
    2. 提前初始化：不要延迟初始化
    >private static Resource resource = new Resource();
    3. 延迟初始化占位类模式：直到开始使用（静态或非静态）内部类时才初始化，静态内部类不依靠外部类
    >private static class ResourceHolder{public static Resource resource = new Resource();}
    >public static Resource getResource() { return ResourceHolder.resource; }
- 需要在**单个原子操作**中更新**所有**相关的状态变量

### 常见关键词
- 内置锁**synchronized**，可重入（父子类synchronized修饰方法，子类调用super.doSomething()不会死锁）
	1. synchronized(this)和synchronized修饰普通方法，获得对象锁，一个对象仅一把锁。a线程访问synchronized修饰的b方法，c线程无法访问该对象的synchronized(this)或synchronized修饰普通方法
	2. synchronized(A.class)和synchronized修饰静态方法，获得类锁，一个类仅一把锁，**与对象锁无关**。
- 定时的锁等待、可中断的锁等待、公平性ReentrantLock
- 可见性，防止重排序的**volatile**。典型用法，检查状态标记是否退出循环
```
volatile boolean asleep;
while(!asleep) 
    countSomeSheep();
```
### 发布与逸出
- 发布，对象能在当前域之外代码使用；逸出，不该被发布的对象被发布
    - public static延迟初始化
    - get private对象
    - 发布内部类实例。this引用逸出的条件是**构造函数中发布this显式或隐式（内部类）引用**。常见错误为构造函数中增加监听器或启动线程

- 安全发布，对象引用及对象状态同时对其他线程可见
    - 静态初始化函数中初始化对象引用`public static Holder holder = new Holder(42);`
    - 对象引用保存在volatile类型的域或者AtomicReferance对象中
    - 对象引用保存在某个正确构造对象的final类型域中
    - 对象引用保存在由锁保护的域中 

## 常见类

### 工具类

- 闭锁
CountDownLatch。闭锁结束前，门一直关闭；闭锁结束时，允许所有线程通过；闭锁结束后，不会改变状态<br>
FuturnTask。三个状态：等待运行、正在运行、运行完成。Future.get方法会阻塞直到运行完成

- 信号量
Semaphore。acquire和release方法来获得和释放信号量

- 栅栏
CyclicBarrier。所有线程到达栅栏位置，才能继续执行<br>
Exchanger。两方栅栏，栅栏位置交换数据

### 构建高效可伸缩结果缓存
先get缓存，为空则计算，后put缓存
1. HashMap用synchronized，太慢
2. HashMap换为ConcurrentHashMap，计算开销很大时，会重复计算
3. ConcurrentHashMap中值类型为Future，判断时会检查某个相应的计算是否已经**开始**（上面判断为计算是否已经**完成**），if代码块非原子，两个线程可能同一时间计算
4. Map的put方法换为ConcurrentMap的原子方法putIfAbsent
```
Future f = cache.get(arg);
if(f==null){
	FutureTask ft = new FutureTask(V)(eval);
	Callable eval = new Callable(){...}
    // 3.没有值则添加值、返回null; 有值则覆盖、返回值
    f = ft;
    cache.put(arg, ft);
    ft.run(); //真正运行

    // 4.没有值则添加值、返回null; 有值则不变、返回值
    f = cache.putIfAbsent(arg, ft);
    if(f == null) { f = ft; ft.run(); }
}

// 任务被取消CancellationException，任务异常封装为ExecutionException，getCause获取原始异常
try {return f.get();}
catch(CancellationException e) {cache.remove(arg, f);}
catch(ExecutionException e) {throw launderThrowable(e.getCause());}
```

### 线程池
正确shutdown Executor，JVM才能关闭
- newCachedThreadPool，线程池规模不受限
- newSingleThreadExecutor单个线程，线程异常，会新建另一个线程
- newScheduledThreadPool相比于Timer，支持相对时间调度、Timer只有一个线程、TimerTask抛出未检查的异常时Timer线程会终止


## 案例展示
### 渲染文本与下载图片并发
CPU密集和IO密集任务区分开
1. 用线程池，使用future = executor.submit(eval)下载任务开始，主线程渲染文本后，调用future.get()得到下载的图片。此处所有图片下载均在一个任务中，即eval
2. ExecutorCompletionService，会新建对象；计算完成时，调用FutureTask的done方法，结果存入BlockingQueue，使用completionService.take()方法获得Future对象。此处每张图片下载在一个任务中，future.get()前会先take()到已经执行完毕的future任务。

```
// 为每个任务新建一个Future对象
for(final ImageInfo image:info) {
    completionService.submit(new Callable<ImageInfo>(){...})
}
for(int i=0; i<info.size(); i++) {
    Future f = ompletionService.take();
    ImageInfo image = f.get();
}
```

### 设置超时
1. future.get()可以设置超时时间，抛出TimeoutException
2. executor.invokeAll()可以设置超时时间并行执行多个Callable任务，返回为future列表

### 递归算法并行化
搬箱子框架，迭代操作都是独立的，不需要等待所有迭代操作都完成再继续执行
```
public interface Puzzle<P, M> {
    P initialPosition();
    boolean isGoal(P p);
    Set<M> leagalMoves(P p);
    P move(P p, M m);
}
static class Node{
    final P p;
    final M m;
    final Node<P, M> prev;// 前一个Node
    Node(P p, M m, Node prev){...}
    List<M> asMoveList() {
        List<M> solution = new LinkedList<>();
        for(Node n = this; n.move != null; n = n.prev){
            solution.add(0, n.move);
        }
        return solution;
    }
}
```
1. 串行：判断点是否已遍历过（HashSet储存），已遍历返回null；否则添加到遍历Set，判断是否为goal，是则返回node.asMoveList()；否则for循环当前node的所有合法move，循环中构建childNode(p,m,node)，执行result = search(childNode)，判断result不为null，则返回
2. 并行：
为解决没有答案的情况可以 done.await()限时；或者使用AtomicInteger计任务数新建CountingSolverTask继承SolverTask，其构造方法中taskCount.incrementAndGet()，run结束finally调用if(taskCount.decrementAndGet() == 0){solution.setValue(null);}
```
final ValueLatch<Node<P,M>> solution = new ValueLatch<>;
private fianl ConcurrentMap<P, Boolean> seen;
public List<M> solve() throws InterruptedException {
    try {
        P p = puzzle.initialPosition();
        exec.execute(new SolverTask(p, null, null));
        //ValueLatch类中会使主线程阻塞直到找到解答
        Node solvNode = solution.getValue();
        return solvNode == null ? null : solvNode.asMoveList();
    } finally {
        exec.shutdown();
    }
    
    class SolverTask extends Node implements Runable {
        ...
        public void run(){
            //找到解答或已经遍历
            if (solution.isSet() || seen.putIfAbsent(p, true) != null)
                return;
            if (puzzle.isGoal(p))
                solution.setValue(this);
            else
                for(M m : puzzle.legalMoves(p))
                    exec.execute(new SolverTask(puzzle.move(p,m), m, this))
        }
    }
    class ValueLatch<T> {
        private T val = null;
        private fianl CountDownLatch done = new CountDownLatch(1);
        public boolean isSet() { return done.getCount==0;}
        public synchronized void setValue(T newVal) {
            if(!isSet()) {
                val = newVal; done.countDown();
            }
        }
        public T getValue() throw InterruptedException {
            //阻塞
            done.await();
            synchronized(this) { return val; }
        }
    }
}
```

## 死锁
### 死锁产生
1. 锁顺序死锁
```
public void leftRight(){
    synchronized(left){synchronized(right){doSomething();}}
}
public void rightLeft(){
    synchronized(right){synchronized(left){doSomethingElse();}}
}
```
2. 动态锁顺序死锁
transferMoney(myAccount, yourAccount)
transferMoney(yourAccount, myAccount)
```
public void transferMoney(Account fromAccount, Account toAccount){
    synchronized(fromAccount){synchronized(toAccount){doSomething();}}
}
```
3. 协作对象间死锁
setLocation先获取Taxi锁，后获得Dispatcher锁；getImage先获得Dispatcher锁，后获得Taxi锁
```
class Taxi{
    ...
    public synchronized void getLocation(Point location) {
        return location;
    }
    public synchronized void setLocation(Point location) {
        this.location = location;
        if(location.equals(destination)){
            dispatcher.notifyAvailable(this);
        }
    }
}
class Dispatcher{
    private final Set<Taxi> availableTaxis;
    public synchronized void notifyAvailable(Taxi taxi){
        availableTaxis.add(taxi);
    }
    public synchronized Image getImage(){
        Image image = new Image();
        for(Taxi t: taxis){ image.drawMarker(t.getLocation()); }
        return image;
    }
}
```
4. 资源死锁
一个任务不同顺序，连接两个数据库。线程A持有数据库a的连接，等待数据库b的连接；线程B持有数据库b的连接，等待数据库a的连接<br>
一个任务提交另一个任务，在单线程的Executor中执行。第一个任务将永远等待，其他任务都将停止
### 避免死锁
1. 规定加载顺序。极少数情况两对象相同散列值，使用加时赛锁（tieLock），保证每次只有一个线程以未知顺序获得两个锁
```
private static final Object tieLock = new Object();
public void tranferMoney(){
    int fromHash = System.identityHashCode(fromAcct);
    int toHash = System.identityHashCode(toAcct);
    if(fromHash < toHash) {
        synchronized(fromAcct){synchronized(toAcct){doSomething;}}
    } else if (fromHash > toHash) {
        synchronized(toAcct){synchronized(fromAcct){doSomething;}}
    } else {
        synchronized(tieLock){
            synchronized(fromAcct){synchronized(toAcct){doSomething;}}
        }
    }
}
```
2. 开放调用，即调用某个方法时不需要持有锁，但可以在方法内使用更细粒度的代码块
```
class Taxi{
    ...
    public void setLocation(Point location) {
        boolean reachedDestination;
        synchronized(this) {
            this.location = location;
            reachedDestination = location.equals(destination);
        }
        if(reachedDestination){
            // notifyAvailable获取Dispatcher锁，但并没有嵌套在Taxi锁内
            dispatcher.notifyAvailable(this);
        }
    }
}
class Dispatcher{
    ...
    public Image getImage(){
        Set<Taxi> copy;
        synchronized(this) {
            copy = new HashSet<Taxi>(this);
        }
        Image image = new Image();
        // getLocation获取Taxi锁，但并没有嵌套在Dispatcher锁内
        for(Taxi t: taxis){ image.drawMarker(t.getLocation()); }
        return image;
    }
}
```