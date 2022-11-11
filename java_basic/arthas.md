### arthas

>一个阿里开源的在线分析诊断工具，可帮助开发与运维人员迅速定位问题并解决问题

#### 一、如何启动

~~~
1. 获取到arthas的安装包，链接为https://alibaba.github.io/arthas/arthas-boot.jar
2. 使用java -jar arthas-boot.jar
3. 启动后，会罗列出所有的java进程，选择需要监控的进程的Id值，Enter进入即可
~~~

#### 二、使用场景

```
1、以全局视角来查看系统的运行状况、健康状况。 dashboard
2、反编译源码，查看jvm加载的是否为预期的文件内容。  jad  reference
3、查看某个方法的返回值，参数等等。  watch {params, returnObj} -x 3
4、方法内调用路径及各方法调用耗时。  trace reference
5、查看jvm运行状况
6、外部.class文件重新加载到jvm里
```

#### 三、具体细节

1. 查看某个方法调用路径与其耗时时间

   ```
   trace com.company.moudule method
   tt -t com.company.moudule method
   方法执行数据的时空隧道，记录下指定方法每次调用的入参和返回信息，并能对这些不同的时间下调用进行观测
   ```

2. 查看某个方法在调用时，入参与出参分别是什么

   ```
   watch com.company.moudule method "{parmas returnObj}" -x   其中x代表参数的深度
   ```

3. 查看与反编译当前环境内的class文件，确认环境内部代码是否为最新代码

   ~~~
   jad com.company.moudule.a.java
   ~~~


4. 使用完毕后，退出环境

   ~~~
   stop
   ~~~

   