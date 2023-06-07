DataStream（数据流）本身是Flink中一个用来表示数据集合的类（Class），我们编写的Flink代码其实就是基于这种数据类型的处理，所以这套核心API就以DataStream命名。对于批处理和流处理，我们都可以用这同一套API来实现。

DataStream在用法上有些类似于常规的Java集合，但又有所不同。我们在代码中往往并不关心集合中具体的数据，而只是用API定义出一连串的操作来处理它们；这就叫作数据流的“转换”（transformations）。

一个Flink程序，其实就是对DataStream的各种转换。具体来说，代码基本上都由以下几部分构成，如下图所示：

![Flink程序构成部分](https://youcai922.github.io/99.src/img/Flink程序构成部分.png)

- 获取执行环境（execution environment）

- 读取数据源（source）
- 定义基于数据的转换操作（transformations）
- 定义计算结果的输出位置（sink）
- 触发程序执行（execute）

其中，获取环境和触发执行，都可以认为是针对执行环境的操作。所以本章我们就从执行环境、数据源（source）、转换操作（transformation）、输出（sink）四大部分，对常用的DataStream API做基本介绍。



## 5.1执行环境（Execution Environment）

Flink程序可以在各种上下文环境中运行：我们可以在本地JVM中执行程序，也可以提交到远程集群上运行。

不同的环境，代码的提交运行的过程会有所不同。这就要求我们在提交作业执行计算时，首先必须获取当前Flink的运行环境，从而建立起与Flink框架之间的联系。只有获取了环境上下文信息，才能将具体的任务调度到不同的TaskManager执行。

### 5.1.1创建运行环境

编写Flink程序的第一步，就是创建执行环境。我们要获取的执行环境，是StreamExecutionEnvironment类的对象，这是所有Flink程序的基础。在代码中创建执行环境的方式，就是调用这个类的静态方法，具体有以下三种。

**1.getExecutionEnvironment**

最简单的方式，它会根据当前运行的上下文直接得到正确的结果：如果程序时独立运行的，就返回一个本地执行环境；如果是创建了jar包，然后从命令行调用它并提交到集群执行，那么就返回集群的执行环境。这种只能的方式不需要我们额外做判断，用起来简单高效。

```java
treamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
```

**2.createLocalEnvironment**

这个方法返回一个本地执行环境。可以在调用时传入一个参数，指定默认的并行度；如果不传入，则默认并行度就是本地的CPU核心数

```java
StreamExecutionEnvironment localEnv = StreamExecutionEnvironment.createLocalEnvironment();
```

**3.createRemoteEnvironment**

这个方法返回集群执行环境。需要在调用时指定JobManager的主机名和端口号，并指定要在集群中运行的jar包。

```java
StreamExecutionEnvironment remoteEnv = StreamExecutionEnvironment
  		.createRemoteEnvironment(
    			"host", // JobManager主机名
    			1234, // JobManager进程端口号
   			"path/to/jarFile.jar"  // 提交给JobManager的JAR包
		); 
```

### 5.1.2执行模式（Execution Mode）

上节中我们获取到的执行环境，是一个StreamExecutionEnvironment，顾名思义它应该是做流处理的。那对于批处理，又应该怎么获取执行环境呢？

在之前的Flink版本中，批处理的执行环境与流处理类似，是调用类ExecutionEnvironment的静态方法，返回它的对象：

```java
// 批处理环境
ExecutionEnvironment batchEnv = ExecutionEnvironment.getExecutionEnvironment();
// 流处理环境
StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
```

基于ExecutionEnvironment读入数据创建的数据集合，就是DataSet；对应的调用的一整套转换方法，就是DataSet API。这些我们在第二章的批处理word count程序中已经有了基本了解。

而从1.12.0版本起，Flink实现了API上的流批统一。DataStream API新增了一个重要特性：可以支持不同的“执行模式”（execution mode），通过简单的设置就可以让一段Flink程序在流处理和批处理之间切换。这样一来，DataSet API也就没有存在的必要了。

- 流执行模式（STREAMING）

这是DataStream API最经典的模式，一般用于需要持续实时处理的无界数据流。默认情况下，程序使用的就是STREAMING执行模式。

- 批执行模式（BATCH）

专门用于批处理的执行模式, 这种模式下，Flink处理作业的方式类似于MapReduce框架。对于不会持续计算的有界数据，我们用这种模式处理会更方便。

- 自动模式（AUTOMATIC）

在这种模式下，将由程序根据输入数据源是否有界，来自动选择执行模式。



**1. BATCH模式的配置方法**

由于Flink程序默认是STREAMING模式，我们这里重点介绍一下BATCH模式的配置。主要有两种方式：

- 通过命令行配置

```
bin/flink run -Dexecution.runtime-mode=BATCH ...
```

在提交作业时，增加execution.runtime-mode参数，指定值为BATCH。

- 通过代码配置

```java
StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
env.setRuntimeMode(RuntimeExecutionMode.BATCH);
```

在代码中，直接基于执行环境调用setRuntimeMode方法，传入BATCH模式。

建议: 不要在代码中配置，而是使用命令行。这同设置并行度是类似的：在提交作业时指定参数可以更加灵活，同一段应用程序写好之后，既可以用于批处理也可以用于流处理。而在代码中硬编码（hard code）的方式可扩展性比较差，一般都不推荐。

**2. 什么时候选择BATCH模式**

我们知道，Flink本身持有的就是流处理的世界观，即使是批量数据，也可以看作“有界流”来进行处理。所以STREAMING 执行模式对于有界数据和无界数据都是有效的；而BATCH模式仅能用于有界数据。

看起来BATCH模式似乎被STREAMING模式全覆盖了，那还有必要存在吗？我们能不能所有情况下都用流处理模式呢？

当然是可以的，但是这样有时不够高效。

我们可以仔细回忆一下word count程序中，批处理和流处理输出的不同：在STREAMING模式下，每来一条数据，就会输出一次结果（即使输入数据是有界的）；而BATCH模式下，只有数据全部处理完之后，才会一次性输出结果。最终的结果两者是一致的，但是流处理模式会将更多的中间结果输出。在本来输入有界、只希望通过批处理得到最终的结果的场景下，STREAMING模式的逐个输出结果就没有必要了。

所以总结起来，一个简单的原则就是：用BATCH模式处理批量数据，用STREAMING模式处理流式数据。因为数据有界的时候，直接输出结果会更加高效；而当数据无界的时候, 我们没得选择——只有STREAMING模式才能处理持续的数据流。

当然，在后面的示例代码中，即使是有界的数据源，我们也会统一用STREAMING模式处理。这是因为我们的主要目标还是构建实时处理流数据的程序，有界数据源也只是我们用来测试的手段。



### 5.1.3触发程序执行

有了执行环境，我们就可以构建程序的处理流程了：基于环境读取数据源，进而进行各种转换操作，最后输出结果到外部系统。

需要注意的是，写完输出（sink）操作并不代表程序已经结束。因为当main()方法被调用时，其实只是定义了作业的每个执行操作，然后添加到数据流图中；这时并没有真正处理数据——因为数据可能还没来。Flink是由事件驱动的，只有等到数据到来，才会触发真正的计算，这也被称为“延迟执行”或“懒执行”（lazy execution）。

所以我们需要显式地调用执行环境的execute()方法，来触发程序执行。execute()方法将一直等待作业完成，然后返回一个执行结果（JobExecutionResult）。

```java
env.execute();
```





## 5.2源算子（Source）

​		创建环境之后，就可以构建数据处理的业务逻辑了。想要处理数据，先得有数据，所以首要任务就是把数据读进来。Flink可以从各种来源获取数据，然后构建DataStream进行转换处理。一般将数据的输入来源成为数据源（data source），而读取数据的算子就是源算子（source operator）。所以，source就是我们整个处理程序的输入端。

​		Flink代码中通用的添加source的方式，是调用执行环境的addSource()方法:

```java
DataStream<String> stream = env.addSource(...)
```

​		方法传入一个对象参数，需要实现SourceFunction接口；返回DataStreamSource。这里的DataStreamSource类继承自SingleOutputStreamOperator类，又进一步继承自DataStream。所以很明显，读取数据的source操作是一个算子，得到的是一个数据流（DataStream）。



#### 5.2.1准备工作

​		为了更好地理解，我们先构建一个实际应用场景。比如网站的访问操作，可以抽象成一个三元组（用户名，用户访问的url，用户访问url的时间戳），所以在这里，我们创建一个类Event，将用户行为包装成它的一个对象

```java
import java.sql.Timestamp;

@Data
@ToString
@AllArgsConstructor
@NoArgsConstructor
public class Event {
    //用户名
    public String user;
    //用户访问url
    public String url;
    //用户访问url的时间戳
    public Long timestamp;
}
```

定义的Event类，拥有这样几个特点：

- 类是公有（public）的
- 有一个无参的构造方法
- 所有属性都是公有（public）的
- 所有属性的类型都是可以序列化的

Flink会把这样的类作为一种特殊的POJO数据类型来对待，方便数据的解析和序列化。

#### 5.2.2从集合中读取数据

​		最简单的读取数据的方式，就是在代码中直接创建一个Java集合，然后调用执行环境的fromCollection方法进行读取。这相当于将数据临时存储到内存中，形成特殊的数据结构后，作为数据源使用，一般用于测试。

```java
public static void main(String[] args) throws Exception {
    StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
    env.setParallelism(1);
    ArrayList<Event> clicks = new ArrayList<>();
    clicks.add(new Event("Mary","./home",1000L));
    clicks.add(new Event("Bob","./cart",2000L));
    DataStream<Event> stream = env.fromCollection(clicks);
	stream.print();
    env.execute();
}
```

我们也可以不构建集合，直接将元素列举出来，调用fromElements方法进行读取数据：

```java
DataStreamSource<Event> stream2 = env.fromElements(
   new Event("Mary", "./home", 1000L),
   new Event("Bob", "./cart", 2000L)
);
```

#### 5.2.3 从文件读取数据

​		真正的实际应用中，自然不会直接将数据写在代码中。通常情况下，我们会从存储介质中获取数据，一个比较常见的方式就是读取日志文件。这也是批处理中最常见的读取方式。

```java
DataStream<String> stream = env.readTextFile("clicks.csv");
```

说明:

- 参数可以是目录，也可以是文件；
- 路径可以是相对路径，也可以是绝对路径；
- 相对路径是从系统属性user.dir获取路径: idea下是project的根目录, standalone模式下是集群节点根目录；
- 也可以从hdfs目录下读取, 使用路径hdfs://..., 由于Flink没有提供hadoop相关依赖, 需要pom中添加相关依赖:

```xml
<dependency>
  <groupId>org.apache.hadoop</groupId>
  <artifactId>hadoop-client</artifactId>
  <version>2.7.5</version>
  <scope>provided</scope>
</dependency>
```

#### 5.2.4 从Socket读取数据

不论从集合还是文件，我们读取的其实都是有界数据。在流处理的场景中，数据往往是无界的。这时又从哪里读取呢？

一个简单的方式，就是我们之前用到的读取socket文本流。这种方式由于吞吐量小、稳定性较差，一般也是用于测试。

```java
DataStream<String> stream = env.socketTextStream("localhost", 7777);
```

#### 5.2.5从Kafka读取数据

Kafka作为分布式消息传输队列，是一个高吞吐、易于扩展的消息系统。而消息队列的传输方式，恰恰和流处理是完全一致的。所以可以说Kafka和Flink天生一对，是当前处理流式数据的双子星。

略微遗憾的是，与Kafka的连接比较复杂，Flink内部并没有提供预实现的方法。所以我们只能采用通用的addSource方式、实现一个SourceFunction了。

好在Kafka与Flink确实是非常契合，所以Flink官方提供了连接工具flink-connector-kafka，直接帮我们实现了一个消费者FlinkKafkaConsumer，它就是用来读取Kafka数据的SourceFunction。

所以想要以Kafka作为数据源获取数据，我们只需要引入Kafka连接器的依赖。Flink官方提供的是一个通用的Kafka连接器，它会自动跟踪最新版本的Kafka客户端。目前最新版本只支持0.10.0版本以上的Kafka，读者使用时可以根据自己安装的Kafka版本选定连接器的依赖版本。这里我们需要导入的依赖如下。

```xml
<dependency>
    <groupId>org.apache.flink</groupId>
    <artifactId>flink-connector-kafka_${scala.binary.version}</artifactId>
    <version>${flink.version}</version>
</dependency>
```

然后调用env.addSource()，传入FlinkKafkaConsumer的对象实例就可以了。

```java
import org.apache.flink.api.common.serialization.SimpleStringSchema;
import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.connectors.kafka.FlinkKafkaConsumer;

import java.util.Properties;

public class SourceKafkaTest {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);
        Properties properties = new Properties();
        properties.setProperty("bootstrap.servers", "hadoop102:9092");
        properties.setProperty("group.id", "consumer-group");
        properties.setProperty("key.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
        properties.setProperty("value.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
        properties.setProperty("auto.offset.reset", "latest");
        DataStreamSource<String> stream = env.addSource(new FlinkKafkaConsumer<String>(
                "clicks",
                new SimpleStringSchema(),
                properties
        ));
        stream.print("Kafka");
        env.execute();
    }
}
```

创建FlinkKafkaConsumer时需要传入三个参数：

- 第一个参数topic，定义了从哪些主题中读取数据。可以是一个topic，也可以是topic列表，还可以是匹配所有想要读取的topic的正则表达式。当从多个topic中读取数据时，Kafka连接器将会处理所有topic的分区，将这些分区的数据放到一条流中去。

- 第二个参数是一个DeserializationSchema或者KeyedDeserializationSchema。Kafka消息被存储为原始的字节数据，所以需要反序列化成Java或者Scala对象。上面代码中使用的SimpleStringSchema，是一个内置的DeserializationSchema，它只是将字节数组简单地反序列化成字符串。DeserializationSchema和KeyedDeserializationSchema是公共接口，所以我们也可以自定义反序列化逻辑。

- 第三个参数是一个Properties对象，设置了Kafka客户端的一些属性。

#### 5.2.6自定义Source

​		大多数情况下，前面的数据源已经能够满足需要。但是凡事总有例外，如果遇到特殊情况，我们想要读取的数据源来自某个外部系统，而flink既没有预实现的方法、也没有提供连接器，就只能使用自定义实现SourceFunction了

- 创建一个自定义的数据源，实现SourceFunction接口。主要重写两个关键方法：run()和cancel()。

  - run()方法：使用运行时上下文对象（SourceContext）向下游发送数据
  - cancel()方法：通过标识位控制退出循环，来达到中断数据源的效果。

  ```java
  import org.apache.flink.streaming.api.functions.source.SourceFunction;
  
  import java.util.Calendar;
  import java.util.Random;
  
  public class ClickSource implements SourceFunction<Event> {
      // 声明一个布尔变量，作为控制数据生成的标识位
      private Boolean running = true;
  
      @Override
      public void run(SourceContext<Event> ctx) throws Exception {
          Random random = new Random();    // 在指定的数据集中随机选取数据
          String[] users = {"Mary", "Alice", "Bob", "Cary"};
          String[] urls = {"./home", "./cart", "./fav", "./prod?id=1", "./prod?id=2"};
  
          while (running) {
              ctx.collect(new Event(
                      users[random.nextInt(users.length)],
                      urls[random.nextInt(urls.length)],
                      Calendar.getInstance().getTimeInMillis()
              ));
              // 隔1秒生成一个点击事件，方便观测
              Thread.sleep(1000);
          }
      }
  
      @Override
      public void cancel() {
          running = false;
      }
  }
  ```

下面的代码我们来读取一下自定义的数据源。有了自定义的source function，接下来只要调用addSource()就可以了：

```java
env.addSource(new ClickSource())
```

下面是完整的代码：

```java
import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;

public class SourceCustom {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);
        //有了自定义的source function，调用addSource方法
        DataStreamSource<Event> stream = env.addSource(new ClickSource());
        stream.print("SourceCustom");
        env.execute();
    }
}
```

这里要注意的是SourceFunction接口定义的数据源，并行度只能设置为1，如果数据源设置为大于1的并行度，则会抛出异常。如下程序所示：

```java
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.functions.source.SourceFunction;

import java.util.Random;

public class SourceThrowException {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.addSource(new ClickSource()).setParallelism(2).print();
        env.execute();
    }
}
```

输出的异常如下：

```java
Exception in thread "main" java.lang.IllegalArgumentException: The parallelism of non parallel operator must be 1.
```

所以如果我们想要自定义并行的数据源的话，需要使用ParallelSourceFunction，示例程序如下：

```java
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.functions.source.ParallelSourceFunction;

import java.util.Random;

public class ParallelSourceExample {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.addSource(new CustomSource()).setParallelism(2).print();
        env.execute();
    }

    public static class CustomSource implements ParallelSourceFunction<Integer> {
        private boolean running = true;
        private Random random = new Random();
        @Override
        public void run(SourceContext<Integer> sourceContext) throws Exception {
            while (running) {
                sourceContext.collect(random.nextInt());
            }
        }
        @Override
        public void cancel() {
            running = false;
        }
    }
}
```

#### 5.2.7 Flink支持的数据类型

我们已经了解了Flink怎样从不同的来源读取数据。在之前的代码中，我们的数据都是定义好的UserBehavior类型，而且在5.2.1小节中特意说明了对这个类的要求。那还有没有其他更灵活的类型可以用呢？Flink支持的数据类型到底有哪些？

**1. Flink的类型系统**

为什么会出现“不支持”的数据类型呢？因为Flink作为一个分布式处理框架，处理的是以数据对象作为元素的流。如果用水流来类比，那么我们要处理的数据元素就是随着水流漂动的物体。在这条流动的河里，可能漂浮着小木块，也可能行驶着内部错综复杂的大船。要分布式地处理这些数据，就不可避免地要面对数据的网络传输、状态的落盘和故障恢复等问题，这就需要对数据进行序列化和反序列化。小木块是容易序列化的；而大船想要序列化之后传输，就需要将它拆解、清晰地知道其中每一个零件的类型。

为了方便地处理数据，Flink有自己一整套类型系统。Flink使用“类型信息”（TypeInformation）来统一表示数据类型。TypeInformation类是Flink中所有类型描述符的基类。它涵盖了类型的一些基本属性，并为每个数据类型生成特定的序列化器、反序列化器和比较器。

**2. Flink支持的数据类型**

简单来说，对于常见的Java和Scala数据类型，Flink都是支持的。Flink在内部，Flink对支持不同的类型进行了划分，这些类型可以在Types工具类中找到：

- 基本类型：所有Java基本类型及其包装类，再加上Void、String、Date、BigDecimal和BigInteger。

- 数组类型：包括基本类型数组（PRIMITIVE_ARRAY）和对象数组(OBJECT_ARRAY)
- 复合数据类型：
  - Java元组类型（TUPLE）：这是Flink内置的元组类型，是Java API的一部分。最多25个字段，也就是从Tuple0~Tuple25，不支持空字段
  - Scala 样例类及Scala元组：不支持空字段
  - 行类型（ROW）：可以认为是具有任意个字段的元组,并支持空字段
  - POJO：Flink自定义的类似于Java bean模式的类

- 辅助类型：Option、Either、List、Map等
- 泛型类型（GENERIC）

Flink支持所有的Java类和Scala类。不过如果没有按照上面POJO类型的要求来定义，就会被Flink当作泛型类来处理。Flink会把泛型类型当作黑盒，无法获取它们内部的属性；它们也不是由Flink本身序列化的，而是由Kryo序列化的。

在这些类型中，元组类型和POJO类型最为灵活，因为它们支持创建复杂类型。而相比之下，POJO还支持在键（key）的定义中直接使用字段名，这会让我们的代码可读性大大增加。所以，在项目实践中，往往会将流处理程序中的元素类型定为Flink的POJO类型。

Flink对POJO类型的要求如下：

- 类是公共的（public）和独立的（standalone，也就是说没有非静态的内部类）；

- 类有一个公共的无参构造方法；

- 类中的所有字段是public且非final的；或者有一个公共的getter和setter方法，这些方法需要符合Java bean的命名规范。

所以我们看到，之前的UserBehavior，就是我们创建的符合Flink POJO定义的数据类型。

**3. 类型提示（Type Hints）**

Flink还具有一个类型提取系统，可以分析函数的输入和返回类型，自动获取类型信息，从而获得对应的序列化器和反序列化器。但是，由于Java中泛型擦除的存在，在某些特殊情况下（比如Lambda表达式中），自动提取的信息是不够精细的——只告诉Flink当前的元素由“船头、船身、船尾”构成，根本无法重建出“大船”的模样；这时就需要显式地提供类型信息，才能使应用程序正常工作或提高其性能。

为了解决这类问题，Java API提供了专门的“类型提示”（type hints）。

回忆一下之前的word count流处理程序，我们在将String类型的每个词转换成（word， count）二元组后，就明确地用returns指定了返回的类型。因为对于map里传入的Lambda表达式，系统只能推断出返回的是Tuple2类型，而无法得到Tuple2<String, Long>。只有显式地告诉系统当前的返回类型，才能正确地解析出完整数据。

```java
.map(word -> Tuple2.of(word, 1L))
.returns(Types.TUPLE(Types.STRING, Types.LONG));
```

这是一种比较简单的场景，二元组的两个元素都是基本数据类型。那如果元组中的一个元素又有泛型，该怎么处理呢？

Flink专门提供了TypeHint类，它可以捕获泛型的类型信息，并且一直记录下来，为运行时提供足够的信息。我们同样可以通过.returns()方法，明确地指定转换之后的DataStream里元素的类型。

```java
returns(new TypeHint<Tuple2<Integer, SomeType>>(){})
```



## 5.3转换算子（Transformation)

​		数据源读入数据之后，我们就可以各种转换算子，将一个或多个DataStream转换未新的DataStream。一个Flink程序的核心，其实就是所有的转换操作，他们决定了处理的业务逻辑。		我们可以针对一条流进行转换处理，也可以进行分流、合流等多流转换操作，从而组合成复杂的数据流拓扑。

#### 5.3.1基本转换算子

**1.映射（map）**

map是大家非常熟悉的大数据操作算子，主要用于将数据流中的数据进行转换，形成新的数据流。简单来说，就是一个“一一映射”，消费一个元素就产出一个元素，如下图

![map算子示意](https://youcai922.github.io/99.src/img/map算子示意.png)

​		我们只需要基于DataStrema调用map()方法就可以进行转换处理。方法需要传入的参数是接口MapFunction的实现；返回值类型还是DataStream，不过泛型（流中的元素类型）可能改变。

​		下面的代码用不同的方式，实现了提取Event中的user字段的功能。

```java
import org.apache.flink.api.common.functions.MapFunction;
import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;

public class TransMapTest {
    public static void main(String[] args) throws Exception{
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);
        DataStreamSource<Event> stream = env.fromElements(
                new Event("Mary", "./home", 1000L),
                new Event("Bob", "./cart", 2000L)
        );
        // 传入匿名类，实现MapFunction
        stream.map(new MapFunction<Event, String>() {
            @Override
            public String map(Event e) throws Exception {
                return e.user;
            }
        });
        // 传入MapFunction的实现类
        stream.map(new UserExtractor()).print();
        env.execute();
    }
    public static class UserExtractor implements MapFunction<Event, String> {
        @Override
        public String map(Event e) throws Exception {
            return e.user;
        }
    }
}
```

​		上面代码中，MapFunction实现类的泛型类型，与输入数据类型和输出数据的类型有关。在实现MapFunction接口的时候，需要指定两个泛型，分别是输入事件和输出事件的类型，还需要重写一个map()方法，定义从一个输入事件转换为另一个输出事件的具体逻辑。

​		另外，细心的读者通过查看Flink源码可以发现，基于DataStream调用map方法，返回的其实是一个SingleOutputStreamOperator。

```
public <R> SingleOutputStreamOperator<R> map(MapFunction<T, R> mapper){} 
```

​		这表示map是一个用户可以自定义的转换（transformation）算子，它作用于一条数据流上，转换处理的结果是一个确定的输出类型。当然，SingleOutputStreamOperator类本身也继承自DataStream类，所以说map是将一个DataStream转换成另一个DataStream是完全正确的。

**2.过滤（filter）**

filter转换操作，顾名思义是对数据流执行一个过滤，通过一个布尔条件表达式设置过滤条件，对于每一个流内元素进行判断，若为true则元素正常数据，若为false则元素被过滤掉，如下图所示

![filter算子示意](https://youcai922.github.io/99.src/img/filter算子示意.png)

进行filter转换之后的新数据流的数据类型与原数据流是相同的。filter转换需要传入的参数需要实现FilterFunction接口，而FilterFunction内要实现filter()方法，就相当于一个返回布尔类型的条件表达式。

下面的代码会将数据流中用户Mary的浏览行为过滤出来 。

```java
import org.apache.flink.api.common.functions.FilterFunction;
import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;

public class TransFilterTest {
    public static void main(String[] args) throws Exception{
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);

        DataStreamSource<Event> stream = env.fromElements(
                new Event("Mary", "./home", 1000L),
                new Event("Bob", "./cart", 2000L)
        );
        // 传入匿名类实现FilterFunction
        stream.filter(new FilterFunction<Event>() {
            @Override
            public boolean filter(Event e) throws Exception {
                return e.user.equals("Mary");
            }
        });
        // 传入FilterFunction实现类
        stream.filter(new UserFilter()).print();
        env.execute();
    }
    public static class UserFilter implements FilterFunction<Event> {
        @Override
        public boolean filter(Event e) throws Exception {
            return e.user.equals("Mary");
        }
    }
}
```

**3.扁平映射（flatMap）**

flatMap操作又称为扁平映射，主要是将数据流中的整体（一般是集合类型）拆分成一个一个的个体使用。消费一个元素，可以产生0到多个元素。flatMap可以认为是“扁平化”（flatten）和“映射”（map）两步操作的结合，也就是先按照某种规则对数据进行打散拆分，再对拆分后的元素做转换处理，如下图。我们此前WordCount程序的第一步分词操作，就用到了flatMap。

![flapMap算子示意](https://youcai922.github.io/99.src/img/flapMap算子示意.png)

同map一样，flatMap也可以使用Lambda表达式或者FlatMapFunction接口实现类的方式来进行传参，返回值类型取决于所传参数的具体逻辑，可以与原数据流相同，也可以不同。

flatMap操作会应用在每一个输入事件上面， FlatMapFunction接口中定义了flatMap方法，用户可以重写这个方法，在这个方法中对输入数据进行处理，并决定是返回0个、1个或多个结果数据。因此flatMap并没有直接定义返回值类型，而是通过一个“收集器”（Collector）来指定输出。希望输出结果时，只要调用收集器的.collect()方法就可以了；这个方法可以多次调用，也可以不调用。所以flatMap方法也可以实现map方法和filter方法的功能，当返回结果是0个的时候，就相当于对数据进行了过滤，当返回结果是1个的时候，相当于对数据进行了简单的转换操作。

flatMap的使用非常灵活，可以对结果进行任意输出，下面就是一个例子：

```java
import org.apache.flink.api.common.functions.FlatMapFunction;
import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.util.Collector;

public class TransFlatmapTest {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);
        DataStreamSource<Event> stream = env.fromElements(
                new Event("Mary", "./home", 1000L),
                new Event("Bob", "./cart", 2000L)
        );
        stream.flatMap(new MyFlatMap()).print();
        env.execute();
    }
    public static class MyFlatMap implements FlatMapFunction<Event, String> {
        @Override
        public void flatMap(Event value, Collector<String> out) throws Exception {
            if (value.user.equals("Mary")) {
                out.collect(value.user);
            } else if (value.user.equals("Bob")) {
                out.collect(value.user);
                out.collect(value.url);
            }
        }
    }
} 
```



#### 5.3.2聚合算子（Aggregation）

直观上看，基本转换算子确实是在“转换”——因为它们都是基于当前数据，去做了处理和输出。而在实际应用中，我们往往需要对大量的数据进行统计或整合，从而提炼出更有用的信息。比如之前word count程序中，要对每个词出现的频次进行叠加统计。这种操作，计算的结果不仅依赖当前数据，还跟之前的数据有关，相当于要把所有数据聚在一起进行汇总合并——这就是所谓的“聚合”（Aggregation），也对应着MapReduce中的reduce操作。

**1. 按键分区（keyBy）**

对于Flink而言，DataStream是没有直接进行聚合的API的。因为我们对海量数据做聚合肯定要进行分区并行处理，这样才能提高效率。所以在Flink中，要做聚合，需要先进行分区；这个操作就是通过keyBy来完成的。

keyBy是聚合前必须要用到的一个算子。keyBy通过指定键（key），可以将一条流从逻辑上划分成不同的分区（partitions）。这里所说的分区，其实就是并行处理的子任务，也就对应着任务槽（task slot）。

基于不同的key，流中的数据将被分配到不同的分区中去，如图5-8所示；这样一来，所有具有相同的key的数据，都将被发往同一个分区，那么下一步算子操作就将会在同一个slot中进行处理了。

![keyBy算子将数据分配到不同分区](https://youcai922.github.io/99.src/img/keyBy算子将数据分配到不同分区.png)

在内部，是通过计算key的哈希值（hash code），对分区数进行取模运算来实现的。所以这里key如果是POJO的话，必须要重写hashCode()方法。

keyBy()方法需要传入一个参数，这个参数指定了一个或一组key。有很多不同的方法来指定key：比如对于Tuple数据类型，可以指定字段的位置或者多个位置的组合；对于POJO类型，可以指定字段的名称（String）；另外，还可以传入Lambda表达式或者实现一个键选择器（KeySelector），用于说明从数据中提取key的逻辑。

我们可以以id作为key做一个分区操作，代码实现如下：

```java
import org.apache.flink.api.java.functions.KeySelector;
import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.datastream.KeyedStream;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;

public class TransKeyByTest {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);
        DataStreamSource<Event> stream = env.fromElements(
                new Event("Mary", "./home", 1000L),
                new Event("Bob", "./cart", 2000L)
        );
        // 使用Lambda表达式
        KeyedStream<Event, String> keyedStream = stream.keyBy(e -> e.user);
        // 使用匿名类实现KeySelector
        KeyedStream<Event, String> keyedStream1 = stream.keyBy(new KeySelector<Event, String>() {
            @Override
            public String getKey(Event e) throws Exception {
                return e.user;
            }
        });
        env.execute();
    }
}
```

需要注意的是，keyBy得到的结果将不再是DataStream，而是会将DataStream转换为KeyedStream。KeyedStream可以认为是“分区流”或者“键控流”，它是对DataStream按照key的一个逻辑分区，所以泛型有两个类型：除去当前流中的元素类型外，还需要指定key的类型。

KeyedStream也继承自DataStream，所以基于它的操作也都归属于DataStream API。但它跟之前的转换操作得到的SingleOutputStreamOperator不同，只是一个流的分区操作，并不是一个转换算子。KeyedStream是一个非常重要的数据结构，只有基于它才可以做后续的聚合操作（比如sum，reduce）；而且它可以将当前算子任务的状态（state）也按照key进行划分、限定为仅对当前key有效。关于状态的相关知识我们会在后面章节继续讨论。

**2. 简单聚合**

有了按键分区的数据流KeyedStream，我们就可以基于它进行聚合操作了。Flink为我们内置实现了一些最基本、最简单的聚合API，主要有以下几种：

l sum()：在输入流上，对指定的字段做叠加求和的操作。

l min()：在输入流上，对指定的字段求最小值。

l max()：在输入流上，对指定的字段求最大值。

l minBy()：与min()类似，在输入流上针对指定字段求最小值。不同的是，min()只计算指定字段的最小值，其他字段会保留最初第一个数据的值；而minBy()则会返回包含字段最小值的整条数据。

l maxBy()：与max()类似，在输入流上针对指定字段求最大值。两者区别与min()/minBy()完全一致。

简单聚合算子使用非常方便，语义也非常明确。这些聚合方法调用时，也需要传入参数；但并不像基本转换算子那样需要实现自定义函数，只要说明聚合指定的字段就可以了。指定字段的方式有两种：指定位置，和指定名称。

对于元组类型的数据，同样也可以使用这两种方式来指定字段。需要注意的是，元组中字段的名称，是以f0、f1、f2、…来命名的。

例如，下面就是对元组数据流进行聚合的测试：

```
import org.apache.flink.api.java.tuple.Tuple2;
import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;

public class TransTupleAggreationTest {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);

        DataStreamSource<Tuple2<String, Integer>> stream = env.fromElements(
                Tuple2.of("a", 1),
                Tuple2.of("a", 3),
                Tuple2.of("b", 3),
                Tuple2.of("b", 4)
        );

        stream.keyBy(r -> r.f0).sum(1).print();
        stream.keyBy(r -> r.f0).sum("f1").print();
        stream.keyBy(r -> r.f0).max(1).print();
        stream.keyBy(r -> r.f0).max("f1").print();
        stream.keyBy(r -> r.f0).min(1).print();
        stream.keyBy(r -> r.f0).min("f1").print();
        stream.keyBy(r -> r.f0).maxBy(1).print();
        stream.keyBy(r -> r.f0).maxBy("f1").print();
        stream.keyBy(r -> r.f0).minBy(1).print();
        stream.keyBy(r -> r.f0).minBy("f1").print();

        env.execute();
    }
}
```

而如果数据流的类型是POJO类，那么就只能通过字段名称来指定，不能通过位置来指定了。

```java
import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;

public class TransPojoAggregationTest {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);
        DataStreamSource<Event> stream = env.fromElements(
                new Event("Mary", "./home", 1000L),
                new Event("Bob", "./cart", 2000L)
        );
        stream.keyBy(e -> e.user).max("timestamp").print();    // 指定字段名称
        env.execute();
    }
}
```

简单聚合算子返回的，同样是一个SingleOutputStreamOperator，也就是从KeyedStream又转换成了常规的DataStream。所以可以这样理解：keyBy和聚合是成对出现的，先分区、后聚合，得到的依然是一个DataStream。而且经过简单聚合之后的数据流，元素的数据类型保持不变。

一个聚合算子，会为每一个key保存一个聚合的值，在Flink中我们把它叫作“状态”（state）。所以每当有一个新的数据输入，算子就会更新保存的聚合结果，并发送一个带有更新后聚合值的事件到下游算子。对于无界流来说，这些状态是永远不会被清除的，所以我们使用聚合算子，应该只用在含有有限个key的数据流上。

**3. 归约聚合（**reduce**）**

如果说简单聚合是对一些特定统计需求的实现，那么reduce算子就是一个一般化的聚合统计操作了。从大名鼎鼎的MapReduce开始，我们对reduce操作就不陌生：它可以对已有的数据进行归约处理，把每一个新输入的数据和当前已经归约出来的值，再做一个聚合计算。

与简单聚合类似，reduce操作也会将KeyedStream转换为DataStream。它不会改变流的元素数据类型，所以输出类型和输入类型是一样的。

调用KeyedStream的reduce方法时，需要传入一个参数，实现ReduceFunction接口。接口在源码中的定义如下：

```java
public interface ReduceFunction<T> extends Function, Serializable {
    T reduce(T value1, T value2) throws Exception;
}
```

ReduceFunction接口里需要实现reduce()方法，这个方法接收两个输入事件，经过转换处理之后输出一个相同类型的事件；所以，对于一组数据，我们可以先取两个进行合并，然后再将合并的结果看作一个数据、再跟后面的数据合并，最终会将它“简化”成唯一的一个数据，这也就是reduce“归约”的含义。在流处理的底层实现过程中，实际上是将中间“合并的结果”作为任务的一个状态保存起来的；之后每来一个新的数据，就和之前的聚合状态进一步做归约。

其实，reduce的语义是针对列表进行规约操作，运算规则由ReduceFunction中的reduce方法来定义，而在ReduceFunction内部会维护一个初始值为空的累加器，注意累加器的类型和输入元素的类型相同，当第一条元素到来时，累加器的值更新为第一条元素的值，当新的元素到来时，新元素会和累加器进行累加操作，这里的累加操作就是reduce函数定义的运算规则。然后将更新以后的累加器的值向下游输出。

我们可以单独定义一个函数类实现ReduceFunction接口，也可以直接传入一个匿名类。当然，同样也可以通过传入Lambda表达式实现类似的功能。

与简单聚合类似，reduce操作也会将KeyedStream转换为DataStrema。它不会改变流的元素数据类型，所以输出类型和输入类型是一样的。

下面我们来看一个稍复杂的例子。

我们将数据流按照用户id进行分区，然后用一个reduce算子实现sum的功能，统计每个用户访问的频次；进而将所有统计结果分到一组，用另一个reduce算子实现maxBy的功能，记录所有用户中访问频次最高的那个，也就是当前访问量最大的用户是谁。

```java
import org.apache.flink.api.common.functions.MapFunction;
import org.apache.flink.api.common.functions.ReduceFunction;
import org.apache.flink.api.java.tuple.Tuple2;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;

public class TransReduceTest {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);
        // 这里的ClickSource()使用了之前自定义数据源小节中的ClickSource()
        env.addSource(new ClickSource())
                // 将Event数据类型转换成元组类型
                .map(new MapFunction<Event, Tuple2<String, Long>>() {
                    @Override
                    public Tuple2<String, Long> map(Event e) throws Exception {
                        return Tuple2.of(e.user, 1L);
                    }
                })
                .keyBy(r -> r.f0) // 使用用户名来进行分流
                .reduce(new ReduceFunction<Tuple2<String, Long>>() {
                    @Override
                    public Tuple2<String, Long> reduce(Tuple2<String, Long> value1, Tuple2<String, Long> value2) throws Exception {
                        // 每到一条数据，用户pv的统计值加1
                        return Tuple2.of(value1.f0, value1.f1 + value2.f1);
                    }
                })
                .keyBy(r -> true) // 为每一条数据分配同一个key，将聚合结果发送到一条流中去
                .reduce(new ReduceFunction<Tuple2<String, Long>>() {
                    @Override
                    public Tuple2<String, Long> reduce(Tuple2<String, Long> value1, Tuple2<String, Long> value2) throws Exception {
                        // 将累加器更新为当前最大的pv统计值，然后向下游发送累加器的值
                        return value1.f1 > value2.f1 ? value1 : value2;
                    }
                })
                .print();
        env.execute();

    }
}
```

reduce同简单聚合算子一样，也要针对每一个key保存状态。因为状态不会清空，所以我们需要将reduce算子作用在一个有限key的流上。