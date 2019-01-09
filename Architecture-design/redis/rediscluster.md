# 构建Redis集群
> 主要介绍在Linux搭建Redis集群，以及如何动态的增加和删除Redis集群的节点

## 环境以及安装包准备
   - yum install gcc-c++  安装gcc环境
   - redis 安装包 需要3.0以上的可以支持集群
   - yum install ruby/yum install rubygems  集群管理工具redis-trib.rb安装需要ruby环境
   - gem install redis-3.0.0.gem     Redis与ruby的接口程序包
## 搭建思路
   - 由于本地环境的局限性 采用伪分布式搭建，端口7001~7006
   - 开启后台启动  cluster-enabled yes
   - 哈希槽一共有16384个，创建成功后会有槽分配的具体情况
   - 修改redis.conf 中端口port / 开启redis集群模式  cluster-enabled yes（放开注释）
   - 复制出6个redis实例在同一目录下
   - make install PREFIX=/usr/local/rediscluster/  redis安装路径
   - 执行集群创建命令
     *  ./redis-trib.rb create --replicas 1 192.168.21.128:7001 192.168.21.128:7002 192.168.21.128:7003 192.168.21.128:7004 192.168.21.128:7005  192.168.21.128:7006
     * 表示有三个主节点，每个主节点有一个从节点
     * replicas 1 表示每个主节点有一个从节点
   - 创建成功后，可以连接任意个节点进行访问，访问的方式为./redis-cli -h IP地址 -p 端口 -c (重要，表示以集群的方式连接)
## 测试集群
   - 客户端测试
     * 可以看到会存储在不同的节点上，在取时会跳转到存储的节点上
     ```java
       [root@xiong bin]# ./redis-cli -h 192.168.21.128 -p 7003 -c
        192.168.21.128:7003> get add
        -> Redirected to slot [1943] located at 192.168.21.128:7001
        "value"
        192.168.21.128:7001> set name 100
        -> Redirected to slot [5798] located at 192.168.21.128:7002
        OK
        192.168.21.128:7002>
      ```
   - java代码测试 加入redis的jar包
     * 需要的jar包  jedis-2.7.0.jar commons-pool2-2.3.jar
     ```java
      private JedisCluster jedisclu;
      @Test
      public void redisTest() {
        Set<HostAndPort> nodes = new HashSet<HostAndPort>();
        nodes.add(new HostAndPort("192.168.21.128", 7001));
        nodes.add(new HostAndPort("192.168.21.128", 7002));
        nodes.add(new HostAndPort("192.168.21.128", 7003));
        nodes.add(new HostAndPort("192.168.21.128", 7004));
        nodes.add(new HostAndPort("192.168.21.128", 7005));
        nodes.add(new HostAndPort("192.168.21.128", 7006));
        jedisclu = new JedisCluster(nodes);
        jedisclu.set("add", "value");
        System.out.println(jedisclu.get("add"));
        	jedisclu.close();
      }
     ```