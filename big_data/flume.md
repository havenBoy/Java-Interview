## flume
>  是一个分布式、高可用的数据收集系统
>
>  可支持不同数据源经过聚合操作后发送到存储系统中，通常用于日志数据的收集

#### 架构与基本概念

- Event

  是flume ng传输的基本单元，是由标题与正文组成

- Source   数据收集组件

  **种类：**

  1. Avro Source 
  2. Thrift Source
  3. Kafka Source
  4. custom Source   实现自定义source，可以体现源的多样性，丰富收集的数据源

  **自定义source实现方法**

  1. 实现PollableSource （轮训拉取）与EventDrivenSource （事件驱动）接口
  2. 二者的区别在于：前者是通过线程不断调用process方法，主动的拉取消息，后者是需要一个触发一个调用的机制
  3. 在实现接口的同时，可以实现configurable接口，以便项目初始化某些配置使用
  4. 在实现的process方法之前，会首先调用start方法

- Channel    是源与接收器之间的管道，用于存储临时数据，可以是内存或者持久化的文件系统

  **种类：**

  1. Memory Channel
  2. File Channel
  3. Kafka Channel
  
- Sink    从channel中获取Event，并将其存入外部存储系统或者下一个Source中，成功后会将Event从Channel中移除

  **种类：**

  1. Hdfs Sink
  2. Hive Sink
  3. Hbase Sink
  4. Avro Sink
  
- Agent

  是一个独立的JVM进程，包括了Source，Channel，Sink
  
- Interceptor  拦截器

  在数据从source端到channel端运输过程中，可以使用拦截器对数据进行处理与加工

  也可对数据进行过滤，主要实现org.apache.flume.interceptor.Interceptor中单个或者多个事件拦截方法

#### 架构模式

source -> interceptor -> channel -> sink

#### 安装部署

- JDK安装

- 下载并解压安装包

- 配置环境变量

- 修改配置，包括java环境路径

- 校验安装是否成功

  flume-ng version

#### 示例使用案例

#### flume整合kafka

- **背景**

  实际的生产环境中，数据采集具有低谷与高峰的时机，如果是处于高峰时间点的话，数据会立马全部流入实时计算框架中，

  有可能会超出实时计算引擎的计算能力，这时kafka会具有消峰的作用，可以很好的抵抗峰值数据的冲击。

- **流程**

  1. 启动kafka
  2. 创建主题
  3. 启动kafka的消费者
  4. 配置flume的配置文件
  5. 启动flume日志采集的进程
  6. 测试，查看消费者端是否有数据消费

