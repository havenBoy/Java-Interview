## redis
> 是比较热门的key-value的NoSql系统之一，数据是缓存在内存中，且可以实现数据的持久化，可以降低数据库的压力

- 数据类型
  - String字符串  set/get  incr/decr key
  - hash  hset student name zhang/  hget key field / hdel key field
  - list lpush key value/rpush key value/ lpop key/ rpop key/
  - set sadd key value / srem key / smemmbers/
- redis持久化
  - aof redis会把一个写请求的记录写入到一个日志中，在重启时会把AOF文件中记录的写操作都执行一遍，以确保数据恢复包最新，默认不开启  使用appendonly yes  开启
    优点：数据基本不会丢失  
    文件在断电时也不会发生损坏，可以进行修复  
    文件格式易读，可修改  
    缺点：AOF文件通常比rdb文件大  性能消耗较高  数据恢复比rdb慢  
  - rdb 默认方式  定期保持数据快照到一个rdb文件，启动时会加载到内存中  
    save 60 100  每60s或者100次变更会触发一次快照保存  
    优点：对性能影响最小，保存快照会fork出子进程进行，不会影响处理客户端请求  
    每次快照会生成一个完整的数据文件  
    使用rdb文件进行数据恢复比AOF快很多  
    缺点：快照是定期生成的，在crash会丢失一部分数据   
    数据集比较大时，影响对外提供的能力  
  - 缓存雪崩
    缓存层挂掉  搭建redis的高可用  主从+哨兵或者redis+cluster
    缓存时间过期，请求全部到数据库进行查询  过期时间加上一个随机值  会大幅度减少在同一时间过期失效  
  - 缓存穿透
    查询一个一定不存在的数据，由于缓存不存在，数据库也不存在，则不会写入缓存，，则反复请求，会导致服务瘫痪  
    解决：使用布隆过滤器或者压缩filter提前拦截，不合法不进行数据的查询到数据库  
          将空对象也设置到缓存中，下次请求就不会进行再次请求  