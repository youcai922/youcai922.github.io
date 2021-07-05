### Hive介绍

​	Hive是基于Hadoop的一个**数据仓库工具**，用来进行数据提取、转化、加载，这是一种可以存储、查询和分析存储在Hadoop中的大规模数据机制。**hive数据仓库工具能将结构化的数据文件映射为一张数据库表，并提供SQL查询功能，能将SQL语句转变成MapReduce任务来执行**。Hive的优点是学习成本低，可以通过类似SQL语句实现快速MapReduce统计，使MapReduce变得更加简单，而不必开发专门的MapReduce应用程序。hive十分适合对数据仓库进行统计分析

#### 架构分析：

Hive架构包括以下组件：CLI（command line interface）、JDBC/ODBC、Thrif Server、WEB GUI、metastore和Driver（Complier、Optmizer和Executor）这些组件分为两类，服务端组件和客户端组件

- 服务端组件

  - Driver组件：包括Complier、Optmizer和Executor，作用是把我们写的HiveQL语句进解析、编译和优化，生成执行计划，然后调用底层mapreduce计算框架
  - metastore组件：元数据服务组件，这个组件存储hive的元数据，hive的元数据存储子啊关系数据库里，hive支持的关系数据库有derby、mysql。元数据对于hive十分重要，因此hive支持把metastore服务独立出来，安装到远程的服务集群里，从而解耦hive服务和metastore服务，保证hive运行的健壮性
  - Thrif服务：thrif是facebook开发的一个软件框架，它用来进行可拓展且跨语言的服务开发，hive集成了该服务，能让不同的编程语言调用hive接口

- 客户端组件：

  - CLI：命令行窗口
  - Thrif客户端：hive架构的许多客户端接口时建立在thrif客户端之上，包括JDBC和ODBC接口
  - WEBGUI：hive客户端提供了一种通过网页的方法访问hive所提供的服务。这个接口对应hive的hwi组件，使用前要启动hwi服务。

  

### 特征：

hive是一种底层封装了Hadoop的数据仓库处理工具，使用类SQL的HiveSQL语言实现数据查询，所有Hive的数据都存储在Hadoop兼容的文件系统中。hive在加载数据过程中不会对数据进行任何修改，只是将数据移动到HDFS中的hive设计的目录下，因此Hive不支持对数据的改写和添加，所有的数据都是在加载的时候确定的。Hive的设计特点如下：

- 支持创建索引，优化数据查询

- 不同的存储类型，例如：纯文本文件、HBase文件

- 将元数据存储在关系性数据库中，大大减少了在查询过程中执行语义检查的时间

- 可以直接使用存储在Hadoop文件系统中的数据

- 内置大量用户函数UDF来操作时间、字符串和其他的数据挖掘工具，支持用户拓展UDF函数来完成内置函数无法实现的操作

- 类SQL的查询方式，将SQL查询转换为MapReduce的job在Hadoop集群上执行

  

#### HiveQL介绍：

HiveQL时一种类似SQL的语言，他与大部分的SQL兼容，但是并不完全支持SQL标准，比如HiveQL不支持更新操作，也不支持索引和事务，它的子查询和join操作也很局限，这是因底层依赖于Hadoop云平台这个特征决定的，但是有很多时SQL无法涉及的，例如多表查询、支持create table as select 和集成MapReduce脚本等

