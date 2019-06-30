# ElasticSearch 笔记

## 通用

1. 数字的相对距离用法：

now-1h，now-1d，2019-02-21T06:50:57||-1h，1550728258000||-1h

时间单位M是月份，m是分钟

注意时区的问题，ES写入时默认使用的是UTC时区，Kibana会自动转换，但是自己的API使用时要自己转换

https://www.elastic.co/guide/en/elasticsearch/reference/current/common-options.html

2. filter_path参数，可以过滤返回的结果

```js
/_search?filter_path=took,hits.hits._id,hits.hits._source
```

3. 多个索引查询

查询api的完整格式应该是：GET /${index}/_search?xxxx，其实用POST方法也可以

其中${index}是要检索的index（如果不写的话，相当于_all，代表查询所有index）

支持同时检索多个index，而且这是一个常态需求。因为一般情况下，比如日志查询中，会根据收集上来的日期来切分index。所以如果想查多个日期的数据，就要用多索引查询。

多个索引查询主要包括三种模式：

1. 用逗号分割开的索引：test1,test2,test3
2. 使用通配符：test1,test2*,test3
3. 使用减号(-)来去掉某些索引：test1, test2\*,-test22\*

## Count API

Count API的endpoint为：/${index..}/\_doc/\_count，后面跟QueryDSL。

QueryDSL和SearchAPI的一样，也可以用URI 和 Request Body 两种形式，参数名也一样



## Search API

查询API首先分为 **URI Search** 和 **Request Body Search** 两种：

**当同时有URI查询条件和RequestBody中的条件时，以URI中的为准**

URI Search

1. 直接将查询参数通过URI参数进行提交
2. 查询语句使用*query_string* 查询，因此可以说是Query DSL中的一个子集
3. 重要的参数包括：q(查询条件), sort(排序条件，格式为：field0:asc,field1desc), from和size（偏移量和返回条数，默认为0, 10）

### 使用RequestBody

使用RequestBody可以表达更强大的查询条件，要注意的是，即使是使用GET方法，也可以提交RequestBody。但是有时客户端会对此做一些限制，比如在curl中，如果使用-d参数指定了RequestBody，则会自动使用POST方法。因此，需要使用-X GET来强制使用GET。而PostMan中，在GET方法的情况下，则直接不能填写RequestBody。有些httpClient库也会有类似的限制，不过经试验，查询API也可以使用POST方法访问。

#### 基本结构

```
{
  "query": {
    "bool": {
      "filter": [
        { "term": { "color": "red"   }},
        { "term": { "brand": "gucci" }}
      ]
    }
  },
  "aggs": {
    "models": {
      "terms": { "field": "model" } 
    }
  }
}
```



#### Sort 排序

如果使用score排序的话，需要使用特殊字段"_score"。

对于_score，默认的排序顺序是desc；对于其他字段，默认排序顺序为asc。

##### 简化写法

对于完整写法，要写成`{"post_date" : {"order" : "asc"}}`

如果只设置顺序的话，也可以简化写为：`{"post_date" : asc"}`

甚至于如果使用默认顺序的话，可以直接用字段名代替：`"post_date"`

##### 数组字段的排序

对于数组字段，可以使用mode 参数，可以选择使用数组中的min，max，sum，medium，avg值进行比较

##### 缺失字段的处理

使用"missing"参数，可选的值为："\_last"和"\_first"两个值，分别表示缺失的情况下排在最后或最前

#### post_filter 过滤

这个过滤的特点在于，它将在整个查询的最后一步进行过滤。首先我们要理解的是，整个查询的过程是先查询，然后进行聚合。post_filter过滤会在聚合完成之后再进行过滤。你可以大致将其理解为SQL中的having子句，它可以帮助你实现，你的聚合结果中依赖某些查询结果数据，但是再最终返回的结果中再过滤掉那些不感兴趣的数据。







