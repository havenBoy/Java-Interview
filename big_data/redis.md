## redis
> 是比较热门的key-value的NoSql系统之一，数据是缓存在内存中，且可以实现数据的持久化，可以降低数据库的压力

#### 一、数据类型

- String  set/get  incr/decr key
- hash  hset student name zhang/  hget key field / hdel key field
- list lpush key value/rpush key value/ lpop key/ rpop key/
- set sadd key value / srem key / smemmbers/

#### 二、redis持久化

- **AOF**
  
  redis会把一个写请求的记录写入到一个日志中，在重启时会把AOF文件中记录的写操作都执行一遍，
  以确保数据恢复包最新，默认不开启  使用appendonly yes  开启  
  
  - 优点：
  1. 数据基本不会丢失  
  2. 文件在断电时也不会发生损坏，可以进行修复  
  3. 文件格式易读，可修改  
  - 缺点：  
  1. AOF文件通常比rdb文件大  
  2. 性能消耗较高  
  3. 数据恢复比rdb慢  
  
- **RDB**
  
   是redis持久化的默认方式  定期保持数据快照到一个rdb文件，启动时会加载到内存中  
  
   配置为：**save 60 100**
  
   表示每60s或者100次变更会触发一次快照保存
  
  - 优点：
    1. 对性能影响最小，保存快照会fork出子进程进行，不会影响处理客户端请求  
    2. 每次快照会生成一个完整的数据文件  
    3. 使用rdb文件进行数据恢复比AOF快很多  
  - 缺点：
    1. 快照是定期生成的，在crash会丢失一部分数据   
    2. 数据集比较大时，影响对外提供的能力  

####  三、缓存雪崩

1. **缓存层挂掉**

  解决方案： 搭建redis的高可用  主从+哨兵或者redis+cluster

2. **缓存时间过期**

  解决方案： 请求全部到数据库进行查询  过期时间加上一个随机值  会大幅度减少在同一时间过期失效  

#### 四、缓存穿透

1. **表现为：**

   查询一个一定不存在的数据，由于缓存不存在，数据库也不存在，则不会写入缓存，则反复请求，会导致服务瘫痪 

2. **解决方案：**

​       使用布隆过滤器或者压缩filter提前拦截，不合法不进行数据的查询到数据库  

​       将空对象也设置到缓存中，下次请求就不会进行再次请求  

#### 五、安装部署启动（单机版）

1. ```shell
   wget   http://download.redis.io/releases/redis-5.0.7.tar.gz
   ```

2. 一般将redis的目录放置在/usr/local/redis下

3. tar -xf redis-5.0.7.tar.gz

4. cd redis-5.0.7  && make  这里是编译的步骤

5. make PREFIX=/usr/local/redis install    这里是安装的步骤

   这里多了一个关键字 **`PREFIX=`** 这个关键字的作用是编译的时候用于指定程序存放的路径

6. ./bin/redis-server& ./redis.conf       

   使用客户端    bin/redis-cli   即可连接

#### 六、集群版安装部署

#### 七、使用注意事项

- [ ] ##### Redis操作set、get等操作出现如下错误

   (error) MOVED 8352 192.168.145.128:6380 

  这种情况一般是因为启动 redis-cli 时没有设置集群模式所导致； 在开启集群后，redis-cli用普通用户登录无法操作集群中的数据，

  需要加上-c 用集群模式登录才可进行操作

- [ ] ##### 集群存在hash槽异常情况

​        ./redis-cli --cluster fix 192.168.200.162:6379   

- [ ] ##### 节点的添加

  cluster meet 192.168.200.161 6379 