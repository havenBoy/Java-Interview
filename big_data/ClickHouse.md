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
  - primary key 可选，稀疏索引，不是所有的数据都会被记录索引，之后在间隔内查询数据，主键必须为排序字段的

##### 数据目录结构：

- 



##### 手动触发分区的合并

- optimize table table_name final;
- 观察数据目录的结构
- 指定合并的分区：optimize table table_name partition ‘20200601’ final;

































