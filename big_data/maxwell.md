# maxwell

#### 一、定义

- 定义

  是由美国的ZenDesk公司开源，是由java编写的MYSQL实时抓取数据，通过实时读取mysql中binlog日志生成Json数据，

  后来会被作为生产者发送给kafka/Redis等应用程序。

- 工作原理

​        主从复制：master把数据的变化写入到binlog日志中，之后，

​         salve数据库将向master数据库发送dump命令，把主库的二进制日志数据拷贝到delay日志中。

#### 二、安装

#### 三、工作原理

#### 四、入门使用

