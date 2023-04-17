### hudi

#### 基础知识

#### 进阶内容

#### hudi-shade打包内容

1. 从gitee上下载源码到本地，使用git命令或者点击下载即可，如果是下载zip文件方式，确定好版本，版本使用的版本为0.12.1

![img](https://wjinfo.feishu.cn/space/api/box/stream/download/asynccode/?code=MDYyOTQwN2Q0MDY0NjZjYTM2NTI2NjEyMzgwZGFlOTlfUmpRUFNLSktZeGZ2bUkyQVpiQTA5SzlPN2d4SkNrdTFfVG9rZW46SzVUMWJQQzYzb0lGOGJ4YkRtZmNtNmp3bmhiXzE2ODE3MjIyMzY6MTY4MTcyNTgzNl9WNA)

2. 为了加速打包，一般采用linux服务器进行，在打包前需要将配置机器的maven环境

3. 在外网下载maven的linux版本，解压，配置maven_home地址以及生效，

   具体参见https://www.cnblogs.com/fuzongle/p/12825048.html

4. 将根目录的pom.xml文件的Maven Repository地址修改为阿里地址：

   https://maven.aliyun.com/repository/public，可加速下载

5. 当前使用的flink版本为1.15.2，hive版本为3.1.2，即可确定最终打包的命令为：

​       mvn clean package -DskipTests -Drat.skip=true -Pflink-bundle-shade-hive3  -Dflink1.15 -Dscala2.12

6. 修改目录packaging/hudi-flink-bundle/pom.xml内hive的版本，当前不需要修改，只是由使用的hive版本来确定

7. 由于有以下4个包在网上不能下载，需要手动在阿里云或者maven中央仓库不能获取，需要手动install到本地

​       以下为命令：下载的路径可以通过当前指定的groupId与artifactedId以及版本号确定

```Shell
   mvn install:install-file -DgroupId=io.confluent -DartifactId=common-config -Dversion=5.3.4 -Dpackaging=jar -Dfile=./common-config-5.3.4.jar
   mvn install:install-file -DgroupId=io.confluent -DartifactId=common-utils -Dversion=5.3.4 -Dpackaging=jar -Dfile=./common-utils-5.3.4.jar
   mvn install:install-file -DgroupId=io.confluent -DartifactId=kafka-avro-serializer -Dversion=5.3.4 -Dpackaging=jar -Dfile=./kafka-avro-serializer-5.3.4.jar
   mvn install:install-file -DgroupId=io.confluent -DartifactId=kafka-schema-registry-client -Dversion=5.3.4 -Dpackaging=jar -Dfile=./kafka-schema-registry-client-5.3.4.jar
```

8. 执行打包命令，耐心等待后，打包成功如下图：

![img](https://wjinfo.feishu.cn/space/api/box/stream/download/asynccode/?code=MjQ5ZmNmZjFhYjU3NTVlMWRmNjM5NDIzZTA2NzMxYWNfekRvMXBZUk1qeXhMSVV2Z0FYZkJXUGpLYWYzU0F6QlBfVG9rZW46WktNWWIzSjVNbzJvQUN4MHNlU2NacUlwbkhlXzE2ODE3MjIyMzY6MTY4MTcyNTgzNl9WNA)

9. 可在根目录/packaging/hudi-flink-bundle/target下找到关于hudi-flink1.15-bundle-0.12.1.jar0

10. 在streampark环境上即可使用该jar包，正式的测试中，发现会报错，大致为：

![img](https://wjinfo.feishu.cn/space/api/box/stream/download/asynccode/?code=ZjFhMTllMDY1NTFlYTg3Njk4N2FjNjI0NzAxNmJhOGZfczBYMnZaWW1sZktqUUZMQ1lGcVlnUUlUNkdPNkY2Y0NfVG9rZW46VVR5b2JxYUJRb0FxN2d4Q0U4bWNQTFd4bmJ1XzE2ODE3MjIyMzY6MTY4MTcyNTgzNl9WNA)

定位到，需要一个关于此类的jar包进入到任务的自定义任务中，下载的路径为：

https://mvnrepository.com/artifact/org.apache.calcite/calcite-core，随便选择一个上传到任务自定义jar包中即可

11. 上线任务并运行，会发现hudi表元数据已经被同步，具体同步信息大致如下：即为成功

![img](https://wjinfo.feishu.cn/space/api/box/stream/download/asynccode/?code=MTZiMzQ5M2ExNzcyMWZhMTM4ZTdlNGU3ODc0NTMxYjNfcVU0WmhUZ2RqUkVQTWRQOHBpbUI3ajRrcG5MMmc2anpfVG9rZW46VzNLMmI3TndRb29iMFd4NlF6TWNiZGlDbmJnXzE2ODE3MjIyMzY6MTY4MTcyNTgzNl9WNA)