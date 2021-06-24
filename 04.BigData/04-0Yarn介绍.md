### yarn

​	一种新的Hadoop资源管理器，是一个通用的资源管理系统，可为上层应用提供统一的资源管理和调度。它极大的提高了集群的利用率、资源统一管理和数据共享。

​	他的基本思想是讲JobTracker的两个主要功能（资源管理和作业调度/监控）分离，主要方法是创建一个全局的ResourceManager和若干个针对应用程序的ApplicationMaster（AM）。

​	Yarn分层结构的本质是ResourceManager。将各个资源部分（计算、内存、带宽等）进行安排给基础NodeManager（Yarn的每个节点代理）

​	ResourceManager 还与 ApplicationMaster 一起分配资源，与 NodeManager 一起启动和监视它们的基础应用程序。在此上下文中，ApplicationMaster 承担了以前的 TaskTracker 的一些角色，ResourceManager 承担了 JobTracker 的角色

**通俗的说：**

	- yarn并不清楚用户提交的程序的运行机制
	- yarn只提供运算资源的调度（用户程序向yarn申请资源，yarn就负责分配资源）
	- yarn的主管角色是ResoureceManager，具体运算资源的角色是NodeManager
	- yarn与运行的用户程序完全解耦，意味着Yarn上可以运行各种类型的分布式运算程序，比如mapreduce、storm、spark、tez...
	- spark、storm等运算框架都可以整合在Yarn上运行，只要他们各自的框架中有符合Yarn规范的资源请求机制即可
	- yarn成为一个通用的资源调度平台，企业中以前存在的各种运算集群都可以整合在一个物理集群上，提高资源利用率，方便数据共享。

