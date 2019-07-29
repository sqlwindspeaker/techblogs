# Spark Pipeline

前面一节我们重点说明了 RDD 的主要 transform 方法的具体算法，这一节我们来重点关注这些transform是怎么串起来执行的

假设我们有这样的如下任务

```scala
var srcRDD = .....
var dstRDD = srcRDD.map {l => l.split(" ")}.filter {w => w.startsWith("/")}.zipWithIndex
```



## 普通执行方式

最简单的执行方式是