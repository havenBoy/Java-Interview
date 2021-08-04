## Spark
> 用于大规模数据处理的同一分析计算引擎  
> 基于内存计算，提高了大数据处理数据的实时性，同时保证了高容错性与高伸缩性 
- hadoop 与 spark 的区别
  - 具有较打优势，但spark不能完全替代hadoop,spark主要用于替代hadoop中的MapReduce，存储可以使用HDFS  
  - spark对硬件的要求较高，对内存与cpu有一定的要求  
  - spark已经融入了Hadoop的生态圈，使用yarn实现资源的调度，借助hdfs来进行存储
- spark的特点
  - 快: 与hadoop比较快，基于内存要快100倍左右，
  - 易用: 支持java、python、scala等的API，支持交互式的python与scala的shell
  - 兼容性：方便与其他开源产品进行兼容
  - spark core、spark sql、spark streaming等
- spark的运行模式
  - local  本地模式，开发测试使用
  - standalone 独立集群模式，开发测试使用
  - standalone-HA  基于zk来搭建高可用的集群
  - on yarn 集群模式，生产环境使用，较多使用场景  
    运行在yarn集群上，由yarn负责资源管理，spark负责任务的调度与计算  
    优点：计算资源按需伸缩，集群利用率高  
- spark shell
  入门使用的工具，方便用户进行交互编程，是测试或者学习时使用  
  spark-shell --master local[*] 表示使用当前的机器上所有可用的资源  

- spark-submit
  - --class  指定主类
  - --master 指定运行节点
  - --executor-memory 1g
  - --total-executor-cores 2 
  - xxx.jar 
  - xxxx 参数
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
  - RDD  
  描述：弹性分布式数据集，是spark最基本的数据抽象，代表一个不可变，可分区，元素可以并行计算的集合，最小的计算单元  
  - 
  
  