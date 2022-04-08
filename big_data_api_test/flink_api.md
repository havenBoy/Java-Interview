# flink

```java
package com.example.pojo;

import java.sql.Timestamp;

public class Event {

    private String username;
    private String url;
    private long timestamp;

    public Event() {
    }

    public Event(String username, String url, long timestamp) {
        this.username = username;
        this.url = url;
        this.timestamp = timestamp;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getUrl() {
        return url;
    }

    public void setUrl(String url) {
        this.url = url;
    }

    public long getTimestamp() {
        return timestamp;
    }

    public void setTimestamp(long timestamp) {
        this.timestamp = timestamp;
    }

    @Override
    public String toString() {
        return "Event{" +
                "username='" + username + '\'' +
                ", url='" + url + '\'' +
                ", timestamp=" + new Timestamp(timestamp) +
                '}';
    }
}
```

```java
##  几种数据源的读取方式
package com.example.source;

import com.example.pojo.Event;
import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;

import java.util.ArrayList;
import java.util.Date;

public class SourceTest {

    public static void main(String[] args) throws Exception {

        StreamExecutionEnvironment executionEnvironment = StreamExecutionEnvironment.getExecutionEnvironment();

        executionEnvironment.setParallelism(1);

        //1.从自定义的数据源获取数据
//        DataStreamSource<Event> eventDataStreamSource = executionEnvironment.addSource(new CustomSource());
//        eventDataStreamSource.print("customSource");

        //2.从集合数据中获取
        ArrayList<Event> list = new ArrayList<>();
        list.add(new Event("user1", "/login", new Date().getTime()));
        DataStreamSource<Event> eventDataStreamFromCollection = executionEnvironment.fromCollection(list);
        eventDataStreamFromCollection.print("from-collection");

        //3.from element
        DataStreamSource<Event> eventDataStreamFromElement = executionEnvironment.fromElements(
                new Event("user1", "/login", new Date().getTime())
        );
        eventDataStreamFromElement.print("from-element");

        //4. from kafka
        Properties pro = new Properties();
        pro.setProperty("bootstrap.servers", "10.58.10.137:6667");
        pro.setProperty("group.id", "consumer-group");
        pro.setProperty("key.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
        pro.setProperty("value.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
        pro.setProperty("auto.offset.reset", "latest");
        DataStreamSource<String> kafkaSourceStream =  executionEnvironment
            .addSource(new FlinkKafkaConsumer<>("test_topic", new SimpleStringSchema(), pro));
        kafkaSourceStream.print("kafka-source");

        //5. from socket
//        DataStreamSource<String> streamFromSocket = executionEnvironment.socketTextStream("10.58.14.201", 9999);
//        streamFromSocket.print("socket");

        //6. from file
        DataStreamSource<String> stringDataStreamSource = executionEnvironment.readTextFile("file/click.txt");
        stringDataStreamSource.print();

        executionEnvironment.execute();
    }
}
```

```xml
    <properties>
        <maven.compiler.source>8</maven.compiler.source>
        <maven.compiler.target>8</maven.compiler.target>
        <flink.version>1.13.0</flink.version>
        <java.version>1.8</java.version>
        <scala.binary.version>2.12</scala.binary.version>
        <slf4j.version>1.7.30</slf4j.version>
    </properties>

    <dependencies>
        <!-- Flink 相关依赖-->
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-java</artifactId>
            <version>${flink.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-streaming-java_${scala.binary.version}</artifactId>
            <version>${flink.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-clients_${scala.binary.version}</artifactId>
            <version>${flink.version}</version>
        </dependency>
        <!-- 日志管理相关依赖-->
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
            <version>${slf4j.version}</version>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-log4j12</artifactId>
            <version>${slf4j.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.logging.log4j</groupId>
            <artifactId>log4j-to-slf4j</artifactId>
            <version>2.14.0</version>
        </dependency>
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-connector-kafka_${scala.binary.version}</artifactId>
            <version>${flink.version}</version>
        </dependency>
    </dependencies>
```

```java
##  自定义数据源
package com.example.source;

import com.example.pojo.ClickEvent;
import java.util.Random;
import org.apache.flink.streaming.api.functions.source.SourceFunction;

public class CustomSource implements SourceFunction<ClickEvent> {

    private boolean running = true;

    @Override
    public void run(SourceContext<ClickEvent> sourceContext) throws Exception {
        String[] users = new String[]{"user1", "user2", "user3"};
        String[] urls = new String[]{"/home", "/login", "/index"};

        Random random = new Random();
        while (running) {
            String user = users[random.nextInt(users.length)];
            String url = urls[random.nextInt(urls.length)];
            sourceContext.collect(new ClickEvent(user, url, System.currentTimeMillis()));

            Thread.sleep(1000L);
        }
    }

    @Override
    public void cancel() {
        running = false;
    }
}
```

```java
## 标准关系型数据库sink

package com.example.sink;

import com.example.pojo.ClickEvent;
import java.util.ArrayList;
import org.apache.flink.connector.jdbc.JdbcConnectionOptions;
import org.apache.flink.connector.jdbc.JdbcConnectionOptions.JdbcConnectionOptionsBuilder;
import org.apache.flink.connector.jdbc.JdbcSink;
import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;

public class StandardMysqlSink {

    public static void main(String[] args) throws Exception {

        StreamExecutionEnvironment executionEnvironment = StreamExecutionEnvironment.getExecutionEnvironment();
        executionEnvironment.setParallelism(1);

        ArrayList<ClickEvent> list = new ArrayList<>();
        list.add(new ClickEvent("user1", "/login", System.currentTimeMillis()));
        list.add(new ClickEvent("user2", "/home", System.currentTimeMillis()));
        DataStreamSource<ClickEvent> clickEventDataStreamSource = executionEnvironment.fromCollection(list);

        String user = "root";
        String password = "123456";
        String jdbcUrl = "jdbc:mysql://10.58.14.201:3306/test?useUnicode=true&characterEncoding=utf-8";
        String driverClass = "com.mysql.jdbc.Driver";

        String sql = "insert into t_click values(?, ?)";

        JdbcConnectionOptions build = new JdbcConnectionOptionsBuilder().withUrl(jdbcUrl)
            .withDriverName(driverClass).withUsername(user).withPassword(password).build();

        clickEventDataStreamSource.addSink(JdbcSink.sink(sql,
            (statement, r) -> {
                statement.setString(1, r.username);
                statement.setString(2, r.url);
            }, build)).name("standardJdbcSink");

        executionEnvironment.execute();
    }
}
```

```java
## 自定义myqsql sink

package com.example.sink;

import com.example.pojo.ClickEvent;
import com.example.sink.function.CustomJdbcSinkFunction;
import java.util.ArrayList;
import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;

public class CustomMysqlSink {

    public static void main(String[] args) throws Exception {

        StreamExecutionEnvironment executionEnvironment = StreamExecutionEnvironment.getExecutionEnvironment();
        executionEnvironment.setParallelism(1);

        ArrayList<ClickEvent> list = new ArrayList<>();
        list.add(new ClickEvent("user1", "/login", System.currentTimeMillis()));
        list.add(new ClickEvent("user2", "/home", System.currentTimeMillis()));
        DataStreamSource<ClickEvent> clickEventDataStreamSource = executionEnvironment.fromCollection(list);

        CustomJdbcSinkFunction customJdbcSink = new CustomJdbcSinkFunction("root", "123456", "test", "10.58.14.201");
        clickEventDataStreamSource.addSink(customJdbcSink).name("customJdbcSink");

        executionEnvironment.execute();
    }
}

## 自定义mysql sink function
package com.example.sink.function;

import com.example.pojo.ClickEvent;
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;
import org.apache.flink.streaming.api.functions.sink.RichSinkFunction;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class CustomJdbcSinkFunction extends RichSinkFunction<ClickEvent> {

    private static final Logger logger = LoggerFactory.getLogger(CustomJdbcSinkFunction.class);

    private Connection connection;
    private PreparedStatement st;
    private final String jdbcUrl;

    private static final String URL_FORMAT = "jdbc:mysql://%s:3306/%s?user=%s&password=%s&useUnicode=true&characterEncoding=utf-8";

    public CustomJdbcSinkFunction(String username, String password, String database, String hostIp) {
        this.jdbcUrl = String.format(URL_FORMAT, hostIp, database, username, password);
        logger.info("connection url: {}", jdbcUrl);
    }

    @Override
    public void invoke(ClickEvent value, Context context) {
        final String sql = "insert into t_click values(?, ?)";

        try {
            Class.forName("com.mysql.jdbc.Driver");
            connection = DriverManager.getConnection(jdbcUrl);
            st = connection.prepareStatement(sql);
            st.setString(1, value.getUsername());
            st.setString(2, value.getUrl());
            st.addBatch();

            st.executeBatch();
        } catch (Exception e) {
            logger.error("connection create error!");
        }
    }

    @Override
    public void close() throws Exception {
        super.close();
        if (st != null) {
            st.executeBatch();
            st.close();
        }
        if (connection != null) {
            connection.close();
        }
    }
}
```

```java
## 自定义Es sink
package com.example.sink;

import com.example.pojo.ClickEvent;
import com.example.sink.function.CustomEsSinkFunction;
import java.util.ArrayList;
import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.connectors.elasticsearch6.ElasticsearchSink;
import org.apache.http.HttpHost;

public class CustomEsSink {

    public static void main(String[] args) throws Exception {

        StreamExecutionEnvironment executionEnvironment = StreamExecutionEnvironment.getExecutionEnvironment();
        executionEnvironment.setParallelism(1);

        ArrayList<ClickEvent> list = new ArrayList<>();
        list.add(new ClickEvent("user1", "/login", System.currentTimeMillis()));
        list.add(new ClickEvent("user2", "/home", System.currentTimeMillis()));
        DataStreamSource<ClickEvent> clickEventDataStreamSource = executionEnvironment.fromCollection(list);

        ArrayList<HttpHost> httpHosts = new ArrayList<>();
        httpHosts.add(new HttpHost("10.58.10.143", 9200, "http"));

        CustomEsSinkFunction tLog0408 = new CustomEsSinkFunction("t_log_0408");
        clickEventDataStreamSource.addSink(new ElasticsearchSink.Builder<>(httpHosts, tLog0408).build()).name("customEsSink");

        executionEnvironment.execute();
    }

}
## 自定义Es sink sourceFunction
package com.example.sink.function;

import com.example.pojo.ClickEvent;
import java.util.HashMap;
import org.apache.flink.api.common.functions.RuntimeContext;
import org.apache.flink.streaming.connectors.elasticsearch.ElasticsearchSinkFunction;
import org.apache.flink.streaming.connectors.elasticsearch.RequestIndexer;
import org.elasticsearch.action.index.IndexRequest;
import org.elasticsearch.client.Requests;

public class CustomEsSinkFunction implements ElasticsearchSinkFunction<ClickEvent> {

    private String indexName;

    private int id = 0;

    public CustomEsSinkFunction(String indexName) {
        this.indexName = indexName;
    }

    @Override
    public void open() throws Exception {

    }

    @Override
    public void close() throws Exception {

    }

    @Override
    public void process(ClickEvent clickEvent, RuntimeContext runtimeContext, RequestIndexer requestIndexer) {
        HashMap<String, String> hashMap = new HashMap<>(2);
        hashMap.put(clickEvent.username, clickEvent.url);
        IndexRequest tLog0408 = Requests.indexRequest().id(String.valueOf(id++)).type("_doc").index(indexName).source(hashMap);
        requestIndexer.add(tLog0408);
    }
}
```

