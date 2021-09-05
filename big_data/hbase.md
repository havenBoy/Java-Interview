# hbase
> 是一个构建在hadoop集群上面向列的数据库管理系统
>
> 基于google big table的数据模型，将数据存储在hdfs上，客户端可对hdfs的数据进行随机访问

## 为什么会有HBASE？

​        我们都知道，hadoop可以通过hdfs来解决存储结构化、半结构化、以及非结构化的数据，它是传统数据库的补充，是海量数据存储的最佳办法，

但是海量数据的随机访问是无法做到的，这个就意味者我们它只能顺序的访问数据，即使是最简单的访问也需要遍历所有的数据集，所以必须要有

一种方法来解决海量数据存储与随机访问问题，所以HBASE就诞生了。

> 数据结构分类
>
> 1. 结构化数据：关系型数据库存储的就是结构化数据
> 2. 半结构化数据：非关系型的，有基本固定的数据结构模式，例如日志文件，Xml文件，json文件等
> 3. 非结构化数据：没有固定结构的数据，如word，PDF，ppt，图片，视频等

## 一、特点

1. 不支持复杂的事务，支持行级事务，保证单行数据的读写原子性
2. 支持半结构化、结构化、非结构化数据
3. 支持数据分片
4. 支持region server之间的自动故障切换
5. 支持block cache 和布隆过滤器
6. 过滤器支持谓词的下推

## 二、HBASE table

​       它是一个面向列族的数据管理系统，表结构只需要定义列族schema，每个列族可以包含多个列，列由多个单元格（cell）组成，每个单元格可以存储多个版本的数据，多个版本的数据按照时间戳区分。Hbase的表结构有以下的特点：

1. RowKey是列的唯一标识，所有的行按照RowKey的字典序进行排序；
2. 容量大：可以存储数百万列，上十亿列数据；
3. 面向列，每一列都单独存放，在查询时可有效降低IO负担
4. 稀疏性：空列可以不占有存储空间
5. 数据多版本：每个单元的数据可以有多个版本，按照时间戳排序，新的数据在上边
6. 存储类型：底层统一采用byte数组存储

## 三、Phoenix

​       它是HBASE开源的SQL中间层，允许使用标准的SQL方式来操作hbase的数据，比hbase的标准API使用起来较为简单与方便，spring与mybatis都可以集成此插件，其次它的性能也是较高，一个查询可以转换为多个表的scan操作，之后并行执行来生成标准的jdbc数据集。可以为较小数据集查询提供毫秒级别的查询，为千万行数据查询提供秒级别查询性能，同时，它还包括了hbase本身不具有的二级索引。

- 与hbase 的集成
- 与spring boot的集成

## 四、关键概念

- ### Row key 行键

  是原来指定检索记录的主键，一般有三种方式来访问hbase中的数据

  1. 通过指定的row key来进行访问
  2. 通过row key的范围来查询，通过设置行键的起始位置与结束位置
  3. 通过全表扫描

  row key是可以为任意的字符串，存储数据时按照字典顺序来进行排序，

  **设计row key的原则为**

  1. 越短越好，一般不超过16个字节，内存一般是8字节对齐，可以利用操作系统的最佳特性（长度原则）

  2. 一般可以为随机数，UUID，hash算法。（**散列/随机原则**）

     > 注意：如果是以时间字符串作为rowkey时，一般建议将时间字符串放在后缀，前边加上随机值，这样可以将时间随机分布在几个region上，降低数据查询时集中在个别regionServer上。

  3. 在设计上必须保证row key的唯一性（**唯一性**）

- ### column family  （列族）

  表中的每一个列必然属于某个列族，列族是表schema的一部分，所以需要在建表时指定。列族的所有列都以列族作为前缀，

  比如:    info:name , info:age等

- ### column qualifier （列限定符）

  可以理解为列名，比如上述描述中name与age字段都可以称为info列族的列限定符，但列限定符不是表schema的一部分，在数据插入时可以动态指定

- ### column （列）

  列是由列族与列限定符组成，一般一个列的组成为**列族：列限定符**

- ### cell (单元格)

  可以理解为指定行和指定列确定的一个单元格，在hbase中可以由行、列族、列限定符、包含值与时间戳组成，一个单元格中的数据是由多个版本的数据组成，每个版本的数据按照时间戳区分

- ### timestamp（时间戳）

  HBASE中通过row key与column来确定的单元格为一个存储单元，每个单元格存放一个数据的多个版本，版本的索引为时间戳，类型为64位整型，在写入数据时自动赋值，当然也可以客户端显示指定，按照时间戳的倒序排序

### 五、存储结构

- Regions
  1. 数据的所有行按照row key的字段字典序排列，按照行键的范围来水平切分为多个region，起始为start key，结束为end key
  2. 每个表的只有一个region，通过数据的不断增加，region会不断增大，当增大到一个阈值时，会切分为2个region
  3. 当列越来越多时，也会切分为越来越多的region
  4. region是HBASE中分布式存储与负载均衡的最小单元，一个region只能分布在一个region Server上，不能拆分
- Region Server
  1. region server运行在hdfs的data node上，包含了以下的组件：
  2. WAL（write ahead log  预写日志） 用于存储未持久化的数据在内存中，以便故障恢复
  3. BlockCache （读缓存），将经常读取的数据缓存在内存中，如果存储不足，按照最近最少使用原则将缓存清除
  4. MemStore（写缓存），用于存储尚未写入磁盘的数据，在数据写入磁盘前会将数据进行排序，region上的每个列族都会有一个
  5. HFile （数据持久化文件），将行数据按照key/value形式存储在文件系统上

### 六、系统架构

HBASE是一个标准的master/slave架构，由三个组件组成：

- ### Zk 

  1. 保证集群中始终只有一个Master
  2. 存储所有Region的入口地址
  3. 实时监控Region server的状态，将其上下线的状态告知master
  4. 存储HBASE的schema，包括表名称以及表的列族信息

- ### Master

  1. 为Region Server分配region
  2. 负责Region Server的负载均衡
  3. 处理schema的变化请求
  4. GFS的垃圾文件回收

- ### Region Server 

  1. 维护Master分配的region，处理region server IO请求
  2. 负责切分单个大的Region

- ### 协作

  

### 七、数据的读写流程

- ### 读数据流程

  1. 客户端从zk上获取meta表的数据，决定数据将通过那个Region Server来处理
  2. 访问数据的所在Region Server，客户端会缓存这些数据并更新缓存
  3. 客户端从行键所在的region server上获取数据
  4. 如果是获取相同的数据，客户端会从缓存中获取行键所在的Region Server，不用每次都请求数据，除非Region失效导致缓存过期

- ### 写数据流程

1. 客户端会向Region Server提交写的请求
2. Region Server会找到对应的Region
3. Region会检测数据的schema是否一致
4. 就数据写入Wal Log中
5. 将数据写入MemStore,当缓存已经满时，需要将数据刷新到HFile中

## 八、常用shell命令

- ### 基本命令

  1. 进入： hbase shell
  2. 获取帮助：help
  3. 查看服务状态：status
  4. 版本信息：version

- ### 表操作

  1. #### 查看所有表

     list

  2. #### 创建表

     命令格式：create tableName columnFamily1 columnFamily2 ...

     eg:  create 'student' 'baseInfo', 'schoolInfo'

  3. #### 查看表基本信息

     describe table    eg:describe 'student'

  4. #### 表的禁用、启用

     禁用：disable ‘student’

     检查表是否被禁用：is_disabled 'student'

     启用表：enable ‘student’

     检查表是否被启用 ： is_enabled ‘student’

  5. #### 表是否存在

     exists 'student'

  6. #### 删除表

     1. 禁用表  disable ‘student’
     2. 删除表  drop 'student'

- ### 增删改

  1. #### 添加列族

     命令格式：alter 表名, 列族名称   eg   :alter 'student', ‘schoolInfo’

  2. #### 删除列族

     alter ‘表名’, {NAME => '表名', METHOD => 'delete'}

     alter 'student',  {NAME => 'teacherInfo', METHOD => 'delete'}

  3. #### 修改列族的存储版本，默认只有一个版本存储

     alter 'student', {NAME => 'teacherInfo', version => 3}

  4. #### 插入数据

     put ‘表名’,'行键','列族:列','值'，如果是行键、列族、列名与原来完全相同，则视为更新

     put ‘student’，'rowkey1',‘schoolInfo:grade’, 'six'

     put ‘student’，'rowkey2',‘baseInfo:name’, 'xiao_ming'

  5. #### 获取指定行列信息

     get '表名' ‘行键值’

  6. #### 获取指定行某列族下的所有列信息

     get '表名','行键值','列族'

  7. #### 获取指定行指定列的数据信息

     get '表名', '行键值', '列族:列名'

  8. #### 删除指定行

     delelte '表名','行键值'

  9. #### 删除指定行、指定列数据

     delete '表名', '行键值','列族:列名'

- ### 查询

  查询一般分为全表与指定行键值

  1. #### 查询
  
     get '表名' ‘行键值’
  
     get '表名','行键值','列族'
  
     get '表名', '行键值', '列族:列名'
  
  2. #### 查询全表数据
  
     scan '表名'
  
  3. #### 查询指定列族数据
  
     scan '表名', {COUMN => '列族'}
  
  4. #### 条件查询
  
     scan ‘表名’, {COLUMN => '列族:列名', STARTROW => 'rowKey1', STOPROW=>'rowKey2',LIMIT=>2, VERSION=>3}
  
     表示：从rowkey1开始，查找2个行的最小3个版本的数据信息
  
  5. #### 条件过滤

## 九、Java API使用

- 创建连接
- 创建表
- 删除表
- 添加数据
- 获取rowkey指定的数据
- 获取指定行指定列最新数据
- 获取所有数据
- 获取表中指定数据
- 其他

## 十、过滤器介绍

- #### 简介

  hbase提供了丰富的过滤器来提高数据的处理效率，用户可以使用自定义的过滤器或者内置的对数据进行过滤

  所有的过滤会在服务器过滤，不会在客户端进行，即**谓词下推**，这样可以降低客户端处理压力与网络传输

- #### 过滤器基础

  无论是内置的还是用户自定义过滤器，只要将定义好的过滤器通过setFilter方法传递给Scan或者PUT实例即可

- #### 过滤器的分类

  1. ##### 比较过滤器

     都继承于CompareFilter，创建一个过滤器需要2个参数，第一个是比较运算符，第二个是比较器实例

  2. ##### 专用过滤器

  3. ##### 包装过滤器
