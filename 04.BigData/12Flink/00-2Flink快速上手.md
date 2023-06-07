#### 1、创建Maven项目

#### 2、添加项目依赖

```xml
<properties>
    <flink.version>1.13.0</flink.version>
    <java.version>1.8</java.version>
    <!--因为Flink的架构中使用了Akka来实现底层的分布式通信，而Akka是用Scala开发的-->
    <scala.binary.version>2.12</scala.binary.version>
    <slf4j.version>1.7.30</slf4j.version>
</properties>

<dependencies>
<!-- 引入Flink相关依赖-->
    <dependency>
        <groupId>org.apache.flink</groupId>
        <artifactId>flink-java</artifactId>
        <version>${flink.version}</version>
    </dependency>
    <dependency>
        <groupId>org.apache.flink</groupId>
        <artifactId>flink-streaming-java_${scala.binary.version}</artifactId>
        <version>${flink.version}</version>
    </dependency>
    <dependency>
        <groupId>org.apache.flink</groupId>
        <artifactId>flink-clients_${scala.binary.version}</artifactId>
        <version>${flink.version}</version>
</dependency>
<!-- 引入日志管理相关依赖-->
    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-api</artifactId>
        <version>${slf4j.version}</version>
    </dependency>
    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-log4j12</artifactId>
        <version>${slf4j.version}</version>
    </dependency>
    <dependency>
        <groupId>org.apache.logging.log4j</groupId>
        <artifactId>log4j-to-slf4j</artifactId>
        <version>2.14.0</version>
	</dependency>
</dependencies>
```

#### 3、配置日志管理

在目录src/main/resources下添加文件：log4j.properties，内容配置如下：

```properties
log4j.rootLogger=error, stdout
log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern=%-4r [%t] %-5p %c %x - %m%n
```



#### 4、批处理示例

- 在根目录下创建input文件夹，创建文本文件words.txt

- 在word中输入一些文字

  ```
  hello world
  hello flink
  hello java
  ```

- 新建BatchWordCount.class

  ```java
  import org.apache.flink.api.common.typeinfo.Types;
  import org.apache.flink.api.java.ExecutionEnvironment;
  import org.apache.flink.api.java.operators.AggregateOperator;
  import org.apache.flink.api.java.operators.DataSource;
  import org.apache.flink.api.java.operators.FlatMapOperator;
  import org.apache.flink.api.java.operators.UnsortedGrouping;
  import org.apache.flink.api.java.tuple.Tuple2;
  import org.apache.flink.util.Collector;
  
  public class BatchWordCount {
      public static void main(String[] args) throws Exception {
          // 1. 创建执行环境
          ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();
          // 2. 从文件读取数据  按行读取(存储的元素就是每行的文本)
          DataSource<String> lineDS = env.readTextFile("input/words.txt");
          // 3. 转换数据格式
          FlatMapOperator<String, Tuple2<String, Long>> wordAndOne = lineDS
                  .flatMap((String line, Collector<Tuple2<String, Long>> out) -> {
                      String[] words = line.split(" ");
                      for (String word : words) {
                          out.collect(Tuple2.of(word, 1L));
                      }
                  })
                  .returns(Types.TUPLE(Types.STRING, Types.LONG));//当Lambda表达式使用 Java 泛型的时候, 由于泛型擦除的存在, 需要显示的声明类型信息
          // 4. 按照 word 进行分组
          UnsortedGrouping<Tuple2<String, Long>> wordAndOneUG = wordAndOne.groupBy(0);
          // 5. 分组内聚合统计
          AggregateOperator<Tuple2<String, Long>> sum = wordAndOneUG.sum(1);
          // 6. 打印结果
          sum.print();
      }
  }
  ```

  代码说明和注意事项：

  - Flink在执行应用程序前应该获取执行环境对象，也就是运行时上下文环境。

  - Flink同时提供了Java和Scala两种语言的API，有些类在两套API中名称是一样的。所以在引入包时，如果有Java和Scala两种选择，需要注意选用Java的包。

  - 直接调用执行环境的readTextFile方法，可以从文件中读取数据。

  - 我们的目标是将每个单词对应的个数统计出来，所以调用flatmap方法可以对一行蚊子进行分词转换。将文件中每一行文字拆分为单词后，要转换成（word,count）形式的二元组，初始count都为1.returns方法指定的返回数据类型Tuple2，就是Flink自带的二元组数据类型。

  - 在分组时调用了groupBy方法，它不能使用分组选择器，只能采用位置索引或属性名称进行分组

    ```java
    //使用索引定位
    dataStream.groupBy(0)
    //使用类属性名称
    dataStream.groupBy("id")
    ```

  - 在分组之后调用sum方法进行聚合，同样只能指定聚合字段的位置索引或属性名称

需要注意的是，这种代码的实现方式，是基于DataSet API的，也就是我们对数据的处理转换，是看作数据集来进行操作的。事实上Flink本身是流批统一的处理架构，批量的数据集本质上也是流，没有必要用两套不同的API来实现。所以从Flink 1.12开始，官方推荐的做法是直接使用DataStream API，在提交任务时通过将执行模式设为BATCH来进行批处理：

```
$ bin/flink run -Dexecution.runtime-mode=BATCH BatchWordCount.jar
```

这样，DataSet API就已经处于“软弃用”（soft deprecated）的状态，在实际应用中我们只要维护一套DataStream API就可以了。这里只是为了方便大家理解，我们依然用DataSet API做了批处理的实现。

#### 5、流处理示例

我们已经知道，用DataSet API可以很容易地实现批处理；与之对应，流处理当然可以用DataStream API来实现。对于Flink而言，流才是整个处理逻辑的底层核心，所以流批统一之后的DataStream API更加强大，可以直接处理批处理和流处理的所有场景。

DataStream API作为“数据流”的处理接口，又怎样处理批数据呢？

在Flink的视角里，一切数据都可以认为是流，流数据是无界流，而批数据则是有界流。所以批处理，其实就可以看作有界流的处理。

对于流而言，我们会在获取输入数据后立即处理，这个过程是连续不断的。当然，有时我们的输入数据可能会有尽头，这看起来似乎就成了一个有界流；但是它跟批处理是截然不同的

- 读取文件

  - 新建Java类BoundedStreamWordCount，在静态main方法中编写代码。具体代码实现如下：

    ```java
    import org.apache.flink.api.common.typeinfo.Types;
    import org.apache.flink.api.java.tuple.Tuple2;
    import org.apache.flink.streaming.api.datastream.DataStreamSource;
    import org.apache.flink.streaming.api.datastream.KeyedStream;
    import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;
    import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
    import org.apache.flink.util.Collector;
    
    import java.util.Arrays;
    
    public class BoundedStreamWordCount {
        public static void main(String[] args) throws Exception {
            // 1. 创建流式执行环境
            StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
            // 2. 读取文件
            DataStreamSource<String> lineDSS = env.readTextFile("input/words.txt");
            // 3. 转换数据格式
            SingleOutputStreamOperator<Tuple2<String, Long>> wordAndOne = lineDSS
                    .flatMap((String line, Collector<String> words) -> {
                        Arrays.stream(line.split(" ")).forEach(words::collect);
                    })
                    .returns(Types.STRING)
                    .map(word -> Tuple2.of(word, 1L))
                    .returns(Types.TUPLE(Types.STRING, Types.LONG));
            // 4. 分组
            KeyedStream<Tuple2<String, Long>, String> wordAndOneKS = wordAndOne
                    .keyBy(t -> t.f0);
            // 5. 求和
            SingleOutputStreamOperator<Tuple2<String, Long>> result = wordAndOneKS
                    .sum(1);
            // 6. 打印
            result.print();
            // 7. 执行
            env.execute();
        }
    }
    ```

- 差异对比（与批处理程序BatchWordCount的不同）

  - 创建执行环境的不同，流处理程序使用的是StreamExecutionEnvironment
  - 每一步处理转换之后，得到的数据对象类型不同
  - 分组操作调用的是keyBy方法，可以传入一个匿名函数作为键选择器（KeySelector），指定当前分组的key是什么
  - 代码末尾需要调用env的execute方法，开始执行任务