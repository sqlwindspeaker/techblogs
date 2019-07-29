# Spark 系统概述

Spark整个生态系统中，我们关注的主要是三个部分：

* 最底层的是SparkCore，包括了Spark的执行环境，RDD相关的一些基础实现
* SQL接口：SparkSQL，包括了SparkSession，DataSet/DataFrame的API，以及SQL的解析，转化为RDD的实现等等
* 流处理：SparkStreaming，包含了将流数据抽象成DStream，再进一步转化为DataSet/DataFrame/RDD的操作

其余的应用库，包括GraphX，MLlib，repl等等，都是特定的应用或者库，从学而为用的角度上来看，不是我们关注的重点。

其中SparkCore部分，包含了整个Spark的基础，包括RDD的概念，数据的分布式执行，如何在集群中部署等等。我们首先关注SparkCore部分的实现，这也将为后续研究SparkSQL和SparkStreaming打下一个坚实的基础。

## SparkCore

学习SparkCore的代码，主要问题就是要弄明白以下这些问题：

* 一个Spark任务，在提交到集群上去执行的时候，都经历了哪些处理过程？
* 数据是如何分片的？每一个分片是如何被Task处理的？
* Task是如何获取到集群的资源的？
* Task在执行的时候是如何使用资源的（主要是存储资源，包括内存和本地硬盘）？
* 什么时候会用到Shuffle？Shuffle过程是怎么实现的？
* 什么时候会用到Broadcast？Broadcast是如何实现的？和Shuffle有什么区别？
* 计算的结果是如何获取的？
* Spark集群内部是怎么协调的？
* Spark集群和宿主集群（比如Yarn）之间是怎么映射的？

为了回答这些问题，我们需要对SparkCore的整体框架有一个比较深入的理解。从功能上来看，SparkCore主要包括 **任务调度模块**，**存储管理模块** ，**执行引擎模块** ，**集群部署模块** 这四大核心模块。在代码结构上，我们可以看到各种各样的包，我们可以分别对他们进行一下归类：

* 任务调度模块，包括 **rdd**，**scheduler** 等
* 存储管理模块，包括 **storage**， **memory** 等
* 执行引擎模块，包括 **executor**，**shuffle**，**broadcast** 等
* 集群部署模块，包括 **launcher**，**deploy** 等

当然还有一些比如 **rpc**，**network**，**io** 等等的包，属于一些基础组件类的代码，还有SparkContext相关的一些没有在特定的子包下的公共代码，会被多个系统共用，也是很重要的代码部分。