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
  2. python3用于ranger的自动化安装
  3. git，用于Ranger的编译
  4. maven3.6.2+，用于Ranger的编译
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
  0. 前置条件，环境中包含python3环境，git环境，npm前端环境，maven环境3.8.2+
  1. 从github或者gitee获取到关于ranger2.0.0的源码，https://gitee.com/mirrors/apache-ranger/repository/archive/release-ranger-2.0.0.zip
  2. 解压下载包到本地，unzip release-ranger-2.0.0.zip
  3. 注释pom.xml文件中repository中的标签，使得直接使用本地仓库，内容如下：
  ```xml
    <repositories> 
	  <!--
    <repository>
        <id>apache.snapshots.https</id>
        <name>Apache Development Snapshot Repository</name>
        <url>https://repository.apache.org/content/repositories/snapshots</url>
        <snapshots>
            <enabled>true</enabled>
        </snapshots>
    </repository>
    <repository>
        <id>apache.public.https</id>
        <name>Apache Development Snapshot Repository</name>
        <url>https://repository.apache.org/content/repositories/public</url>
        <releases>
            <enabled>true</enabled>
        </releases>
        <snapshots>
            <enabled>false</enabled>
        </snapshots>
    </repository>
    <repository>
      <id>repo</id>
      <url>file://${basedir}/local-repo</url>
      <snapshots>
         <enabled>true</enabled>
      </snapshots>
    </repository>
    -->
    </repositories>
  ```
  4. 将pom文件的各个关联组件，如hive,hdfs,yarn等组件的版本进行修改，修改问当前环境内的版本。以下为当前系统的版本号，建议进行修改。
  
  |  组件名称   |   版本号   |
  |  ----      |   ----     |
  |  hdfs      |   3.1.1    |
  |  yarn      |   3.1.1    |
  |  hive      |   3.1.0    |
  |  hbase     |   2.3.4    |
  |  kafka     |   2.0.0    |
  |  kylin     |   4.0.0    |
  |  elasticsearch  | 7.1.1 |
  5. mvn clean compile package assembly:assembly install -DskipTests=true -Drat.skip=true 跳过单元测试编译打包
  6. 命令执行成功后在当前文件下生成target目录，如下：  
  ```
  drwxr-xr-x. 2 root root        28 Aug 11 15:03 antrun
  drwxr-xr-x. 2 root root         6 Aug 11 15:06 archive-tmp
  drwxr-xr-x. 3 root root        22 Aug 11 15:03 maven-shared-archive-resources
  -rw-r--r--. 1 root root 169704679 Aug 11 15:09 ranger-2.0.0-admin.tar.gz
  -rw-r--r--. 1 root root 170617862 Aug 11 15:10 ranger-2.0.0-admin.zip
  -rw-r--r--. 1 root root  19496641 Aug 11 15:10 ranger-2.0.0-atlas-plugin.tar.gz
  -rw-r--r--. 1 root root  19527415 Aug 11 15:10 ranger-2.0.0-atlas-plugin.zip
  -rw-r--r--. 1 root root  17429311 Aug 11 15:07 ranger-2.0.0-hbase-plugin.tar.gz
  -rw-r--r--. 1 root root  17452830 Aug 11 15:07 ranger-2.0.0-hbase-plugin.zip
  -rw-r--r--. 1 root root  17193156 Aug 11 15:06 ranger-2.0.0-hdfs-plugin.tar.gz
  -rw-r--r--. 1 root root  17219795 Aug 11 15:06 ranger-2.0.0-hdfs-plugin.zip
  -rw-r--r--. 1 root root  17231265 Aug 11 15:07 ranger-2.0.0-hive-plugin.tar.gz
  -rw-r--r--. 1 root root  17253844 Aug 11 15:07 ranger-2.0.0-hive-plugin.zip
  -rw-r--r--. 1 root root  34477186 Aug 11 15:08 ranger-2.0.0-kafka-plugin.tar.gz
  -rw-r--r--. 1 root root  34516939 Aug 11 15:08 ranger-2.0.0-kafka-plugin.zip
  -rw-r--r--. 1 root root  74774626 Aug 11 15:10 ranger-2.0.0-kms.tar.gz
  -rw-r--r--. 1 root root  74882538 Aug 11 15:10 ranger-2.0.0-kms.zip
  -rw-r--r--. 1 root root  22985106 Aug 11 15:07 ranger-2.0.0-knox-plugin.tar.gz
  -rw-r--r--. 1 root root  23008841 Aug 11 15:07 ranger-2.0.0-knox-plugin.zip
  -rw-r--r--. 1 root root  17166995 Aug 11 15:11 ranger-2.0.0-kylin-plugin.tar.gz
  -rw-r--r--. 1 root root  17201439 Aug 11 15:11 ranger-2.0.0-kylin-plugin.zip
  -rw-r--r--. 1 root root     34225 Aug 11 15:10 ranger-2.0.0-migration-util.tar.gz
  -rw-r--r--. 1 root root     37740 Aug 11 15:10 ranger-2.0.0-migration-util.zip
  -rw-r--r--. 1 root root  18638604 Aug 11 15:10 ranger-2.0.0-ranger-tools.tar.gz
  -rw-r--r--. 1 root root  18648366 Aug 11 15:10 ranger-2.0.0-ranger-tools.zip
  -rw-r--r--. 1 root root  19525929 Aug 11 15:08 ranger-2.0.0-solr-plugin.tar.gz
  -rw-r--r--. 1 root root  19559534 Aug 11 15:08 ranger-2.0.0-solr-plugin.zip
  -rw-r--r--. 1 root root  17176648 Aug 11 15:11 ranger-2.0.0-sqoop-plugin.tar.gz
  -rw-r--r--. 1 root root  17207318 Aug 11 15:11 ranger-2.0.0-sqoop-plugin.zip
  -rw-r--r--. 1 root root   3726950 Aug 11 15:10 ranger-2.0.0-src.tar.gz
  -rw-r--r--. 1 root root   5690757 Aug 11 15:10 ranger-2.0.0-src.zip
  -rw-r--r--. 1 root root  35770483 Aug 11 15:07 ranger-2.0.0-storm-plugin.tar.gz
  -rw-r--r--. 1 root root  35797908 Aug 11 15:07 ranger-2.0.0-storm-plugin.zip
  -rw-r--r--. 1 root root  26929850 Aug 11 15:10 ranger-2.0.0-tagsync.tar.gz
  -rw-r--r--. 1 root root  26938089 Aug 11 15:10 ranger-2.0.0-tagsync.zip
  -rw-r--r--. 1 root root  11199628 Aug 11 15:10 ranger-2.0.0-usersync.tar.gz
  -rw-r--r--. 1 root root  11215668 Aug 11 15:10 ranger-2.0.0-usersync.zip
  -rw-r--r--. 1 root root  17181950 Aug 11 15:08 ranger-2.0.0-yarn-plugin.tar.gz
  -rw-r--r--. 1 root root  17213353 Aug 11 15:08 ranger-2.0.0-yarn-plugin.zip
  -rw-r--r--. 1 root root         5 Aug 11 15:11 version
  ```

- ranger-admin安装部署
>  此服务是ranger界面集成服务，完成部署后可登陆界面进行安全规则配置，同时服务的安全规则配置也支持API的调用  
>  安装节点为集群内的任意节点，建议为主节点  
  1. 使用编译命令mvn clean compile package assembly:assembly install -DskipTests=true -Drat.skip=true
  2. 解压编译打包完成后target目录下ranger-2.0.0-admin.tar.gz文件到某个路径下。  
     tar -zxvf ranger-2.0.0-admin.tar.gz -C /opt/
  3. 修改install.properties文件内容，内容如下：  
  ```
  SQL_CONNECTOR_JAR=/usr/share/java/mysql-connector-java.jar #需要指定一个数据库连接jar包路径
  db_root_user=root       #数据库管理员用户名
  db_root_password=*****  #数据库管理员密码
  db_host=localhost       #数据库主机，如果不在一台机器，请修改
  db_name=ranger          #数据库名字，用于保存记录安全规则
  db_user=root            #对应管理ranger数据库的用户
  db_password=****        #对应管理ranger数据库的密码
  policymgr_external_url=http://localhost:6080  #界面服务提供默认服务地址，如果有需要修改端口，在此修改
  ```
  4. 赋予数据库用户root用户在localhost与当前节点管理网地址的所有权限，原因是需要使用root用户创建关于ranger数据库，如下示例：
  ```sql
  flush privileges;
  grant system_user on *.* to 'root';
  drop user'root'@'localhost';
  create user 'root'@'localhost' identified by '123456';
  grant all privileges on *.* to 'root'@'localhost' with grant option;

  drop user'root'@'10.58.14.201';
  create user 'root'@'10.58.14.201' identified by '123456';
  grant all privileges on *.* to 'root'@'10.58.14.201' with grant option;
  flush privileges;
  ```
  5. 初始化服务，执行命令./setup.sh
  6. 启动ranger-admin，执行命令ranger-admin start 
  7. 界面输入policymgr_external_url中设置的url，进行登录ranger界面，默认账户/密码为admin/admin

- 与各个组件集成
  - hdfs 
    1. 登录ranger-admin的管理界面后，创建与hdfs的权限认证集成。
    2. 在编译后的target文件中找到hdfs相关的插件包为ranger-2.0.0-hdfs-plugin.tar.gz
    3. 解压到/opt/下，tar -zxvf ranger-2.0.0-hdfs-plugin.tar.gz -C /opt/
    4. 修改install.properties文件中的如下内容，其余内容可以保持不动：  
    ```
    POLICY_MGR_URL=http://10.58.14.201:6080  #策略web地址
    REPOSITORY_NAME=dev_hdfs                 #策略名称，在创建策略时会用到，与界面配置保持一致
    COMPONENT_INSTALL_DIR_NAME=/software/hadoop-2.7.2  #hdfs安装路径，即hadoop家目录地址
    ```
    5. 启动插件./enable-hdfs-plugin.sh，启动后需要重启hadoop以保证生效，安装部署时需要注意组件的先后顺序，同时升级也需要考虑
    6. 停止插件./disable-hdfs-plugin.sh，与启动类似，需要注意执行脚本后需要重启关联组件，以确保生效。
    7. 登陆ranger界面添加一个hdfs的安全策略服务，service_name与步骤4中的REPOSITORY_NAME保持一致，其余按照提示进行填写
    8. 添加一个策略的具体配置，可以选择配置相关的用户、用户组以及操作权限包括可读、可写、可执行。
    9. 配置完成后可以进行测试，验证用户如果没有权限时会出现报错信息。
  - kafka
  - yarn 
  - hive 
  - hbase
  - elasticsearch
- 原理描述
  - 通过读取安装组件时生成的配置文件以及组件自带的jar包，通过hook机制调用各个组件服务达到权限管理
  - 执行./enable-xx-plugin.sh建立hook机制
  - 将插件自带conf更新到系统安装服务下
  - 将插件自带的lib更新到系统安装的lib下
  - 将install.properties文件内容生成.xml文件，更新到系统组件安装服务的conf下
  - 服务重新启动，使得配置项生效

- ranger接口使用 （策略下发具有延迟）
  1. 服务相关接口（只管理本集群组件，在创建策略前创建服务数据） 当服务删除时，所有策略失效；服务具有互斥性，先定义先生效
    - 通过Id值查询服务详细信息
    - 获取所有服务详细信息
    - 创建服务接口
    - 服务删除接口

  2. 策略相关接口
  3. 用户与用户组相关接口
