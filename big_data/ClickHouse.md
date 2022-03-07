# ClickHouse

## 一、简介

​        Yandex在2016年开源的列式存储数据库，使用了C++语言进行开发的，主要是用于在线分析处理查询（OLAP）

​        能够使用SQL查询实时数据并生成实时分析的数据报告。

OLTP：

OLAP：适合一次插入，多次查询

## 二、特点

1. 列式存储（理解）

   行式存储：

   - 比较代表的是常用的关系型数据库，适合数据的增删改查

   列式存储：

   - 更加适合查询场景，容易进行压缩操作，压缩比较好，
   - 对于列的聚合，分析比较友好

2. DBMS

   几乎涵盖了所有的SQL大部分的语法，包括DML与DDL，数据的存储管理与恢复，同时具有权限操作

3. 多样的引擎

4. 高吞吐的写入能力

   - 采用了类LMS Tree的结构，写入数据时，先写入缓存，后期会进行合并

   - 顺序写入，不可更改
   - 数据合并时，会有性能的损耗，几乎不能提供服务

5. 数据分区与线程并行

   - 数据进行分区，避免数据的全表扫描
   - 单个查询就可以吃掉所在机器的所有CPU
   - 对高qps的查询场景支持不是较好

6. 适合存储已经计算过的，且字段较多的大款表的数据存储

7. 比较擅长的是单表查询，而非多表的关联查询

   - 始终将右表加载在内存中，然后与左表进行匹配

## 三、安装部署

1. 确认防火墙关闭

2. centos取消打开文件数量的限制

   - ulimit -a
   - /etc/security/limits.conf          {user}@{group} hard/soft nofile/nproc 65536/131072
   - /etc/security/limits.d/20-nproc.conf
   - 重新登录即可生效
   - 集群环境内的文件覆盖

   - open files  文件打开数量
   - max user processes   用户最多打开的进程

3. 安装相关依赖

   yum install -y libtool

   yum install -y *ODBC*

4. 取消selinux

   - 修改/etc/selinux/config中的SELINUX=disable
   - 分发集群内的集群

5. 重启三台服务器即可

   - 临时失效使用命令：setenforce  0

6. 四个安装包   执行rpm -ivh *.rpm即可

7. rpm -qa | grep clickhouse

8. 修改config.xml文件，可以让当前机器的clickhouse服务让其他机器访问

9. 分发配置文件

10. 启动服务 clickhouse start

11. 连接时使用  clickhouse-client -m (可以使用空行)   --query "sql"

12. 默认使用9000端口

13. 要注意集群的安装部署：

## 四、数据类型

- 整型    int8   int16  int32  int64
- 浮点型  Float32  Float64
- 布尔值   Uint8  取值为0或者为1
- decimal32  decimal64 decimal128
- 字符串  String/fixedString(5)  补全字符数
- 枚举类型  Enum8/Enum16
- 时间类型Date/Datetime/Datetime64
- 数组类型
- Nullable

## 五、表引擎

在建表时，必须指定表的引擎，名称大小写敏感

- 指定数据存储的位置
- 不同的引擎支持的语法不同
- 并发的数据访问，有些不能支持
- 索引的使用
- 是否可以多线程的请求
- 数据复制的请求

##### 有以下的几种类型：

- TinyLog       以列文件的文件形式存储，不支持索引，不支持并发，多线程查询
- Memory      内存的引擎，以未压缩的形式存储在内存中，不支持索引
- MergeTree   最强大的引擎，是一个系列，支持索引与分区
  - partition by  可选 ，分区  避免全表扫描，优化查询速率，不存在则为all目录
  - order by  必须，只是提供数据的一级索引，但不是数据的唯一约束，为了能够使用主键
  - primary key 可选，稀疏索引，不是所有的数据都会被记录索引，之后在间隔内查询数据，主键必须为排序字段的前缀字段

##### 数据目录结构：

- 



##### 手动触发分区的合并

- optimize table table_name final;
- 观察数据目录的结构
- 指定合并的分区：optimize table table_name partition ‘20200601’ final;

##### 二级索引(一级索引不好使用)

- 在版本20.1.24后会自动打开，在这之前的版本需要手动将参数开启

  set allow_experrimental_data_skipping_indices = 1

- 在字段定义中声明

  INDEX a  field_name TYPE minmax GRANUARTITY 5

  **GRANUARTITY  N**    表示二级索引对于一级索引粒度的粒度

- 表是否包含索引

  - 在表数据存储路径下是否有跳表的索引目录
  - show create time table_name 可以展示是否有索引

##### 数据TTL

- 描述：引擎MergeTree可以管理数据表或者列的生命周期的功能

- 使用：

  字段级别：

  - 在列字段声明时加上  **TTL create_time+interval 10 SECOND**

  - 约束1：不能为主键的
  - 约束2：必须为时间类型
  - 修改：alter table xxx modify column xxx TTL create_time + interval 10 second

  表级别：

  - alter table xxx MODIFY TTL create_time + INTERVAL10 second

##### ReplacingMergeTree

- 描述

  是MergeTree的一个变种，多一个去重的功能，保证了primary key的唯一的约束

- 去重的时机

  只会在合并的过程中才会出现，合并的执行是在后台执行的且未知时间

- 去重的范围

  只能保证分区内的去重，但是不能保证跨区的唯一性，且在同一批插入或合并分区时才会去重

  认为重复的数据保留，版本字段值最大值会保留

SummingMergeTree

- 描述

  只能在分区内聚合，且不是实时聚合，同批数据插入会进行预聚合，如果不是在同一批，需要手动执行合并

  engine=SummingMergeTree(filed_name)

  对order by的字段进行预聚合

- 注意：

  - 其他的列以插入的顺序保留第一行

## 六、SQL操作

- insert 

  - insert into table values
  - insert into table select xxx from table 
- update/delete

  - 间接操作：alter table xxx delete/update 

  - 执行步骤：新增数据，新增分区，并把旧的分区打上失效的标记

  - 不支持事务
  - 每次进行操作时，都会创建新的分区，所以尽量使用批量的变更，不要单一的修改数据
- 查询

  - 支持子查询
  - 支持CTE，使用with语句
  - 支持使用join，但不建议
  - 窗口函数，暂时未开放
  - 不支持自定义函数
  - group by 操作增加 
    - with rollup  上卷（维度从右向左进行进行一次）
    - with cube  多维分析（所有的维度都要进行一次）
    - with total   只计算总计
- alter操作（与mysql的关键字alter基本保持一致）
  - 新增字段
  - 修改字段
  - 删除字段
- 导出数据
  - clickhouse-client --query "sql" > file

## 七、副本

- 主要是为了数据的高可性，保证一台节点宕机后，数据也可以从其他机器获取到相同的数据

- 配置的步骤：

  - zookeeper标签或者外部加载xml文件，默认的文件名称为metrika.xml

  - 修改xml文件的所属用户与组为clickhouse

  - 分发配置文件到其他节点上，包括config.xml以及配置本身文件

  - 重启   clickhouse restart，所有的节点

  - 表引擎必须为MergeTree的家族，参数为zk的数据路径，以及副本的名称，需要不同才行

    注意：副本只能同步数据，但是不能同步表结构，需要在新的节点上创建表结构

## 八、分片集群

将一块完整的数据分布在不同的节点上，称之为分布式表，本身是不存储表数据的

- 写入流程
  - 内部同步，由本身进行副本的同步（生产打开）
  - 非内部同步，由分布式表进行同步，但会有影响
- 读取流程
  - 是该读取那个副本的数据？优先选择error_count小的副本，
  - 如果是相同的，顺序，随机等任意算法
- 集群配置
  - 在资源不足的情况下会选择类似与kafka的分区与副本的存放方式
  - 资源充足的情况下还是采用多个节点存储多个分区与副本的方式
- 配置过程
  - metrika-shard.xml，注意文件的所属并修改
  - 分发到其他的节点上以及主配置文件本身
  - 其他的节点需要修改外部文件内的标签对应的分区与副本的名称
  - 重启，所有节点
- 建表语句
  - create table on xx cluster xxx
  - 指定分区的路径，以及副本的名称
  - 展示集群的命令：show clusters 
  -  如果是集群表，某一台建表语句会自动在其他的节点上创建
- 分布式表的创建来管理本地表

## [九、高级部分](https://github.com/havenBoy/Java-Interview/blob/master/big_data/clickhouse-high.md)















