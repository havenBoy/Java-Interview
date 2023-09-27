### ThreadLocal

* 对于ThreadLocal的理解

  - 线程本地变量，为每个线程都提供一个副本，使得每个线程可以访问自己内部的本地变量；

* 深入理解ThreadLocal类

  具有的方法：

  - get()
  - set()
  - remove()
  - initialValue()

  工作流程：

  - 首先ThreadLocal中定义了一个ThreadLocalMap内部类，它继承了WeakReference（存在内存泄漏的风险），其中的key值是ThreadLocal的实例对象；
  - 在为ThreadLocal类的对象set或者get对象时，需要先获取ThreadLocalMap类属性，以对象为key，设置value；在get()之前，需要先set，或者重写initialValue方法；
  - 当前线程终止后，会被垃圾回收；

* ThreadLocal应用的场景

  - 数据的连接
  - session的管理