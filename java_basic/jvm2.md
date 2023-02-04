### JVM内存结构

需要了解JVM内存的管理与回收，不合理的利用导致很长时间的垃圾回收；

内存的泄漏会导致内存不够用；

#### JVM内存结构

- 堆，是GC的主要区域
  - 存放内容：对象实例，数组
  - 所有new出来的对象都在堆内存，分为年轻代与年老代
  - 抛出错误：outOfMemoryError,可能由于堆内存无法扩展时；
  - 新生代= Eden+from+to   比例8:1:1

- 堆区划分与理解：java对象的生命周期
  - Eden space
  
     伊甸园，对象被创建的时候首先放到这个区域，进行垃圾回收后，不能被回收的对象被放入到空的survivor区域
  
  - survivor
  
    用于保存在eden space内存区域中经过垃圾回收后没有被回收的对象。Survivor有两个，分别为To Survivor、 From Survivor，
  
    这个两个区域的空间大小是一样的。
  
    执行垃圾回收的时候Eden区域不能被回收的对象被放入到空的survivor（也就是To Survivor，同时Eden区域的内存会在垃圾回收的过程中全部释放），另一个survivor（即From Survivor）里不能被回收的对象也会被放入这个survivor（即To Survivor），然后To Survivor 和 From Survivor的标记会互换，始终保证一个survivor是空的
  
  - old gen
  
    用于存放新生代中经过多次垃圾回收仍然存活的对象，也有可能是新生代分配不了内存的大对象会直接进入老年代。
  
    经过多次垃圾回收都没有被回收的对象，这些对象的年代已经足够old了，就会放入到老年代。
  
    当老年代被放满的之后，虚拟机会进行垃圾回收，称之为Major GC。由于Major GC除并发GC外均需对整个堆进行扫描和回收，因此又称为Full GC。
  
  - 理解：
  
    我是一个普通的Java对象，我出生在Eden区，在Eden区我还看到和我长的很像的小兄弟(其他java对象)，我们在Eden区中玩了挺长时间。有一天Eden区中的人实在是太多了(会触发Young GC,每次GC加一岁)，我就被迫去了Survivor区的“To”区，自从去了Survivor区，我就开始漂了，有时候在Survivor的“From”区，有时候在Survivor的“To”区，居无定所(每次Young GC都需要Survivor区中的from区和to区"对调")。直到我18岁的时候(进行了18次Young GC)，爸爸说我成人了，该去社会上闯闯了。于是我就去了年老代那边，年老代里人很多，并且年龄都挺大的，我在这里也认识了很多人。在年老代里，我生活了20年，然后被回收(Old GC)。

- 栈，（不会发生GC）
  - 存放内容：基本数据类型，以及对象的引用
  - 每个线程会对应一个栈，每个栈中具有多个栈帧；
  - 支持本地方法栈，即native方法；
  - 抛出错误：outOfMemoryError,StackOverFlowError
- PC寄存器、程序计数器  （不会发生GC）
  - 当前线程执行字节码的行号指示器，每个线程有一个程序计数器，多个线程之间互不干扰；
  - 记录当前方法执行到哪一行代码
- 方法区（持久代）
  - 存放内容：静态变量，方法，常量，类的信息
  - 使用不当，也会导致内存溢出，异常信息为metadata
- 内存，速度高于堆内存，但在申请内存方面，申请速度方面会低于堆内存；

https://blog.csdn.net/hyman_c/article/details/103008165