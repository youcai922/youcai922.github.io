## 初识Flink

#### Flink的起源和设计概念

- Flink是Apache基金会旗下的一个开源大数据处理框架。
- 起源于一个叫做Stratosphere的项目

- 官网：https://flink.apache.org/
- 核心目标：数据流上的有状态计算（Stateful Computations over Data Streams）

- 具体定位是一个框架和分布式处理引擎，如下图所示，用于对误解和游街数据流进行有状态计算。Flink被设计在所有常见的集群环境中运行，以内存执行速度和任意规模来执行计算。

![Flink框架处理流程](https://youcai922.github.io/99.src/img/Flink框架处理流程.png)