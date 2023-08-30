### JobManager和TaskManager

Flink中的节点可以分为JobManager和TaskManager。

JobManager处理器也称之为Master，用于协调分布式任务执行。他们用来调度Task进行具体的任务。TaskManager处理器也称为Worker，用于实际执行任务。



### 并发度和Slots

每一个TaskManager是一个独立的JVM进程，它可以在独立的线程上执行一个或多个任务task。为了控制一个taskManager能接受多少task，TaskManager上就会划分多个slot来进行控制。每个slot表示的是TaskManager上拥有资源的一个固定大小的子集。flink-confg.yaml配置文件中的

```
# 集群的并行计算的能力
taskmanager.numberOfTaskSlots: 1
# 任务需要的并行度（代码配置优先度>web配置优先度>flink配置文件优先度）
parallelism.default: 1
```

