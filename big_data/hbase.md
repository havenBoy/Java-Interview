# hbase
> 是一个构建在hadoop集群上面向列的数据库管理系统
>
> 基于google big table的数据模型，将数据存储在hdfs上，客户端可对hdfs的数据进行随机访问

## 为什么会有HBASE？

​        我们都知道，hadoop可以通过hdfs来解决存储结构化、半结构化、以及非结构化的数据，它是传统数据库的补充，是海量数据存储的最佳办法，但是海量数据的随机访问是无法做到的，这个就意味者我们它只能顺序的访问数据，即使是最简单的访问也需要遍历所有的数据集，所以必须要有一种方法来解决海量数据存储与随机访问问题，所以HBASE就诞生了。

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

- ### 协作说明

  

### 七、数据的读写流程

- ### 读数据流程

  1. 客户端从zk上获取meta表的数据，决定数据将通过那个Region Server来处理

  2. 访问数据的所在Region Server，客户端会缓存这些数据并更新缓存

  3. 客户端从行键所在的region server上获取数据

  4. 如果是获取相同的数据，客户端会从缓存中获取行键所在的Region Server，不用每次都请求数据，

     除非Region失效导致缓存过期

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

- pom依赖添加

  ~~~xml
      <dependency>
        <groupId>org.apache.hbase</groupId>
        <artifactId>hbase-client</artifactId>
        <version>2.3.4</version>
      </dependency>
      <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-api</artifactId>
        <version>1.7.25</version>
      </dependency>
      <dependency>
        <groupId>javax.servlet</groupId>
        <artifactId>javax.servlet-api</artifactId>
        <version>4.0.1</version>
        <scope>provided</scope>
      </dependency>
  ~~~

- 创建连接

  ~~~java
  Configuration configuration = HBaseConfiguration.create();
  configuration.set("hbase.zookeeper.property.clientPort", "2181");
  configuration.set("hbase.zookeeper.quorum", "127.0.0.1");
  configuration.set("zookeeper.znode.parent", "/hbase-unsecure");
  try {
      connection = ConnectionFactory.createConnection(configuration);
      admin = (HBaseAdmin) connection.getAdmin();
      TableName[] tableNames = admin.listTableNames();
  } catch (IOException e) {
      logger.error("连接创建失败！");
  }
  
  assert (connection != null && admin != null);
  logger.info("连接创建成功！");
  ~~~

- 创建表

  ~~~java
  TableName tableName = TableName.valueOf("test_1");
  if (admin.tableExists(tableName)) {
      return;
  }
  TableDescriptorBuilder builder = TableDescriptorBuilder.newBuilder(tableName);
  
  //列族创建指定
  List<ColumnFamilyDescriptor> familyDescriptors = new ArrayList<ColumnFamilyDescriptor>();
  ColumnFamilyDescriptorBuilder familyDescriptorBuilder1 = ColumnFamilyDescriptorBuilder
      .newBuilder(Bytes.toBytes("f1"));
  ColumnFamilyDescriptorBuilder familyDescriptorBuilder2 = ColumnFamilyDescriptorBuilder
      .newBuilder(Bytes.toBytes("f2"));
  familyDescriptors.add(familyDescriptorBuilder1.build());
  familyDescriptors.add(familyDescriptorBuilder2.build());
  builder.setColumnFamilies(familyDescriptors);
  
  admin.createTable(builder.build());
  ~~~

- 删除表

  ~~~java
  public static void dropTable() throws IOException {
      //这里是表名称
      TableName tableName = TableName.valueOf("test_1");
      if (!admin.isTableDisabled(tableName)) {
          admin.disableTable(tableName);
      }
      admin.deleteTable(tableName);
  }
  ~~~

- 添加数据

  ~~~java
      public static void put() throws IOException {
          //指定表名称
          TableName tableName = TableName.valueOf("test_1");
          //构建put请求时，需要将rowkey传入作为构造参数
          Put put = new Put(Bytes.toBytes("r1"));
          //指定列族，列名以及值
          put.addColumn(Bytes.toBytes("f1"), Bytes.toBytes("name"), Bytes.toBytes("xiong"));
          put.addColumn(Bytes.toBytes("f2"), Bytes.toBytes("school"), Bytes.toBytes("street"));
  
          HTable table = (HTable) connection.getTable(tableName);
          table.put(put);
      }
  ~~~

- 获取rowkey指定的数据

  ~~~java
  public static void selectOne2() throws IOException {
      String rowKey = "r1";
      Get get = new Get(Bytes.toBytes(rowKey));
  
      TableName tableName = TableName.valueOf("test_1");
      HTable table = (HTable) connection.getTable(tableName);
      Result result = table.get(get);
      logger.info("获取指定行键数据：{}", result);
  }
  ~~~

- 获取指定行指定列最新数据

  ~~~java
      public static void selectOne1() throws IOException {
          Get get = new Get(Bytes.toBytes("r1"));
  
          TableName tableName = TableName.valueOf("test_1");
          HTable table = (HTable) connection.getTable(tableName);
          Result result = table.get(get);
  
          logger.info("name:{}", Bytes.toString(result.getValue(Bytes.toBytes("f1"),                 Bytes.toBytes("name"))));
          logger.info("school:{}", Bytes.toString(result.getValue(Bytes.toBytes("f2"), Bytes.toBytes("school"))));
      }
  ~~~

- 获取所有数据

  ~~~java
      public void tableScanner() throws IOException {
          TableName tableName = TableName.valueOf("test_1");
  
          HTable table = (HTable) connection.getTable(tableName);
          Scan scan = new Scan();
          ResultScanner scanner = table.getScanner(scan);
          for (Result result : scanner) {
              logger.info("遍历的每一个结果：{}", result);
          }
      }
  ~~~

- 获取表中指定数据

  ~~~java
      public void tableScannerFilter() throws IOException {
          //指定表名称
          TableName tableName = TableName.valueOf("test_1");
          HTable table = (HTable) connection.getTable(tableName);
          //取10条数据
          PageFilter pageFilter = new PageFilter(10);
          //值过滤器，value = 10
          ValueFilter valueFilter = new ValueFilter(CompareOperator.EQUAL, new LongComparator(10));
          //列族过滤器，列族前缀等于f1
          FamilyFilter familyFilter = new FamilyFilter(CompareOperator.EQUAL, new BinaryPrefixComparator(Bytes.toBytes("f1")));
          //列限定发符=f1:name
          QualifierFilter qualifierFilter = new QualifierFilter(CompareOperator.EQUAL,
              new BinaryComparator(Bytes.toBytes("f1:name")));
          //rowkey = 子字符串包含f1
          RowFilter rowFilter = new RowFilter(CompareOperator.NOT_EQUAL, new SubstringComparator("f1"));
  
          //必须符合一个
          FilterList filterList = new FilterList(Operator.MUST_PASS_ONE);
          //必须所有的过滤器都通过
          //FilterList filterList = new FilterList(Operator.MUST_PASS_ALL);
          filterList.addFilter(pageFilter);
          filterList.addFilter(familyFilter);
          filterList.addFilter(valueFilter);
          filterList.addFilter(qualifierFilter);
          filterList.addFilter(rowFilter);
          Scan scan = new Scan();
          scan.setFilter(filterList);
          ResultScanner scanner = table.getScanner(scan);
          for (Result result : scanner) {
              logger.info("遍历的每一个结果：{}", result);
          }
      }
  ~~~

- 删除数据

  ~~~java
      public static void deleteData() throws IOException {
          //指定表名称
          HTable table = (HTable) connection.getTable(TableName.valueOf("test_1"));
          //指定rowKey
          Delete delete = new Delete(Bytes.toBytes("r1"));
          //指定列族
          delete.addFamily(Bytes.toBytes("f1"));
          //指定列族、列名
          delete.addColumn(Bytes.toBytes("f1"), Bytes.toBytes("name"));
          table.delete(delete);
      }
  ~~~

- 连接销毁

  ~~~java
      public static void destroy() throws IOException {
          admin.close();
          connection.close();
      }
  ~~~

## 十、过滤器介绍

- #### 简介

  hbase提供了丰富的过滤器来提高数据的处理效率，用户可以使用自定义的过滤器或者内置的对数据进行过滤

  所有的过滤会在服务器过滤，不会在客户端进行，即**谓词下推**，这样可以降低客户端处理压力与网络传输

- #### 过滤器基础

  无论是内置的还是用户自定义过滤器，只要将定义好的过滤器通过setFilter方法传递给Scan或者PUT实例即可

- #### 过滤器的分类

  1. ##### 比较过滤器

     都继承于CompareFilter，创建一个过滤器需要2个参数，第一个是比较运算符，第二个是比较器实例

     - 比较运算符包括：

       ~~~java
       @Public
       public enum CompareOperator {
           LESS,  //小于
           LESS_OR_EQUAL,//小于等于
           EQUAL,//等于
           NOT_EQUAL,//不等于
           GREATER_OR_EQUAL,//大于等于
           GREATER,//大于
           NO_OP;//非，取反
       
           private CompareOperator() {
           }
       }
       ~~~

     - 比较器实例都继承于**ByteArrayComparable**， 包括以下几种：

       | 比较器实例                | 描述                                                         |
       | ------------------------- | ------------------------------------------------------------ |
       | BigDecimalComparator      | 按照字典序对字节数组进行排                                   |
       | BinaryComparator          | 按照字典序对字节数组进行排序                                 |
       | BinaryComponentComparator |                                                              |
       | BinaryPrefixComparator    | 按照给出的字符串前缀对字节数组进行排序                       |
       | BitComparator             | 按位进行比较                                                 |
       | LongComparator            | 判断给定的数值是否相等                                       |
       | NullComparator            | 判断给定的值是否为空                                         |
       | RegexStringComparator     | 使用指定的正则表达式与指定字节数组进行比较，只能支持等于或者不等于 |
       | SubstringComparator       | 判断指定的字符串是否出现在指定的字节数组中                   |

     - 比较过滤器种类

       | 比较过滤器种类        | 描述                                     |
       | --------------------- | ---------------------------------------- |
       | DependentColumnFilter | 一个ValueFilter + 一个时间戳过滤器的组合 |
       | FamilyFilter          | 基于列族来过滤数据                       |
       | QualifierFilter       | 基于列名进行过滤数据                     |
       | RowFilter             | 基于行键过滤数据                         |
       | ValueFilter           | 基于单元格数据进行过滤                   |

  2. ##### 专用过滤器

     继承于filterBase类，适合更加精确，范围更小的查询

  3. ##### 包装过滤器

     通过包装现有的过滤器实现某些拓展的过滤功能

  4. ##### FilterList

  ​       当需要多个过滤条件进行查询时使用

  ​       可以使用filterList的构造器与addFilters方法进行传入多个过滤器

## 十一、协处理器

- #### 简介

  允许将业务的代码放在服务端regionServer执行，将处理好的数据返回给客户端，而不是将所有数据全部返回客户端，在客户端进行数据的过滤，这样可以降低需要传输的数据量，从而提高处理效率。在实际的应用中，权限的校验、二级索引都是使用协处理器来实现的。

- #### 类型

  - **Observer**

    类似于关系型数据库中的触发器，当某些事件发生后，这类协处理器会被server端调用，一般来说，权限的校验（在数据查询或者插入时，会调用preGet或者prePut方法）、二级索引（使用协处理器来维护二级索引）。

    一般来说分为以下四种类型：

    - RegionObserver     如Get或者Put操作  hbase.coprocessor.region.classes
    - RegionServerObserver   如服务的启动、停止、或者文件合并
    - MasterObserver  观察Hmaster的事件发生，如表的创建、删除或者结构的修改    hbase.coprocessor.master.classes
    - WalObserver  支持与预写日志相关的事件   hbase.coprocessor.wal.classes

    以上的四种类型的协处理器都继承于Coprocessor接口

  - **Endpoint**

    类似于关系型数据库中的存储过程，客户端可以在服务端调用此类协处理器来对数据进行处理，然后再返回数据。

    以求最大值为例，如果是把所有的数据全部返回到客户端再进行求聚合操作，客户端的压力会很大，如果采用协处理器，用户可以将求最大值的操作部署在RegionServer上，Hbase会触发每个节点并发执行求最大值的操作，将每个Region 的最大值返回，最后只要在客户端计算各个Region返回数据的最大值即可。

- #### **协处理的加载方式**

  - 静态加载

    静态加载一般为系统级的加载，作用的范围是HBASE上所有的表，需要重启Hbase服务才能生效；

  - 动态加载

    动态加载一般为表级别的处理器，作用于指定的表，而不是所有的表，不需要重启Hbase服务；

- #### 静态加载与卸载

  1. 在hbase-site.xml文件中定义需要加载的协处理器

     ~~~xml
     <property>
         <name>hbase.coprocessor.region.classes</name>  必须是特定的class，否则不能识别，类型在各个协处理器后边
         <value>org.myname.hbase.coprocessor.endpoint.SumEndPoint</value> 这里是自定义的协处理器实现类
     </property
     ~~~

  2. 将打包编译好的jar包放在hbase的安装目录下

  3. 重启hbase即可

  4. 卸载时，只要将xml文件中的内容移除，然后重启hbase即可。

- #### 动态加载与卸载

  一般是有2种方式，一个是Hbase shell ,一种是 java Api方式

  - **Hbase shell**    在动态加载表时必须要使得表脱机，即disable表
    1. 首先需要将表脱机，disable table_name  
    2. 使用如下的命令进行加载协处理器
    3. 启用表
    4. 验证是否启用成功
    5. 卸载时，也需要将表首先disable掉
    6. alter 'table_name' 
    7. enable table_name
  - **Java API**

## 十二、容灾与备份

- **copyTable**

  可以支持从现有的表的数据拷贝到新的表中，具有以下的特点：

  1. 在执行命令之前，需要创建相同结构的表
  2. 表拷贝的操作是使用JavaAPI，使用scan获取到数据，然后使用put将数据导入
  3. 支持时间区间、row区间，改变表的名称，列族名称等

  - **同集群下的拷贝：**

    ~~~shell
    hbase org.apache.hadoop.hbase.mapreduce.CopyTable \
    --new.name=table_new table_old
    ~~~

  - **跨集群的拷贝：**

    ~~~shell
    hbase org.apache.hadoop.hbase.mapreduce.CopyTable \
    --peer.adr=dstClusterZK:2181:/hbase 
    --new.name=new_table table_old(旧表名)
    ~~~

- **import/export**

- **snapshot**

## 十三、二级索引 
