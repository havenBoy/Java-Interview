### ReentrantLock（可重入锁）

- **介绍**：
  - 它是以Lock作为底层的接口，而Synchronize是java语言内置的实现；
  - 支持可重入，即当前线程如果再次获取该对象的锁时，不会被阻塞
  - 支持公平锁与非公平锁，可相应中断，也可设置超时；
  - 内部基于AbstractQueuedSynchronizer(AQS)原理实现；
  - 二者都是用于线程的同步，但前者的功能比后者更为丰富；
  - 继承关系：ReentrantLock的内部类Sync，而Sync的父类即为：AQS


- **原理**

  - 可重入：对于同一个对象的锁，相同线程在进入同步代码块时，不会被阻塞；

  - 公平与非公平：是指线程获取锁的方式，默认是非公平

    - 公平：按照FIFO（先来先得）获取锁，使得每个线程都能获取锁，但是效率低下；

    - 非公平：抢占式方式，抢不到的进入等待队列，效率变高，但有时也会出现饥饿现象；

    - 在调用Lock的构造方法时，可以指定是否为公平锁；

    - 性能问题的比较：

      按道理来说，有序的进行要比无序的速率要快，不然公众场合也不会要排队，但是为什么这里的非公平锁要比公平锁要快呢？是因为假如线程A已经在持有锁，线程B去竞争，假如A没有释放，那么线程B会被挂起，此时如果线程A释放了锁，同时线程C也来竞争锁，那么首先回去对线程B进行唤醒，但是也许在唤醒线程B的同时，线程C早已获得锁，并且在时间的间隙已经使用完，那么这时系统的吞吐量就提高了；

- **源码分析**：

  - AQS同步队列，

    - 是一个基于双向链表的同步队列，如果线程在获取锁失败时，会被封装成一个节点，加入队列；
    - 在当头节点释放同步状态后，会唤醒其他的后继节点，后继节点升级为头结点，以前的头结点被删除

  - 公平锁（获取锁流程）

    - 调用Acquire方法，把线程放入同步队列中进行等待

    - 如果线程获取锁，那么把自己设置为持锁线程返回；

    - 如果当前状态值不为0，且当前线程持有锁，那么直接重入即可；

    - 代码：

      ```java
      +--- ReentrantLock.FairSync.java
      final void lock() {
          // 调用 AQS acquire 获取锁
          acquire(1);
      }

      +--- AbstractQueuedSynchronizer.java
      /**
       * 该方法主要做了三件事情：
       * 1. 调用 tryAcquire 尝试获取锁，该方法需由 AQS 的继承类实现，获取成功直接返回
       * 2. 若 tryAcquire 返回 false，则调用 addWaiter 方法，将当前线程封装成节点，
       *    并将节点放入同步队列尾部
       * 3. 调用 acquireQueued 方法让同步队列中的节点循环尝试获取锁
       */
      public final void acquire(int arg) {
          // acquireQueued 和 addWaiter 属于 AQS 中的方法，这里不展开分析了
          if (!tryAcquire(arg) &&
              acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
              selfInterrupt();
      }

      +--- ReentrantLock.FairSync.java
      protected final boolean tryAcquire(int acquires) {
          final Thread current = Thread.currentThread();
          // 获取同步状态
          int c = getState();
          // 如果同步状态 c 为0，表示锁暂时没被其他线程获取
          if (c == 0) {
              /*
               * 判断是否有其他线程等待的时间更长。如果有，应该先让等待时间更长的节点先获取锁。
               * 如果没有，调用 compareAndSetState 尝试设置同步状态。
               */ 
              if (!hasQueuedPredecessors() &&
                  compareAndSetState(0, acquires)) {
                  // 将当前线程设置为持有锁的线程
                  setExclusiveOwnerThread(current);
                  return true;
              }
          }
          // 如果当前线程为持有锁的线程，则执行重入逻辑
          else if (current == getExclusiveOwnerThread()) {
              // 计算重入后的同步状态，acquires 一般为1
              int nextc = c + acquires;
              // 如果重入次数超过限制，这里会抛出异常
              if (nextc < 0)
                  throw new Error("Maximum lock count exceeded");
              // 设置重入后的同步状态
              setState(nextc);
              return true;
          }
          return false;
      }

      +--- AbstractQueuedSynchronizer.java
      /** 该方法用于判断同步队列中有比当前线程等待时间更长的线程 */
      public final boolean hasQueuedPredecessors() {
          Node t = tail;
          Node h = head;
          Node s;
          /*
           * 在同步队列中，头结点是已经获取了锁的节点，头结点的后继节点则是即将获取锁的节点。
           * 如果有节点对应的线程等待的时间比当前线程长，则返回 true，否则返回 false
           */
          return h != t &&
              ((s = h.next) == null || s.thread != Thread.currentThread());
      }
      ```

  - 非公平锁（获取锁流程）

    - 调用CAS方法抢占锁，抢占成功后把自己设置为持锁线程，并返回；

    - 如果加锁失败，那么调用acquire方法，把线程置于同步队列尾部等待；

    - 线程在同步队列抢占时获得锁，把自己设置为持锁线程并返回；

    - 如果同步状态不为0，那么当前线程为持锁线程，执行重入逻辑；

    - 代码：

      ```java
      +--- ReentrantLock.NonfairSync
      final void lock() {
          /*
           * 这里调用直接 CAS 设置 state 变量，如果设置成功，表明加锁成功。这里并没有像公平锁
           * 那样调用 acquire 方法让线程进入同步队列进行排队，而是直接调用 CAS 抢占锁。抢占失败
           * 再调用 acquire 方法将线程置于队列尾部排队。
           */
          if (compareAndSetState(0, 1))
              setExclusiveOwnerThread(Thread.currentThread());
          else
              acquire(1);
      }

      +--- AbstractQueuedSynchronizer
      /** 参考上一节的分析 */
      public final void acquire(int arg) {
          if (!tryAcquire(arg) &&
              acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
              selfInterrupt();
      }

      +--- ReentrantLock.NonfairSync
      protected final boolean tryAcquire(int acquires) {
          return nonfairTryAcquire(acquires);
      }

      +--- ReentrantLock.Sync
      final boolean nonfairTryAcquire(int acquires) {
          final Thread current = Thread.currentThread();
          // 获取同步状态
          int c = getState();
          
          // 如果同步状态 c = 0，表明锁当前没有线程获得，此时可加锁。
          if (c == 0) {
              // 调用 CAS 加锁，如果失败，则说明有其他线程在竞争获取锁
              if (compareAndSetState(0, acquires)) {
                  // 设置当前线程为锁的持有线程
                  setExclusiveOwnerThread(current);
                  return true;
              }
          }
          // 如果当前线程已经持有锁，此处条件为 true，表明线程需再次获取锁，也就是重入
          else if (current == getExclusiveOwnerThread()) {
              // 计算重入后的同步状态值，acquires 一般为1
              int nextc = c + acquires;
              if (nextc < 0) // overflow
                  throw new Error("Maximum lock count exceeded");
              // 设置新的同步状态值
              setState(nextc);
              return true;
          }
          return false;
      }
      ```

  - **公平锁与非公平锁实现的对比**：

    ​

  - **释放锁**

    - 不区分公平与非公平，较为简单

      ~~~java
      public void unlock() {
          // 调用 AQS 中的 release 方法
          sync.release(1);
      }

      public final boolean release(int arg) {
          // 调用 ReentrantLock.Sync 中的 tryRelease 尝试释放锁
          if (tryRelease(arg)) {
              Node h = head;
              /*
               * 如果头结点的等待状态不为0，则应该唤醒头结点的后继节点。
               * 这里简单说个结论：
               *     头结点的等待状态为0，表示头节点的后继节点线程还是活跃的，无需唤醒
               */
              if (h != null && h.waitStatus != 0)
                  // 唤醒头结点的后继节点，该方法的分析请参考我写的关于 AQS 的文章
                  unparkSuccessor(h);
              return true;
          }
          return false;
      }
      protected final boolean tryRelease(int releases) {
          /*
           * 用同步状态量 state 减去释放量 releases，得到本次释放锁后的同步状态量。
           * 当将 state 为 0，锁才能被完全释放
           */ 
          int c = getState() - releases;
          // 检测当前线程是否已经持有锁，仅允许持有锁的线程执行锁释放逻辑
          if (Thread.currentThread() != getExclusiveOwnerThread())
              throw new IllegalMonitorStateException();
              
          boolean free = false;
          // 如果 c 为0，则表示完全释放锁了，此时将持锁线程设为 null
          if (c == 0) {
              free = true;
              setExclusiveOwnerThread(null);
          }
          
          // 设置新的同步状态
          setState(c);
          return free;
      }
      ~~~

- **Synchronize与ReetrantLock的比较**

  - 二者均支持可重入和非公平锁
  - 后者支持响应中断，超时等待，公平锁；
  - Synchronize在出现异常时，会自动释放锁，而Lock的锁一般需要手动，且在finally中；
  - Lock支持查询对象是否处于锁定状态（isLocked），获取加锁次数（getHoldCount）,而Synchronize只是支持查询当前线程是否持有锁（holdsLock）；