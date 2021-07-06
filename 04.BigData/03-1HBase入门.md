**HBase提供多种方法访问接口**

- Native Java API，最常规和高效的访问方式，适合Hadoop MapReduce Job并行批处理HBase表数据
- HBase Shell，HBase的命令行工具，最简单的接口，适合HBase管理使用
- Thrift Gateway，里YoonThrift序列化技术，支持C++，PHP，Python等多种语言，适合其他异构系统在线访问HBase表数据
- REST Gateway：支持REST风格的Http API访问HBase，解除了语言限制
- Pig，可以使用Pig Latin流式编程语言来操作HBase中的数据，和Hive类型，本质最终也是编译成MapReduce Job来处理HBase表数据，适合做数据统计
- Hive，在Hive 0.7.0及之后的版本支持HBase，可以使用类似SQL语言来访问HBase



----------

#### HBase Shell常用命令

| 名称                       | 命令表达式                                                   |
| -------------------------- | ------------------------------------------------------------ |
| 创建表                     | create '表名称','列名称1','列名称2','列名称N'                |
| 添加记录                   | put '表名称','行名称','列名称','值'                          |
| 查看记录                   | get '表名称','行名称'                                        |
| 查看表中的记录总数         | count '表名称'                                               |
| 删除记录                   | delete '表名称','行名称','列名称'                            |
| 删除一张表                 | 先要屏蔽该表，才能对表进行删除，第一步disable '表名称'<br />第二步 drop '表名称' |
| 查看所有记录               | scan                                                         |
| 查看某一表某个列中所有数据 | scan "表名称"，['列名称:']                                   |
| 更新记录                   | 就是重写一个遍进行覆盖                                       |





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

```
HBaseAdmin admin=new HBaseAdmin(Config);
admin.disableTable("tablename");
```

