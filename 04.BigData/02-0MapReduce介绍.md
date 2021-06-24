# MapReduce

#### MapReduce是什么

1. MapReduce是一种分布式计算框架 ，以一种可靠的，具有容错能力的方式并行地处理上TB级别的海量数据集。主要用于搜索领域，解决海量数据的计算问题。
2. MR有两个阶段组成：Map和Reduce，用户只需实现map()和reduce()两个函数，即可实现分布式计算。
   1. Map()负责把一个大的block块进行切片并计算。
   2. Reduce() 负责把Map()切片的数据进行汇总、计算。



#### 工作原理

![img](https://gimg2.baidu.com/image_search/src=http%3A%2F%2Fimage.mamicode.com%2Finfo%2F201805%2F20180505214236856637.png&refer=http%3A%2F%2Fimage.mamicode.com&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=jpeg?sec=1627005998&t=21e7cf0d2aaa3de3ea1abb19015219e4)

1. 第一步对输入的数据进行切片，每个切片分配一个map()任务，map()对其中的数据进行计算，对每个数据用键值对的形式记录，然后输出到环形缓冲区。
2. map（）中输出的数据在环形缓冲区内进行快排，每个环形缓冲区默认大小100M，当数据达到80M时（默认），把数据输出到磁盘上。形成很多个内部有序整体无序的小文件。
3. 框架把磁盘中的小文件传到Reduce()中来，然后进行归并排序，最终输出。
