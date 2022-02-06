# Canal

## 一、什么是canal

- canal组件

​       通过读取数据库中的日志数据，来获取增量的数据

​       使用java开发的基于数据库增量日志数据解析，提供增量数据的消费与生产

- binlog 

​       是mysql的二进制的日志，记录了所有的DDL与DML，是以事件的形式进行记录，包含查询的消耗时间

​       开启二进制会大概有1%的性能损耗

- 应用场景
  - 主从同步
  - 

3.1 mysql的主从复制

3.2 伪装为从节点

## 二、mysql的准备

- 修改配置文件路径为/etc/my,cnf

- 重启mysql

- 设置canal连接用户的权限

三、canal安装与部署
