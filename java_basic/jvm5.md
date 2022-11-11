### CMS与G1垃圾回收器的比较

- CMS

- G1

  ~~~properties
     G1要做到这样的效果，也是有前提的，一方面是硬件环境的要求，必须是多核的CPU以及较大的内存（从规范来看，512M以上就满足条件了），另外一方面是需要接受吞吐量的稍微降低，对于实时性要求高的系统而言，这点应该是可以接受的。
  
  ​为了能够达到这样的效果，G1在原有的各种GC策略上进行了吸收和改进，在G1中可以看到增量收集器和CMS的影子，但它不仅仅是吸收原有GC策略的优点，并在此基础上做出了很多的改进，简单来说，G1吸收了增量GC以及CMS的精髓，将整个jvm Heap划分为多个固定大小的region，扫描时采用Snapshot-at-the-beginning的并发marking算法（具体在后面内容详细解释）对整个heap中的region进行mark，回收时根据region中活跃对象的bytes进行排序，首先回收活跃对象bytes小以及回收耗时短（预估出来的时间）的region，回收的方法为将此region中的活跃对象复制到另外的region中，根据指定的GC所能占用的时间来估算能回收多少region，这点和以前版本的Full GC时得处理整个heap非常不同，这样就做到了能够尽量短时间的暂停应用，又能回收内存，由于这种策略在回收时首先回收的是垃圾对象所占空间最多的region，因此称为Garbage First。
  
  ​ 看完上面对于G1策略的简短描述，并不能清楚的掌握G1，在继续详细看G1的步骤之前，必须先明白G1对于JVM Heap的改造，这些对于习惯了划分为new generation、old generation的大家来说都有不少的新意。
  
  ​ G1将Heap划分为多个固定大小的region，这也是G1能够实现控制GC导致的应用暂停时间的前提，region之间的对象引用通过remembered set来维护，每个region都有一个remembered set，remembered set中包含了引用当前region中对象的region的对象的pointer，由于同时应用也会造成这些region中对象的引用关系不断的发生改变，G1采用了Card Table来用于应用通知region修改remembered sets，Card Table由多个512字节的Card构成，这些Card在Card Table中以1个字节来标识，每个应用的线程都有一个关联的remembered set log，用于缓存和顺序化线程运行时造成的对于card的修改，另外，还有一个全局的filled RS buffers，当应用线程执行时修改了card后，如果造成的改变仅为同一region中的对象之间的关联，则不记录remembered set log，如造成的改变为跨region中的对象的关联，则记录到线程的remembered set log，如线程的remembered set log满了，则放入全局的filled RS buffers中，线程自身则重新创建一个新的remembered set log，remembered set本身也是一个由一堆cards构成的哈希表。
  
  ​ 尽管G1将Heap划分为了多个region，但其默认采用的仍然是分代的方式，只是仅简单的划分为了年轻代（young）和非年轻代，这也是由于G1仍然坚信大多数新创建的对象都是不需要长的生命周期的，对于应用新创建的对象，G1将其放入标识为young的region中，对于这些region，并不记录remembered set logs，扫描时只需扫描活跃的对象，G1在分代的方式上还可更细的划分为：fully young或partially young，fully young方式暂停的时候仅处理young regions，partially同样处理所有的young regions，但它还会根据允许的GC的暂停时间来决定是否要加入其他的非young regions，G1是运行到fully-young方式还是partially young方式，外部是不能决定的，在启动时，G1采用的为fully-young方式，当G1完成一次Concurrent Marking后，则切换为partially young方式，随后G1跟踪每次回收的效率，如果回收fully-young中的regions已经可以满足内存需要的话，那么就切换回fully young方式，但当heap size的大小接近满的情况下，G1会切换到partially young方式，以保证能提供足够的内存空间给应用使用。
  ~~~

  ```properties
        除了分代方式的划分外，G1还支持另外一种pure G1的方式，也就是不进行代的划分，pure方式和分代方式的具体不同在下面的具体执行步骤中进行描述。掌握了这些概念后，继续来看G1的具体执行步骤：
     
     1.Initial Marking
     
     ​ G1对于每个region都保存了两个标识用的bitmap，一个为previous marking bitmap，一个为next marking bitmap，bitmap中包含了一个bit的地址信息来指向对象的起始点。
     
     ​ 开始Initial Marking之前，首先并发的清空next marking bitmap，然后停止所有应用线程，并扫描标识出每个region中root可直接访问到的对象，将region中top的值放入next top at mark start（TAMS）中，之后恢复所有应用线程。
     
     ​ 触发这个步骤执行的条件为：
     
       G1定义了一个JVM Heap大小的百分比的阀值，称为h，另外还有一个H，H的值为(1-h)*Heap Size，目前这个h的值是固定的，后续G1也许会将其改为动态的，根据jvm的运行情况来动态的调整，在分代方式下，G1还定义了一个u以及soft limit，soft limit的值为H-u*Heap Size，当Heap中使用的内存超过了soft limit值时，就会在一次clean up执行完毕后在应用允许的GC暂停时间范围内尽快的执行此步骤；
     
       在pure方式下，G1将marking与clean up组成一个环，以便clean up能充分的使用marking的信息，当clean up开始回收时，首先回收能够带来最多内存空间的regions，当经过多次的clean up，回收到没多少空间的regions时，G1重新初始化一个新的marking与clean up构成的环。
     
     2.Concurrent Marking
     
     按照之前Initial Marking扫描到的对象进行遍历，以识别这些对象的下层对象的活跃状态，对于在此期间应用线程并发修改的对象的以来关系则记录到remembered set logs中，新创建的对象则放入比top值更高的地址区间中，这些新创建的对象默认状态即为活跃的，同时修改top值。
     
     3.Final Marking Pause
     
     当应用线程的remembered set logs未满时，是不会放入filled RS buffers中的，在这样的情况下，这些remebered set logs中记录的card的修改就会被更新了，因此需要这一步，这一步要做的就是把应用线程中存在的remembered set logs的内容进行处理，并相应的修改remembered sets，这一步需要暂停应用，并行的运行。
     
     4.Live Data Counting and Cleanup
     
        值得注意的是，在G1中，并不是说Final Marking Pause执行完了，就肯定执行Cleanup这步的，由于这步需要暂停应用，G1为了能够达到准实时的要求，需要根据用户指定的最大的GC造成的暂停时间来合理的规划什么时候执行Cleanup，另外还有几种情况也是会触发这个步骤的执行的：
     
        G1采用的是复制方法来进行收集，必须保证每次的”to space”的空间都是够的，因此G1采取的策略是当已经使用的内存空间达到了H时，就执行Cleanup这个步骤；
     
        对于full-young和partially-young的分代模式的G1而言，则还有情况会触发Cleanup的执行，full-young模式下，G1根据应用可接受的暂停时间、回收young regions需要消耗的时间来估算出一个yound regions的数量值，当JVM中分配对象的young regions的数量达到此值时，Cleanup就会执行；partially-young模式下，则会尽量频繁的在应用可接受的暂停时间范围内执行Cleanup，并最大限度的去执行non-young regions的Cleanup。
     
        这一步中GC线程并行的扫描所有region，计算每个region中低于next TAMS值中marked data的大小，然后根据应用所期望的GC的短延时以及G1对于region回收所需的耗时的预估，排序region，将其中活跃的对象复制到其他region中。
  ```
  
  
  
  ​     