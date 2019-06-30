# SparkSQL

## DataSet 和 DataFrame

DataFrame in Scala/Java is a DataSet of Rows.

 Dataset 是新版本的RDD，它使用了专用的encoder来实现高效的网络间传输

使用SparkSession来获取SparkSQL的执行环境





SparkSession和SparkContext的关系

sparkContext读取的是RDD，sparkSession.read.xxx读取的是dataframe；dataframe中的api是sparkSQL相关的；rdd相关的api是传统spark的



RDD和DataFrame的转换: 如果有确定的Schema（比如数据Model类已经存在，则可以直接通过RDD构建对象，变成dataframe）

否则，可以动态创建Row对象，并指定schema



## RDD 和 DataSet/DataFrame的区别

## coalesce 和 repartition 的区别

coalesce（合并的意思）接口：coalesce(numPartitions:Int，shuffle:Boolean=false):RDD[T]

和 repartition接口的区别：repartition(numPartitions:Int):RDD[T]

可以看出，coalesce接口是多了一个shuffle参数的，默认是不进行shuffle。

实际上，<span style='color:red'>repartition ＝ coalesce(shuffle = true)</span>

这两个函数的作用都是用来对数据进行重新分区，而区别只在于coalesce函数允许在分区的时候不进行shuffle

为什么可以在重新分区的时候不进行shuffle呢？这主要是面向当修正后的分区比原分区数要少的时候，这时，其实可以通过将若干个分区合并成新分区的方式实现，而没必要进行shuffle操作。

而当增加分区个数的时候，则必须要求shuffle ＝ true。对于修改后分区比修改前分区数多的情况，如果不开启shuffle，则该函数是无效的。

因此，一般情况下，需要shuffle的时候，会使用repartition函数；不需要shuffle的时候，会使用coalesce函数

## cache 和 persist 的区别

这两个函数都是在 Spark 中负责数据持久化的函数。本质上，cache是一个persist的包装，persist函数可以选择将数据持久化到不同的级别，比如纯内存，纯硬盘 或者 内存＋硬盘等。而cache只允许数据保存在纯内存中。

具体说到持久化的作用，其实就是如果一份计算后的数据需要重复被使用的话，可以将这份数据缓存起来，以备后续使用时不用再重新计算。

持久化的时候，cache()函数后面不要再有额外的action，否则仍会重新触发计算（待考证）

cache()需要是一系列transformation后的操作，而不能是另起一行单独进行一次cache（待考证）





Spark SQL 写hive表，列名大小写不敏感，转换为RDD的时候，名字可能会有点问题











