## HIVE

> 是一个构建在hadoop上的数据仓库工具，可将结构化的数据文件映射为表，并提供SQL查询功能  
> 用于查询的SQL语句会被映射为MapReduce任务执行，提交到hadoop上执行

### 特点

- 简单，容易上手  
- 灵活性高，可以自定义udf与存储格式 
- 可以用于超大数据量的查询，集群扩展方便
- 统一的元数据管理，可以与presto/impala/sparksql等共享元数据
- 执行的延迟较高，不适合做数据的实时处理，适合海量数据的离线处理  

### HIVE体系架构

- command line shell 通过hive命令行的方式来操作数据

- thift/jdbc  通过thift协议按照标准JDBC来操作数据  

- meta_store 表名，表结构，字段名，字段类型等称为元数据，所有的元数据会存储在RDMS中，
  
  默认为derby,且元数据可被共享

### HQL执行流程

- 语法解析：Antlr定义出SQL的语法规则，完成语法与词法的分析，将SQL转化为AST Tree语法树  
- 语义解析：遍历AST Tree 抽象出基本组成单元QueryBlock
- 生成逻辑执行计划： 遍历Query Block,翻译为执行树operatorTree 
- 优化逻辑执行计划： 逻辑层
- 生成物理执行计划
- 优化物理执行计划

### hive 数据模型

- db: 在hive表现为hive.metastore.warehouse.dir下的文件夹
- table  表
- external table 数据存放
- partition  分区
- bucket 同一张表下哈希散列下的多个文件

### HIVE数据类型

- 整型：tinyint,smallint,int,bigint
- 布尔：boolean 
- 浮点型：float double
- 定点数：decimal(7,2) 用户自定义精度
- 字符串：string
- 日期类型：timestamp, date, timestamp with local timezone
- 二进制类型：binary

### 常见DDL语句

- 查看数据库  show databases;

- 使用数据库  use database_name;

- 新建数据库  create database xx if not exist database_name [comment 'xxx'] 
  
  [location 'hdfs://path'] [with dbproperties]

- 查看数据库信息  desc database [extended] database_name

- 创建分区

- 删除分区  会删除对应文件夹下的文件

- 特点 
  
  - 简单，容易上手  
  - 灵活性高，可以自定义udf与存储格式 
  - 可以用于超大数据量的查询，集群扩展方便
  - 统一的元数据管理，可以与presto/impala/sparksql等共享元数据
  - 执行的延迟较高，不适合做数据的实时处理，适合海量数据的离线处理  

- HIVE体系架构
  
  - command line shell 通过hive命令行的方式来操作数据
  
  - thift/jdbc  通过thift协议按照标准JDBC来操作数据  
  
  - meta_store 表名，表结构，字段名，字段类型等称为元数据，所有的元数据会存储在RDMS中，
    
    默认为derby,且元数据可被共享
  
  - HQL执行流程如下：  
    
    - 语法解析：Antlr定义出SQL的语法规则，完成语法与词法的分析，将SQL转化为AST Tree语法树  
    - 语义解析：遍历AST Tree 抽象出基本组成单元QueryBlock
    - 生成逻辑执行计划： 遍历Query Block,翻译为执行树operatorTree 
    - 优化逻辑执行计划： 逻辑层
    - 生成物理执行计划
    - 优化物理执行计划

- hive 数据模型
  
  - db: 在hive表现为hive.metastore.warehouse.dir下的文件夹
  - table  表
  - external table 数据存放
  - partition  分区
  - bucket 同一张表下哈希散列下的多个文件

- HIVE数据类型
  
  - 整型：tinyint,smallint,int,bigint
  - 布尔：boolean 
  - 浮点型：float double
  - 定点数：decimal(7,2) 用户自定义精度
  - 字符串：string
  - 日期类型：timestamp, date, timestamp with local timezone
  - 二进制类型：binary

- 常见DDL语句
  
  - 查看数据库  show databases;
  
  - 使用数据库  use database_name;
  
  - 新建数据库  create database xx if not exist database_name [comment 'xxx'] 
    
    [location 'hdfs://path'] [with dbproperties]
  
  - 查看数据库信息  desc database [extended] database_name
  
  - 创建分区
  
  - 删除分区  会删除对应文件夹下的文件
  
  - 

- join
  
  - inner join
  - left join
  - right join

- json字符串
  
  - insert overwrite table select get_json_object(line, '$.id') as id from xxx

- 函数
  
  - round   取整
  - floor   去除小数，最大的整数
  - cell    向上取整
  - rand    随机数
  - abs     绝对值
  - 日期函数  year/month/day/weekofyear
  - 字符串函数  length/concat/lower/reverse/split
  - 类型转换   cast(value as TYPE)
  - 爆炸函数   lateral view explode(col)
  - udf/udtf/udaf  一进一出 (extends UDF)/输入一行输出多行()/多行输入一行输出（extends UDAF）
  - 窗口函数  
    - NTILE
    - ROW_NUMBER  按照

### 内部表与外部表的区别以及转换

- 内部表也叫管理表，表的目录会放置在HDFS下
- 外部表会在创建时在指定的路径下创建表的目录，如果指定location路径不存在，则与内部表一样。
- 在删除表时，内外部表存在差异：  
  1. 内部表在删除时，会删除元数据信息以及hdfs上的目录以及数据  
  2. 外部表在删除时，不会删除hdfs上的目录以及数据，只会清除元数据信息
- 转换： alter table  xxx set tblproperties ('EXTERNAL' = 'TRUE/FALSE')  //注意这里是大写

### HIVE查询优化

- ##### JVM重用
  
  对于小文件较多，以及多个小任务的场景影响较大
  
  hadoop会默认使用派生的JVM进行运行map与reduce任务，如果一个任务中包含多个小任务，就会消耗大量资源
  
  JVM的重用可以优化一个任务中包含多个小任务的情况，使得一个JVM实例可以在一个job中使用多次
  
  这个参数可以在core-site.xml文件中配置，参数为mapreduce.job.jvm.numtasks
  
  如果开启JVM的重用，则一直会占用到task slot，直到任务执行完毕

- ##### 严格模式的含义
  
  hive.mapred.mode=strict/notstrict
  
  开启strict模式，
  
  对于分区表，不予许全表扫描，除非过滤条件中包含分区字段
  
  对于order by查询语句，必须要使用limit关键字，因为执行orderby后，会把数据分发到一个reduce中，加上limit避免执行时间长
  
  一般的解决办法是搭配使用此： clustered by col1   = distributed by col sort by col1
  
  hive的join语句与关系型数据库不同，不会自动加上on条件，所以会使用笛卡尔积join，会出现不可控制的情况

- ##### limit语句优化
  
  一般的limit语句是执行查询的语句后对结果进行截取，应该避免这样做，对于数据量大的表
  
  开启对结果的数据抽样以及设置最大抽取的条数：set hive.limit.optimize.enable=true；hive.limit.row.max.size
  
  设置控制读取结果的文件数量：hive.limit.optimize.limit.file
  
  **缺点是**：对于部分没有抽取到的数据，则永远不会被处理到

- ##### 并行执行
  
  一般来说，一个hive查询的任务会分为若干个阶段，包含MR阶段，抽取阶段，合并阶段，limit阶段
  
  但只是按照阶段顺序执行，对于没有依赖的阶段来说，没有必要按顺序执行，所以可以设置任务执行的并行度
  
  开启任务执行并行：hive.exec.parallel=true
  
  设置并行度的大小：hive.exec.parallel.thread.number

- ##### 输出合并小文件
  
  可设置在仅仅有执行map任务后，设置合并小文件的参数：hive.merge.mapfiles=true
  
  在执行完MapReduce后，进行合并小文件： hive.merge.mapredfiles=true
  
  可设置合并小文件大小的阈值，代表当大小超过此值后，会进行小文件的合并：hive.merge.samllfiles.avgsize
  
  注意这里是单独启动一个MR任务进行小文件的合并

- ##### fetch抓取
  
  保证某些特殊的请求，不走底层的hive查询
  
  hive.fetch.task.conversion = more(默认)/none/minimal
  
  more:  会在表抽样，limit语句情景下不走MR
  
  none: 所有的查询都要走MR查询
  
  minimal: 部分limit 过滤分区时不走MR

- ##### 本地模式
  
  对于触发任务到集群查询与查询本身所花费的时间差距太大，数据量小到没有必要提交任务到集群
  
  这种情况下可以使用本地模式；
  
  hive.exec.mode.local.auto = true

- ##### MR优化
  
  1. 使用explain语句来查看当前查询的阶段，并进行适当的优化，合并stage
  2. 在MAP执行前执行小文件的合并，这样可以减少MAP的任务，具体的类为CombineHiveInputFormat
  3. 如果input处理的文件较大或者较为复杂时，可适当增加map的数量来降低map的处理数据量，每个切片的数据量尽量接近blockSize的大小

- ##### 表的优化
  
  1. 在查询列时，尽量使用明确的列名称，不要使用select *
  2. 使用join条件时，join的表应该先过滤，不要在后续的where语句中在进行过滤，可减少查询数据的量
  3. join优化，使得小表在前，大表在后，原因是前面的表会被装载在内存中
  4. group by的优化， 首先会在map端进行聚合，可设置聚合触发的条目，当有数据倾斜时，进行负载均衡
  5. count(distinct) 首先这个查询会在一个reduce中进行，当数据量大时，会很耗时。需要提前准备执行group by,这样会多一个任务执行
