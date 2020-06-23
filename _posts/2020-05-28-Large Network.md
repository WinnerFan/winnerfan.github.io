## 大型网站

### 架构变化
- 透明代理，发起方以为中间代理提供服务；服务方以为中间代理请求服务。增加网络开销，包括流量和延迟；代理出现问题，考虑热备
    - 硬件负载均衡，F5
    - 软件负载均衡，LVS
- 请求和处理间没有代理服务器，并行的有名称服务，与请求和服务端通信。其作用收集服务器地址，提供这些地址给请求方
- 并行的有规则服务器，只与请求通信，不与服务器通信
- Master节点分配给Worker

### 难点
- 全局时钟
- 故障独立性，一部分有问题也正常运行
- 单点故障
- 事务挑战

### Session问题

带有Cookie的Session到达上次Session的单机
- Session Stricky，负载均衡器根据Cookie进行请求转发，常用
- Session Replication，Server进行Session同步
- Session集中存储，Server从同一个地方读取Session，常用
- Cookie Based，Cookie传递Session数据

### 数据库读压力大
- 新建读库，与原库数据复制
- 利用搜索引擎，倒排表排序，建立索引。索引可以全量、增量，还可以实时、非实时划分
- 缓存，数据缓存和页面缓存
- 分布式存储，分布式文件系统、分布式KV系统、分布式数据库
- 垂直拆分，根据业务拆分；水平拆分，同一张表拆分

## 中间件基础

### 多线程编程
- synchronized：syn修饰普通方法或this，syn普通方法或this互斥；syn修饰静态方法或class，syn静态方法或class互斥。**两把锁**均不影响无syn修饰的方法
- ReentrantLock：tryLock方法，锁被其他线程持有，返回false；当前线程持有锁，返回true。公平锁。读写锁
- volatile：保证可见性，但是不能控制并发
- Atomics：AtomicInteger类代码简洁，性能提升。通过JNI方式使用硬件CAS
- wait、notify、notifyAll：在对象的synchronized(this)中
- CountDownLatch：线程达到预期或完成时，调用countDownLatch.countDown()，当此数目达到threadNum数时，唤醒countDownLatch.await()线程
```
final CountDownLatch countDownLatch = new CountDownLatch(threadNum);
```
- CyclicBarrier：协同多个线程，多个线程等待，直到多个线程都达到屏障，cyclicBarrier.await()数量达到threadNum数。可以循环使用
- Semaphore：管理信号量，构造传入可供管理的信号量数值，控制并发数量。semaphores.acquire()和semaphore.release()
- Exchanger：两个线程间交换数据，阻塞在exchange方法上，直到另一个线程调用exchange方法
- Future：调用函数后立马返回，异步调用耗时函数。isDone，get，cancel可能会失败

### 代理
- 静态代理：多个类代理，代理类功能实现相同，则需要为每个类都完成代理类
- 动态代理
```
public interface Calculator {
    int add (int a, int b);
}
public class CalculatorImpl implements Calculator {
    public int add(int a, int b) {
        return a + b;
    }
}
public class LogHandler implements InvocationHandler {
    public Object invoke(Object obj1, Method method, Object[] args) throws Throwable {
        Object o = method.invoke(obj, args);
        return o;
    }
    ...
}
public void testProxy() {
    Calculator c = new CalculatorImpl();
    LogHandler l = new LogHandler(c);
    Calculator proxy = (Calculator) Proxy.newProxyInstance(c.getClass().getClassLoader(), c.getClass().getInterfaces(), l);
    proxy.add(1, 1);
}
```
### 反射

## 数据库
### 事务处理
2PC
Paxos
### Sequence && AutoID
- 唯一性，UUID，使用种子(IP，MAC，时间，机器名)
- 连续性，机器->ID生成器->存储

### 跨库join
- 多次查找，先根据手机号查id，再根据id查相关商品总数
- 数据冗余，手机号，id，相关商品总数在一个表中
- 借助搜索引擎

### 读写分离
1. 主库从库非对称
- 数据结构相同，多从库对应一主库：根据分库规则，通过消息系统、数据同步服务器，基于被修改或新增的数据主键获取内容，采取行复制；基于数据库的日志，进行行复制
- 主备库分库方式不同：主库根据买家id分库，从库根据卖家id分库
- 引入数据变更平台：Extractor把数据源变更信息加入平台，平台由多条pipeline组成，Applier把变更应用到相应目标
2. 数据库平滑迁移
数据迁移不接受长时间停机，记录增量日志，迁移结束对增量处理（逐渐收敛）。最后可以停止源数据库中对于要迁移走的数据的写操作，保证增日志处理完毕，使得新库表数据最新，再切换分库规则，放开所有写

## 消息中间件
解决一致性的方法
1. 发送消息给中间件
2. 消息中间件入库
3. 消息中间件返回结果
4. 业务操作
5. 发送业务操作结果给消息中间件
6. 更改存储中消息状态

三种异常
1. 业务操作未进行，消息未入存储
2. 业务操作未进行，消息存入存储，状态为待处理
3. 业务操作成功，消息存入存储，状态为待处理

第2、3需要了解业务操作结果，定时重复
1. 消息中间件寻味待处理消息的操作结果
2. 检查操作结果
3. 发送给消息中间件操作结果
4. 更新消息状态

JMS连接方式
1. JMS Queue顺序入队，消息不确定被那个应用消费
2. JMS Topic顺序入队，消息被所有订阅的应用消费
由于集群的存在，考虑将Topic和Queue级联

订阅方式
1. 非持久订阅，应用重启后，不会执行停机期间的消息
2. 持久订阅，应用重启后，会执行停机期间的消息