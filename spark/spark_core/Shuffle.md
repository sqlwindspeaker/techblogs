# Spark Shuffle

在Spark的计算框架中，数据（也就是RDD）之间的依赖关系主要分为OneToOneDependency（1:1）和ShuffleDependency（M:N）两种。其中所有连续的OneToOneDependency关系会被整合在一个Stage之间计算，数据通过pipeline的方式进行计算和传递。而对于ShuffleDependency，Spark会把计算分成相依赖的两个Stage，上游的Stage将结果通过被称之为ShuffleWrite的方式写入Storage，下游的Stage通过ShuffleRead的方式，从Storage中读取输入，再进行接下来的计算。因此，Spark Shuffle从宏观上看，就是Spark对于上下游的Stage之间的数据进行传递的过程，如下图所示



图片



## Shuffle Write 和 Shuffle Read 的触发点

根据前面我们说的，Shuffle过程是发生在上下游Stage之间的数据交换。因此，对于只有一个Stage的Spark任务，它这个Stage是一个FinalStage，这个任务中是没有Shuffle发生的。对于有多个Stage的任务，那么：

* 最后一个Stage是FinalStage，它的输入是ShuffleRead，输出取决于具体的Action
* 对于第一级Stages（注意不是第一个，可能有多个输入），它们都是ShuffleMapStage，他们的输入是特定的输入源，输出是ShuffleWrite
* 对于所有其他的Stage，他们都是ShuffleMapStage，他们的输入都是ShuffleRead，输出是ShuffleWrite

进一步归纳就是：

* 对于每一个非FinalStage（就是ShuffleMapStage），它都会产生一个Shuffle Write操作。
* 对于每一个非第一级的Stage，它都会产生Shuffle Read。

因为每一个Stage的具体执行，都是由它所包含的Task的执行来实现的。因此Stage之间的ShuffleWrite/ShuffleRead操作也就是相关的Task的ShuffleWrite/ShuffleRead。具体到Task来说，

* Task是否需要进行ShuffleWrite是由Task的类型决定的，只要是ShuffleMapTask，就需要进行ShuffleWrite；
* 一个Task是否需要进行ShuffleRead，是由这个Task的InputRDD类型决定的，也就是说ShuffleRead其实是某一种RDD(比如 ShuffledRDD，CoGroupedRDD)的输入；

因此，ShuffleWrite和ShuffleRead的触发点是不太一样的，ShuffleWrite的触发是通过ShuffleMapTask在RDD的计算逻辑之后主动插入的，Task对此是有感知的。而ShuffleRead的执行是在RDD内部的，作为RDD的计算逻辑中的一部分进行的，Task面对发生ShuffleRead的RDD和其他的RDD是完全相同的，因此从Task执行层面上讲，对ShuffleRead的存在是无感知的


## 不是所有的跨网络数据传输都是Shuffle操作

## Shuffle 涉及的组件

### ShuffleId

每一个Shuffle过程都会有一个ShuffleID与之对应，这个ID在ShuffleDependency创建的时候被创建，并保存在ShuffleDependency中

###ShuffleDependency

一个ShuffleDependency代表着一个特定的Shuffle步骤，内部会包含 `Partitioner`，`Serializer` ，`KeyOrderFunc` ，`Aggregator` 等Shuffle操作会用到的对象。

ShuffleDependency的创建是在相关的RDD（ShuffledRDD，CoGroupedRDD）的 getDependencies 方法中创建的。但是要注意的是，创建的ShuffleDependency中包含的RDD是当前RDD的ParentRDD。创建之后，DAGScheduler会在创建Stage的过程中，从后向前遍历RDD Chain，每找到一个ShuffleDependency，就根据这个ShuffleDependency创建一个Stage。因此，**ShuffleDependency会被两个对象持有，一个是根据它创建的Stage，另外一个是创建它的RDD**。这两个对象，分别是我们前文所提到的ShuffleWrite和ShuffleRead的入口。举个例子，假设RDD关系如下：

RDD3 <- - - RDD2 <- - - RDD1 <- - - RDD0

我们知道 RDD3应该是InputRDD，它没有ParentRDD，因此他没有Dependency。假设（RDD3，RDD2），（RDD1，RDD0）之间都是OneToOneDependency，RDD2和RDD1之间是ShuffleDependency，那么这个RDD Chain会被分为两个Stage，首先创建Stage0，是一个ShuffleMapStage，包含RDD2< - RDD1之间的ShuffleDependency，对应的RDD是RDD2；接下来是Stage1，是一个FinalStage，对应的RDD包含RDD3 。

在执行的时候，Stage0先执行，它的每个ShuffleMapTask会通过ShuffleDependency中的ShuffleHandle迭代每一条RDD2的数据，从而Pipeline的方式触发RDD3的计算，得到数据后，由ShuffleWriter进行ShuffleWrite写入Storage系统。在Stage1执行时，FinalStage的ResultTask会通过用户的actionFunc，驱动迭代读取RDD0的数据，从而同样以Pipeline的方式触发RDD1的计算。而RDD1在计算的过程中，它的输入是ShuffleRead，它会通过自己持有的ShuffleDependency中的shuffleHandle，获取ShuffleReader，进而读取ShuffleWrite的结果

### ShuffleManager

### ShuffleWriteProcessor

### ShuffleWriter

### ShuffleReader

### ShuffleBlockResolver







s