## sqoop

> 是一个常用的数据迁移的工具，主要用于不同存储系统之间数据的导入与导出

- **sqoop简介**
  - 导入数据：从关系型数据库导入数据到HDFS，hive,hbase等分布式文件存储
  - 导出数据：从分布式文件系统中导出数据到关系型数据库中
  - 原理：是将执行命令转化为MapReduce作业来执行数据的迁移，其不涉及reduce操作，是一个特殊的MapReduce作业

- **安装部署**

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

- **mysql与sqoop的相关操作**

  - 

- ##### HDFS的导入与导出

- ##### HIVE的导入与导出

- ##### Hbase的导入与导出

