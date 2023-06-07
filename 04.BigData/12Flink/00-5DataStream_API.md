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

