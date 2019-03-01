### CAS    AQS

> 主要针对CAS与AQS的思想理解以及辅助类的解析

- CAS

  - 思想：（无锁）

    CAS有3个操作数，内存值V，旧的预期值A，要修改的新值B。当且仅当预期值A和内存值V相同时，将内存值V修改为B，否则什么都不做

  - ​

- AQS

  - 思想：

    ​       有很多线程请求共享资源，就把进入的线程设置为有效的工作线程，且把当前的共享的资源进行锁定，当其他的请求需要访问共享资源时，发现当前共享资源是占用的，那么采取的策略是把当前请求的线程放入一个先进先出的双向队列的阻塞锁，当同步状态释放时，会自动唤醒当前队列的头结点；


  - CountDownLatch、CyclicBarrier、Semaphore的区别

    - CountDownLatch

      - 应用场景：

        所有的线程都准备好了才开始，所有的线程跑完了才算是结束，可以实现类似计数器的功能；

      - 构造器 :  

        ```java
        public CountDownLatch(int count) {  }; //其中count是计数器
        ```

      - ```java
        public void await() throws InterruptedException { };   
        //调用await()方法的线程会被挂起，它会等待直到count值为0才继续执行
        ```

      - ```java
        public boolean await(long timeout, TimeUnit unit) throws InterruptedException { }; 
        //和await()类似，只不过等待一定的时间后count值还没变为0的话就会继续执行
        ```

      - ```java
        public void countDown() { };  //将count值减1
        ```

    - CyclicBarrier

      - 字面意思是回环栅栏，可以让一组线程都等待至某个状态时再执行，并且可复用；

      - 应用场景：

        当有一个大任务时，要分配多个子任务去执行，只有当所有的子任务执行完成，才能执行大任务

      - 构造器：

        ```java
        1. public CyclicBarrier(int parties, Runnable barrierAction) {}
        2. public CyclicBarrier(int parties) {}
        //其中的parties是指多少个任务等待至barrier状态，barrierAction是指当指定线程达到指定的状态后会执行的内容；
        ```

      - ```java
        public int await() throws InterruptedException, BrokenBarrierException { };
        //用来挂起当前线程，当前状态到达barrier状态后执行后续任务；
        ```

      - ```java
        public int await(long timeout, TimeUnit unit)throws InterruptedException,BrokenBarrierException,TimeoutException { };
        //让等待的线程等待一段时间，如果还有一些任务没有到达barrier状态，那么直接让到达barrier状态的线程执行后续的任务；
        ```

    - Semaphore

      - 信号量，可以控制访问的个数，如停车位，如果已经满了，那么只有等以前停的车走了，才能让下一个车进入；

      - 构造器：

        1. ```java
           public Semaphore(int permits) {
               sync = new NonfairSync(permits);
               //参数permits表示许可数目，即同时可以允许多少线程进行访问
           }
           ```

        2. ```java
           public Semaphore(int permits, boolean fair) {    
               sync = (fair)? new FairSync(permits) : new NonfairSync(permits);
               //这个多了一个参数fair表示是否是公平的，即等待时间越久的越先获取许可
           }
           ```

      - 重要的方法

        ```java
        public void acquire() throws InterruptedException {  }     //获取一个许可
        public void acquire(int permits) throws InterruptedException { } 
        //获取permits个许可，acquire获取许可，如果获取不到，会一直等待，直到获取到
        public void release() { }          //释放一个许可
        public void release(int permits) { }    
        //释放permits个许可
        //release是释放一个许可，在释放之前必须获取一个许可

        //以上的方法都可能被阻塞，请尝试以下方法;
        public boolean tryAcquire() { };    
        //尝试获取一个许可，若获取成功，则立即返回true，若获取失败，则立即返回false
        public boolean tryAcquire(long timeout, TimeUnit unit) throws InterruptedException { };  
        //尝试获取一个许可，若在指定的时间内获取成功，则立即返回true，否则则立即返回false
        public boolean tryAcquire(int permits) { }; 
        //尝试获取permits个许可，若获取成功，则立即返回true，若获取失败，则立即返回false
        public boolean tryAcquire(int permits, long timeout, TimeUnit unit) throws InterruptedException { }; 
        //尝试获取permits个许可，若在指定的时间内获取成功，则立即返回true，否则则立即返回false
        ```