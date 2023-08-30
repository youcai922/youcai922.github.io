### 1、窗口和水位线

- 窗口：一般就是划定的一段时间范围，也就是“时间窗”；对在这范围内的数据进行处理，就是所谓的窗口计算。所以窗口和时间往往是分不开的。

- 水位线的特性：
  - 水位线是插入到数据流中的一个标记，可以认为是一个特殊的数据
  - 水位线主要的内容是一个时间戳，用来表示当前事件时间的进展
  - 水位线是基于数据的时间戳生成的
  - 水位线的时间戳必须单调递增，以确保任务的事件时间时钟一直向前推进
  - 水位线可以通过设置延迟，来保证正确处理乱序数据
  - 一个水位线 Watermark(t)，表示在当前流中事件时间已经达到了时间戳 t, 这代表 t 之前的所有数据都到齐了，之后流中不会出现时间戳 t’ ≤ t 的数据；
    F
- link 中的水位线，其实是流处理中对低延迟和结果正确性的一个权衡机制，而且把控制的权力交给了程序员，我们可以在代码中定义水位线的生成策略



### 2、水位线
 在 Flink 的 DataStream API 中 ， 有 一 个 单 独 用 于 生 成 水 位 线 的 方法：`.assignTimestampsAndWatermarks()`，它主要用来为流中的数据分配时间戳，并生成水位线来指示事件时间：

```java
public SingleOutputStreamOperator<T> assignTimestampsAndWatermarks(WatermarkStrategy<T> watermarkStrategy)
```

具体使用时，直接用 DataStream 调用该方法即可，与普通的 transform 方法完全一样。

```java
DataStream<Event> stream = env.addSource(new ClickSource());
DataStream<Event> withTimestampsAndWatermarks = stream.assignTimestampsAndWatermarks(<watermark strategy>);
```

- .assignTimestampsAndWatermarks()方法需要传入一个 WatermarkStrategy 作为参数，这就是所谓的 “ 水位线生成策略 ” 。 WatermarkStrategy 中包含了一个 “ 时间戳分配器”TimestampAssigner 和一个“水位线生成器”WatermarkGenerator。

  - TimestampAssigner：主要负责从流中数据元素的某个字段中提取时间戳，并分配给元素。时间戳的分配是生成水位线的基础。
  - WatermarkGenerator：主要负责按照既定的方式，基于时间戳生成水位线。在WatermarkGenerator 接口中，主要又有两个方法：onEvent()和 onPeriodicEmit()。
  - onEvent：每个事件（数据）到来都会调用的方法，它的参数有当前事件、时间戳，以及允许发出水位线的一个 WatermarkOutput，可以基于事件做各种操作
  - onPeriodicEmit：周期性调用的方法，可以由 WatermarkOutput 发出水位线。周期时间为处理时间，可以调用环境配置的.setAutoWatermarkInterval()方法来设置， 默认为200ms。

  - env.getConfig().setAutoWatermarkInterval(60 * 1000L);

### 3、窗口

Flink 是一种流式计算引擎，主要是来处理无界数据流的，数据源源不断、无穷无尽。想要更加方便高效地处理无界流，一种方式就是将无限数据切割成有限的“数据块”进行处理，这就是所谓的“窗口”（Window）。

#### 3.1 窗口分为时间窗口和计数窗口

时间窗口和计数窗口，只是对窗口的一个大致划分；在具体应用时，还需要定义更加精细的规则，来控制数据应该划分到哪个窗口中去。不同的分配数据的方式，就可以有不同的功能 应用。根据分配数据的规则，窗口的具体实现可以分为 4 类：滚动窗口（Tumbling Window）、滑动窗口（Sliding Window）、会话窗口（Session Window），以及全局窗口（Global Window）。

在定义窗口操作之前，首先需要确定，到底是基于按键分区（Keyed）的数据流 KeyedStream来开窗，还是直接在没有按键分区的 DataStream 上开窗。也就是说，在调用窗口算子之前， 是否有 keyBy 操作。

（1） 按键分区窗口:

​	经过按键分区 keyBy 操作后，数据流会按照 key 被分为多条逻辑流（logical streams），这就是 KeyedStream。基于 KeyedStream 进行窗口操作时， 窗口计算会在多个并行子任务上同时执行。相同 key 的数据会被发送到同一个并行子任务，而窗口操作会基于每个 key 进行单独的处理。所以可以认为，每个 key 上都定义了一组窗口，各自独立地进行统计计算。在代码实现上，我们需要先对 DataStream 调用.keyBy()进行按键分区，然后再调用.window()定义窗口。


```java
 stream.keyBy(...).window(...)
```
（2） 非按键分区

​	实际上就是没有调用keyby的数据流，这时窗口逻辑只能在一个任务（task）上执行，就相当于并行度变成了 1。实际应用中一般不推荐使用这种方式。

```java
stream.windowAll(...)
```
#### 3.2 代码中如何使用窗口：

窗口操作主要有两个部分：窗口分配器（Window Assigners）和窗口函数（Window Functions）。
stream.keyBy() .window() .aggregate()

其中.window()方法需要传入一个窗口分配器，它指明了窗口的类型；而后面的.aggregate()方法传入一个窗口函数作为参数，它用来定义窗口具体的处理逻辑。

-  窗口分配器

  按照具体的分配规则，又有滚动窗口、滑动窗口、会话窗口、全局窗口四种。除去需要自定义的全局窗口外，其他常用的类型 Flink
  中都给出了内置的分配器实现，可以方便地调用实现各种需求。

  - 滚动处理时间窗口：
    窗口分配器由类 TumblingProcessingTimeWindows 提供，需要调用它的静态方法.of()。

  ```java
    stream.keyBy(...).window(TumblingProcessingTimeWindows.of(Time.seconds(5))).aggregate(...);
  ```

  - 滑动处理时间窗口：
    窗口分配器由类 SlidingProcessingTimeWindows 提供，同样需要调用它的静态方法.of()。

  ```java
  stream.keyBy(...).window(SlidingProcessingTimeWindows.of(Time.seconds(10), Time.seconds(5))).aggregate(...)
  ```

  ​		这里.of()方法需要传入两个 Time 类型的参数：size 和 slide，前者表示滑动窗口的大小，后者表示滑动窗口的滑动步长。我们这里创建了一个长度为 10 秒、滑动步长为 5 秒的滑动窗口。

  - 滚动计数窗口
    滚动计数窗口只需要传入一个长整型的参数 size，表示窗口的大小。

  ```java
  //定义了一个长度为 10 的滚动计数窗口，当窗口中元素数量达到 10 的时候，就会触发计算执行并关闭窗口。
  stream.keyBy(...).countWindow(10)
  ```

  - 滑动计数窗口

     与滚动计数窗口类似，不过需要在.countWindow()调用时传入两个参数：size 和 slide，前者表示窗口大小，后者表示滑动步长。

  ```java
  //定义了一个长度为 10、滑动步长为 3 的滑动计数窗口。每个窗口统计 10 个数据，每隔 3 个数据就统计输出一次结果。
  stream.keyBy(...).countWindow(10，3);
  ```

  - 全局窗口
    全局窗口是计数窗口的底层实现，一般在需要自定义窗口时使用。它的定义同样是直接调用.window()，分配器由 GlobalWindows 类提供。

  ```java
  stream.keyBy(...).window(GlobalWindows.create());
  ```

     需要注意使用全局窗口，必须自行定义触发器才能实现窗口计算，否则起不到任何作用。

- 窗口函数
  窗口函数定义了要对窗口中收集的数据做的计算操作，根据处理的方式可以分为两类：增量聚合函数和全窗口函数。
  经窗口分配器处理之后，数据可以分配到对应的窗口中，而数据流经过转换得到的数据类型是 WindowedStream。这个类型并不是 DataStream，所以并不能直接进行其他转换，而必须进一步调用窗口函数，对收集到的数据进行处理计算之后，才能最终再次得到 DataStream.
  - 增量聚合函数
    窗口将无界数据转换成了有界数据，但是并不是收集齐了这个窗口里面的所有数据之后，才开始处理，而是跟流式处理一样，来一条处理一条 就像 DataStream 的简单聚合一样，每来一条数据就立即进行计算，中间只要保持一个简单的聚合状态就可以了
    典型的增量聚合函数有两个：ReduceFunction 和 AggregateFunction。
    - 归约函数（ReduceFunction）

```java
public class WindowReduceExample {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env =  StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);
        // 从自定义数据源读取数据，并提取时间戳、生成水位线
        SingleOutputStreamOperator<Event> stream = env.addSource(new ClickSource())
            .assignTimestampsAndWatermarks(WatermarkStrategy.<Event>forBoundedOutOfOrderness(Duration.ZERO)
                                           .withTimestampAssigner(new SerializableTimestampAssigner<Event>() {
                                               @Override
                                               public long extractTimestamp(Event element, long recordTimestamp){
                                                   return element.timestamp;
                                               }
                                           })); 
        stream.map(new MapFunction<Event, Tuple2<String,Long>>() {
            @Override
            public Tuple2<String, Long> map(Event value) throws Exception {
                // 将数据转换成二元组，方便计算
                return Tuple2.of(value.user, 1L);
            }
        })
            .keyBy(r -> r.f0)
            // 设置滚动事件时间窗口
            .window(TumblingEventTimeWindows.of(Time.seconds(5)))
            .reduce(new ReduceFunction<Tuple2<String, Long>>() {
                @Override
                public Tuple2<String, Long> reduce(Tuple2<String, Long> value1,Tuple2<String, Long> value2) throws Exception {
                    // 定义累加规则，窗口闭合时，向下游发送累加结果
                    return Tuple2.of(value1.f0, value1.f1 + value2.f1);
                }
            }) 
            .print();
        env.execute();
    }
}
```
