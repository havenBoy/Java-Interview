## Spark
> 用于大规模数据处理的同一分析计算引擎  
> 基于内存计算，提高了大数据处理数据的实时性，同时保证了高容错性与高伸缩性 
- ##### hadoop 与 spark 的区别
  
  - 具有较打优势，但spark不能完全替代hadoop,spark主要用于替代hadoop中的MapReduce，存储可以使用HDFS  
  - spark对硬件的要求较高，对内存与cpu有一定的要求  
  - spark已经融入了Hadoop的生态圈，使用yarn实现资源的调度，借助hdfs来进行存储
  
- spark的特点
  - 快: 与hadoop比较快，基于内存要快100倍左右，
  - 易用: 支持java、python、scala等的API，支持交互式的python与scala的shell
  - 兼容性：方便与其他开源产品进行兼容
  - spark core、spark sql、spark streaming等
  
- spark的运行模式
  - local 
  
     本地模式，开发测试使用
  
  - standalone 
  
    独立集群模式，开发测试使用
  
  - standalone-HA  
  
    基于zk来搭建高可用的集群
  
  - on yarn 
  
    集群模式，生产环境使用，较多使用场景  
    运行在yarn集群上，由yarn负责资源管理，spark负责任务的调度与计算  
    优点：计算资源按需伸缩，集群利用率高  
  
- spark shell
  入门使用的工具，方便用户进行交互编程，是测试或者学习时使用 
  spark-shell --master local[*] 表示使用当前的机器上所有可用的资源  
  
- spark的wordCount
  
  ~~~scala
  sc.textFile(path).flatMap(_.spilt(" ")).map((_, 1)).reduceByKey(_+_).collect  
  ~~~
  
- spark运行环境
  - Local模式即本地环境 练习使用
  - standalone 即独立部署模式
    1. conf/slaves 填写自己机器的主机名
    2. 修改spark-env.sh脚本中的java_home，以及运行master节点与端口，端口默认为7077
  - yarn模式
    1. 配置yarn中的
    2. 修改spark-env.sh
    3. 启动hdfs与yarn
    4. 提交任务
  
- 配置历史服务
  需要查看历史服务运行的状况，需要修改以下2个文件
  1. spark-default.conf
  2. spark-env.sh
  3. 分发配置项
  4. 重启spark以及历史服务脚本
  - 配置多个master  
    1. 使用多个zookeeper，需要重新修改spark-env.sh,后分发
    2. 重启集群，需要在多个master上启动，会形成一个HA的环境
  
- spark运行框架
  - Driver
  - Executor
  - Master&worker  独立部署环境中使用,资源组件
  - Application Master 
  
- 核心概念
  - Executor  运行在工作节点的一个JVM进程
  - core  指定CPU core数量，虚拟核数
  - 并行度  并行执行任务的核数
  - 有向
  
- spark-submit
  - --class  指定主函数类
  - --master 指定运行节点
  - --executor-memory 1g  计算节点内存
  - --total-executor-cores 2  所有执行总的核数
  - xxx.jar  所要运行的jar
  - xxxx 命令行参数
  
- spark-submit  部署在yarn上
  - --class
  - --master yarn 
  - --deploy-mode cluster  (client 会在会话窗口，关闭就会停止)
  - --driver-memory 1g 
  - --executor-memory 1g
  - --executor-cores 2
  - --queue default 
  - xxx.jar 
  - xxx参数
  
- wordcount
  要求：写出spark的wordcount
  
- 三大数据类型
  - RDD  装饰者模式，其数据处理模式类似于IO
    描述：弹性分布式数据集，是spark最基本的数据抽象，代表一个不可变，可分区，元素可以并行计算的集合，
  
    是spark中可以运行的最小计算单元，在调用collect时才会进行真正的方法调用  
    
    - 弹性：包括存储的弹性，容错的弹性，计算的弹性，分片的弹性
    - 分布式：数据是存储在大数据集群的不同节点上
    - 数据集： RDD封装了计算逻辑，不保存数据
    - 数据抽象： 是一个抽象类，需要实现子类具体实现
    - 不可变: 封装的逻辑不可变，如果需要改变，需要实现新的RDD
    - 可分区，并行计算 
    
  -  五大主要属性
    - 分区列表：执行并行计算，分布式计算  getPartitions
    - 分区计算函数：分区    compute
    - RDD的依赖关系：getDependencies 
    - 分区器：数据分区的处理
    - 首选位置：判断计算发送到那个节点效率最优
    
  - 执行原理
  
- RDD创建
  -  外部文件创建  
    1. 文件  文件、文件夹、hdfs，通配符文件夹或者文件
    2. hdfs
    
  - 从内存或者集合中创建 

    1. 准备环境
       
       ~~~scala
       val sc = new SparkContext(new SparkConf().setMaster("local[*]").setAppName("app"));
       ~~~
       
    2. 创建RDD

       ~~~scala
       val seq = Seq[Int](1,2,3,4)
       //并行，以下是等价的
       //val rdd: RDD[Int] = sc.parallelize(seq)
       val rdd: RDD[Int] = sc.makeRDD(seq)
       rdd.collect.foreach(println)
       ~~~

    3. 关闭环境

       sc.stop()

    4. 从其他RDD创建

- 分区与并行度（注意概念的区分）
  - 默认并行度defaultParallelism

    为当前运行环境的可用最大核数

  - 分区规则：

    文件分区还是有区别的：最小分区数量，取当前的并行度值与2之间最小值 

    如果不想使用默认分区数，则  val rdd = sc.textFile('path', 3) 

    分区方式计算：字节数/minPartitions = 每个分区的字节数  最后余加一个新的分区  1.1倍原则 

    hadoop存放文件，当文件大于源文件的1.1倍是才会新生成新的分区 

  - 数据分区的分配：  

    1. 以行为单位进行读取，与字节数没有关系
    2. 数据读取时，需要以偏移量为单位，
    3. 数据分区的偏移量范围的计算
    4. 偏移量不会被重复读取
    5. 如果为多个文件时，会以文件为单位进行

- RDD常用方法
  - **转换算子**
  
    即功能的补充，装饰方法的内容，flatMap  Map 
  
    - mapPartitions  会将整个分区的数据全部加载在内存中，可能会导致内存的溢出，传递一个迭代器，返回一个迭代器
    - map  一个一个数据的处理，比较慢，串行操作，不清楚数据来源于那个分区 
    - 区别：性能：前者较高，会长时间的占用内存，内存有限的情况下，不建议使用
  
  - **行动算子**
  
    触发任务的调度与作业的执行
  
  



