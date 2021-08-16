## Ranger
> 是大数据领域一个集中式安全管理框架，目的是通过制定策略实现对Hadoop组件的集中式安全管理
> 实现对集群中数据的安全访问
- ranger组成
  1. Ranger Admin  用户管理策略，提供WebUI与RestFul接口
  2. Ranger UserSync 用于将Unix系统同步到Ranger Admin
  3. Ranger TagSync 同步atlas中Tag信息，基于标签的权限管理
  4. Ranger KMS 对hadoop KMS的管理策略与秘钥管理
- 依赖组件
  1. JDK用于运行RangerAdmin/RangerKMS
  2. python 2.7用于ranger的自动化安装
  3. git用于Ranger的编译
  4. maven3.6用于Ranger的编译
  5. RDMS用于存储授权策略，存储Ranger用户、组，存储审核日志 
- 支持扩展性组件
  1. HDFS
  2. HBASE
  3. HIVE
  4. YARN
  5. KAFKA
  6. ELASTICSEARCH
  7. KYLIN
- 编译打包
  1. 从github或者gitee获取到关于ranger2.1.0的源码
  2. 解压下载包到本地
  3. 注释pom文件中repository中的标签，使得直接访问本地仓库
  4. 