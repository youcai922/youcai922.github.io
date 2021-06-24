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

