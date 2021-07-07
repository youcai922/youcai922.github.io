**HBase提供多种方法访问接口**

- Native Java API，最常规和高效的访问方式，适合Hadoop MapReduce Job并行批处理HBase表数据
- HBase Shell，HBase的命令行工具，最简单的接口，适合HBase管理使用
- Thrift Gateway，里YoonThrift序列化技术，支持C++，PHP，Python等多种语言，适合其他异构系统在线访问HBase表数据
- REST Gateway：支持REST风格的Http API访问HBase，解除了语言限制
- Pig，可以使用Pig Latin流式编程语言来操作HBase中的数据，和Hive类型，本质最终也是编译成MapReduce Job来处理HBase表数据，适合做数据统计
- Hive，在Hive 0.7.0及之后的版本支持HBase，可以使用类似SQL语言来访问HBase



----------

#### HBase Shell常用命令

| 名称                       | 命令表达式                                                   | 实例                                 |
| -------------------------- | ------------------------------------------------------------ | ------------------------------------ |
| 创建表                     | create '表名称','列名称1','列名称2','列名称N'                | create 'test', 'cf'                  |
| 添加记录                   | put '表名称','行名称','列名称','值'                          | put 'test', 'row1', 'cf:a', 'value1' |
| 查看记录                   | get '表名称','行名称'                                        | get 'test', 'row1'                   |
| 查看表中的记录总数         | count '表名称'                                               | count 'test'                         |
| 删除记录                   | delete '表名称','行名称','列名称'                            | delete 'test','row1','cf:a'          |
| 删除一张表                 | 先要屏蔽该表，才能对表进行删除，第一步disable '表名称'<br />第二步 drop '表名称' | disable 'test'<br />drop 'test'      |
| 查看所有记录               | scan                                                         | scan 'test'                          |
| 查看某一表某个列中所有数据 | scan "表名称"，['列名称:']                                   |                                      |
| 更新记录                   | 就是重写一个遍进行覆盖                                       |                                      |





------

#### HBase API接口

<table>
	<tr>
	    <th>HBase数据模型</th>
	    <th>java类</th>
	</tr >
	<tr >
        <td rowspan="3">数据库(DataBase)</td>
	    <td>HBaseAdmin</td>
	</tr>
	<tr>
	    <td>HBaseConfiguration</td>
	</tr>
	<tr>
	    <td>HTable</td>
	</tr>
	<tr>
        <td>表（Table）</td>
	    <td>HTableDescriptor</td>
	</tr>
	<tr>
        <td>列族（Column Family）</td>
        <td>button</td>
    </tr>
    <tr>
        <td rowspan="3">列修饰符(Column Qualifier)</td>
        <td>Put</td>
    </tr>
    <tr>
        <td>Get</td>
    </tr>
    <tr>
        <td>Scanner</td>
    </tr>
</table>

- HBaseConfiguration
  - 关系：org.apache.hadoop.hbase.HBaseConfiguration
  - 作用：对HBase进行配置

| 返回值 | 函数                                         | 描述                                                         |
| ------ | -------------------------------------------- | ------------------------------------------------------------ |
| void   | addResource(Path file)                       | 通过给定的路径所指的文件来添加资源                           |
| void   | clear()                                      | 清空所有以设置的属性                                         |
| string | get(String name)                             | 获取属性名对应的值                                           |
| string | getBoolean(String name,boolean defaultValue) | 获取为boolean类型的属性值，如果其属性值类型部位boolean，则返回默认属性值 |
| void   | set(String name,String value)                | 通过属性名来设置值                                           |
| void   | setBoolean(String name,boolean value)        | 设置boolean类型的属性值                                      |

用法示例：

```java 
HBaseConfiguration hconfig=new HBaseConfiguration();
hconfig.set("hbase.zookeeper.property.clientPort","2181");
```

该方法设置了“hbase.zookeeper.property.clientPort”的端口号为2181。一般情况下，HBaseConfiguration会使用构造函数进行初始化，然后在使用其他方法。

- HBaseAdmin
  - 关系：Org.apache.hadoop.hbase.client.HBaseAdmin
  - 作用：提供了一个接口来管理HBase数据库的表信息。它提供的方法包括：创建表，删除表，列出表项，使表有效或无效，以及添加或删除表列族成员等。

<table>
	<tr>
	    <th>返回值</th>
	    <th>函数</th>
	    <th>描述</th>
	</tr >
	<tr >
        <td rowspan="6">void</td>
	    <td>addColumn(String tableName,HColumnDescriptor column)</td>
	    <td>向一个已经存在的表添加列</td>
	</tr>
	<tr>
	    <td>checkHBaseAvailable(HBaseConfiguration conf)</td>
	    <td>静态函数，查看HBase是否处于运行状态</td>
	</tr>
	<tr>
	    <td>createTable(HTableDescriptor desc)</td>
	    <td>创建一个表，同步操作</td>
	</tr>
	<tr>
        <td>deleteTable(byte[] tableName)</td>
	    <td>删除一个已经存在的表</td>
	</tr>
	<tr>
        <td>enableTable(byte[] tableName)</td>
	    <td>使表处于有效状态</td>
	</tr>
	<tr>
        <td>disableTable(byte[] tableName)</td>
	    <td>使表处于无效状态</td>
	</tr>
	<tr>
        <td>HTableDescriptor[]</td>
        <td>listTables</td>
        <td>列出所有用户控件表项</td>
    </tr>
	<tr>
        <td>void</td>
        <td>modifyTable(byte[] tableName,HTableDescriptor htd)</td>
        <td>修改表的模式，是异步的操作，可能需要花费一定的时间</td>
    </tr>
	<tr>
        <td>boolean</td>
        <td>tableExists(String tableName)</td>
        <td>检查表是否存在</td>
    </tr>
</table>

用法示例：

```java
HBaseAdmin admin=new HBaseAdmin(Config);
admin.disableTable("tablename");
```

- HTableDescriptor
  - 关系：org.apache.hadoop.hbase.HTableDescriptor
  - 作用：包含了表的名字机器对应表的列祖

| 返回值            | 函数                              | 描述         |
| ----------------- | --------------------------------- | ------------ |
| void              | addFamily(HColumnDescriptor)      | 添加一个列族 |
| HVolumnDescriptor | removeFamily(byte[] column)       | 移除一个列族 |
| byte[]            | getName()                         | 获取表的名字 |
| byte[]            | getValue(byte[] key)              | 获取属性的值 |
| void              | setValue(String key,String value) | 设置属性的值 |

用法示例：

```java
HTableDescriptor htd=new HTableDescriptor(table);
htd.addFamily(new HcolumnDescritor("family"));
```

- HColumnDescriptor
  - 关系：org.apache.hadoop.hbase.HColumnDescriptor
  - 作用：维护着关于列族的信息，例如版本号，压缩设置等。它通常在创建表或者为表添加列族的时候使用。列族被创建后不能直接修改，只能通过删除然后重新创建的方式。列族被删除的时候，列族里面的数据也会同时被删除。

| 返回值 | 函数                              | 描述               |
| ------ | --------------------------------- | ------------------ |
| byte[] | getName()                         | 获取列族的名字     |
| byte[] | getValue(byte[] key)              | 获取对应的属性的值 |
| void   | setValue(String key,String value) | 设置对应属性的值   |

用法示例：

```java 
HTableDescriptor htd=new HTableDescriptor(tablename);
HColumnDescriptor col=new HColumnDescriptor("content");
htd.addFamily(col);
```

- HTable
  - 关系：org.apache.hadoop.hbase.clien.HTable
  - 作用：可以用来和HBase表直接通信。此方法对于更新操作来说是非线程安全的。

| 返回值           | 函数                                                         | 描述                                             |
| ---------------- | ------------------------------------------------------------ | ------------------------------------------------ |
| void             | checkAdnPut(byte[] row,byte[] family,byte[] qualifler,byte[] value,Put put) | 自动的检查row/family/qualifler是否于给定的值匹配 |
| void             | close()                                                      | 释放所有资源或挂起内部缓冲区中的更新             |
| Boolean          | exists(Get get)                                              | 检查Get实例所指定的值是否存在于HTable的列中      |
| Result           | get(Get get)                                                 | 获取指定行的某些单元格所对应的值                 |
| byte[] []        | getEndKeys()                                                 | 获取当前一行打开的表每个区域的结束键值           |
| ResultScanner    | getScanner(byte[] family)                                    | 获取当前给定列族的scanner实例                    |
| HTableDescriptor | getTableDescriptor()                                         | 获取当前表的HTableDescriptor实例                 |
| byte[]           | getTableName()                                               | 获取表名                                         |
| static boolean   | isTableEnabled(HBaseConfiguration conf,String tableName)     | 检查表是否有效                                   |
| void             | put(Put put)                                                 | 向表中添加值                                     |

用法示例：

```java
HTable table=new HTable(conf,Bytes.toBytes(tablename));
ResultScanner scanner=table.getScanner(family);
```

- Put
  - 关系：org.apache.hadoop.hbase.client.Put
  - 作用：对于单个行执行添加操作

| 返回值  | 函数                                                    | 描述                                        |
| ------- | ------------------------------------------------------- | ------------------------------------------- |
| Put     | add(byte[] family,byte[] qualifier,byte[] value)        | 将指定的列和对应的值添加到Put实例中         |
| Put     | add(byte[] family,byte[]qualifier,long ts,byte[] value) | 将指定的列和对应的值及时间戳添加到Put实例中 |
| byte[]  | getRow()                                                | 获取Put实例的行                             |
| RowLock | getRowLock()                                            | 获取Put实例的行锁                           |
| long    | getTimeStamp()                                          | 获取Put实例的时间戳                         |
| boolean | isEmpty()                                               | 检查familyMap是否为空                       |
| Put     | setTimeStamp(long timeStamp)                            | 设置Put实例的时间戳                         |

用法示例：

```
HTable table=new HTable(conf,Byte.toBytes(tablename));
Put p=new Put(brow);//为指定行创建一个Put操作
p.add(family,qualifier,value);
table.put(p);
```

- Get
  - 关系：org.apache.hadoop.hbase.client.Get
  - 作用：用来获取单个行的相关信息

| 返回值 | 函数                                       | 描述                               |
| ------ | ------------------------------------------ | ---------------------------------- |
| Get    | addColumn(byte[] family,byte[] qualifier)  | 将指定列族和列修饰符对应的列       |
| Get    | addFamily(byte[] family)                   | 通过指定的列族获取其对应列的所有列 |
| Get    | setTimeRange(long minStamp,long manxStamp) | 获取指定区间的列版本号             |
| Get    | setFilter(Filter filter)                   | 当执行Get操作时设置服务器的过滤器  |

用法示例：

```java
HTable table=new HTable(conf,Bytes.toBytes(tablename));
Get g=new Get(Bytes.toBytes(row));
```

- Result
  - 关系：org.apache.hadoop.hbase.client.Result
  - 作用：存储Get或者Scan操作后获取表的单行值。使用此类提供的方法可以直接获取值或者各种Map结构(Key-value对)

| 返回值                      | 函数                                            | 描述                                   |
| --------------------------- | ----------------------------------------------- | -------------------------------------- |
| boolean                     | contaionsColumn(byte[] family,byte[] qualifier) | 检查指定的列是否存在                   |
| NavigableMap<byte[],byte[]> | getFamilyMap(byte[] family)                     | 获取对应列族所包含的修饰符与值的键值对 |
| byte[]                      | getValue(byte[] family,byte[] qualifier)        | 获取对应列的最新值                     |

- ResultScanner
  - 关系：Interface
  - 作用：客户端获取值的接口

| 返回值 | 函数    | 描述                            |
| ------ | ------- | ------------------------------- |
| void   | close() | 关闭Scanner并释放分配给它的资源 |
| Result | next()  | 获取下一行的值                  |



------

#### HBase 数据导入/导出

**HBase导出到HDFS：**

如果只是对hbase数据做备份和恢复操作，可以直接使用HBase提供的export和import工具

- 使用export命令将表数据写为文件

```
$HBASE+HOME/bin/hbase org.apache.hadoop.hbase.mapreduce.Export oldtable/backup
```

- 使用import命令导入存储文件，恢复HBase数据

```
$HBASE+HOME/bin/habse org.apache.hadoop.hbase.mapreduce.Import newtable/backup
```

- 如果要按照指定格式导出HBase中数据并生成文本文件，目前只能通过MapReduce程序进行导出



**HDFS文件导入到HBase有三种方案：**

- 通过Java API进行导入：

  使用HBase的API中的Put是最直接的方法，只适用于实时数据写入，海量数据导入时效率问题较为明显。

- 通过工具导入：

  HBase提供importtsv工具支持从TSV文件中数据导入HBase。使用该工具将文件数据加载至HBase十分高效，因为它是通过MapReduce Job来实施导入的。

- 通过MapReduce程序导入

  MapReduce读取hdfs上的文件，以HTable.put(put)的方式在map中完成数据写入，无reduce过程。

