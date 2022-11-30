### GC分析，命令调优（CMD + GUI）

#### jps

查看正在运行的java进程

```
jps [ -q ] [ -mlvV ][hostid ]
jps [ -help ]
```

#### jinfo

用来实时地查看和调整虚拟机的各项参数

```
jinfo [option] pid
```

#### jmap

用于生成堆转储快照，用于分析Java虚拟机堆中的对象，后续结合图形化界面进行分析

```
jmap [options] pid
jmap -F -dump:format=b,file=file.log pid
```

#### jstack

可以用来打印目标 Java 进程中各个线程的栈轨迹，以及这些线程所持有的锁。

通过线程的栈轨迹可以定位线程长时间停顿的原因，如线程间死锁、死循环、请求外部资源导致长时间等待等。

```
jstack [options] pid
jstack -l pid
```

#### jstat

是用于监视虚拟机各种运行状态信息的命令行工具，

它可以显示本地或者远程虚拟机进程中的类加载、内存、垃圾回收等信息

```
jstat generalOptions
eg: jstat -gc pid interval count 
$ jstat -options
    -class
    -compiler
    -gc
    -gccapacity
    -gccause
    -gcmetacapacity
    -gcnew
    -gcnewcapacity
    -gcold
    -gcoldcapacity
    -gcutil
    -printcompilation
```

S0C/S0U 第一幸存区的总大小与已经使用大小

S1C/S1U  第二幸存区的总大小与已经使用大小

EC/EU     新生区的大小与已经使用大小

OC/OC    老年区的大小与已经使用的大小

MC/MU   方法去的大小与已经使用的大小

CCSC/CCSU  压缩区的大小与已经使用的大小

YGC/YGCT    从程序启动到监控时间节点，新生区发生GC的次数与占用的时间多少，时间单位为s

FGC/FGCT        从程序启动到监控时间节点，老年区发生GC的次数与占用的时间多少，时间单位为s

GCT         新生去与老年区发生GC的总的时间

#### jcmd

可以向运行中的Java虚拟机(JVM)发送诊断命令。

```
jcmd <pid | main class> <command ... | PerfCounter.print | -f  file>
jcmd -l
jcmd -h
```

#### ECLIPSE_MAT（免费）

#### JCONSOLE（JDK自带）

#### JHAT(JDK自带)

用来分析java堆的命令，可以将堆中的对象以html的形式显示出来，包括对象的数量，大小等等

默认web访问端口7000，启动方式    jhat -J  -Xmx512M（最大堆内存） a1.log（dump文件）

```
jhat中的OQL（对象查询语言） 
如果需要根据某些条件来过滤或查询堆的对象，这是可能的，可以在jhat的html页面中执行OQL，来查询符合条件的对象

基本语法： 
select <javascript expression to select>
[from [instanceof] <class name> <identifier>]
[where <javascript boolean expression to filter>]

解释： 
(1)class name是java类的完全限定名，如：java.lang.String, java.util.ArrayList, [C是char数组, [Ljava.io.File是java.io.File[]
(2)类的完全限定名不足以唯一的辨识一个类，因为不同的ClassLoader载入的相同的类，它们在jvm中是不同类型的
(3)instanceof表示也查询某一个类的子类，如果不明确instanceof，则只精确查询class name指定的类
(4)from和where子句都是可选的
(5)java域表示：obj.field_name；java数组表示：array[index]

举例： 
（1）查询长度大于100的字符串
select s from java.lang.String s where s.count > 100
（2）查询长度大于256的数组
select a from [I a where a.length > 256
（3）显示匹配某一正则表达式的字符串
select a.value.toString() from java.lang.String s where /java/(s.value.toString())
（4）显示所有文件对象的文件路径
select file.path.value.toString() from java.io.File file
（5）显示所有ClassLoader的类名
select classof(cl).name from instanceof java.lang.ClassLoader cl
（6）通过引用查询对象
select o from instanceof 0xd404d404 o
```

#### JPROFILER(收费)
