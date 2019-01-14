### Hibernate与MyBatis区别

> 都是比较流行的ORM框架，HIbernate是全自动的，mybatis是半自动的；

* 1.最大的区别

  HIbernate不需要开发中编写sql，更专注于流程的逻辑，而Mybatis需要自己手动编写sql语句；

* 开发的难度相比

  HIbernate开发的难度高于Mybatis，由于HIbernate的学习周期较大；

  Mybatis是相对简单的，主要依赖于sql的编写，比较灵活；

* 数据扩展性

  HIbernate与数据库的关联存在于Xml配置文件中；

  Mybatis依赖于sql语句的编写，所以扩展性与迁移性较差；

* 缓存机制的比较

  相同点：存在系统默认的二级缓存，同时具有扩展自己设置的三级缓存的能力；

  不同点：

  HIbernate： SessionFactory配置文件中详细配置，然后在表对象映射配置中配置那种缓存；

  Mybatis： 每个具体表映射对象中详细配置，不同的表可以实现不同的缓存机制；

  二者比较：HIbernate如果在使用二级缓存中出现错误，那么系统会报出错误，但Mybatis需要注意cache的使用，容易出现脏数据对数据库的影响；

* Session的获取

  都是由SessionFactoryBuilder创建出SessionFactory，然后由SessionFactory创建出Session，由Session执行sql语句与事务；

  Mybatis是可以灵活的sql编码，简单容易实现；但数据的迁移性不太好；

  ​