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
        executionEnvironment.addSource();

        //5. from socket

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
        <!-- 引入 Flink 相关依赖-->
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
        <!-- 引入日志管理相关依赖-->
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