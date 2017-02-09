MongoDB的索引的机制与普通数据库基本相似，主要有如下几部分：

## 单字段索引

MongoDB默认为所有集合创建了一个\_id字段的单字段索引，该索引唯一，且不能删除（\_id为集合的主键）

索引的创建方法：

```
db.customers.ensureIndex({name:1},{unique:false} )
```

查询索引：

```
 db.system.indexes.find()
```

查询结果：

```
{ "v" : 1, "name" : "_id_", "key" : { "_id" : 1 }, "ns" : "test.customers" }
{ "v" : 1, "name" : "name_1", "key" : { "name" : 1 }, "ns" : "test.customers" }
```

对有索引的查询选择器进行解释：

```
> db.customers.find({name:'zhangsan'}).explain()
{
    "cursor" : "BtreeCursor name_1",//表示该查询用到了索引
    "isMultiKey" : false,//未使用多键复合索引
    "n" : 10,//查询选择匹配到的记录数量
    "nscannedObjects" : 10,//执行查询扫描到的文档对象数量
    "nscanned" : 10,//扫描到的文档或索引总数
    "nscannedObjectsAllPlans" : 10,//扫描文旦总数在所有查询计划中
    "nscannedAllPlans" : 10,//在所有查询计划中扫描的文档或索条目的总数量
    "scanAndOrder" : false,//从游标取出查询到的数据时，是否对数据进行排序
    "indexOnly" : false,//
    "nYields" : 0,//产生的读锁数
    "nChunkSkips" : 0,
    "millis" : 0,//查询耗时（ms）
    "indexBounds" : {
        "name" : [
            [
                "zhangsan",
                "zhangsan"
            ]
        ]
    },
    "server" : "raspberrypi:27017"
}
```

对上文含义进行解释看//以后的部分；

**注意：以上部分注释以后也会用到，同时在分析查询时会经常用到，最好记下来。**

## 复合索引

复合索引主要是指对多个字段同时添加索引，故而复合索引支持匹配多个字段的查询。
创建复合索引：


```
db.customers.ensureIndex({id:1,age:1})

```
查询索引结果：


```
> db.system.indexes.find()
{ "v" : 1, "name" : "_id_", "key" : { "_id" : 1 }, "ns" : "test.customers" }
{ "v" : 1, "name" : "name_1", "key" : { "name" : 1 }, "ns" : "test.customers" }
{ "v" : 1, "name" : "id_1_age_1", "key" : { "id" : 1, "age" : 1 }, "ns" : "test.customers" }

```
使用索引示例：


```
> db.customers.find({id:{$lt:5},age:{$gt:12}})
{ "_id" : ObjectId("589835c41c85cb68725f789b"), "id" : 3, "name" : "zhangsan", "age" : 13 }
{ "_id" : ObjectId("589835c41c85cb68725f789c"), "id" : 4, "name" : "zhangsan", "age" : 14 }

```
解释执行示例：


```
> db.customers.find({id:{$lt:5},age:{$gt:12}}).explain()
{
	"cursor" : "BtreeCursor id_1_age_1",
	"isMultiKey" : false,
	"n" : 2,
	"nscannedObjects" : 2,
	"nscanned" : 4,
	"nscannedObjectsAllPlans" : 5,
	"nscannedAllPlans" : 9,
	"scanAndOrder" : false,
	"indexOnly" : false,
	"nYields" : 0,
	"nChunkSkips" : 0,
	"millis" : 0,
	"indexBounds" : {
		"id" : [
			[
				-1.7976931348623157e+308,
				5
			]
		],
		"age" : [
			[
				12,
				1.7976931348623157e+308
			]
		]
	},
	"server" : "raspberrypi:27017"
}

```







## 数组的多键索引

对一个值为数组类型的字段创建索引，则会默认对数组中的每一个元素都创建索引


```
> db.customers.ensureIndex({'orders.orders_id':1})

```
查看：


```
> db.system.indexes.find()
{ "v" : 1, "name" : "_id_", "key" : { "_id" : 1 }, "ns" : "test.customers" }
{ "v" : 1, "name" : "name_1", "key" : { "name" : 1 }, "ns" : "test.customers" }
{ "v" : 1, "name" : "id_1_age_1", "key" : { "id" : 1, "age" : 1 }, "ns" : "test.customers" }
{ "v" : 1, "name" : "orders.products.product_name_1", "key" : { "orders.products.product_name" : 1 }, "ns" : "test.customers" }
{ "v" : 1, "name" : "orders.orders_id_1", "key" : { "orders.orders_id" : 1 }, "ns" : "test.customers" }

```
解释执行：


```
> db.customers.find({'orders.orders_id':{$gte:0}}).explain()
{
	"cursor" : "BtreeCursor orders.orders_id_1",
	"isMultiKey" : false,
	"n" : 1,
	"nscannedObjects" : 1,
	"nscanned" : 1,
	"nscannedObjectsAllPlans" : 1,
	"nscannedAllPlans" : 1,
	"scanAndOrder" : false,
	"indexOnly" : false,
	"nYields" : 0,
	"nChunkSkips" : 0,
	"millis" : 0,
	"indexBounds" : {
		"orders.orders_id" : [
			[
				0,
				1.7976931348623157e+308
			]
		]
	},
	"server" : "raspberrypi:27017"
}

```


另外一个：



```
> db.customers.find({'orders.products.product_name':'iphone'})
{ "_id" : ObjectId("58983d0fc55e261327343eab"), "id" : 11, "name" : "lisi", "orders" : [ 	{ 	"orders_id" : 1, 	"create_time" : "2017-02-06", 	"products" : [ 	{ 	"product_name" : "MiPad", 	"price" : "$100.00" }, 	{ 	"product_name" : "iphone", 	"price" : "$399.00" } ] } ], "mobile" : "13161020110", "address" : { "city" : "beijing", "street" : "taiyanggong" } }
> db.customers.find({'orders.products.product_name':'iphone'}).explain()
{
	"cursor" : "BtreeCursor orders.products.product_name_1",
	"isMultiKey" : true,
	"n" : 1,
	"nscannedObjects" : 1,
	"nscanned" : 1,
	"nscannedObjectsAllPlans" : 1,
	"nscannedAllPlans" : 1,
	"scanAndOrder" : false,
	"indexOnly" : false,
	"nYields" : 0,
	"nChunkSkips" : 0,
	"millis" : 0,
	"indexBounds" : {
		"orders.products.product_name" : [
			[
				"iphone",
				"iphone"
			]
		]
	},
	"server" : "raspberrypi:27017"
}

```






## 索引管理

索引的创建我们已经在上文中有过讲述，下面总结下索引的创建格式：


```
db.collection.ensureIndex(keys,options)
```
其中key是一个document文档，包含需要添加索引的字段以及索引的排序方向；option可选，控制索引的创建方式；
所有的索引都保存在集合**system.indexes**中;
索引的删除方式为：


```
db.collection.dropIndex(indexName)

```
如：删除customers的name_1索引：


```
> db.customers.dropIndex('name_1')
{ "nIndexesWas" : 5, "ok" : 1 }
> db.system.indexes.find()
{ "v" : 1, "name" : "_id_", "key" : { "_id" : 1 }, "ns" : "test.customers" }
{ "v" : 1, "name" : "id_1_age_1", "key" : { "id" : 1, "age" : 1 }, "ns" : "test.customers" }
{ "v" : 1, "name" : "orders.products.product_name_1", "key" : { "orders.products.product_name" : 1 }, "ns" : "test.customers" }
{ "v" : 1, "name" : "orders.orders_id_1", "key" : { "orders.orders_id" : 1 }, "ns" : "test.customers" }

```





## 慢查询监控

1、MongoDB会自动的将查询语句执行时间超过100ms的输出到日志中，其中100ms可以通过mongod的启动选项 slowms设置，默认100ms
2、还可以通过打开数据库的监视功能，默认是关闭的，通过如下命令打开


```
db.setProfilingLevel(level,[slowms])

```
level：监视级别，值为0为关闭，1：只记录慢日志，2：记录所有的操作
监视的结果都保存在system.profile中。

示例如下：


```
> db.setProfilingLevel(2)
{ "was" : 0, "slowms" : 100, "ok" : 1 }
> db.customers.find()
{ "_id" : ObjectId("589835c41c85cb68725f7899"), "id" : 1, "name" : "zhangsan", "age" : 11 }
{ "_id" : ObjectId("589835c41c85cb68725f789a"), "id" : 2, "name" : "zhangsan", "age" : 12 }
{ "_id" : ObjectId("589835c41c85cb68725f789b"), "id" : 3, "name" : "zhangsan", "age" : 13 }
{ "_id" : ObjectId("589835c41c85cb68725f789c"), "id" : 4, "name" : "zhangsan", "age" : 14 }
{ "_id" : ObjectId("589835c41c85cb68725f789d"), "id" : 5, "name" : "zhangsan", "age" : 15 }
{ "_id" : ObjectId("589835c41c85cb68725f789e"), "id" : 6, "name" : "zhangsan", "age" : 16 }
{ "_id" : ObjectId("589835c41c85cb68725f789f"), "id" : 7, "name" : "zhangsan", "age" : 17 }
{ "_id" : ObjectId("589835c41c85cb68725f78a0"), "id" : 8, "name" : "zhangsan", "age" : 18 }
{ "_id" : ObjectId("589835c41c85cb68725f78a1"), "id" : 9, "name" : "zhangsan", "age" : 19 }
{ "_id" : ObjectId("589835c41c85cb68725f78a2"), "id" : 10, "name" : "zhangsan", "age" : 20 }
{ "_id" : ObjectId("58983d0fc55e261327343eab"), "id" : 11, "name" : "lisi", "orders" : [ 	{ 	"orders_id" : 1, 	"create_time" : "2017-02-06", 	"products" : [ 	{ 	"product_name" : "MiPad", 	"price" : "$100.00" }, { 	"product_name" : "iphone", 	"price" : "$399.00" } ] } ], "mobile" : "13161020110", "address" : { "city" : "beijing", "street" : "taiyanggong" } }
> db.system.profile.find()
{ "op" : "query", "ns" : "test.system.indexes", "query" : { "expireAfterSeconds" : { "$exists" : true } }, "ntoreturn" : 0, "ntoskip" : 0, "nscanned" : 4, "keyUpdates" : 0, "numYield" : 0, "lockStats" : { "timeLockedMicros" : { "r" : NumberLong(294), "w" : NumberLong(0) }, "timeAcquiringMicros" : { "r" : NumberLong(13), "w" : NumberLong(15) } }, "nreturned" : 0, "responseLength" : 20, "millis" : 0, "ts" : ISODate("2017-02-09T03:45:41.845Z"), "client" : "0.0.0.0", "allUsers" : [ ], "user" : "" }
{ "op" : "query", "ns" : "test.customers", "query" : {  }, "ntoreturn" : 0, "ntoskip" : 0, "nscanned" : 11, "keyUpdates" : 0, "numYield" : 0, "lockStats" : { "timeLockedMicros" : { "r" : NumberLong(252), "w" : NumberLong(0) }, "timeAcquiringMicros" : { "r" : NumberLong(19), "w" : NumberLong(16) } }, "nreturned" : 11, "responseLength" : 995, "millis" : 0, "ts" : ISODate("2017-02-09T03:45:49.972Z"), "client" : "127.0.0.1", "allUsers" : [ ], "user" : "" }
> 

```

