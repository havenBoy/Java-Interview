# Canal java API 

```java
package com.xiong.test.canal;

import com.alibaba.otter.canal.client.CanalConnector;
import com.alibaba.otter.canal.client.CanalConnectors;
import com.alibaba.otter.canal.protocol.CanalEntry.Column;
import com.alibaba.otter.canal.protocol.CanalEntry.Entry;
import com.alibaba.otter.canal.protocol.CanalEntry.EntryType;
import com.alibaba.otter.canal.protocol.CanalEntry.RowChange;
import com.alibaba.otter.canal.protocol.CanalEntry.RowData;
import com.alibaba.otter.canal.protocol.Message;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.net.InetSocketAddress;
import java.net.SocketAddress;
import java.util.List;

public class CanalTest {

    private static final Logger logger = LoggerFactory.getLogger(CanalTest.class);

    public static void canalTcpTest() throws Exception {

        //canal的端口为11111默认
        SocketAddress address = new InetSocketAddress("10.58.14.201", 11111);
        //这里是canal启动前，conf下的实例，可以拥有多个实例，后面的账户与密码是指客户端连接canal的信息，默认不传入即可
        CanalConnector canalConnector = CanalConnectors.newSingleConnector(address, "example", "", "");

        while (true) {
            //建立连接
            canalConnector.connect();

            //订阅对应的数据库  这里的格式为database.table
            canalConnector.subscribe("test_canal.test1");

            //获取10个这样的数据，不会阻塞，即使没有10个，也会立马返回
            Message message = canalConnector.get(10);

            //一个Message对象
            List<Entry> entries = message.getEntries();

            if (entries.size() > 0) {
                for (Entry entry : entries) {
                    String tableName = entry.getHeader().getTableName();
                    logger.info("当前变化的表名称为：{}", tableName);
                    if (entry.getEntryType().equals(EntryType.ROWDATA)) {
                        RowChange rowChange = RowChange.parseFrom(entry.getStoreValue());
                        //一个新增操作，不会包含before数据
                        //一个删除操作，不会包含after数据
                        //一个更新操作，会既包含before数据与after数据
                        List<RowData> rowDataList = rowChange.getRowDatasList();
                        for (RowData rowData : rowDataList) {
                            List<Column> beforeColumnsList = rowData.getBeforeColumnsList();
                            for (Column column : beforeColumnsList) {
                                logger.info("before: column_name: {}, value: {}", column.getName(), column.getValue());
                            }
                            List<Column> afterColumnsList = rowData.getAfterColumnsList();
                            for (Column column : afterColumnsList) {
                                logger.info("after: column_name: {}, value: {}", column.getName(), column.getValue());
                            }
                        }
                    } else {
                        logger.info("不为ROWDATA类型的数据，不解析...");
                    }
                }
            } else {
                logger.warn("暂时没有数据，等待5s后进行再次获取...");
                Thread.sleep(5000);
            }
        }
    }

}
```

```xml
<dependency>
    <groupId>com.alibaba.otter</groupId>
    <artifactId>canal.client</artifactId>
    <version>1.1.2</version>
</dependency>
```

