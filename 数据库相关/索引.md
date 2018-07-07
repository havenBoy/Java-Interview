# 索引

> 主要讲述索引概念、索引种类、索引原理、使用索引需要注意的问题，以及如何开启索引的慢日志

## 什么是索引，索引的优缺点，索引的选择性
它是数据库中一列或者多列值进行排序的一种结构，用于包含数据表中所有记录的引用指针
- 优点：极大的提高搜索的速率；
- 缺点：提出索引就需要对其进行维护，成本较高；
- 索引的选择性：索引的不重复的索引值占数据库所有记录的比例，如果所有的选择性越高，那么索引的性能就越好，唯一索引就是如此；

## 索引的原理
- 首先创建索引的内容并进行排序;
- 把排序的结果倒排，并拼接数据的地址；
- 查询时会先拿到索引的内容后通过地址来拿到具体的数据；

## 创建索引需要注意的问题
- 索引的默认值不要设置为null，否则无效；
- 不使用负匹配的规则作用于索引，比如不使用 not in ，而是用 not exists代替，并且使用以下的运算符会停止索引的匹配：!=   like  < > 等；
- 不要在索引的列上进行运算，可以先对数据进行运算，然后在与索引列的数据进行比较；
- 定义有外键的列建议建立索引；
- 数据库类型为text、longtext、bit尽量不要建立索引，考虑：如果非要对比较长的字符串进行加索引，可以先对字符串进行hash求值，然后在加索引，或者索引开始的部分字符，；
- 索引遵循最左前缀匹配的规则，要求如下：
  * 1.索引的作用从左到右依次递减，经常创建索引把最常用的列放在左边；
  * 2.如果本列更新比较频繁，不建议建立索引；
  * 3.重复值较多的不建议建立索引，例如性别等；

## 索引的种类
- 普通索引：是一种最基本的索引，没有任何限制；
  * 创建索引：create index index_name on table_name(column_name(length));
  * 添加索引：alter table_name add idnex index_name on (column_name(length));
  * 删除索引：drop index index_name on table_name;
- 唯一索引：唯一的要求是索引值必须唯一,一般为表的主键设置，允许有空值；
  * 创建索引：create unique index index_name on table_name(column_name(length));
  * 添加索引：alter table_name add unique idnex index_name on (column_name(length));
  * 删除索引：drop unique index index_name on table_name;
- 主键索引：要求是索引值是唯一的，且不允许有空值，一般是在表的设计时，primary key('column_name');
- 组合索引：在多个列上创建的索引，遵循最左前缀的原则；
- 全文索引：主要用于查询文本中的关键字，一般是在fulltext字段，目前只能是char,varchar,text字段能够使用；（关键字fulltext）
  * 创建索引：create  fullindex index_name on table_name(column_name);
  * 添加索引：alter table_name add fulltext index_name on (column_name);
  * 删除索引：drop fulltext index_name on table_name;

## mysql如何开启慢日志
- 找出安装目录下的my.ini文件
- slow_query_log = on     
- slow_query_path = 慢日志文件存放的路径；
- long_query_time = 60  阈值的设定(如果查询的时间大于此设定的时间，则会记录)；