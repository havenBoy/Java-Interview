# JDK线程池

> 关于java自己提供的线程池的介绍以及使用
> 在jdk1.4之前线程池的实现是很简陋的，在1.5以后加入了java.lang.concurrent包，主要是关于线程以及线程池的包

## 线程池的作用
- 主要是为了限制系统中线程的执行的数量
  * 线程池可以手动的创建以及销毁，可以指定线程池中线程的数量，为的是达到系统运行的最佳效果;
  * 如果当前的线程数大于线程池的最大size，那么新到来的线程会进入等候的状态，等待之前的线程退出线程池，如果当前的线程池中没有运行的线程，那么就直接进入线程池，如果没有线程进入到线程池，那么线程池处于等待的状态，直到有新的线程进入；
- 可以进行重复的利用，减少了创建与销毁线程的开销，提高系统的性能；
- 可以指定线程池的大小，如果线程池维持的线程的数量过多会造成系统的拥挤，如果线程池数量太少，又会浪费了系统的资源；

## ThreadPoolExcutor  具体实现类
- Executor -> ExecutorService -> AbstractExecutorService -> ThreadPoolExecutor
- 构造函数的解释
```java
public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue) {
this(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue,
             Executors.defaultThreadFactory(), defaultHandler);
```
* corecorePoolSize: 核心线程数目
* maximumPoolSize: 最大线程数目
* keepAliveTime: 线程存活时间，如果当前的线程数量大于核心线程数，那么多余的线程会被终结
* unit: keepAliveTime的单位；
* workQueue：Runnable的阻塞队列；