# RDD 实现

配合着一起看：<https://so.csdn.net/so/search/s.do?p=1&q=RDD&t=blog&domain=&o=&u=legotime&s=&l=&f=false&rbg=0>





## scala 基础

* classTag
* Either类：参见：<https://colobu.com/2015/06/11/Scala-Either-Left-And-Right/>

## 基础函数

* clean：ClosureCleaner类的作用就是递归清理外围类中无用域，降低序列化的开销，防止不必要的不可序列化异常，具体还没细看，参见：<https://www.jianshu.com/p/51f5a34e2785>

* withScope：用来展示DAG可视化的，参见：<https://blog.csdn.net/legotime/article/details/51289351>

* toDebugString：可以以文字的方式打印RDD的logical Plan

  



## transform的实现

**基于MapPartitionsRDD实现**：

包括 map, flatMap, filter, mapPartitions, glom, 操作：

**By类型操作**：

包括by类型的操作，groupBy，keyBy和sortBy，首先通过map方法将原RDD提取出key变成(k, v)的PairRDD，然后通过PairRDDFunctions实现

**集合操作**：

* 笛卡尔积：生成cartesianRDD
* union：通过UnionRDD
* intersection：首先通过x => (x, null) 转换为PairRDD，然后通过coGroup操作，再过滤出两个value都不为空的keys
* subtract：首先x => (x, null)， 然后用subtractByKey实现

**zip操作**：基于zipPartitionsRDD实现

**distinct操作**：这个方法会尽量不进行shuffle，如果对于已经用shuffle过并且分区数不变的情况，分区之间是不会有重复的，因此分区内去重即可；否则，需要进行x => (x, null)，然后通过一次shuffle，把所有的key收集起来实现（partitioner不是每个rdd都有的，一般只有刚shuffle后的RDD才会有，shuffle之后再进行其他非shuffle的transform操作会创建一个不含partitioner的RDD）

**repartition/coalesce操作**：待续



cartesianRDD 的计算，是一个N:1dependency，这种dependency 有两种情况，如果是直接从数据源读，每个结果rdd的partition会读多个数据源，也就是说一个数据源的数据会被读多次；



但是如果是shuffle之后再进行N:1 depedency，则中间会进行一次broadcast exchange；也就是说，exchange分两种，一种是shuffle exchange，这种是shuffle dependency，特点是child partition只需要parent partition的部分数据，所以需要shuffle一把；而N：1dependcy是child partiition需要parent 的多个partition，但是要全部数据，所以不需要shuffle，直接broadcast即可；













