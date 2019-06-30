# RDD：Spark编程模型

RDD 的类型：

* 基础RDD
* PairRDD，会有额外的一些定义在PairRDDFunctions中的方法可用，
* DoubleRDD，会有额外定义在DoubleRDDFunctions中的方法可用
* 输入类型的RDD，比如JdbcRDD，NewHadoopRDD等等，代表各种InputRDD



## RDD操作



### 基础RDD操作

**transform操作**：

* 集合类操作
  * 求并(union/++)操作：很直白的操作，把多个RDD合成一个RDD，多用于数据从多个数据源中读入然后合并成一个RDD进行后续操作；合并后的Partitions数是各个子RDD的Partition个数之和；
  * 笛卡尔积(cartesian)：求两个RDD的笛卡尔积
  * 求交(intersection)操作：求两个RDD中数据的交集，并且去重，因此内部会有一个Shuffle操作
  * 求差(subtract)操作：求两个RDD的差集，使用左边RDD的partitioner和partition个数

* zip相关操作：
  * **普通zip**：zip的意思就是 a, b => (a, b)；因此，普通zip操作就是两个RDD的每个分区的每个元素做zip操作，因此**要求两个RDD有相同的partition个数，并且每个partition有相同的元素**
  * **zipPartitions操作**：也是两个RDD的zip操作，并且要求两个RDD的partition个数相同；但是和普通zip不同的是，它要求提供一个函数(Iterator[T], Iterator[U]) ＝> Iterator[V]，把两个分区的数据进行全局zip，而不是像普通zip操作那样将分区内的数据1:1的zip；**zipPartitions**操作不要求partition中有相同的元素个数
  * **zipWithIndex操作**：这是一个RDD自己的操作，简单来说就是自己和自己的下标进行zip；这个下标是个连续的Long，排序方式是按partition升序，然后再按partition内的顺序升序；同时要注意的是这个操作的顺序对于某些shuffle类的RDD是不能保证Index每次都一样的，因此如果要保证的话，需要自己进行sort操作
  * **zipWithUniqueId操作**：同样是一个RDD自己的操作，和withIndex不同的是，它的uniqueId计算方法并不保证是连续的，因此它只需要自己的partition内的数据，不需要提交spark 任务就可以完成；而zipWithIndex在partition个数超过1的时候需要提交Spark任务完成计算
* repartition/coalesce操作：
* distinct：去重
* map 系列操作：转换数据为新的RDD，但是要求原RDD中的每个元素必需转换成一个新元素
  * map/mapPartitions：接口不太一样，但是底层实现上目前已经没有区别了
  * mapPartitionsWithIndex：允许自己的转换函数获取partitionIndex的信息
* flatMap：转换数据到新的RDD，但是允许原RDD中的一个元素转换为0个，1个或多个元素
* filter：过滤出想要的数据的RDD
* by 系列操作：从元素中提取出一个key，然后进行操作
  * groupBy：转换为pairRDD，将原数据按key进行分组，key用作k，value是分组后的数据汇总的列表
  * keyBy：将原数据转换为pairRDD，key用作k，value是原数据
  * sortBy：仍然是原类型的RDD，但是根据key进行排序

**action操作**： action操作的特点是返回值不是一个RDD，而是一个Scala类型，比如数组，或者数字类型等

* 收集数据类
  * top/takeOrdered：获取降序/升序的前若干个数据
  * take：按index获取前若干个数据
  * takeSample：采样获取若干个数据
  * first：相当于take(1)，获取第一个数据
  * toLocalIterator/collect：收集数据到本地
* 归并类操作：
  * reduce：同类型的将多个值汇总为一个值，无法设置ZeroValue，返回值和RDD元素类型相同
  * fold：reduce的扩展版，同样要求生成相同类型的结果，但是可以提供相同类型的ZeroValue值
  * aggregate：fold的扩展版，可以提供一个不同类型的ZeroValue，返回值的类型和ZeroValue的类型相同
* 统计类操作：
  * max/min：获取最大值/最小值
  * count/countApprox：获取精确总数/获取估计的总数
  * countApproxDistinct：使用HyperLogLog算法，获取近似的去重总数，用作大数据量时候的去重计算方案
  * countByValue/countByValueApprox：统计每个元素的个数/估计个数，
* foreach/foreachPartition：这两个操作是比较特殊的action，因为他们不返回结果。他们提供一个对生成的RDD自定义数据处理的能力
* 写入文件：saveAs...：将RDD持久化到文件中，因为目前都用DataFrame了，这个api用的相对少；



**其他操作**： 

* unpersist，persist，cache： 将一个已经生成的RDD进行暂存，从而加速对该RDD的重用；这个操作一般在一系列transform的最后执行，因为它的返回值不是一个RDD，因此无法进行链式操作；
* checkpoint：checkpoint和前面的persist的本质区别是，persist是保存在执行节点本地的资源上的，checkpoint会将其存储在一个可靠存储上（一般是HDFS）；另外一个要点是，checkpoint之后会将RDD的依赖数据删除，因此后续再需要计算需要重复计算一遍，因此强烈建议checkpoint之前先对数据做一次persist，防止数据重复被重复计算



### PairRDD操作

PairRDD的操作主要是在基础RDD操作的基础上，增加了大量的按Key进行的操作，并且这些操作大多数是返回RDD的transform操作，只有count和lookup，saveAsxxx是action

**transform操作**

* cogroup/groupWith操作：cogroup和groupWith是同一个操作，类似于先将两个RDD分别进行groupByKey，然后再对这两个RDD进行join操作
* partitionBy：提供一个partitioner，对原RDD重新进行分区
* groupByKey：和基础RDD的groupBy操作一样，只是这里不用提供一个提取key的函数，直接用Pair的key值
* join，leftOuterJoin，rightOuterJoin，fullOuterJoin：类似SQL的join操作，将(k, v) 和 (k, u) 转换为 (k, (v, u))；如果是OuterJoin的话，可能为空的列使用Option类型
* subtractByKey：类似于基础RDD的subtract操作，但是通过key来做差，也就是返回第一个RDD中的key不在在第二个RDD中的所有数据
* mapValues：和基础RDD不同的是，这里的mapValues只应用到value上，也就是说通过(v) => u 将原始的(k, v) 转变为(k, u)，并保持原来的k不变
* flatMapValues：和mapValues类似
* keys/values：获取所有的key/value，注意返回的也是RDD，不是数组列表
* foldBykey，aggregateByKey，reduceByKey，combineByKey：和基础RDD操作的含义类似，只是在每个key内进行操作
* countApproxDistinctByKey：计算每个key里面的去重总数估计

**action操作**

* countByKey/countByKeyApprox：对每一个key进行计数
* lookup：提供一个key，获取这个key下所有values的列表；类似于filter(k => k == key).values.collect
* saveAsxxx：保存到文件



### OrderedRDD操作

OrderedRDD操作也是面向 **PairRDD**，但是是当Pair的key是可排序的情况下，也就是有Ordering[K]存在的情况。这种情况下，可以支持一些对Key排序相关的操作：

* filterByRange：提供一个key的range，过滤出原RDD中，key在这个range中的pair，生成一个新RDD
* sortByKey：和基础RDD的sortBy类似，只是key用pair中的key
* repartitionAndSortWithinPartitions：这是个组合操作，基础的repartiton操作是不保证内部的顺序的，这个方法会保证partition内部的数据按key进行排序



### DoubleRDD操作

DoubleRDD引入的操作主要是针对double类型的数据可以实现的一些统计分析结果，包括**histogram**，**sum**， **stdev**，**variance**， **mean** 这五类，都是action，并且含义都不言而喻，不再过多赘述



### 

## 按操作类型汇总







