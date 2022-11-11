# JDK线程池

> 介绍java.lang.concurrent包，主要是关于线程以及线程池的包

## 线程池的作用
- 主要是为了限制系统中线程的执行的数量
  * 线程池可以手动的创建以及销毁，可以指定线程池中线程的数量，为的是达到系统运行的最佳效果;
  * 如果当前的线程数大于线程池的最大size，那么新到来的线程会进入等候的状态，等待之前的线程退出线程池，如果当前的线程池中没有运行的线程，那么就直接进入线程池，如果没有线程进入到线程池，那么线程池处于等待的状态，直到有新的线程进入；
- 可以进行重复的利用，减少了创建与销毁线程的开销，提高系统的性能；
- 可以指定线程池的大小，如果线程池维持的线程的数量过多会造成系统的拥挤，如果线程池数量太少，又会浪费了系统的资源；

## ThreadPoolExcutor
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
## Excutors 提供的五个静态的工厂类
- newSingleThreadPool()   单线程线程池
  ```java
      public static ExecutorService newSingleThreadExecutor() {
        return new FinalizableDelegatedExecutorService
            (new ThreadPoolExecutor(1, 1,
                                    0L, TimeUnit.MILLISECONDS,
                                    new LinkedBlockingQueue<Runnable>()));
      }
  ```
  * 解释：

    此方法返回单线程的Excutors,线程池会依次执行请求的线程，如果有异常，会有新的对象来接管，且处理是按照次序来处理的；
- newfixedThreadPool()
  ```java
      public static ExecutorService newFixedThreadPool(int nThreads) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>());
    }
  ```
  * 解释：

    创建创建固定数目的线程池，达到最大线程数时，不再会扩容，会等待，直到有其他的任务退出线程池；
- newscheduleThreadPool()
  * 解释：是一种没有固定大小的线程池，可以用于执行定时或者周期性的任务；
  * 使用示例：
  ```java
  	ScheduledExecutorService service = Executors.newScheduledThreadPool(10);
  	service.scheduleWithFixedDelay(new Runnable() {
  		@Override
  		public void run() {
  			System.out.println(new Date().getSeconds());
  		}
  	}, 1000, 200000, TimeUnit.MICROSECONDS);
  ```
- newCachedThreadPool()
  ```java
      public static ExecutorService newCachedThreadPool() {
            return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                          60L, TimeUnit.SECONDS,
                                          new SynchronousQueue<Runnable>());
        }
  ```
  * 解释：
    - 此线程池的大小不是固定的，如果当前的线程大于当前任务的线程数，就会回收一部分资源，如果有新的任务创建时，则会生成新的线程来处理；
    - 线程池的corePoolSize是0，需要新创建线程时，大小是由当前的jVM来决定；

- newSingleThreadScheduledExcutors()
  * 执行单个数目定时任务的线程池

