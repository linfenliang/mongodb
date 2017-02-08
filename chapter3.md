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
	"nscannedObjectsAllPlans" : 10,
	"nscannedAllPlans" : 10,
	"scanAndOrder" : false,
	"indexOnly" : false,
	"nYields" : 0,
	"nChunkSkips" : 0,
	"millis" : 0,
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

对上文含义进行解释看//以后的部分

## 复合索引

## 数组的多键索引

## 索引管理

## 慢查询监控



