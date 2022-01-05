# flink_cdc

~~~java
package com.xiong.test;

import com.alibaba.ververica.cdc.connectors.mysql.MySQLSource;
import com.alibaba.ververica.cdc.connectors.mysql.table.StartupOptions;
import com.alibaba.ververica.cdc.debezium.DebeziumSourceFunction;
import com.alibaba.ververica.cdc.debezium.StringDebeziumDeserializationSchema;
import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;

public class FlinkCdCTest {

    public static void main(String[] args) throws Exception {
        //1.获取执行的环境
        StreamExecutionEnvironment enviroment = StreamExecutionEnvironment.getExecutionEnvironment();

        enviroment.setParallelism(1);
        //enviroment.enableCheckpointing(1000L);
        //2.通过
        DebeziumSourceFunction<String> sourceFunction = MySQLSource.<String>builder()
                .hostname("localhost")
                .port(3306)
                .databaseList("test")
                .tableList("test.test1")
                .deserializer(new StringDebeziumDeserializationSchema())
                .startupOptions(StartupOptions.initial())
                .build();

        DataStreamSource<String> stringDataStreamSource = enviroment.addSource(sourceFunction);
        stringDataStreamSource.print();

        enviroment.execute("my_job");
    }
}
~~~

```xml
```

