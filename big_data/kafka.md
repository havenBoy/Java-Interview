## kafka
>  是由scala语言编写的，是一个分布式，分区，多副本，多订阅者的消息系统

- #### 优势对比
  
  - 可靠性：分布式，分区，复制与容错
  - 可扩展性：可扩容与缩容，无需停机
  - 性能：发布与订阅信息具有高吞吐量
  - 快：保证零停机与零数据丢失
  
- #### kafka组成
  
  - **broker**    一个或者多个实例  服务端实例
  - **topic**      一个消息的类别  每个topic包含一个或者partition
  - **partition**    分区，是物理概念
  - **producer**      发布消息到kafka的broker
  - **consumer**    消费者，从kafka的broker消费消息
  - **consumer group**     每一个consumer属于一个特定的consumer group 
  
- #### Shell

  - 查看topic
  - 创建topic
  - 删除topic
  - 生产数据
  - 消费数据
  - 主题信息描述

- #### kafka架构
  
  - producer
  - zookeeper
  - consumer
  
- #### kafka集群的数量计算

- #### kafka消费是否有序？
  
  kafka能够保证分区内消息的顺序，但不能保证整体的有序性 
  如果需要保证有序性，只要有一个分区即可
  
- #### 消费者组与分区的关系

- #### 生产者分区策略
  
  - 没有指定分区号，轮询
  - 没有指定分区号，指定key 会根据key%hashcode 分发到不同的分区内
  
- #### 数据丢失
  
  - 生产者保证数据的不丢失
    同步模式 ack = 1 只要leader收到，认为会后续同步，
    ack = 0 会分发数据，但不等待 ack = -1会等待所有副本收到数据
    
  - broker 采用分区副本机制，保证数据的高可用
  
  - 消费者保证消息不丢失 
    
    拿到数据后，数据会存储在hbase或者mysql中，但有时存储组件不可靠，导致数据没有写入但offset已经发生改变，
    
    主要是因为offset的异步提交 
    **解决方法**： 修改为手动提交，由于默认是采用自动提交offset导致的消息不同步
  
- #### 数据重复问题
  
  - 落表时，创建唯一主键或者索引，避免重复数据
  - 业务逻辑处理，查询时，判断是否存在，如果存在则不处理，如果不存在，则先插入redis，在进行业务的逻辑处理
  
- #### 数据查找过程
  
  - 通过offset确定数据保存在那个分区内的segement   .index(稀疏索引) .log（真正数据存储）
  - 查找对应segment里边的index文件， 如果没有找到offset会查询到最近一条数据，下一条数据即为查询的数据  
  
- #### auto.offset.reset值解析
  
  - latest    各分区下有已经提交的offset时，从提交的offset开始消费，无提交的offset的消息时，会消费新产生的
  
    该分区下的数据，是默认值
  
  - earliest  如果有提交的offset时，从提交的offset开始消费，但如果没有提交的offset时，会从头开始消费
  
  - none  只要有一个分区不存在已经提交的offset时，会抛出异常
  
- #### offset的手动提交与自动提交