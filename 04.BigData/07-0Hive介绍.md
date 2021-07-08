[官方文档](https://hive.apache.org/)

### Hive介绍

​	Hive是基于Hadoop的一个**数据仓库工具**，用来进行数据提取、转化、加载，这是一种可以存储、查询和分析存储在Hadoop中的大规模数据机制。**hive数据仓库工具能将结构化的数据文件映射为一张数据库表，并提供SQL查询功能，能将SQL语句转变成MapReduce任务来执行**。Hive的优点是学习成本低，可以通过类似SQL语句实现快速MapReduce统计，使MapReduce变得更加简单，而不必开发专门的MapReduce应用程序。hive十分适合对数据仓库进行统计分析

#### 架构分析：

Hive架构包括以下组件：CLI（command line interface）、JDBC/ODBC、Thrif Server、WEB GUI、metastore和Driver（Complier、Optmizer和Executor）这些组件分为两类，服务端组件和客户端组件

- 服务端组件

  - Driver组件：包括SQL Parser（解析器）、Complier（编译器）、Optmizer（优化器）和Executor（执行器），作用是把我们写的HiveQL语句进解析、编译和优化，生成执行计划，然后调用底层mapreduce计算框架
  - metastore组件：元数据服务组件，这个组件存储hive的元数据，hive的元数据存储子啊关系数据库里，hive支持的关系数据库有derby、mysql。元数据对于hive十分重要，因此hive支持把metastore服务独立出来，安装到远程的服务集群里，从而解耦hive服务和metastore服务，保证hive运行的健壮性
  - Thrif服务：thrif是facebook开发的一个软件框架，它用来进行可拓展且跨语言的服务开发，hive集成了该服务，能让不同的编程语言调用hive接口

- 客户端组件：

  - CLI：命令行窗口
  - Thrif客户端：hive架构的许多客户端接口时建立在thrif客户端之上，包括JDBC和ODBC接口
  - WEBGUI：hive客户端提供了一种通过网页的方法访问hive所提供的服务。这个接口对应hive的hwi组件，使用前要启动hwi服务。

  

#### 特征：

hive是一种底层封装了Hadoop的数据仓库处理工具，使用类SQL的HiveSQL语言实现数据查询，所有Hive的数据都存储在Hadoop兼容的文件系统中。hive在加载数据过程中不会对数据进行任何修改，只是将数据移动到HDFS中的hive设计的目录下，因此Hive不支持对数据的改写和添加，所有的数据都是在加载的时候确定的。Hive的设计特点如下：

- 支持创建索引，优化数据查询

- 不同的存储类型，例如：纯文本文件、HBase文件

- 将元数据存储在关系性数据库中，大大减少了在查询过程中执行语义检查的时间

- 可以直接使用存储在Hadoop文件系统中的数据

- 内置大量用户函数UDF来操作时间、字符串和其他的数据挖掘工具，支持用户拓展UDF函数来完成内置函数无法实现的操作

- 类SQL的查询方式，将SQL查询转换为MapReduce的job在Hadoop集群上执行

#### 优缺点：

优点：

- 操作接口采用类SQL语法，提供快速开发的能力（简单、容易上手）
- 避免了去写MapReduce，减少开发人员的学习成本
- Hive的执行延迟比较高，因此Hive常用于数据分析，对实时性要求不高的场合
- Hive的优势在于处理大数据，对于小数据的处理没有优势，因为Hive的执行延迟比较高
- Hvie支持用户自定义函数，可以根据自己的需求来实现自己的函数

缺点：

- Hive的HQL表达能力有限
  - 迭代式算法无法表达
  - 数据挖掘方面不擅长，由于MapReduce数据处理流程的限制，效率更高的算法却无法实现
- Hive的效率比较低
  - Hive自动生成MapReduce作业，通常情况下不够智能化
  - Hive调优比较困难，粒度比较粗



#### Hive对比数据库

- 查询语言：SQL被广泛的应用在数据仓库中，因此，专门针对Hive的特性设计了类SQL的查询语言HQL。熟悉SQL开发的开发者可以很方便的使用Hive进行开发
- 数据存储位置：Hive是建立在Hadoop之上的，所有Hive的数据都是存储在HDFS中的。而数据库则可以将数据保存在块设备或者本地文件系统中。
- 数据更新：Hive是针对数据仓库应用设计的。而**数据仓库的内容是读多写少的**，因此**Hive中不建议对数据的改写，所有数据都是在加载的时候确定好的**。而数据库中的数据通常是需要经常进行修改的。因此可以使用INSERT INTO ... VALUES添加数据，使用UPDATE ...SET修改数据
- 执行：Hive中大多数查询的执行是通过Hadoop提供的MapReduce来实现的。而数据库通常有自己的执行引擎。
- 执行延迟：Hive在查询数据的时候，由于没有索引，需要扫描整个表，因此延迟比较高。另外一个导致Hive执行延迟高的因素是MapReduce框架。由于MapReduce本身具有较高的延迟，因此在利用MapReduce执行Hive查询时，也会有较高的延迟，相对的，数据库的执行延迟较低。当然，这个低时有条件的，即数据规模较小，当数据规模大到超过数据库的处理能力的时候，Hive的并行计算显然能体现出优势
- 可拓展性：由于Hive建立在Hadoop之上，因此Hive的可拓展性是一致的。而数据库由于ACID语义的严格限制，拓展性非常有限。目前最先进的并行数据库Oracle在理论上的拓展能力也只有100台左右
- 数据规模：由于Hive建立在集群上并可以利用MapReduce进行并行计算，因此可以支持很大规模的数据；对应的，数据库可以支持的数据规模较小

#### HiveQL介绍：

HiveQL时一种类似SQL的语言，他与大部分的SQL兼容，但是并不完全支持SQL标准，比如HiveQL不支持更新操作，也不支持索引和事务，它的子查询和join操作也很局限，这是因底层依赖于Hadoop云平台这个特征决定的，但是有很多时SQL无法涉及的，例如多表查询、支持create table as select 和集成MapReduce脚本等

