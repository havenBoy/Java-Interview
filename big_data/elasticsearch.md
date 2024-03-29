### elasticsearch

基于Lucene的restful的分布式实时全文搜索引擎，可以快速的存储、搜索、分析海量的数据

全文检索是指对每一个词进行建立索引，指明该词在文章中的位置与出现次数，

当需要查询时，会首先根据建立的索引查找，从而起到加速查询的作用

#### 基本概念

1. 索引（index）: 

   类似于表的概念，是存放数据的地方

2. 类型（type）：

   用来定义数据结构，是一个逻辑数据分类

3. 文档（document）：

   是一条数据记录，是最小的数据单元 

4. 字段（field）：

   一个文档中可以包含多个字段，用来解释数据对象的属性

5. 分片（shard）：

   将一个索引中的数据切分为多个分片，分布在多态机器。后续横向扩展，提高吞吐量与性能

6. 副本（replication）：

   服务器在故障或宕机时， shard 可能会丢失，因此可以为每个 shard 创建多个 replica 副本。

   副本可以在shard故障时提供备用服务，保证数据不丢失，多个replica还可以提升搜索操作的吞吐量和性能

7. 索引模板（template）:

   可以帮助简化创建索引的语句，将模板中的配置和映射应用到创建的索引中。

   新建索引时，索引名称满足`index_patterns`条件的，将会使用索引模板中的配置和映射。

   可设置索引的匹配规则，分片数量，刷新时间、匹配order级别

#### 倒排索引

一个倒排索引由文档中所有不重复词的列表构成，对于其中每个词，有一个包含它的文档列表；

创建倒排索引时，将所有拆分的词条罗列，记录该词条在哪个文档中出现与出现位置，此称为倒排表；

倒排表顺序的存储在磁盘的某个文件中，这些文件称为倒排文件；

docvalues解决了？

在有聚合操作时，

#### text/keword

keyword不会进行分词，进而创建倒排索引，在匹配时必须完全匹配才行，而text会进行分词

#### query/filter

前者会进行查询，计算相关度以及相似度

后者只是判断是否会满足条件，不会进行相关度的分析，但会缓存结果，提高性能

#### Es写入数据流程

1. 客户端选择一个coordinator节点发送请求，节点会根据文档内容进行路由，转发到分区的主节点上
2. 实际的节点处理写入操作，之后会将数据同步到从副本节点上
3. 协同节点等待所有从副本的节点同步完毕，返回响应给到客户端

写入注意点：

1. 数据是先写入内存buffer中，当到达一定阈值后，刷新内容到文件缓存中，最后落盘；
2. 但由于内存buffer与文件系统缓存都是以系统内存为基础的，所以当发生断电或者宕机后，会将文件内容丢失
3. 基于此，新产生一个translog，先将文件内容写入内存buffer后，迅速将数据按照顺序写入此文件，在文件大小到达阈值后，写入文件系统缓存
4. 如果发生宕机，那么系统在恢复时，先将此变化的内容同步到文件系统缓存中
5. 从translog到文件系统缓存数据写入的过程看做是一次flush，只有进入文件系统缓存，此时数据才会被查询到
6. 更新与删除文档操作都是写操作，文档都是标记删除的，编辑是先标记删除，后新增一个新的文档
7. 在真正写入文件系统时，会进行文件段内容的合并，将标记删除的文档进行删除才是真正删除

#### 查询过程（query+fetch）

1. 客户端选择一个coordinator节点进行查询请求，该节点将查询请求广播到所有的主分片节点与副分片节点上；
2. 每个分片在本地创建一个page+size大小的优先队列，然后返回给协调节点，由协调节点进行合并，排序，分页并产出最终结果；
3. 协调节点通过doc_id值，去各个节点上进行获取真实的数据，返回协调节点；
4. 获取数据时，为了保证读操作可以随机分布在各个节点上，会对文档Id值进行hash处理，随机选择一个副本节点拉取数据；

#### Es读写一致性的保证

1. 对于数据的更新操作，每个文档即数据都会有一个版本号，乐观锁方式并发控制；
2. 对于写入数据，支持三种级别quorum/one/all，意义分别为：大多数节点/主分片节点/所有节点，默认为大多数分片节点；
3. 对于quorum/one可能都会导致数据的写入失败，其中的one代表只要有一个主分片节点；
4. 对于读操作，默认为主副本节点与从副本节点都返回才会最终返回；

#### Es主节点的选举细节

- 当大多数的master候选节点认为master不存在了，就发起master的选举
- 从所有的master候选节点进行选择，

#### 提高索引建立的优化方式

1. 使用SSD作为存储介质，因为索引的构建都是在内存中完成的；
2. 做大批量的数据导入时，建议先将索引的副本先设置为0；
3. 如果对查询结果没有实时的要求时，可将索引的刷新时间设置较大值，如30s
4. 默认的translog的溢写值为512M，可以调大为1G ；

#### 深度分页问题

- 一般的实现是以pageno + pagesize实现的，但由于数据返回具有最大backets数量限制，导致查询会出现错误

- 使用scroll_id，此方法实现的主要的优点是使用一个唯一的翻滚Id值去查询数据，数据返回快，

  但底层是基于数据的快照实现。无法查询快照后新增或者修改的数据。

- 使用search_after来实现，此方法实现的原理是，记录每次查询结果的最大值，然后作为下次查询的最小值，数据也是实时的

  缺点是：必须要有一个唯一键来约束递增值，
