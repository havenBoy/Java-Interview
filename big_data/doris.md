# Doris

### 一、简介

- 是由百度大数据研发开源的项目
- 是一个现代化的MPP（大规模并行处理）分析型数据库产品，只需要亚秒级的响应时间，有效支持实时数据分析
- 采用分布式的架构模式，易于运维，支持超大数据集，数量级在PB
- 架构角色包含了，FE（前端），BE（后台），MYSQL Client （与mysql集成的客户端），Broker（文件系统接口进程）

### 二、架构与角色

- ##### FE

  负责维护与存储集群的元数据、接收客户端的查询请求，规划查询请求，调度查询执行以及返回查询的结果

  主要包含以下3个角色：

  - leader&follower    达到元数据的高可用，可保证在单节点宕机的情况下，元数据可以实时在线恢复

  - Observer   用来扩展查询节点，同时起到备份元数据的作用，也可扩展查询的能力，其不参与任何的写入，只是读取。

- ##### BE

  负责物理数据的存储与计算，根据FE生成的物理计划，分布式执行查询。

  它对数据的存储是多副本的，这样可以保证数据的可靠性

- ##### Broker

  它是一个独立的无状态的进程，封装了文件系统接口，提供了远程读取存储系统文件的能力，如HDFS，S3等

- ##### Mysql Client

  Doris可以借助Mysql 协议，使用任意的mysql的JDBC或者ODBC或者Mysqld的客户端，可直接访问Doris

### 二、编译与安装

- ##### 安装Docker环境

- ##### 使用镜像环境编译

- ##### 注意事项

- ##### 默认端口

- ##### 集群部署规划

- ##### 部署FE

- ##### 部署BE

- ##### 在FE中添加或者声明BE节点

- ##### FE扩容与缩容

- ##### BE扩容或者缩容

### 三、数据表操作与语法

- ##### 用户与表或库的创建

  - 创建数据库      create database test_db;
  - 用户授权          grant all on test_db to test;

- ##### 基本概念

  ##### 数据以表的形式在逻辑上描述，下面使用Row与Column来说明表

  - Row是一行数据，Column是一行数据中不同的字段信息。
  - Column分为非排序列与排序列，存储引擎会按照排序列对数据进行排序存储，建立稀疏索引，方柏霓查找数据
  - 在聚合模型中，列可以分为2类，一种是Key，一种是Value

  ##### 在Doris的存储引擎中，用户数据会首先被分为多个Partition，划分的维度一般为用户制定的分区，比如时间。

  - 在每个分区内，数据会被按照Hash进行分桶，分桶的列是有用户制定，每个分桶是一个Tablet，是数据划分的最小单元
  - Tablet之间的数据是没有交集的，独立存储，它是数据移动或者拷贝的最小单元
  - 是逻辑上最小的管理单元

- ##### 建表语法

  ```sql
  CREATE [EXTERNAL] TABLE [IF NOT EXISTS] [database.]table_name
   (column_definition1[, column_definition2, ...]
   [, index_definition1[, index_definition12,]])
   [ENGINE = [olap|mysql|broker|hive]]
   [key_desc]
   [COMMENT "table comment"];
   [partition_desc]
   [distribution_desc]
   [rollup_index]
   [PROPERTIES ("key"="value", ...)]
   [BROKER PROPERTIES ("key"="value", ...)];
  ```

  - 支持单分区与复合分区，其中，复合分区代表既有分区也有分桶，单分区只是进行分桶，HASH分布即可
  - 第一分区指的是用户可以指定某一列作为分区列，支持整型数据以及时间类型，需要指定每个分区的范围
  - 第二分区为分桶，用户可以指定一个或者多个维度列，以及桶数

- ##### 字段类型

  | 字段类型        | 字段长度（字节） | 备注 |
  | --------------- | ---------------- | ---- |
  | TINYINT         | 1                |      |
  | SMALLINT        | 2                |      |
  | INT             | 4                |      |
  | BIGINT          | 8                |      |
  | LARGEINT        | 16               |      |
  | FLOAT           | 4                |      |
  | DOUBLE          | 12               |      |
  | DECIMAL         | 16               |      |
  | DATE            | 3                |      |
  | DATETIME        | 8                |      |
  | CHAR[(length)]  |                  |      |
  | VCHAR[(length)] |                  |      |
  | BOOLEAN         |                  |      |
  | HLL             |                  |      |
  | BITMAP          |                  |      |
  | STRING          |                  |      |


### 四、数据的导入与导出

### 五、查询

六、集成其他系统

七、监控与报警

八、优化

九、数据备份与恢复

