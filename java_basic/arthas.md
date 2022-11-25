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
   stop：会将所有的客户端与服务端全部退出，如果需要使用，则需要重新启动
   quit: 只是退出当前的客户端，不会退出服务端
   要注意使用时不要长时间挂载，时间长以后，严重时会导致监控进程发生堆栈的溢出
   ~~~
   

5. 临时替换编译class，在环境内调试代码

   ~~~
   1. 首先需要获取到需要调试的java文件，文件一般有2中来源，一种是拿到自己线下环境java文件，但是会存在与线上环境的代码不符，
      另外一种是直接反编译在环境中的class文件，命令为：  这里注意将文件的路径写对，否则不能识别
      jad --source-only com.example.demo.aaController > /root/aaController.java
   2. 对已经反编译好的文件进行编辑，写出正确逻辑的代码
   3. 将正确代码进行编译，在这之前需要查到当前待编译文件的classLoader,
      使用sc -d aaController | grep classLoaderHash （这里也可以直接使用线下环境编译获得class文件）
   4. 使用上一个步骤中获取到编译器的hash值来指定classLoader进行编译， mc -c *hash* /tmp/aaController.java -d /tmp
   5. 重新加载新编译好的class文件，retransform /tmp/aa.class 
   6. 在退出当前sessoin后会自动回滚已经修改的class,如果手动，使用reset命令
   ~~~

6. 导出heap文件

   ~~~
   heapdump path                导出全部
   heapdump --live  path        只是导出存活的对象
   heapdump                     不指定路径默认为临时路径
   ~~~

7. 判断某个类属于或者来源于那个jar，方便定位jar包的冲突问题

   ~~~
   sc -d com.company.common.Utils
   ~~~

8. 恢复增强类

   ~~~
   重置增强类，将被 Arthas增强过的类全部还原，Arthas服务端stop时会重置所有增强过的类
   reset 
   reset -E *User
   reset aaUser
   ~~~