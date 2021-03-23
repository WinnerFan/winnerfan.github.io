---
layout: post
title: Thread Class
tags: Java
---
## Thread Class
### 背景
@Async的坑
1. 为每一个任务启动一个线程
2. 默认线程数不受限制
3. 不复用线程

### 线程池基础
#### 区分Executors，ThreadPoolExecutor和ThreadPoolTaskExecutor
![avatar](/assets/images/2021-02-02-Thread.png) 
1. Executors

工具类，相当于一个工厂类，实际调用ThreadPoolExecutor创建线程池。

- newSingleThreadExecutor：堆积的请求处理队列占用内存很多
- newFixedThreadPool：堆积的请求处理队列占用内存很多
- newCachedThreadPool：线程数无上限
- newScheduledThreadPool：线程数无上限，延迟或定时任务

2. ThreadPoolExecutor

JDK中的类，ExecutorService为线程池接口，ScheduledExceutorService 线程调度接口

3. ThreadPoolTaskExecutor

Spring包下的类，实质是ThreadPoolExecutor的包装，通过XML自动注入或者配置类注入

- 方式一：
```
<bean id="taskExecutor" class="org.springframework.scheduling.concurrent.ThreadPoolTaskExecutor">
  <property name="corePoolSize" value="10"/>
  <property name="maxPoolSize" value="200"/>
  <property name="queueCapacity" value="10"/>
  <property name="keepAliveSeconds" value="20"/>
  <property name="rejectedExecutionHandler">
    <bean class="java.util.concurrent.ThreadPoolExecutor$CallerRunsPolicy"/>
  </property>
</bean>


@Resource(name="taskExecutor")
ThreadPoolTaskExecutor taskExecutor;
// 或者可以直接@Autowried
@AutoWired
ThreadPoolTaskExecutor taskExecutor
```

- 方式二：
```
@Configuration
public class ... {
    @Bean("taskExecutor")
    public Executor taskExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        //详见后文
    }
}
```

#### 处理流程
- 先核心线程，后入缓冲队列，最后最大线程
- 线程池中线程数大于核心线程数，在缓冲队列中的线程空闲时间超过设置，线程被销毁
- execute没有返回值，submit有返回值，通过future.get获取
- 拒绝策略
    -  AbortPolicy，默认，抛出异常可以捕获
    -  CallerRunPolicy，回退给调用者
    -  DiscardPolicy，抛弃任务
    -  DiscardOldPolicy，抛弃最旧任务 

### @Async
#### @Async使用
```
<task:annotation-driven executor="threadPool"/>
<task:executor id="threadPool" pool-size="5-10" queue-capacity="10" keep-alive="5" reject-policy="DISCARD_OLDEST"/>
```
@Async的默认线程池为SimpleAsyncTaskExecutor，这个类不重用线程，默认每次调用都会创建一个新的线程。

- 修饰无返回值方法，异常会被AsyncUncaughtExceptionHandler处理掉，若需要抛出异常，可以手动new一个异常抛出
- 修饰有返回值方法，返回值是Future，不会被捕获，需要`try-catch`处理异常

#### @Async自定义线程池

1. 重新实现AsyncConfigurer接口，有且只有一个
```
@Component
public class Task {
    @Async("taskExecutor")
    public void doSomething() {
        ...
    }
}
```

```
@EnableAsync
@Configuration
public class MyConfigurer implements AsyncConfigurer {
    @Bean("taskExecutor")
    public Executor taskExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(10);
        executor.setMaxPoolSize(20);
        executor.setQueueCapacity(200);
        executor.setKeepAliveSeconds(60);
        executor.setThreadNamePrefix("taskExecutor-");
        executor.setRejectedExecutionHandler(new ThreadPoolExecutor.CallerRunsPolicy());
        executor.initialize();
        return executor;
    }
    @Override
    public Executor getAsyncExecutor() {
        return taskExecutor();
    }
    @Override
    public AsyncUncaughtExceptionHandler getAsyncUncaughtExceptionHandler() {
        return null;
    }
}
```
2. 修改XML
```
<task:annotation-driven executor="threadPool"/>
<bean id="threadPool"  class="org.springframework.scheduling.concurrent.ThreadPoolTaskExecutor"/>
  <property name="corePoolSize" value="5"/>
  ...
  <property name="rejectedExecutionHandler">
    <bean class="java.util.concurrent.ThreadPoolExecutor$CallerRunsPolicy" />
  </property>
</bean>
```

3. 继承AsyncConfigurerSupport
```
@EnableAsync
@Configuration
class SpringAsyncConfigurer extends AsyncConfigurerSupport {  
    @Bean  
    public ThreadPoolTaskExecutor asyncExecutor() {  
        ThreadPoolTaskExecutor threadPool = new ThreadPoolTaskExecutor();
        ...
        return threadPool;  
    }  
  
    @Override  
    public Executor getAsyncExecutor() {  
        return asyncExecutor;  
}  

  @Override  
    public AsyncUncaughtExceptionHandler getAsyncUncaughtExceptionHandler() {
        return null;
    }
}
```

4. 配置自定义的TaskExecutor

- AsyncConfigurer的默认线程池在源码中为空
- Spring通过beanFactory.getBean(TaskExecutor.class)查看是否有线程池
- 未配置时，通过beanFactory.getBean(DEFAULT_TASK_EXECUTOR_BEAN_NAME, Executor.class)，查询名称为TaskExecutor的线程池

```
@EnableAsync
@Configuration
class TaskPoolConfig {  
    @Bean(name = AsyncExecutionAspectSupport.DEFAULT_TASK_EXECUTOR_BEAN_NAME) 
    public Executor taskExecutor() {  
        ThreadPoolTaskExecutor threadPool = new ThreadPoolTaskExecutor();
        ...
        return threadPool;  
    }  
```