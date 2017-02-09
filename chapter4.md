## 插入语句

MongoDB的 插入语句之前也有过介绍了，这里我们只做一个简单的示例：

```
> db.customers.insert({id:11,name:'lisi',orders:[{orders_id:1,create_time:'2017-02-06',products:[{product_name:'MiPad',price:'$100.00'},{product_name:'iphone',price:'$399.00'}]}],mobile:'13161020110',address:{city:'beijing',street:'taiyanggong'}})

> db.customers.find({'orders.0.products.1.product_name':'iphone'})
{ "_id" : ObjectId("58983d0fc55e261327343eab"), "id" : 11, "name" : "lisi", "orders" : [     {     "orders_id" : 1,     "create_time" : "2017-02-06",     "products" : [     {     "product_name" : "MiPad",     "price" : "$100.00" },     {     "product_name" : "iphone",     "price" : "$399.00" } ] } ], "mobile" : "13161020110", "address" : { "city" : "beijing", "street" : "taiyanggong" } }
```

注意：  
1、MongoDB第一次插入数据时，不需要事先创建集合collection，插入时自动创建；
2、每次插入时，不需要显示的指定_id，MongoDB默认创建该字段作为主键（当然，也可以自己指定），_id为ObjectId类型的值，由12个字节组成，分别是时间戳，精确到秒（4字节），机器唯一标识（3字节），进程ID（2字节），随机数（3）字节，如 58983d0fc55e261327343eab解释为58983d0f，为1970年到如今的时间差（秒），c55e26，为机器唯一标示符，1327为进程ID（十六进制），343eab，为随机数；
3、MongoDB中每一个collection中必须有一个_id字段，且必须值唯一。


## 修改语句

MongoDB的修改语句语法如下所示：


```
db.collection.update(query,update,<upsert>,<multi>)
```
update表示需要修改的地方，如果只包含字段，而没有操作符，则会发生取代性修改，该字段一般尽量携带操作符，防止导致因修改一个字段而删除其他字段
upsert可选，默认false，true时表示如果没有匹配到记录，则插入
multi可选，默认false，默认只更新匹配到的第一个文档，true时表示更新所有匹配到的记录

示例：



```
> db.user.insert({id:11,name:'lisi',orders:[{orders_id:1,create_time:'2017-02-06',products:[{product_name:'MiPad',price:'$100.00'},{product_name:'iphone',price:'$399.00'}]}],mobile:'13161020110',address:{city:'beijing',street:'taiyanggong'}})
> db.user.find()
{ "_id" : ObjectId("589c0fc4c55e261327343eb1"), "id" : 11, "name" : "lisi", "orders" : [ 	{ 	"orders_id" : 1, 	"create_time" : "2017-02-06", 	"products" : [ 	{ 	"product_name" : "MiPad", 	"price" : "$100.00" }, 	{ 	"product_name" : "iphone", 	"price" : "$399.00" } ] } ], "mobile" : "13161020110", "address" : { "city" : "beijing", "street" : "taiyanggong" } }
> db.user.update({id:11},{$set:{name:"lisisi",age:18}})
> db.user.find()
{ "_id" : ObjectId("589c0fc4c55e261327343eb1"), "address" : { "city" : "beijing", "street" : "taiyanggong" }, "age" : 18, "id" : 11, "mobile" : "13161020110", "name" : "lisisi", "orders" : [ 	{  "orders_id" : 1, 	"create_time" : "2017-02-06", 	"products" : [ 	{ 	"product_name" : "MiPad", 	"price" : "$100.00" }, 	{ 	"product_name" : "iphone", 	"price" : "$399.00" } ] } ] }

```
更改多个文档，或找不到则插入（不可同时）：



```
> db.user.update({id:12},{$set:{name:"lisisi",age:18}},{multi:true})
> db.user.update({id:12},{$set:{name:"lisisi",age:18}},{upsert:true})
> db.user.find()
{ "_id" : ObjectId("589c0fc4c55e261327343eb1"), "address" : { "city" : "beijing", "street" : "taiyanggong" }, "age" : 18, "id" : 11, "mobile" : "13161020110", "name" : "lisisi", "orders" : [ 	{  "orders_id" : 1, 	"create_time" : "2017-02-06", 	"products" : [ 	{ 	"product_name" : "MiPad", 	"price" : "$100.00" }, 	{ 	"product_name" : "iphone", 	"price" : "$399.00" } ] } ] }
{ "_id" : ObjectId("589c11513b5012ba1ce5500e"), "age" : 18, "id" : 12, "name" : "lisisi" }

```



## 删除语句

MongoDB中的删除语句与RDB中的delete类似，只是名字叫remove，其语法如下：


```
db.collection.remove(<query>,<justOne>)
```
query为可选参数，类似where
justOne是一个Boolean值，表示只删除匹配到的第一个文档（即记录）
**注意：如果不指定任何参数，则将删除所有记录，但是索引数据不会被删除，如果需要删除记录以及索引，则需要单独去drop掉索引**

示例如下：



```
> db.user.remove({id:12},false)
> db.user.find()
{ "_id" : ObjectId("589c0fc4c55e261327343eb1"), "address" : { "city" : "beijing", "street" : "taiyanggong" }, "age" : 18, "id" : 11, "mobile" : "13161020110", "name" : "lisisi", "orders" : [ 	{  "orders_id" : 1, 	"create_time" : "2017-02-06", 	"products" : [ 	{ 	"product_name" : "MiPad", 	"price" : "$100.00" }, 	{ 	"product_name" : "iphone", 	"price" : "$399.00" } ] } ] }
> db.user.remove({},false)
> db.user.find()
> 

```
可以看到删除后的结果，为空，
去看下索引：


```
> db.system.indexes.find()
{ "v" : 1, "name" : "_id_", "key" : { "_id" : 1 }, "ns" : "test.customers" }
{ "v" : 1, "name" : "id_1_age_1", "key" : { "id" : 1, "age" : 1 }, "ns" : "test.customers" }
{ "v" : 1, "name" : "orders.products.product_name_1", "key" : { "orders.products.product_name" : 1 }, "ns" : "test.customers" }
{ "v" : 1, "name" : "orders.orders_id_1", "key" : { "orders.orders_id" : 1 }, "ns" : "test.customers" }
{ "v" : 1, "name" : "_id_", "key" : { "_id" : 1 }, "ns" : "test.user" }

```
索引则还存在。



## 锁机制


MongoDB的读写锁，基本与RDB类似，没有什么特殊的地方，需要注意的是，目前来讲，MongoDB的锁机制还是与RDB（关系型数据库）存在一定的差距的。
