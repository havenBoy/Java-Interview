# flinkCDC

### 一、什么是CDC？

CDC是change data capture（变更数据获取）的简称

核心的思想是：监控与捕获数据库的变动，对于数据表与数据的增删改，将变更的数据记录下来，写入到消息的中间件中提供其他组件

### 二、CDC的种类

|                              | 基于查询的CDC            | 基于监控binglog        |
| ---------------------------- | ------------------------ | ---------------------- |
| 开源的产品代表               | sqoop、kafka jdbc source | Canal/Maxwell/flinkCDC |
| 执行的模式                   | 批处理                   | 流处理                 |
| 是否可以监控到所有数据的变化 | 否                       | 是                     |
| 延迟性                       | 高延迟                   | 低延迟                 |
| 是否会对数据库增加压力       | 是                       | 否                     |
| 对于全量数据的友好型         | 是                       | 否                     |

### 三、flinkCDC

这个是一个可以从关系型数据库（包括MYSQL、Postgresql，暂时不支持其他关系型数据库）直接获取全量数据或者部分的增量数据的source组件

### 四、组件对比（技术选型）

|                            | flinkcdc           | maxwell      | canal                                                        |
| -------------------------- | ------------------ | ------------ | ------------------------------------------------------------ |
| 初始化功能（读取全量数据） | 有，可支持多库多表 | 对单表的全量 | 不能支持                                                     |
| 封装格式种类               | 自定义反序列化格式 | JSON         | JSON                                                         |
| 高可用状态                 | 集群高可用即可     | 无           | 使用ZK                                                       |
| 断点续传                   | 支持，checkpoint   | mysql        | 本地磁盘存储                                                 |
| SQL->数据的对应关系        | 1对多              | 1对多        | 1对1，即使是影响到多条数据，<br />也会以一条数据返回所以需要单独对数据进行explode处理 |

五、cdc底层如何保证数据的一致性 ？

从原始文件与变更的数据进行比较合并方面。需要具体分析
