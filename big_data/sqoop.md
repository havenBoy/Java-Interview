## sqoop

> 是一个常用的数据迁移的工具，主要用于不同存储系统之间数据的导入与导出

#### **sqoop简介**

- 导入数据：从关系型数据库导入数据到HDFS，hive,hbase等分布式文件存储
- 导出数据：从分布式文件系统中导出数据到关系型数据库中
- 原理：是将执行命令转化为MapReduce作业来执行数据的迁移，其不涉及reduce操作，是一个特殊的MapReduce作业

#### **安装部署**

建议使用sqoop1.x版本，还有sqoop2版本，但二者的版本不兼容。

- **下载与解压**

- **配置环境变量**

- **修改配置文件sqoop-env.sh.template**

- **拷贝mysql的驱动类到lib文件夹下**

- **验证**

  sqoop version

- ##### 基本使用命令

  - sqoop help  查看sqoop所有的命令，帮助命令
  - sqoop help 命令名称   查看指定命令的使用

#### **mysql与sqoop的相关操作**

- 查看所有的数据库

  ```shell
  bin/sqoop list-databases --connect jdbc:mysql://sangfor-abdi-node1:3306 --username factory --password Admin123@
  ```

- 查看某个数据库下所有的数据表信息

  ```shell
  bin/sqoop list-tables --connect jdbc:mysql://sangfor-abdi-node1:3306/data_factory --username factory --password Admin123@
  ```

- 导出mysql数据到hdfs中

  ```shell
  sqoop import \
  --connect jdbc:mysql://sangfor-abdi-node1:3306/data_factory \   
  --username factory \
  --password Admin123@ \
  --table t_project \
  --target-dir /zxx/person \  # 指定导出目录
  --delete-target-dir \ # 如果导出目录存在，就先删除
  --where "id = 1" \    # where指定条件
  --fields-terminated-by '\t' \  # 指定字段数据分隔符  \t 这里指的是制表符
  --m 1
  
  ##验证
  hadoop fs -ls /zxx/person
  hadoop fs -cat /zxx/person/part-m-00000
  
  # 增量模式 (根据某个主键值进行数据的筛选，前提是拿到上次最后同步的id值)
  sqoop import \
  --connect jdbc:mysql://sangfor-abdi-node1:3306/data_factory \
  --username factory \
  --password Admin123@ \
  --table t_project \
  --incremental append \ # append模式（追加模式，会对新的数据进行末尾追加）
  --check-column id \ # 检查列为id列
  --last-value 7 \ # id列上一个记录的值为7
  --target-dir /zxx/person/increment \
  --m 1
  # lastmodify模式(指定的字段必须为时间字段)
  sqoop import \
  --connect jdbc:mysql://sangfor-abdi-node1:3306/data_factory \
  --username factory \
  --password Admin123@ \
  --table t_project \
  # lastmodified模式
  --incremental lastmodified \ 
  # 检查列为数据更新的列
  --check-column update_time \ 
  # id列上一个记录的值
  --last-value '2021-10-19 11:21:01' \
  --target-dir /zxx/person/increment \ 
  --m 1 \
  ## （此参数会对已经map的数据块进行合并，根据指定的字段，生成一个jgu文件）
  --merge-key id 
  ```

#### HIVE的导入与导出

- 自动创建hive表（预先创建一个表）

  ```shell
  sqoop import \
  --connect jdbc:mysql://sangfor-abdi-node1:3306/data_factory \
  --username factory \
  --password Admin123@ \
  --table t_todo_application \
  # 这里需要和hive中分隔指定的一样
  --fields-terminated-by ',' \ 
  --delete-target-dir \
  # 导入hive
  --hive-import \  
   #hive表
  > --hive-database test \
  > --hive-table test \
  # 覆盖hive表中已有数据
  --hive-overwrite \ 
  --m 1
  ```

- 非自动创建hive表

  ```shell
  sqoop import \
  --connect jdbc:mysql://sangfor-abdi-node1:3306/data_factory \
  --username factory \
  --password Admin123@ \
  --table t_todo_application \
  # 这里需要和hive中分隔指定的一样
  --fields-terminated-by ',' \ 
  --delete-target-dir \
  # 导入hive
  --hive-import \  
   #hive表 (这里的需要注意：直接指定数据表，需要预先创建表才行。包括)
  > --hive-table test.test \
  # 覆盖hive表中已有数据
  --hive-overwrite \ 
  --m 1
  ```

- hive表数据导入到mysql数据库中

  ```shell
  bin/sqoop export \
  --connect jdbc:mysql://sangfor-abdi-node1:3306/data_factory \
  --username factory \
  --password Admin123@ \
  --table t_todo_application_copy \
  --fields-terminated-by '\u0001' \
  --export-dir '/warehouse/tablespace/managed/hive/zxx.db/test'
  ```

#### Hbase的导入与导出

```shell
sqoop import \
--connect jdbc:mysql://sangfor-abdi-node1:3306/data_factory \
--username factory \
--password Admin123@ \
--table t_todo_application \
## 指定hbase表名(自动创建)
--hbase-table hbase_table \ 
--hbase-create-table \ 
# 指定列族
--column-family f1 \ 
# 执行rowkey
--hbase-row-key id \ 
--m 1
```

