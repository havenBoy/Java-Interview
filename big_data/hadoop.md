## 介绍hadoop
  > 泛指hadoop生态圈，狭义指的是这个开源框架，包括hdfs+yarn+map_reduce  
  > HDFS 分布式文件系统，解决海量文件的存储  
  > YARN是作业调度与集群资源管理的框架 解决资源任务的调度  
  > MAPREDUCE 分布式运算编程框架 解决海量数据的计算   

### hadoop1.0与hadoop2.0的区别？

hadoop1.0 包含hdfs与MapReduce，不包含yarn，其资源调度使用的是MapReduce内置的资源管理去调度  
hadoop2.0引入yarn作为资源调度框架，mapreduce只负责计算  
Yarn框架中将JobTracker资源分配和作业控制分开,这也就是resource_manager与application_master的原型  

### hadoop特性优点？

- 扩容能力--容易将节点扩充到一个hadoop集群中
- 成本低--使用廉价的机器组成服务器集群
- 高效率--并发数据处理机制
- 可靠性--会保存数据的多个副本，当任务发生错误时，会将失败的任务重新分配

### hadoop集群中hadoop需要启动那些进程，作用是什么？

- namenode 负责文件系统元数据信息，管理对集群存储文件与文件系统命名空间的访问
- datanode  负责某个节点的真实数据存储，提供来自文件系统客户端的读写请求，块的创建以及删除
- secondary_namenode  并不是namenode的守护进程，而是负责namenode合并editlog，减少namenode启动的时间
- resource_manager 是yarn平台的守护进程，负责所有资源的分配的调度
- node_manager 单个节点的资源管理 用于resource_manager具体命令的执行
- DFS_ZK_Failover_Controller 如果是高可用的hdfs，会有这个进程，负责监控namenode的状态，并把状态写入zk

### hadoop 主要配置项文件

- hadoop-env.sh  主要配置jdk路径
- core-site.xml  主要配置
- hdfs-site.xml  主要配置为
- mapred-site.xml  主要配置
- yarn-site.xml  指定yarn中的resourceManager的地址

### hadoop 主要的命令

- 初始化 hadoop namenode -format  只在第一次时使用，后续不要轻易执行
- 启动dfs start-dfs.sh
- 启动yarn start-yarn.sh
- 启动历史服务器 
- 一键启动  start-all.sh 不建议使用，已经过时
- 启动完成后 常见的端口50070（hdfsweb界面端口） 8088 （yarn任务运行状态web界面） 18888 （历史服务器端口）9000 客户端内部端口
- 常见的dfs命令  hadoop fs ...

### hdfs 的垃圾回收机制 

- 在core-site.xml配置

- fs.trash.interval 是以分钟为单位。默认删除会直接删除掉，不会以垃圾的方式存在 

### hdfs的写数据流程

1. 客户端通过FileSystem模块向namenode请求上传文件，namenode检查目标文件是否存在，检查父目录是否存在
2. namenode返回是否可以上传此文件
3. 客户端请求第一个block上传到那几个datanode服务器上
4. namenode返回几个datanode的节点
5. 客户端请求第一个datanode上传数据，第一个datanode接受请求后会调用datanode2等
6. datanode逐级应答客户端
7. 客户端向第一个datanode上传block,在datanode1接受到第一个block会传给下一个datanode,以此类推
8. 在传输完第一个block后会进行下一个block的上传

### hdfs读数据流程

1. 客户端通过fs向namenode请求下载文件，namenode会查询元数据，找到文件块所在的节点
2. 就近原则，同节点->同机架，请求读取数据
3. datanode开始传输文件给客户端
4. 客户端接收，以本地缓存，然后写入目标文件

### secondaryNameNode的作用

1. namenode是管理元数据信息 datanode是负责具体数据存储

2. 负责协作namenode把editlog 到fsimage文件中

3. 达到触发的条件 1h或者100W 把namenode积累的所有edits与一个最新的fsimage下载到本地，

   在内存中进行合并，这个过程称为checkpoint

### Hadoop参数调优

1. 在hdfs-site.xml配置多目录，需要提前配置好，否则修改后需要重启集群
2. namenode有一个工作线程池，用来处理DataNode的并发心跳与客户端并发的元数据操作  
   dfs.namenode.handler.count = 20 * log2，一般集群大小为10台时，参数设置为60  
3. 编辑日志存储路径dfs.namenode.edits.dir设置与镜像文件存储路径分开，达到最低写入延迟
4. 服务器节点上yarn可使用的物理内存总量，默认为8G，如果内存比这个值小时。需要手动修改辞职
5. 单个任务需要可申请的最多物理内存量，默认最大为8G

### hadoop宕机

1. 如果是MR造成的系统宕机，需要控制yarn同时运行任务数量以及每个任务申请的最大内存
2. 如果是写入文件过量造成的namenode宕机，写入上一级的增加消息的缓冲，增加kafka的存储大小

### hadoop小文件优化与处理

- **小文件问题解释**

  小文件指的是文件的大小明显小于hdfs的block的大小，当前默认block的大小为128M

  block的大小的设置取决于当前操作系统与磁盘读取文件的速率，原始为64M

  hdfs中任意一个文件，都会在namenode中存在一个元数据信息，这个信息的大小约等于150byte

  如果小文件的数量较大时，namenode会占用较大的内存，对于文件的读取与写入及其困难

  后续在处理小文件时，任务的启动与停止占用了比任务处理更多的时间

  一般可以启用JVM参数重用，达到在一个JVM中跑多个MAP任务，可以减少JVM启动与停止的花销

- **小文件出现的情况**

  1. 日志文件，日志是翻滚的，每天都会有一个文件产生
  2. 实际上，每个文件就是很小，然后文件的数量就是很大，然后还没有办法合并的文件

- **解决方式**

  1. 在实际的环境中，需要设置小文件的阈值，业务在定期监控此大小，达到阈值执行业务上的文件合并
  2. hadoop本身也提供了hadoop archive以及combineInputFormat类，可对小文件进行合并

### HDFS的扩容与缩容

### HDFS的安全模式

- 所处的特殊模式，只接受读数据请求，不接受删除修改，删除等操作，用来保护数块的安全性

- 在namenode启动时，hdfs会首先进入安全模式，集群会先检查数据快的完整性，

  datanode会向namenode汇报可用的block信息，等到达到安全标准后会自动退出安全模式

- 命令：hdfs dfsadmin -safemode enter/leave

### 机架感知

- 通过配置一个脚本来进行映射  放在hadoop core-site.xml配置文件中，

  关键字为topology.script.file.name，用于namenode与jobtracker进行调用

- 实现DNSToSwitchMapping的接口中的resolve接口来完成网络位置的映射