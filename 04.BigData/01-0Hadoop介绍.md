# Hadoop介绍

### [官方文档](http://hadoop.apache.org/docs/r1.0.4/cn/quickstart.html)

### Hadoop介绍

- Hadoop是开源的分布式计算和存储框架，由Apache基金会开发和维护
- 为庞大的计算机集群提供可靠的、可伸缩的应用层计算和存储支持，它允许使用简单的编程模型跨计算机群集分布式处理大型数据集，并且支持在单台计算机到几千台计算机之间进行扩展。
- 使用 java 开发，所以可以在多种不同硬件平台的计算机上部署和使用。其核心部件包括分布式文件系统 (Hadoop DFS，HDFS) 和 MapReduce。

#### Hadoop特点

- 高度可拓展性：支持4000个节点，10P以上数据
- 数据高度可靠：数据具有多个副本，保证数据可靠性
- 系统高可用性：通过多个元数据服务器实现同步，实现系统高可用性
- 自动并行处理：支持MapReduce并行计算框架，计算任务调度到数据所在节点，减少网络开销，提高性能
- 灵活易管理：节点可以领过加入和退出

### Hadoop作用

- 在多计算机集群环境中营造一个统一而稳定的存储和计算环境，并能为其他分布式应用服务提供平台支持
- 简单来说，就是解决了多个计算机同时做一件事情，HDFS就相当于硬盘，而MapReduce就是计算机的CPU控制器



### Hadoop三大核心：HDFS、MapReduce、Yarn

可以把HDFS理解为电脑，Yarn相当于分布式的操作系统平台，MapReduce相当于运行操作系统上的应用程序

### 生态圈介绍

![img](https://img-blog.csdnimg.cn/20190712100517880.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI1MDYyMjk5,size_16,color_FFFFFF,t_70)

![微信截图_20210705143117.png](https://youcai922.github.io/99.src/img/微信截图_20210705143117.png)

- HDFS（Hadoop Distributed File System分布式文件系统）：

  ​	hadoop体系的基础，负责数据存储和管理，拥有高容错性的特点。

  - client：切分文件
  - NameNode：master节点，每个HDFS只有一个，管理HDFS的名称空间和数据块映射信息，配置相关副本信息，处理客户端请求
  - DataNode：slave节点，存储实际数据，并汇报状态信息给NameNode，默认一个文件会备份3份在不同的DataNode中，提高可靠性和容错性
  - Secondary NameNode：辅助NameNode，实现高可靠性，定期合并Fsimage和Fsedits，推送给NameNode；紧急情况下辅助和修复NameNode，但其并非NameNode的热备份。**Hadoop2为HDFS引入了两个重要的新功能**
  - Federation允许集群中出现多个NameNode，之间互相独立且不需要互相协助，各自分工，管理自己的区域。DataNode被用作通用的数据块存储设备。meigeDataNode要向集群中所有NameNode注册，并发送心跳报告，执行所有NameNode的命令
  - HDFS中的高可用消除了Hadoop1中存在的单节点故障，其中NameNode故障将导致集群中断，HDFS的高可用性提供故障转移功能（备用节点从失败的主Name Node接管工作的过程中）以实现自动化

- MapReduce（分布式计算框架（磁盘））

  ![图1 MapReduce执行流程](https://bkimg.cdn.bcebos.com/pic/5882b2b7d0a20cf49b9b0c7b76094b36acaf990f?x-bce-process=image/resize,m_lfit,w_1280,limit_1/format,f_auto)

  是一种基于 **磁盘** 的分布式并行批处理计算模型，用于处理大数据量的计算。其中**Map对应数据集上的独立元素进行指定的操作，生成键-值对形式中间，Reduce则对中间结果中相同的键的所有值进行规约，以得到最终结果**。

  - Jbotrackerr：master节点，只有一个，管理所有作业，任务/作业的监控，错误处理等，将任务分解成一系列任务，并分派给Tasktracker。
  - Tacktracker：slave节点，运行 Map task和Reduce task；并与Jobtracker交互，汇报任务状态。
  - Map task：解析每条数据记录，传递给用户编写的map()函数并执行，将输出结果写入到本地磁盘（如果为map—only作业，则直接写入HDFS）。
  - Reduce task：从Map 它深刻地执行结果中，远程读取输入数据，对数据进行排序，将数据分组传递给用户编写的Reduce()函数执行。

- Spark（分布式计算框架（内存））

  基于 **内存** 的分布式并行计算框架，不同于MapReduce的是——Job中间输出结果可以保存在内存中，从而不再需要读写HDFS，因此Spark能更好地适用于数据挖掘与机器学习等需要迭代的MapReduce的算法。

  - Cluster Manager：在standalone模式中即为Master主节点，控制整个集群，监控worker。在YARN模式中为资源管理器
  - Worker节点：从节点，负责控制计算节点，启动Executor或者Driver。
  - Driver： 运行Application 的main()函数
  - Executor：执行器，是为某个Application运行在worker node上的一个进程
  - Spark将数据抽象为RDD（弹性分布式数据集），内部提供了大量的库，包括Spark Core、Spark SQL、Spark Streaming、MLlib、GraphX。 开发者可以在同一个应用程序中无缝组合使用这些库。
  - Spark Core：包含Spark的基本功能；尤其是定义RDD的API、操作以及这两者上的动作。其他Spark的库都是构建在RDD和Spark Core之上的
  - Spark SQL：提供通过Apache Hive的SQL变体Hive查询语言（HiveQL）与Spark进行交互的API。每个数据库表被当做一个RDD，Spark SQL查询被转换为Spark操作。
  - Spark Streaming：对实时数据流进行处理和控制。Spark Streaming允许程序能够像普通RDD一样处理实时数据，通过短时批处理实现的伪流处理。
  - MLlib：一个常用机器学习算法库，算法被实现为对RDD的Spark操作。这个库包含可扩展的学习算法，比如分类、回归等需要对大量数据集进行迭代的操作。
  - GraphX：控制图、并行图操作和计算的一组算法和工具的集合。GraphX扩展了RDD API，包含控制图、创建子图、访问路径上所有顶点的操作

- Flink（分布式计算框架）

  基于内存的分布式并行处理框架，类似于Spark，但在部分设计思想有较大出入。对 Flink 而言，其所要处理的主要场景就是流数据，批数据只是流数据的一个极限特例而已。

  - Flink 和 Spark对比：

    Spark中，RDD在运行时是表现为Java Object，而Flink主要表现为logical plan。所以在Flink中使用的类Dataframe api是被作为第一优先级来优化的。但是相对来说在spark RDD中就没有了这块的优化了。

    Spark中，对于批处理有RDD，对于流式有DStream，不过内部实际还是RDD抽象；在Flink中，对于批处理有DataSet，对于流式我们有DataStreams，但是是同一个公用的引擎之上两个独立的抽象，并且Spark是伪流处理，而Flink是真流处理。

- Yarn/Mesos（分布式资源管理器 ）

  YARN是下一代MapReduce，即MRv2，是在第一代MapReduce基础上演变而来的，主要是为了解决原始Hadoop扩展性较差，不支持多计算框架而提出的。

  Yarn是下一代 Hadoop 计算平台，yarn是一个通用的运行时框架，用户可以编写自己的计算框架，在该运行环境中运行。

  Mesos诞生于UC Berkeley的一个研究项目，现已成为Apache项目，当前有一些公司使用Mesos管理集群资源，比如Twitter。与yarn类似，Mesos是一个资源统一管理和调度的平台，同样支持比如MR、steaming等多种运算框架

- Zookeeper（分布式协作服务）

  解决分布式环境下的数据管理问题：统一命名，状态同步，集群管理，配置同步等。

  Hadoop的许多组件依赖于Zookeeper，它运行在计算机集群上面，用于管理Hadoop操作。

  - Hadoop2.0使用Zookeeper来实现HA(高可用，有多个NameNode)，同时使用他的事务处理，保证只有一个活跃的NameNode，存储配置信息等
  - HBase使用Zookeeper的事务来保证整个集群只有一个HMaster，察觉HRegionServer联机和宕机,存储访问控制列表等

- Sqoop（数据ETL/同步工具）

  Sqoop是SQL-to-Hadoop的缩写，主要用于传统数据库和Hadoop之前传输数据。数据的导入和导出本质上是Mapreduce程序，充分利用了MR的并行化和容错性。

  Sqoop利用数据库技术描述数据架构，用于在关系数据库、数据仓库和Hadoop之间转移数据

- Hive/Impala（基于Hadoop的数据仓库）

  Hive定义了一种类似SQL的查询语言(HQL),将SQL转化为MapReduce任务在Hadoop上执行。通常用于离线分析。

  HQL用于运行存储在Hadoop上的查询语句，Hive让不熟悉MapReduce开发人员也能编写数据查询语句，然后这些语句被翻译为Hadoop上面的MapReduce任务。

  Impala是用于处理存储在Hadoop集群中的大量数据的MPP（大规模并行处理）SQL查询引擎。 它是一个用C ++和Java编写的开源软件。 与Apache Hive不同，Impala不基于MapReduce算法。 它实现了一个基于守护进程的分布式架构，它负责在同一台机器上运行的查询执行的所有方面。因此执行效率高于Apache Hive。

- HBase（分布式列存储数据库）

  HBase是一个建立在HDFS之上，面向列的针对结构化数据的**可伸缩、高可靠、高性能、分布式和面向列的动态模式数据库**。

  HBase采用了BigTable的数据模型：增强的稀疏排序映射表（Key/Value），其中，键由行关键字、列关键字和时间戳构成。

  HBase提供了对大规模数据的随机、实时读写访问，同时，HBase中保存的数据可以使用MapReduce来处理，它将数据存储和并行计算完美地结合在一起。

- Flume（日志收集工具）

  Flume是一个可扩展、适合复杂环境的海量日志收集系统。拥有：**分布式、高可靠、高容错、易于定制和扩展**的特点。它将数据从产生、传输、处理并最终写入目标的路径的过程抽象为数据流，在具体的数据流中，数据源支持在Flume中定制数据发送方，从而支持收集各种不同协议数据。

  同时，Flume数据流提供对日志数据进行简单处理的能力，如过滤、格式转换等。此外，Flume还具有能够将日志写往各种数据目标（可定制）的能力。

  Flume以Agent为最小的独立运行单位，一个Agent就是一个JVM。单个Agent由Source、Sink和Channel三大组件构成

  ![img](https://img-blog.csdnimg.cn/20190712141031493.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI1MDYyMjk5,size_16,color_FFFFFF,t_70)

  Source：从客户端收集数据，并传递给Channel。

  Channel：缓存区，将Source传输的数据暂时存放。

  Sink：从Channel收集数据，并写入到指定地址。

  Event：日志文件、avro对象等源文件。

- Kafka（分布式消息队列）

  Kafka是一种高吞吐量的分布式发布订阅消息系统，它可以处理消费者规模的网站中的所有动作流数据。实现了主题、分区及其队列模式以及生产者、消费者架构模式。

  生产者组件和消费者组件均可以连接到KafKa集群，而KafKa被认为是组件通信之间所使用的一种消息中间件。KafKa内部氛围很多Topic（一种高度抽象的数据结构），每个Topic又被分为很多分区（partition），每个分区中的数据按队列模式进行编号存储。被编号的日志数据称为此日志数据块在队列中的偏移量（offest），偏移量越大的数据块越新，即越靠近当前时间。生产环境中的最佳实践架构是Flume+KafKa+Spark Streaming。

- Pig（ad-hoc脚本）

  设计动机是提供一种基于MapReduce的ad-hoc(计算在query时发生)数据分析工具

  Pig定义了一种数据流语言—Pig Latin，它是MapReduce编程的复杂性的抽象，Pig平台包括运行环境和用于分析Hadoop数据集的脚本语言(Pig Latin)。

  其编译器将Pig Latin翻译成MapReduce程序序列将脚本转换为MapReduce任务在Hadoop上执行。通常用于进行离线分析。

- Oozie（工作流调度器）

  Oozie是一个可扩展的工作体系，集成于Hadoop的堆栈，用于协调多个MapReduce作业的执行。它能够管理一个复杂的系统，基于外部事件来执行，外部事件包括数据的定时和数据的出现。

  Oozie工作流是放置在控制依赖DAG（有向无环图 Direct Acyclic Graph）中的一组动作（例如，Hadoop的Map/Reduce作业、Pig作业等），其中指定了动作执行的顺序。

  Oozie使用hPDL（一种XML流程定义语言）来描述这个图。

- Mahout（数据挖掘算法库）

  Mahout的主要目标是创建一些可扩展的机器学习领域经典算法的实现，旨在帮助开发人员更加方便快捷地创建智能应用程序。

  Mahout现在已经包含了聚类、分类、推荐引擎（协同过滤）和频繁集挖掘等广泛使用的数据挖掘方法。

  除了算法，Mahout还包含数据的输入/输出工具、与其他存储系统（如数据库、MongoDB 或Cassandra）集成等数据挖掘支持架构。

- Tachyon（分布式内存文件系统）

  以内存为中心的分布式文件系统，拥有高性能和容错能力，

  能够为集群框架（如Spark、MapReduce）提供可靠的内存级速度的文件共享服务。

- Tez(DAG计算模型)

  Tez是Apache最新开源的支持DAG作业的计算框架，它直接源于MapReduce框架，核心思想是将Map和Reduce两个操作进一步拆分，

  即Map被拆分成Input、Processor、Sort、Merge和Output， Reduce被拆分成Input、Shuffle、Sort、Merge、Processor和Output等，

  这样，这些分解后的元操作可以任意灵活组合，产生新的操作，这些操作经过一些控制程序组装后，可形成一个大的DAG作业。

  目前hive支持mr、tez计算模型，tez能完美二进制mr程序，提升运算性能。

- Giraph(图计算模型)

  Apache Giraph是一个可伸缩的分布式迭代图处理系统， 基于Hadoop平台

- GraphX(图计算模型）

  Spark GraphX是一个分布式图计算框架项目，目前整合在spark运行框架中，为其提供BSP大规模并行图计算能力。

- MLib（机器学习库）

  Spark MLlib是一个机器学习库，它提供了各种各样的算法，这些算法用来在集群上针对分类、回归、聚类、协同过滤等。

- Streaming（流计算模型）

  Spark Streaming支持对流数据的实时处理，以微批的方式对实时数据进行计算

- Phoenix（hbase sql接口）

  Apache Phoenix 是HBase的SQL驱动，Phoenix 使得Hbase 支持通过JDBC的方式进行访问，并将你的SQL查询转换成Hbase的扫描和相应的动作。

- ranger(安全管理工具）

  Apache ranger是一个hadoop集群权限框架，提供操作、监控、管理复杂的数据权限，它提供一个集中的管理机制，管理基于yarn的hadoop生态圈的所有数据权限。

- knox（hadoop安全网关）

  Apache knox是一个访问hadoop集群的restapi网关，它为所有rest访问提供了一个简单的访问接口点，能完成3A认证（Authentication，Authorization，Auditing）和SSO（单点登录）等

- falcon（数据生命周期管理工具）

  Apache Falcon 是一个面向Hadoop的、新的数据处理和管理平台，设计用于数据移动、数据管道协调、生命周期管理和数据发现。它使终端用户可以快速地将他们的数据及其相关的处理和管理任务“上载（onboard）”到Hadoop集群。

- Ambari（安装部署配置管理工具）

  Apache Ambari 的作用来说，就是创建、管理、监视 Hadoop 的集群，是为了让 Hadoop 以及相关的大数据软件更容易使用的一个web工具。



### 理解

- Hadoop生态组件很多，功能也很多有重复，就好像我们可以利用锅炖汤，我们也可以利用它来吃饭，甚至可以用来装菜。

- HDFS的作用就是存储数据：对大数据进行处理的前提就是我们首先需要存下这些数据，在多个计算机中存储这些数据，就好像你在windows计算机存储文件一样，你的路径概念对于磁盘来说也是虚拟的，有可能你的一个文件分布在磁盘的各个磁道和扇区，因为操作系统帮你管理了，那么HDFS就是做这个事情
- 存储完数据之后我们就可以对数据进行处理了，我们需要解决怎么分配数据处理的工作，如果一个机器挂了，怎么重新启动对应的任务，机器中间怎么互相通信交换数据这些问题，MapReduce / Tez / Spark就是为了解决这些问题，MapReduce是第一代计算引擎，Tez和Spark是第二代。MapReduce简化了计算模型，分为Map和Reduce两个过程（中间采用Shuffle串联）
  - Map就是几百个计算器同时读取这个文件的各个部分，分别执行各自的任务，产生数据分析结果
  - Reduce就是把所有之前Map处理的结果及进行统计
- 再有了MapReduce之后，我们就可以完成大数据的处理了，但是程序员发现MapReduce的编写过程很麻烦，就好像拥有了汇编，什么都可以制作了，但是制作过程十分繁琐，且成本高，这个时候就需要有抽象的语言来实现算法和处理流程。这个时候就诞生了Pig和Hive
  - Pig是接近脚本的方式去描述MapReduce，Hive则用的是SQL。他们把脚本和SQL语言编译成MapReduce程序，丢给计算引擎去计算。
  - Hive使得非程序员也能够参与到大数据的开发过程中，而且容易维护。
- 但是Hive同时又具有缺点，就是Hive在MapReduce上的运行速度缓慢，而有的时候数据分析需要实时的数据，比如淘宝的首页推荐。所以这个时候Impala，Presto，Drill就出现了，他们保证了更快的计算速率，同时舍弃了系统的容错性保护（如果系统出错大不了重启任务）。
- 这个基础上还是很慢，这个时候就需要Spark这种内存的计算引擎提高效率了
- Yarn是一个资源调度系统，所有的计算机都要听从yarn的命令
- Zookeeper是一个存储协同系统
- sqoop是把关系型表导入到Hadoop中，实现数据同步
- 当任务比较多的时候，有可能后端的数据跟不上前端的访问速度，难道就一直处于请求状态，这样用户的体验就会很差，这个时候就需要Kafka做消息队列了，请求全部进入到队列当中，后端顺序执行处理。

