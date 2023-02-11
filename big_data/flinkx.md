### flinkx(chunjun)

是一款稳定、易用、高效、批流一体的数据集成框架，

目前基于实时计算引擎Flink实现多种异构数据源之间的数据同步与计算，

不同的数据源头被抽象成不同的Reader插件，不同的数据目标被抽象成不同的Writer插件。

理论上，FlinkX框架可以支持任意数据源类型的数据同步工作

#### 模式

单机模式：flink集群的单机模式

standalone模式：对应flink集群的分布式模式

yarn模式：对应flink集群的yarn模式

#### flinkx相关任务参数

主要以setting与content组成

content主要定义了reader与writer的结构，主要结构参见各个reader与writer的参数

setting参数主要包含了speed/ditry/errorlimit/restore参数等