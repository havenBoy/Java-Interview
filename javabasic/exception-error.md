### 异常与error

- 二者都是异常处理机制的基本类型
- Exception是程序在运行过程中可以预料的意外状态，能被捕获并进行处理，它分为可检查（checked）与不可检查（unchecked）,可检查异常在程序编译期间的一部分，如IOException，需要使用try, catch和finally关键字在编译期进行处理，而不可检查一般是代码的逻辑问题，如NullPointException，indexOutOfBoundsException等
- Error在正常的情况下，一般不会出现，一旦出现就是不可恢复的，如OutOfMemoryError，StackOverflowError等
- Exception与Error都是继承于Throwable类；
- 注意：尽量不要用try catch包住一个大的代码块，否则会产生额外的性能开销；
- throws ?  throw ?
  - throws是声明在方法头上，而throw是声明在方法里边
  - throws表现是异常的可能性，但不一定会发生这个异常，但throw表示已经发生这个异常，并将其抛出
- **Java中final,finalize,finally关键字的区别**
  - final：可以作为修饰符修饰变量、方法和类，被final修饰的变量只能一次赋值；被final修饰的方法不能够在子类中被重写（override）；被final修饰的类不能够被继承。
  - finally用在异常处理中定义总是执行代码，无论try块中的代码是否引发异常，catch是否匹配成功，finally块中的代码总是被执行，除非JVM被关闭（System.exit(1)），通常用作释放外部资源(不会被垃圾回收器回收的资源)。
  - finalize()方法是Object类中定义的方法，**当垃圾回收器将无用对象从内存中清除时，该对象的finalize()方法被调用。由于该方法是protected方法，子类可以通过重写（override）该方法以整理资源或者执行其他的清理工作。(有可能对象会复活)**

