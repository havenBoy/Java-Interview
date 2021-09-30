## flume
>  是一个分布式、高可用的数据收集系统
>
>  可支持不同数据源经过聚合操作后发送到存储系统中，通常用于日志数据的收集

#### 架构与基本概念

- Event

  是flume ng传输的基本单元，

- Source

  数据收集组件

  **种类：**

  1. Avro Source 
  2. Thrift Source
  3. Kafka Source
  4. Jms Source

- Channel

  是源与接收器之间的管道，用于存储临时数据，可以是内存或者持久化的文件系统

  **种类：**

  1. Memory Channel
  2. File Channel
  3. Kafka Channel

- Sink

  主要是从channel中获取Event，并将其存入外部存储系统或者下一个Source中，成功后会将Event从Channel中移除

  **种类：**

  1. Hdfs Sink
  2. HIve Sink
  3. Hbase Sink
  4. Avro Sink

- Agent

  是一个独立的JVM进程，包括了Source，Channel，Sink

#### 架构模式

#### 安装部署

#### 示例使用案例