# MongoDB自己的查询语言

MongoDB不支持SQL，本节主要讲在MongoDB中如何实现类似SQL的查询功能

SQl中的写法如

```
select column from table join table2 where condition 
```

在MongoDB中有一套类似的写法，先做名称解释

| SQL | MongoDB |
| :---: | :--- |
| select | 查询选择器实现 |
| where | 查询投射项 |
| join | 无，依靠MongoDB中字段数据类型可以嵌套实现关联查询，字段类型为数组时，MongoDB做了特殊处理 |

为了更好的说明，现在MongoDB中插入了一条记录：

```
db.customers.insert({id:11,name:'lisi',orders:[{orders_id:1,create_time:'2017-02-06',products:[{product_name:'MiPad',pric10',address:{city:'beijing',street:'taiyanggong'}})
```

查询语句：

```
db.customers.find()
```

类似于SQL中的：

```
select * from customers
```

添加查询条件：

第一个订单中的第一个产品名称必须为iphone，同时返回的结果不显示MongoDB中自带的\_id选项，只返回id与orders列

```
db.customers.find({'orders.0.products.1.product_name':'iphone'},{_id:0,id:1,orders:1})
```

查询结果为：

```
{ "id" : 11, "orders" : [     {     "orders_id" : 1,     "create_time" : "2017-02-06",     "products" : [     {     "e" : "iphone",     "price" : "$399.00" } ] } ] }
```

其结构如下：

```
db.collection.find( <query filter>, <projection> )
```

参数都可以为空，不过当返回字段不为空时，查询条件必须存在或{}；

另外，查询条件中如&lt;，&lt;=，in，not in，or，and，exists等如下所示：

```
db.customers.find({$or:[{age:{$exists:false}},{id:{$lte:10}}]},{_id:0,id:1,name:1,age:1}).sort({age:1})
```

转换成SQL为：

```
select id,name,age from customers where exists(age) or id <=10 order by age asc
```

```
db.customers.find({age:{$in:[15,18]}})
```

注意：由于MongoDB的松散的数据组织方式，所以可以存在在某一些记录中其中某一列不存在而在另一些记录中存在的情况，如最开始创建customers的时候，没有age列，随着业务的发展，需要为其添加age列，则可以直接在后面的记录添加时，加入age，查询时，前面的记录将查询不出age列，后面的记录则可以查询到。

以下为常用的表达式：

| 表达式 | 语义 |
| :--- | :--- |
| $gt | great than大于 &gt; |
| $lt | less than小于 &lt; |
| $gte | great than equals大于等于 &gt;= |
| $lte | 小于等于 &lt;= |
| $ne | not equals不等于&lt;&gt; |
| $in | 在某一个集合里 |
| $nin | 不在某一个集合中 |
| $exists | 验证一个元素是否存在 |
| 正则表达式 | 支持 |
| $elemMatch | 匹配数组内的元素 |
| user.score | 查询嵌套对象的值（左为示例user列中score对象的值） |
| $not | 取反 |
| $all | 匹配所有 |
| $size | 匹配数组大小 |
| $slice | 返回数组的一个子集，即对以某属性为基础，返回多少条（范围）。也可以接受偏移值和要返回的元素数量，来返回中间的结果 |
| $where | 可执行任务JavaScript作为查询的一部分，可为function也可为字符串，查询较慢 |



