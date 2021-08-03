## HIVE
> 是一个构建在hadoop上的数据仓库，可将结构化的数据文件映射为表，并提供SQL查询功能  
> 用于查询的SQL语句会被映射为MapReduce任务执行，提交到hadoop上执行

- 特点 
  - 简单，容易上手  
  - 灵活性高，可以自定义udf与存储格式 
  - 可以用于超大数据量的查询，集群扩展方便
  - 统一的元数据管理，可以与presto/impala/sparksql等共享元数据
  - 执行的延迟较高，不适合做数据的实时处理，适合海量数据的离线处理  
- HIVE体系架构
  - command line shell 通过hive命令行的方式来操作数据
  - thift/jdbc  通过thift协议按照标准JDBC来操作数据  
  - meta_store 表名，表结构，字段名，字段类型等称为元数据，所有的元数据会存储在RDMS中，默认为derby,且元数据可被共享
  - HQL执行流程如下：  
    - 语法解析：Antlr定义出SQL的语法规则，完成语法与词法的分析，将SQL转化为AST Tree语法树  
    - 语义解析：遍历AST Tree 抽象出基本组成单元QueryBlock
    - 生成逻辑执行计划： 遍历Query Block,翻译为执行树operatorTree 
    - 优化逻辑执行计划： 逻辑层
    - 生成物理执行计划
    - 优化物理执行计划
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
  - 新建数据库  create database xx if not exist database_name [comment 'xxx'] [location 'hdfs://path'] [with dbproperties]
  - 查看数据库信息  desc database [extended] database_name
  - 
