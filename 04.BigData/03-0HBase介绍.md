### HBase介绍：

HBase是高可靠性、高性能、面向列、可伸缩的分布式存储系统。不同于一般的关系型数据库，Hbase是一个适合于非结构化数据存储的数据库。另一个不同的是HBase基于列而不是基于行的模式。

### HBase数据模型术语

- 表（Table）

  HBase 会将数据组织进一张张的表里面，一个 HBase 表由多行组成。

- 行（Row）

  HBase 中的一行包含一个行键和一个或多个与其相关的值的列。在存储行时，行按字母顺序排序。出于这个原因，行键的设计非常重要。目标是以相关行相互靠近的方式存储数据。常用的行键模式是网站域。如果你的行键是域名，则你可能应该将它们存储在相反的位置（org.apache.www，org.apache.mail，org.apache.jira）。这样，表中的所有 Apache 域都彼此靠近，而不是根据子域的第一个字母分布。

- 列（Column）

  HBase 中的列由一个列族和一个列限定符组成，它们由`:`（冒号）字符分隔。

- 列族（Column Family）

  出于性能原因，列族在物理上共同存在一组列和它们的值。在 HBase 中每个列族都有一组存储属性，例如其值是否应缓存在内存中，数据如何压缩或其行编码是如何编码的等等。表中的每一行都有相同的列族，但给定的行可能不会在给定的列族中存储任何内容。

  列族一旦确定后，就不能轻易修改，因为它会影响到 HBase 真实的物理存储结构，但是列族中的列标识(Column Qualifier)以及其对应的值可以动态增删。 

- 列限定符（Column Qualifier）

  列限定符被添加到列族中，以提供给定数据段的索引。鉴于列族的`content`，列限定符可能是`content:html`，而另一个可能是`content:pdf`。虽然列族在创建表时是固定的，但列限定符是可变的，并且在行之间可能差别很大。

- 单元格（Cell）

  单元格是行、列族和列限定符的组合，并且包含值和时间戳，它表示值的版本。

- 时间戳（Timestamp）

  时间戳与每个值一起编写，并且是给定版本的值的标识符。默认情况下，时间戳表示写入数据时 RegionServer 上的时间，但可以在将数据放入单元格时指定不同的时间戳值。



### HBase结构介绍

HBase是列式存储：

概念上的HBase存储：

|  行键（Row Key）  | 时间戳（Time Stamp） |    ColumnFamily`contents`    |     ColumnFamily`anchor`      |   ColumnFamily `people`    |
| :---------------: | :------------------: | :--------------------------: | :---------------------------: | :------------------------: |
|   “com.cnn.www”   |          T9          |                              |   anchor：cnnsi.com =“CNN”    |                            |
|   “com.cnn.www”   |          T8          |                              | anchor：my.look.ca =“CNN.com” |                            |
|   “com.cnn.www”   |          T6          | contents：html =“<html> ...” |                               |                            |
|   “com.cnn.www”   |          T5          | contents：html =“<html> ...” |                               |                            |
|   “com.cnn.www”   |          T3          | contents：html =“<html> ...” |                               |                            |
| “com.example.www” |          T5          | contents：html =“<html> ...” |                               | people:author = "John Doe" |

实际上的HBase存储过程中，是不会存储这些为空的数据的，存储形式入下

```
{
  "com.cnn.www": {
    contents: {
      t6: contents:html: "<html>..."
      t5: contents:html: "<html>..."
      t3: contents:html: "<html>..."
    }
    anchor: {
      t9: anchor:cnnsi.com = "CNN"
      t8: anchor:my.look.ca = "CNN.com"
    }
    people: {}
  }
  "com.example.www": {
    contents: {
      t5: contents:html: "<html>..."
    }
    anchor: {}
    people: {
      t5: people:author: "John Doe"
    }
  }
}
```

#### HBase表特点：

- 大：一个表可以有数十亿行，上百万列
- 无模式：每行都有一个可排序的主键和任意多的列，列可以根据需要动态的增加，同一张表中不同的行可以有截然不同的列；
- 面向列：面向列（族）的存储和权限控制，列（族）独立检索
- 稀疏：空（null）列并不占用存储空间，表可以设计的非常稀疏；
- 数据多版本：每个单元中的数据可以有多个版本，默认情况下版本号自动分配，时单元格插入时的时间戳
- 数据类型单一：HBase中的数据都是字符串，没有类型

#### 使用场景：

- 大数据量存储，大数据量高并发操作
- 需要对数据随机读写操作
- 独写访问均是非常简单的操作

#### HBase与HDFS对比

- 两者都具有良好的容错性和拓展性，都可以拓展到成百上千个节点
- HDFS适合批处理场景
- 不支持数据随即查找
- 不支持增量数据处理
- 不支持数据更新

#### HBase基本组件：

- Client：
  - 包含访问HBase的接口，并维护Cache来加快对HBase的访问，比如region的位置信息
- Master
  - 为Region server分配region
  - 负责Region server的负载均衡
  - 发现失效的Region server并重新分配其上的region
  - 管理用户对table的增删改查操作
- Region Server
  - Region Server维护region，处理对这些region的IO请求
  - Region Server负责切分在运行过程中变得过大的region

**zookeeper的作用：**

- 通过选举，保证任何时候，集群中只有一个master、Master与RegionServers启动时会向zookeeper注册
- 存贮所有Region的寻址入口
- 实时监控Region server的上线和下线信息。并实时通知给Master
- 存储HBase的schema和table元数据
- 默认情况下，HBase管理zookeeper实例，比如，启动或者停止zookeeper
- zookeeper的引入使得Master不再是单点故障

**HBase容错性：**

- Master容错：zookeeper重新选择一个新的Master
  - 无Master过程中，数据读取仍照抄进行
  - 无Master过程中，Region切分、负载均衡等无法进行

- RegionServer容错：定时向zookeeper汇报心跳，如果一旦时间内未出现心跳，Master将该RegionServer上的Region重新分配到其他RegionServer上，失效服务器上”预写“日志由主服务器进行分割并派送给新的RegionServer
- Zookeeper容错：zookeeper是一个可靠的服务，一般配置3或5个zookeeper实例

