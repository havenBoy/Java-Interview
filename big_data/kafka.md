## kafka
>  是由scala语言编写的，是一个分布式，分区，多副本，多订阅者的消息系统
>
>  主要用来处理流式信息，在实时计算中广泛使用

### part1: 基础信息

#### 优势对比

- 可靠性：分布式，分区，复制与容错
- 可扩展性：可扩容与缩容，无需停机
- 性能强大：发布与订阅信息具有高吞吐量
- 速度快：保证零停机与零数据丢失
- 缓冲与消峰：上游数据会有突发的流量，消息组件会有一个消息缓冲的作用
- 冗余性：小个生产者发布消息，可被多个订阅的topic消费到

#### kafka组成

- **broker**    一个或者多个实例 。是消息的代理点，是存储消息的服务站
- **topic**      一个消息的类别  每个topic包含一个或者多个partition
- **partition**    分区，是物理概念
- **producer**      发布消息到kafka的broker
- **consumer**    消费者，从broker拉取并消费消息
- **consumer group**     每一个consumer属于一个特定的consumer组

#### Shell

- 查看topic

  ```shell
   bin/kafka-topics.sh --zookeeper hostname:2181 --list
   
   --zookeeper 指的是zk的集群地址
  ```

- 创建topic

  ```shell
  bin/kafka-topics.sh --zookeeper hostname:2181 --create --replication-factor 3 --partitions 1 --topic topic_name
  
  --topic 定义topic名
  --replication-factor 定义副本数
  --partitions 定义分区数
  ```

- 删除topic

  ```shell
  bin/kafka-topics.sh --zookeeper hostname:2181 --delete --topic topic_name
  
  --zookeeper 指的是zk的集群地址
  ```

- 生产数据

  ```shell
  bin/kafka-console-producer.sh --broker-list hostname:9092 --topic topic_name
  --produce.config config/produce.properties
  --broker-list 指的是broker实例的集群节点
  ```

- 消费数据

  ```shell
  bin/kafka-console-consumer.sh  --bootstrap-server hostname:9092 --from-beginning --topic topic_name
  --from-beginning：会把first主题中以往所有的数据都读取出来，根据业务场景选择是否增加该配置,默认为latest
  --bootstrap-server：生产消息的服务器
  # 带有特定消费参数的消费数据，也可以适用于jass的权限体系查询
  bin/kafka-console-consumer.sh  --bootstrap-server hostname:9092 --topic topic_name 
  --consumer.config config/consumer.properties
  ```

- 主题信息描述

  ```shell
  bin/kafka-topics.sh --zookeeper hostname:2181  --describe --topic topic_name
  
  --zookeeper 指的是zk的集群地址
  ```

#### kafka架构

- producer
- zookeeper
- consumer

#### kafka高级配置项

- **cleanup.policy**

​       过期或达到日志上限的清理策略（delete-删除；compact-压缩）

​       默认值为delete

- **compression.type**

​       指定给该topic最终的压缩类型（uncompressed；snappy；lz4；gzip；producer）

​       默认值为producer

- **delete.retention.ms**

​       压缩日志保留的最长时间，也是消费端消息的最长时间。单位为毫秒

​      （默认值：86400000）

- **retention.bytes**

​       topic每个分区的最大文件大小。

​      （默认值：-1没有限制）

- **segment.bytes**

​       topic的文件是以segment文件存储的，该参数控制每个segment文件的大小

​      （默认值：1073741824）

- **segment.index.bytes**

​       对于segment日志的索引文件大小限制

​       （默认值：10M）

- **retention.ms**

​       日志文件保留的分钟数。数据存储的最大时间超过这个时间会根据cleanup.policy策略处理数据。

​       默认为7 * 24 * 60  min

#### kafka消费是否有序？

- kafka能够保证分区内消息的顺序，但不能保证整体的有序性 ，如果需要保证有序性，只要有一个分区即可

- 想办法在生产数据的时间，将需要顺序的数据放在同一个分区内。在生产数据时指定分区的位置

#### 消费者组与分区的关系

对于某个特定的分区只能被一个消费组中的一个消费实例订阅，不允许相同消费组内的不同消费实例订阅

如果想要提高消费的速率，那么设置一个消费组中存在主题分区数个消费实例进行消费

消费组的实例数不要超过分区数

#### 生产者分区策略

- 没有指定分区号，轮询，即Range
- 没有指定分区号，指定key 会根据key%hashcode 分发到不同的分区内，Hash

#### 数据丢失

- **生产者方面**
  ack = 1 只要leader收到，认为会后续同步，如果在接受到后leader宕机，并为进行同步操作，会出现数据丢失
  
  ack = 0 会分发数据，但不等待，不能保证数据不丢失
  
  ack = -1会等待所有副本收到数据，可保证数据不丢失
  
- **broker 方面**

  采用分区副本机制，保证数据的高可用

- **消费者方面**
  
  拿到数据后，数据会存储在hbase或者mysql中，但有时存储组件不可靠，导致数据没有写入但offset已经发生改变，
  
  主要是因为offset的异步提交 
  **解决方法**： 修改为手动提交，由于默认是采用自动提交offset导致的消息不同步

#### 数据重复问题

- 对于消费组消费offset值的记录分别有2套API可以使用，一种是高版本，一种是低版本
- 低版本消费的offset会自己维护，高版本的offset可以实现手动维护
- 高版本的API使用过程中，如果消息已经消费，在提交offset时，broker挂掉，就会导致offset不能正确记录，下次恢复会重复

- 解决的办法是：落表时，创建唯一主键或者索引，避免重复数据
- 业务逻辑处理，查询时，判断是否存在，如果存在则不处理，如果不存在，则先插入redis，在进行业务的逻辑处理

#### offset（消息偏移量）

**auto.offset.reset值**

- latest    

  各分区下有已经提交的offset时，从提交的offset开始消费，无提交的offset的消息时，会消费新产生的该分区下的数据，是默认值

- earliest  

  如果有提交的offset时，从提交的offset开始消费，但如果没有提交的offset时，会从头开始消费

- none  

  只要有一个分区不存在已经提交的offset时，会抛出异常

消费位移在老版本中被记录在zookeeper的目录中，但这个无疑会降低吞吐性

新版本的消费位移被记录在broker中的一个内置主题中

#### zookeer什么作用，可以不使用吗 ？ 

- 早期版本的kafka使用zk作为元数据的信息存储，包括消费组的管理，主题消费的状态以及offset值的记录
- 中后期的broker依然依赖zookeeper，作为controller的选举，broker存活的判断依据
- kafka发布到3版本后，彻底将zookeeper去掉，使用KRaft 模式，将元数据的信息存入kafka本身的架构中
- KRaft 模式使得kafka更加容易管理、监控与支持
- zookeeper依然可以在3版本中使用，后续4版本会将其彻底移除

#### follower是怎么和leader同步信息的？

- 副本概念：每个分区都会有多个副本，副本是由一个leader与多个follower组成，后续的消息的接受与消息的消费都是从leader中进行

  而follower的作用是时刻备份主副本的消息，尽可能的和主副本的消息保持一致

- ISR + OSR = AR   与主副本保持同步状态的副本 + 与主副本不能保持同步状态的副本 = 所有的副本，正常情况下OSR的个数应该为0

- 当主副本挂掉时，只有ISR中的副本才有机会成为leader， ISR中的副本如果长时间没有保持同步状态或者落后太多，就会变为OSR

- HW、LW、LEO    每个副本都会维护最后一条信息的offset + 1 ， 且副本的HW即为此值，代表最多可消费到此消息的前面消息

- 同步方式既不是单纯的同步方式，也不是单纯的异步同步，同步方式太消耗性能，异步方式不能保证消息的完整性

#### kafka为什么会这么快？

- 本身是分布式集群，采用分区技术，并发度比较高
- 写入时顺序写入，一直追加到文件的末尾，比内存的随机写入效率都高
- 零拷贝技术的使用，可以减少内核缓冲区到用户进程缓存区之间反复的IO拷贝操作，减少资源的竞争

#### leader发生宕机，ISR也不存在，发生什么？

- 默认配置项unclean.leader.election = true ,，可使得不同步的副本变为主副本。此时需要考虑数据的一致性；
- 如果配置为false，那么会一直等待leader恢复，此时kafka不可使用

#### message的结构体什么样子?

- 当生产者有发送消息时，会先将信息写入内存，内存满了后，会将消息写入副本

- topic消息的存储是按照分区存储的，分区名称会按照topic_name-1/topic_name-2。。。
- 一个分区可以分为多个segment，每个segment的大小可设置，当到达阈值时，就不再写入数据
- 分区内的一类是以log为后缀的文件，另一类是以index为后缀的文件，每一个log文件和一个index文件相对应，这一对文件就是一个Segment File
- log文件即为数据文件，index文件为索引文件，其中记录了对应消息文件中的物理偏移量
- 消息的文件名称以偏移量的大小命名，当需要查找消息在那个文件时，二分法查询文件名称，迅速定位所属文件，后续按照索引文件到对应消息文件查询
- 消息体的内容由固定长度的头信息与变化长度的消息体组成
- 头信息: 8位的唯一递增的序列ID值，4位的消息长度值，4位的CRC校验信息，1位服务协议版本号，1位的压缩或者编码类型等
- value bytes payload表示实际消息数据

#### kafka中发生rebalance的时机？

- 消费者发生变化时
- 分区数发生变化时
- 订阅主题的范围发生变化时

### part2: 权限管理相关

#### 一、基本概念

- **认证（authentication）**：包括client与broker以及broker与zookeeper之间的互相认证
- **信道加密（encryption）**：指的是client端与broker以及broker与broker之间数据传输时配置SSL，是一种非对称的加密方式，默认是关闭的
- **授权（authorization）** ：通过ACL命令行来使用用户的权限控制
- **SASL（Simple Authentication and Security Layer）**，是一个安全层框架，具体实现可以是kerberos、PLAIN或者SCRAM

对于中小型的kafka集群来说，用户系统并不复杂，集群用户也不会很多，而且会运行在内网环境，SSL加密也不会是很必要。

SASL/PLAIN而言，运维的成本较小，适合中小型kafka集群，弊端是不能动态进行用户的增减。

| 认证机制    | 引入版本 | 使用场景                                                     |
| ----------- | -------- | ------------------------------------------------------------ |
| SASL/GSSAPI | 0.9      | 适用于本身已经实现Kerberos认证的场景，需要给集群中的broker与访问用户申请principals |
| SASL/PLAIN  | 0.10.2   | 适应与中小型kafka集群，账户文件会配置到一个静态文件中，不支持动态增加减用户, 需要重启broker |
| SASL/SCRAM  | 0.10.2   | 适应于中小型kafka集群，可以将用户的信息存储在zk中，无需重启集群即可支持认证用户的动态增加减 |

#### 二、kafka集成SASL/PLAIN步骤

- **zookeeper配置SASL（可配置）**

    1. zoo.cfg文件配置

        ```properties
        authProvider.1=org.apache.zookeeper.server.auth.SASLAuthenticationProvider
        requireClientAuthScheme=sasl
        ```

    2. 新建zk_server_jaas_config文件，为zk添加账户认证信息

        ```properties
        Server {
            org.apache.kafka.common.security.plain.PlainLoginModule required
            ##这里的用户名与密码指的是zk集群之间认证的用户与密码
            username="admin	"
            password="Test123."
            ##这里使用user_{user} 指的是user用户的密码，用来定义zk客户端的用户与密码，按需添加
            user_kafka="zk_kafka_client_passwd";
        };
        ```

    3. 将kafka认证相关jar包拷贝到lib下

        ```
        kafka-clients-2.4.1.jar
        lz4-java-1.6.0.jar
        slf4j-api-1.7.28.jar
        slf4j-log4j12-1.7.28.ja
        snappy-java-1.1.7.3.jar
        ```

    4. 修改zkEnv.sh文件

        ```shell
        export SERVER_JVM_OPTIONS="-Djava.security.auth.login.config=/usr/hdp/3.1.0.0-78/zookeeper/conf"
        ```

- **kafka配置SASL/PLAIN**

    1. 创建kafka_server_jaas_conf文件， 内容如下:

        ```properties
        KafkaServer {
            org.apache.kafka.common.security.plain.PlainLoginModule required
        	    # username和password是broker用于初始化连接到其他的broker
                username="admin"
                password="admin" 
                # 下面定义了所有连接到broker和broker验证的所有的客户端连接包括其他broker的用户密码
                user_admin="admin"   
                user_alice="alice"; # 这里增加了alice用户，密码alice
        };
        kafkaClient {
            org.apache.kafka.common.security.plain.PlainLoginModule required
            username="admin" # 这里是kafka客户端连接broker的用户名
            password="admin"; # 这里是kafka客户端连接broker的密码
            # 上面应该是对应KafkaServer中的user_admin配置项
        };
        ```

    2. 传递配置信息到启动参数中，在kafka-env.sh文件中添加如下一行：

        ```shell
        export KAFKA_KERBEROS_PARAMS="-Djava.security.auth.login.config=/usr/hdp/3.1.0.0-78/kafka/conf/kafka_jaas.conf"
        ```

    3. 配置server.properties文件，内容追加如下参数：

        ```properties
        advertised.listeners=SASL_PLAINTEXT://host.name:port
        sasl.enabled.mechanisms=PLAIN
        sasl.mechanism.inter.broker.protocol=PLAIN
        security.inter.broker.protocol=SASL_PLAINTEXT
        allow.everyone.if.no.acl.found=false   ## 需要配置，否则所有的权限会暴露出来
        
        authorizer.class.name=kafka.security.auth.SimpleAclAuthorizer
        advertised.listeners=SASL_PLAINTEXT://ip:port
        super.users=User:admin
        ```

#### 三、kafka配置SASL/SCRAM

1. 创建一个关于jass_conf的文件

    ```properties
    KafkaServer {
      org.apache.kafka.common.security.scram.ScramLoginModule required
      ##org.apache.kafka.common.security.plain.PlainLoginModule required
      username="admin"
      password="Test123."
      user_admin="Test123."
      user_admin1="Test123.";
    };
    
    KafkaClient {
      org.apache.kafka.common.security.scram.ScramLoginModule required
      ##org.apache.kafka.common.security.plain.PlainLoginModule required
      username="admin"
      password="Test123."
      user_admin="Test123.";
    };
    ```

2. server.properties文件的配置新增如下：

    ```properties
    advertised.listeners=SASL_PLAINTEXT://host.name:port
    sasl.enabled.mechanisms=SCRAM-SHA-512
    sasl.mechanism.inter.broker.protocol=SCRAM-SHA-512
    security.inter.broker.protocol=SASL_PLAINTEXT
    allow.everyone.if.no.acl.found=false   ## 需要配置，否则所有的权限会暴露出来
    
    authorizer.class.name=kafka.security.auth.SimpleAclAuthorizer
    advertised.listeners=SASL_PLAINTEXT://ip:port
    ##这里指定出最高权限的用户
    super.users=User:admin

3. 使用脚本kafka-configs.sh初始化出最高权限用户admin的密码：

   ```shell
   ## 创建用户，并指定密码
   /usr/hdp/3.1.0.0-78/kafka/bin/kafka-configs.sh --zookeeper 192.168.0.35:2181 --alter --add-config 'SCRAM-SHA-512=[password=Test123.]' --entity-type users --entity-name admin
   ```

   **后续普通用户的创建也可以基于此命令行，原因是没有与之对应的java_api可以使用，使用版本需要在2.7.0以上才可以**

   ```shell
   ##用户列表
   /usr/hdp/3.1.0.0-78/kafka/bin/kafka-configs.sh --zookeeper 192.168.0.35:2181 --describe --entity-type users
   ##创建用户admin1
   /usr/hdp/3.1.0.0-78/kafka/bin/kafka-configs.sh --zookeeper 192.168.0.35:2181 --alter --add-config 'SCRAM-SHA-512=[password=Test123.]' --entity-type users --entity-name admin1
   ##如果是编辑用户的密码时，使用相同的命令行，修改下密码即可
   ##创建用户组aa关于用户admin1
   /usr/hdp/3.1.0.0-78/kafka/bin/kafka-acls.sh --authorizer kafka.security.auth.SimpleAclAuthorizer --authorizer-properties zookeeper.connect=192.168.0.35:2181 --add --allow-principal User:admin1  --group aa
   ##删除用户组aa
   /usr/hdp/3.1.0.0-78/kafka/bin/kafka-acls.sh --authorizer kafka.security.auth.SimpleAclAuthorizer --authorizer-properties zookeeper.connect=192.168.0.35:2181 --remove --allow-principal User:admin1  --group aa
   ##删除用户admin1，并且移除掉相关zk的节点信息
   /usr/hdp/3.1.0.0-78/kafka/bin/kafka-configs.sh --zookeeper 192.168.0.35:2181 --alter --delete-config 'SCRAM-SHA-512=[password=Test123.]' --entity-type users --entity-name admin1
   ./zkCli.sh
   rmr /config/users/admin2
   ```

#### 四、java Api 操作ACL

```java
   /**
     * 操作Acl  **不支持更新**
     * @param resourceType  当前版本主要包括TOPIC以及GROUP
     * @param resourceName  资源名称，主要体现为主题的名称或者是组的名称
     * @param username      当前的acl对那个用户生效
     * @param operation     
     */
public void opreateACL(ResourceType resourceType, String resourceName, String username, AclOperation operation) {
    ResourcePattern resource = new ResourcePattern(resourceType, resourceName, PatternType.LITERAL);
    AccessControlEntry accessControlEntry = new 
        AccessControlEntry("User:" + username, "*", operation,      AclPermissionType.ALLOW);
    AclBinding aclBinding = new AclBinding(resource, accessControlEntry);

    //创建ACl
    CreateAclsResult createAclsResult = client.createAcls(Collections.singletonList(aclBinding));
    //删除Acl
    CreateAclsResult createAclsResult = client.deleteAcls(Collections.singletonList(AclBindingFilter.ANY));
    for (Map.Entry<AclBinding, KafkaFuture<Void>> e : createAclsResult.values().entrySet()) {
        KafkaFuture<Void> future = e.getValue();
        try {
            future.get();
            boolean success = !future.isCompletedExceptionally();
            if (success) {
                logger.info("acl create success");
            }
        } catch (Exception exc) {
            logger.warn("acl create error: {0}", exc);
            exc.printStackTrace();
        }
    }
}
/**
*  罗列出所有的ACL规则列表
**/
public void describeACL() {

    DescribeAclsResult describeAclsResult = client.describeAcls(AclBindingFilter.ANY);
    try {
        Collection<AclBinding> aclBindings = describeAclsResult.values().get();
        for (AclBinding get : aclBindings) {
            System.out.println(get.pattern().name());
            System.out.println(get.pattern().patternType());
            System.out.println(get.pattern().resourceType());
            System.out.println(get.entry().principal());
            System.out.println(get.entry().permissionType());
            System.out.println(get.entry().operation());
            System.out.println("-------------------------");
        }
    } catch (Exception e) {
        logger.error("acl list error");
    }
}

   /**
     * 罗列出所有的消费组
     */
public Set<String> listGroup() {
    Set<String> consumers = new HashSet<>();
    try {
        ListConsumerGroupsResult listConsumerGroupsResult = client.listConsumerGroups();
        consumers = listConsumerGroupsResult.valid()
            .get(10L, TimeUnit.SECONDS).stream().map(ConsumerGroupListing::groupId).collect(Collectors.toSet());
        System.out.println(consumers);
    } catch (Exception e) {
        logger.error("kafka consumer list error！", e);
    }
    return consumers;
}
```

| 问题描述                                                     | 答复                                  | 备注                                               |
| ------------------------------------------------------------ | ------------------------------------- | -------------------------------------------------- |
| 1. ACL策略是否可以支持更新？                                 | 不支持                                | 支持新增与删除，但不支持更新操作，不具有主键的概念 |
| 2. ACL策略的资源名称是否可以随意定义，即在添加时，是否进行资源的校验 | 不会                                  |                                                    |
| 3. ACL删除需要指定哪些参数？                                 | 如添加ACL的参数，具体参数参见新增参数 |                                                    |
| 4. 新增2条相同的ACL策略，是否会报错？                        | 不会报错，最终展示一条                | 自动覆盖旧的策略                                   |

#### 五、资源与权限点控制

| 资源类型 | 操作类型                                              | 备注                                                         |
| -------- | ----------------------------------------------------- | ------------------------------------------------------------ |
| TOPIC    | READ（读取）/WRITE（写入）/DELETE（删除）/ALL（全部） | 根据业务适当添加权限点，当前业务一般需要勾选ALL              |
| GROUP    | READ（读取）                                          | 不建议其他权限点，当前业务只有source有添加消费组标识<br />需要识别到，消费组随机生成，要有”所有“选项才可行 |

```java
UNKNOWN((byte)0), 未知
ANY((byte)1), 任意权限，带有任意的过滤条件
ALL((byte)2), 所有权限点
READ((byte)3), 读取
WRITE((byte)4), 写入
CREATE((byte)5), 创建
DELETE((byte)6), 删除
ALTER((byte)7), 修改，包括分区以及副本的修改
DESCRIBE((byte)8), 主题的描述权限
CLUSTER_ACTION((byte)9), 集群操作权限
DESCRIBE_CONFIGS((byte)10), 配置的描述权限
ALTER_CONFIGS((byte)11), 配置的修改权限
IDEMPOTENT_WRITE((byte)12); 幂等写权限
```

#### 六、框架选择

| 框架名称 | yahoo/kafka-manager                         | didi/LogiKm                            |
| -------- | ------------------------------------------- | -------------------------------------- |
| 版本号   | 2.6.                                        | 2.6.0                                  |
| 研发公司 | yahoo                                       | didi                                   |
| 开发语言 | scala                                       | java                                   |
| 体验地址 | https://10.58.56.180:9010/   admin/Test123. | http://10.58.56.180:8080/  admin/admin |
| 优势     |                                             |                                        |
| 劣势     |                                             |                                        |

#### 七、SSL/TSL安全连接方式

- 生成ssl相关证书（服务端）

  ~~~shell
  # 创建相关目录
  mkdir -p /usr/ca/{root,server,client,trust}
  
  # 生成server.keystore.jks文件，即服务端的keystore
  keytool -keystore /usr/ca/server/server.keystore.jks -alias ds-sangfor-abdi-node1 -validity 365 -genkey -keypass ds1994 -keyalg RSA -dname "CN=sangfor-abdi-node1,OU=aspire,O=aspire,L=beijing,S=beijing,C=cn" -storepass ds1994 -ext SAN=DNS:sangfor-abdi-node1
  
  # 生成CA认证证书
  openssl req -new -x509 -keyout /usr/ca/root/ca-key -out /usr/ca/root/ca-cert -days 365 -passout pass:ds1994 -subj "/C=cn/ST=beijing/L=beijing/O=aspire/OU=aspire/CN=sangfor-abdi-node1"
  
  # 通过CA证书创建一个客户端信任证书
  keytool -keystore /usr/ca/trust/client.truststore.jks -alias CARoot -import -file /usr/ca/root/ca-cert -storepass ds1994
  
  # 通过CA证书创建一个服务端器端信任证书
  keytool -keystore /usr/ca/trust/server.truststore.jks -alias CARoot -import -file /usr/ca/root/ca-cert -storepass ds1994
  
  #导出服务器端证书server.cert-file
  keytool -keystore /usr/ca/server/server.keystore.jks -alias ds-sangfor-abdi-node1 -certreq -file /usr/ca/server/server.cert-file -storepass ds1994
  
  #用CA给服务器端证书进行签名处理
  openssl x509 -req -CA /usr/ca/root/ca-cert -CAkey /usr/ca/root/ca-key -in /usr/ca/server/server.cert-file -out /usr/ca/server/server.cert-signed -days 365 -CAcreateserial -passin pass:ds1994
  
  # 将CA证书导入到服务器端keystore
  keytool -keystore /usr/ca/server/server.keystore.jks -alias CARoot -import -file /usr/ca/root/ca-cert -storepass ds1994
  
  # 将已签名的服务器证书导入到服务器keystore
  keytool -keystore /usr/ca/server/server.keystore.jks -alias ds-sangfor-abdi-node1 -import -file /usr/ca/server/server.cert-signed -storepass ds1994
  ~~~

- 生成ssl相关证书（客户端）

  ~~~shell
  # 导出客户端证书
  keytool -keystore /usr/ca/client/client.keystore.jks -alias ds-sangfor-abdi-node1 -validity 365 -genkey -keypass ds1994 -dname "CN=sangfor-abdi-node1,OU=aspire,O=aspire,L=beijing,S=beijing,C=cn" -ext SAN=DNS:sangfor-abdi-node1 -storepass ds1994
  
  # 将证书文件导入到客户端keystore
  keytool -keystore /usr/ca/client/client.keystore.jks -alias ds-sangfor-abdi-node1 -certreq -file /usr/ca/client/client.cert-file -storepass ds1994
  
  # 用CA给客户端证书进行签名处理
  openssl x509 -req -CA /usr/ca/root/ca-cert -CAkey /usr/ca/root/ca-key -in /usr/ca/client/client.cert-file -out /usr/ca/client/client.cert-signed -days 365 -CAcreateserial -passin pass:ds1994
  
  # 将CA证书导入到客户端keystore
  keytool -keystore /usr/ca/client/client.keystore.jks -alias CARoot -import -file /usr/ca/root/ca-cert -storepass ds1994
  
  # 将已签名的证书导入到客户端keystore
  keytool -keystore /usr/ca/client/client.keystore.jks -alias ds-sangfor-abdi-node1 -import -file /usr/ca/client/client.cert-signed -storepass ds1994
  ~~~

  

- ssl客户端设置步骤

- ssl客户端生产消费数据示例

- ssl java客户端代码设置

  ~~~java
  //一共是五个参数需要设置
  //configure the following three settings for SSL Encryption
  props.put(CommonClientConfigs.SECURITY_PROTOCOL_CONFIG, "SSL");
  props.put(SslConfigs.SSL_TRUSTSTORE_LOCATION_CONFIG, "/var/private/ssl/kafka.client.truststore.jks");
  props.put(SslConfigs.SSL_TRUSTSTORE_PASSWORD_CONFIG,  "test1234");
  
  // configure the following three settings for SSL Authentication
  props.put(SslConfigs.SSL_KEYSTORE_LOCATION_CONFIG, "/var/private/ssl/kafka.client.keystore.jks");
  props.put(SslConfigs.SSL_KEYSTORE_PASSWORD_CONFIG, "test1234");
  props.put(SslConfigs.SSL_KEY_PASSWORD_CONFIG, "test1234");
  ~~~

  

