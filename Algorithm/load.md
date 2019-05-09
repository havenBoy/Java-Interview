### 常见的负载均衡算法及其实现

- **随机**

  通过系统的随机函数，随机选取一台服务器访问，当访问数量的增加，实际效果越来越趋向于轮询效果；

  ~~~java
  public class Random
  {
      public static String getServer()
      {
          // 重建一个Map，避免服务器的上下线导致的并发问题
          Map<String, Integer> serverMap = 
                  new HashMap<String, Integer>();
          serverMap.putAll(IpMap.serverWeightMap);
          
          // 取得Ip地址List
          Set<String> keySet = serverMap.keySet();
          ArrayList<String> keyList = new ArrayList<String>();
          keyList.addAll(keySet);
          
          java.util.Random random = new java.util.Random();
          int randomPos = random.nextInt(keyList.size());
          
          return keyList.get(randomPos);
      }
  }
  ~~~

  ​

- **轮询**

  按访问的顺序，轮流分配到每一台机器上，而不关心其他的环境因素;

  优点：做到请求转发的绝对平衡；

  缺点：为了使得请求转发的绝对平衡而加入synchronized，这样造成的代码吞吐量发生明显下降；

  ~~~java
  public class RoundRobin
  {
      private static Integer pos = 0;
      
      public static String getServer()
      {
          // 重建一个Map，避免服务器的上下线导致的并发问题
          Map<String, Integer> serverMap = new HashMap<String, Integer>();
          serverMap.putAll(IpMap.serverWeightMap);
          // 取得Ip地址List
          Set<String> keySet = serverMap.keySet();
          ArrayList<String> keyList = new ArrayList<String>();
          keyList.addAll(keySet);
          
          String server = null;
          synchronized (pos) {
              if (pos > keySet.size())
                  pos = 0;
              server = keyList.get(pos);
              pos ++;
          }
          return server;
      }
  }
  ~~~

- **加权轮询**

  由于每台服务器负载能力的不同，所以给配置高，负载低的服务器配置较高的权重，使其处理更多的请求，而配置低，负载高的服务器配置较低的权重，使其处理较少的请求，这样就更大的发挥了服务器的优势；

  ~~~java
  public class WeightRoundRobin {
      private static Integer pos;
      public static String getServer() {
          // 重建一个Map，避免服务器的上下线导致的并发问题
          Map<String, Integer> serverMap = new HashMap<String, Integer>();
          serverMap.putAll(IpMap.serverWeightMap);
          
          // 取得Ip地址List
          Set<String> keySet = serverMap.keySet();
          Iterator<String> iterator = keySet.iterator();
          
          List<String> serverList = new ArrayList<String>();
          while (iterator.hasNext()) {
              String server = iterator.next();
              int weight = serverMap.get(server);
              for (int i = 0; i < weight; i++)
                  serverList.add(server);
          }
          String server = null;
          synchronized (pos) {
              if (pos > keySet.size())
                  pos = 0;
              server = serverList.get(pos);
              pos ++;
          }
          return server;
      }
  }
  ~~~


- **加权随机**

  --类似于随机法，是随机法与加权轮询法的结合

  ~~~java
  public class WeightRandom {
      public static String getServer() {
          // 重建一个Map，避免服务器的上下线导致的并发问题
          Map<String, Integer> serverMap = new HashMap<String, Integer>();
          serverMap.putAll(IpMap.serverWeightMap);
          
          // 取得Ip地址List
          Set<String> keySet = serverMap.keySet();
          Iterator<String> iterator = keySet.iterator();
          
          List<String> serverList = new ArrayList<String>();
          while (iterator.hasNext()) {
              String server = iterator.next();
              int weight = serverMap.get(server);
              for (int i = 0; i < weight; i++)
                  serverList.add(server);
          }
          java.util.Random random = new java.util.Random();
          int randomPos = random.nextInt(serverList.size());
          return serverList.get(randomPos);
      }
  }
  ~~~

  ​

- **源地址哈希**

  首先获取访问的IP地址，计算出IP地址的哈希值，然后对服务器的数量进行取模运算，得到服务器的序号；

  优点 :  保证了同一IP地址会被哈希到同一台服务器上，

  缺点：如果有服务器的上下线，会导致获取不到服务器的问题，一致Hash算法可以解决的问题；

  ~~~java
  public class Hash {
      public static String getServer() {
          // 重建一个Map，避免服务器的上下线导致的并发问题
          Map<String, Integer> serverMap = new HashMap<String, Integer>();
          serverMap.putAll(IpMap.serverWeightMap);
          
          // 取得Ip地址List
          Set<String> keySet = serverMap.keySet();
          ArrayList<String> keyList = new ArrayList<String>();
          keyList.addAll(keySet);
          
          // 在Web应用中可通过HttpServlet的getRemoteIp方法获取
          String remoteIp = "127.0.0.1";
          int hashCode = remoteIp.hashCode();
          int serverListSize = keyList.size();
          int serverPos = hashCode % serverListSize;
          
          return keyList.get(serverPos);
      }
  }
  ~~~

- 最小连接数

  根据后端服务器当前的连接情况，动态地选取其中当前积压连接数最少的一台服务器来处理当前请求，尽可能地提高后端服务器的利用效率，将负载合理地分流到每一台机器；