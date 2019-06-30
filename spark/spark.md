创建RDD的两个方法：

1. 使用parallel方法，把driver内存中的一个collection转换成RDD;
2. 通过读取外部数据的方式生成一个RDD



设置partition，一般情况下，一个cpu上处理2～4个partition比较合适



Operations:

*transformation* ：从一个RDD创建一个新的RDD；因为RDD是不可变的，因此不会在原RDD上做修改

*action*：在一个RDD上进行计算，并且将结果返回到driver上。这里要注意的是，reduce是action；而reduceByKey 是一个transformation, 不是 action，但是countByKey是一个action。



因为transformation是惰性的，因此对于一个transformation生成的RDD，如果在上面做重复的action，其实会导致这个生成的RDD会被重复计算。因此，可以通过persist机制，来实现中间RDD结果的重用；

注意，即使是读取外部生成RDD也是一个transformation。因此，读入的数据如果会被重复使用，也需要persisit，否则会导致重复读取；





map 和 mapPartitions





