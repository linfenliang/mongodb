## 简介

聚集操作实际上是对数据进行统计分析时使用的，简单的说，可以理解为SQL中的聚合操作，MongoDB中的聚集操作是为了大数据分析做准备的，尤其是MapReduce可以在分片集群上进行操作，本节主要讲了简单的一些操作，没有涉及到MongoDB中的聚集框架，MongoDB中对数据进行分析计算的方式主要有：管道模式、MapReduce模式以及简单的函数或命令这三种聚集分析方式。

## 管道模式聚集分析

MongoDB的聚合框架是参考UNIX上的管道命令实现的，数据通过一个多步骤的管道，每个步骤都会对数据进行加工处理，最后返回需要的结果集。  
管道聚集是可以操作一个分片的集合的（The aggregation pipeline can operate on a sharded collection.）  
管道聚集在某一些阶段可以利用索引提高性能，另外，管道聚集还有一个一个内部优化阶段（后面管道聚集优化会讲到）。

下面是一个示例：

![](https://docs.mongodb.com/manual/_images/aggregation-pipeline.png)

常见的管道操作符有如下：

| 操作符 | 说明 |
| :--- | :--- |
| $match | 过滤文档 |
|$limit| 限制管道中文件的数据|
|$skip|跳过指定的文档数量|
|$sort|对所输入的文档进行排序|
|$group|对文档进行分组后计算聚集结果|
|$out|输出文档到具体的集合中（必须是管道操作的最后一步）|
与$group一起使用的聚集操符：

| 操作符 | 说明 |
| :--- | :--- |
|$first|返回group后的第一个值|
|$last|返回group后的最后一个值|
|$max|group后的最大值|
|$min|group后的最小值|
|$avg|group后的平均值|
|$sum|group后求和|
## MapReduce模式聚集分析
MongoDB也提供map-reduce操作来高效聚集，通常，map-reduce操作有两部分：一个Map阶段处理每一个文档，并未每一个输入的文档产生一个或多个对象，在reduce阶段，对上一步Map产生的输出结果进行合并。map-reduce也可以有一个最终的阶段来对最后的输出结果进行修改，就像其他聚集操作一样，map-reduce能够指定一个查询条件来对输入文档的查询结果进行排序以及部分输出（sort and limit）。
Map-reduce一般采用自定义JavaScript函数来处理map操作与reduce操作以及可选的最后一个最终操作，采用自定义的JavaScript能够比管道聚集更灵活，然而一般情况下map-reduce比管道聚集更加低效与低效也更加复杂。
Map-reduce也是可以操作分片的集合的，Map reduce操作也能够输出到一个分片的集合中（将操作结果写入一个分片的集合中）
下面是一个采用map-reduce进行聚集的示例：
![](https://docs.mongodb.com/manual/_images/map-reduce.png)



## 简单聚集函数分析

MongoDB也能提供如：


```
db.collection.count() 
```
以及


```
db.collection.distinct() 
```
这种操作。

所有的聚集操作都是从单一的collection中进行的。虽然这些操作能够简单的访问公共的常见聚合函数，然而他们缺乏灵活性也没有管道聚集以及Map-reduce聚集功能强大。
示例：
![](https://docs.mongodb.com/manual/_images/distinct.png)




## 参考

[https://docs.mongodb.com/manual/aggregation/](https://docs.mongodb.com/manual/aggregation/)

