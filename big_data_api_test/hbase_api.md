# Hbase Java API

~~~java
package com.xiong.test.hbase;

import java.io.IOException;
import java.util.ArrayList;
import java.util.List;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.hbase.CompareOperator;
import org.apache.hadoop.hbase.HBaseConfiguration;
import org.apache.hadoop.hbase.TableName;
import org.apache.hadoop.hbase.client.*;
import org.apache.hadoop.hbase.filter.*;
import org.apache.hadoop.hbase.util.Bytes;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;


public class HbaseTest {

    private static final Logger logger = LoggerFactory.getLogger(HbaseTest.class);

    private static Connection connection = null;
    private static HBaseAdmin admin = null;

    public static void init() {
        Configuration configuration = HBaseConfiguration.create();
        configuration.set("hbase.zookeeper.property.clientPort", "2181");
        configuration.set("hbase.zookeeper.quorum", "localhost");
        configuration.set("zookeeper.znode.parent", "/hbase-unsecure");
        try {
            connection = ConnectionFactory.createConnection(configuration);
            admin = (HBaseAdmin) connection.getAdmin();
            TableName[] tableNames = admin.listTableNames();
        } catch (IOException e) {
            logger.error("连接创建失败！");
        }

        assert (connection != null && admin != null);
        logger.info("连接创建成功！");
    }

    public static void listTable() throws IOException {
        TableName[] tableNames = admin.listTableNames();
        for (TableName tableName : tableNames) {
            logger.info(tableName.getNameAsString());
        }
    }

    public static void createTable() throws IOException {
        TableName tableName = TableName.valueOf("test_1");
        if (admin.tableExists(tableName)) {
            return;
        }
        TableDescriptorBuilder builder = TableDescriptorBuilder.newBuilder(tableName);

        //列族创建指定
        List<ColumnFamilyDescriptor> familyDescriptors = new ArrayList<ColumnFamilyDescriptor>();
        ColumnFamilyDescriptorBuilder familyDescriptorBuilder1 = ColumnFamilyDescriptorBuilder
                .newBuilder(Bytes.toBytes("f1"));
        ColumnFamilyDescriptorBuilder familyDescriptorBuilder2 = ColumnFamilyDescriptorBuilder
                .newBuilder(Bytes.toBytes("f2"));
        familyDescriptors.add(familyDescriptorBuilder1.build());
        familyDescriptors.add(familyDescriptorBuilder2.build());
        builder.setColumnFamilies(familyDescriptors);

        admin.createTable(builder.build());
    }

    public static void put() throws IOException {
        TableName tableName = TableName.valueOf("test_1");
        Put put = new Put(Bytes.toBytes("r1"));

        put.addColumn(Bytes.toBytes("f1"), Bytes.toBytes("name"), Bytes.toBytes("xiong"));
        put.addColumn(Bytes.toBytes("f2"), Bytes.toBytes("school"), Bytes.toBytes("street"));

        HTable table = (HTable) connection.getTable(tableName);
        table.put(put);
    }

    public static void dropTable() throws IOException {
        TableName tableName = TableName.valueOf("test_1");
        if (!admin.isTableDisabled(tableName)) {
            admin.disableTable(tableName);
        }
        admin.deleteTable(tableName);
    }

    public static void deleteData() throws IOException {
        HTable table = (HTable) connection.getTable(TableName.valueOf("test_1"));
        Delete delete = new Delete(Bytes.toBytes("r1"));
        delete.addFamily(Bytes.toBytes("f1"));
        delete.addColumn(Bytes.toBytes("f1"), Bytes.toBytes("name"));
        table.delete(delete);
    }

    public static void selectOne() throws IOException {
        Get get = new Get(Bytes.toBytes("r1"));

        TableName tableName = TableName.valueOf("test_1");
        HTable table = (HTable) connection.getTable(tableName);
        Result result = table.get(get);

        logger.info("name:{}", Bytes.toString(result.getValue(Bytes.toBytes("f1"), Bytes.toBytes("name"))));
        logger.info("school:{}", Bytes.toString(result.getValue(Bytes.toBytes("f2"),                          Bytes.toBytes("school"))));
    }

    public static void selectOne1() throws IOException {
        Get get = new Get(Bytes.toBytes("r1"));

        TableName tableName = TableName.valueOf("test_1");
        HTable table = (HTable) connection.getTable(tableName);
        Result result = table.get(get);

        logger.info("name:{}", Bytes.toString(result.getValue(Bytes.toBytes("f1"), Bytes.toBytes("name"))));
        logger.info("school:{}", Bytes.toString(result.getValue(Bytes.toBytes("f2"), Bytes.toBytes("school"))));
    }

    public static void selectOne2() throws IOException {
        String rowKey = "r1";
        Get get = new Get(Bytes.toBytes(rowKey));

        TableName tableName = TableName.valueOf("test_1");
        HTable table = (HTable) connection.getTable(tableName);
        Result result = table.get(get);
        logger.info("获取指定行键数据：{}", result);
    }

    public void tableScanner() throws IOException {
        TableName tableName = TableName.valueOf("test_1");

        HTable table = (HTable) connection.getTable(tableName);
        Scan scan = new Scan();
        ResultScanner scanner = table.getScanner(scan);
        for (Result result : scanner) {
            logger.info("遍历的每一个结果：{}", result);
        }
    }

    public void tableScannerFilter() throws IOException {
        TableName tableName = TableName.valueOf("test_1");

        HTable table = (HTable) connection.getTable(tableName);

        PageFilter pageFilter = new PageFilter(10);
        ValueFilter valueFilter = new ValueFilter(CompareOperator.EQUAL, new LongComparator(10));
        FamilyFilter familyFilter = new FamilyFilter(CompareOperator.EQUAL, 
                                                     new BinaryPrefixComparator(Bytes.toBytes("f1")));
        QualifierFilter qualifierFilter = new QualifierFilter(CompareOperator.NOT_EQUAL,
                new BinaryComparator(Bytes.toBytes("f1:name")));
        RowFilter rowFilter = new RowFilter(CompareOperator.EQUAL, new SubstringComparator("f1"));

        FilterList filterList = new FilterList(FilterList.Operator.MUST_PASS_ONE);
        //FilterList filterList = new FilterList(Operator.MUST_PASS_ALL);
        filterList.addFilter(pageFilter);
        filterList.addFilter(familyFilter);
        filterList.addFilter(valueFilter);
        filterList.addFilter(qualifierFilter);
        filterList.addFilter(rowFilter);
        Scan scan = new Scan();
        scan.setFilter(filterList);
        ResultScanner scanner = table.getScanner(scan);
        for (Result result : scanner) {
            logger.info("遍历的每一个结果：{}", result);
        }
    }

    public static void selectCell() throws IOException {
        String rowKey = "r1";
        Get get = new Get(Bytes.toBytes(rowKey));

        TableName tableName = TableName.valueOf("test_1");
        HTable table = (HTable) connection.getTable(tableName);
        Result result = table.get(get);
        logger.info("获取指定行键数据：{}", result);
    }

    public static void destroy() throws IOException {
        admin.close();
        connection.close();
    }
}
~~~

~~~xml
    <dependencies>
        <dependency>
            <groupId>org.apache.hbase</groupId>
            <artifactId>hbase-client</artifactId>
            <version>2.3.4</version>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
            <version>1.7.25</version>
        </dependency>
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>javax.servlet-api</artifactId>
            <version>4.0.1</version>
            <scope>provided</scope>
        </dependency>
    </dependencies>
~~~

