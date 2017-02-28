## 分片集群简介
在之前有说过关于MongoDB的复制集，复制集主要用来实现自动故障转移从而达到高可用的目的，然而，随着业务规模的增长和时间的推移，业务数据量会越来越大，当前业务数据可能只有几百GB不到，一台DB服务器足以搞定所有的工作，而一旦业务数据量扩充大几个TB几百个TB时，就会产生一台服务器无法存储的情况，此时，需要将数据按照一定的规则分配到不同的服务器进行存储、查询等，即为分片集群。分片集群要做到的事情就是数据分布式存储。

## 分片部署架构

### 架构设计
先看一张图：

![](https://docs.mongodb.com/manual/_images/sharded-cluster-production-architecture.bakedsvg.svg)

先做一些解释：
1.shard片：一般为一台单独的服务器，即为一个数据存储节点，这个存储节点需要做复制集，实现高可用以及自动故障转移，一般一个分片集群有多个shard片；
2. config服务器：主要是记录shard的配置信息（元信息metadata），如数据存储目录，日志目录，端口号，是否开启了journal等信息，为了保证config服务器的可用性，也做了复制集处理，注意，一旦配置服务器无法使用，则整个集群就不能使用了，**一般是独立的三台服务器实现**冗余备份，这三台可能每一台是独立的复制集架构。
3.Mongos路由进程（Router）：应用程序通过驱动程序直接连接router，router启动时从配置服务器复制集中读取shared信息，然后将数据实际写入或读取（路由）到具体的shard中。

### 部署配置

注意：此处由于我这边只有一台服务器（PC机），所以这里实现的是一个伪集群，部署架构如图所示：
![](https://docs.mongodb.com/manual/_images/sharded-cluster-test-architecture.bakedsvg.svg)

一个Shard（复制集模式），一个config进程，一个router进程，均在同一台服务器上面

沿用上一节的复制集作为shard，

启动：

```
mongod --dbpath=D:\MongoDB\Server\3.2\data\rs0_0 --logpath=D:\MongoDB\Server\3.2\logs\rs0_0.log --port=40000 --replSet=rs0
mongod --dbpath=D:\MongoDB\Server\3.2\data\rs0_1 --logpath=D:\MongoDB\Server\3.2\logs\rs0_1.log --port=40001 --replSet=rs0
mongod --dbpath=D:\MongoDB\Server\3.2\data\rs0_2 --logpath=D:\MongoDB\Server\3.2\logs\rs0_2.log --port=40002 --replSet=rs0
```
配置服务器启动：



```
mongod --dbpath=D:\MongoDB\Server\3.2\data\db_config --logpath=D:\MongoDB\Server\3.2\logs\dbconfig.log --port=40003 --configsvr
```
路由服务器启动



```
mongos --logpath=D:\MongoDB\Server\3.2\logs\dbrouter.log --port=40004 --configdb=linfl-PC:40003
```
添加分片信息到集群：



```
mongo --port 40004
mongos> use admin
switched to db admin
mongos> sh.addShard("rs0/linfl-PC:40000,linfl-PC:40001")
{
        "ok" : 0,
        "errmsg" : "can't add shard 'rs0/linfl-PC:40000,linfl-PC:40001' because a local database 'config' exists in another config",
        "code" : 96
}
```

报错了，原来的rs0由于已经存在config数据库了，去把他删掉：



```
D:\MongoDB\Server\3.2\bin>mongo --port 40000
2017-02-27T16:14:51.454+0800 I CONTROL  [main] Hotfix KB2731284 or later update
is not installed, will zero-out data files
MongoDB shell version: 3.2.9
connecting to: 127.0.0.1:40000/test
rs0:PRIMARY> show dbs
cms     0.000GB
config  0.000GB
local   0.000GB
test    0.000GB
rs0:PRIMARY> use config
switched to db config
rs0:PRIMARY> db.dropDatabase()
{ "dropped" : "config", "ok" : 1 }
```
然后重新连接到4004执行添加分片即可，看下状态：



```
D:\MongoDB\Server\3.2\bin>mongo --port 40004
2017-02-27T16:15:17.286+0800 I CONTROL  [main] Hotfix KB2731284 or later update
is not installed, will zero-out data files
MongoDB shell version: 3.2.9
connecting to: 127.0.0.1:40004/test
mongos> sh.addShard("rs0/linfl-PC:40000,linfl-PC:40001")
{ "shardAdded" : "rs0", "ok" : 1 }
mongos> sh.status()
--- Sharding Status ---
  sharding version: {
        "_id" : 1,
        "minCompatibleVersion" : 5,
        "currentVersion" : 6,
        "clusterId" : ObjectId("58b3d9df84493cb599359c8b")
}
  shards:
        {  "_id" : "rs0",  "host" : "rs0/linfl-PC:40000,linfl-PC:40001" }
  active mongoses:
        "3.2.9" : 1
  balancer:
        Currently enabled:  yes
        Currently running:  no
        Failed balancer rounds in last 5 attempts:  5
        Last reported error:  remote client 192.168.56.1:51845 tried to initiali
ze this host as shard rs0, but shard name was previously initialized as config
        Time of Reported error:  Mon Feb 27 2017 16:15:48 GMT+0800
        Migration Results for the last 24 hours:
                No recent migrations
  databases:
        {  "_id" : "cms",  "primary" : "rs0",  "partitioned" : false }
        {  "_id" : "test",  "primary" : "rs0",  "partitioned" : false }

```
对config数据库中的各个集合做如下解释：



```
mongos> use config
switched to db config
mongos> show collections
actionlog //
changelog //保存被分片的集合的任何元数据的改变信息，如chunks的迁移，分割等
chunks //集群中所有的块信息，块的数据范围以及块所在的片
databases //集群中所有的数据库
lockpings //追踪集群中的激活组件
locks//均衡器产生的锁信息
mongos//所有路由信息
settings//分片集群的配置信息，如chunk大小，均衡器状态等
shards//集群中所有的片信息
tags
version//元信息版本
```

**注意：对集群的操作应该都是通过客户端连接mongos路由来执行。**

正常来讲，一个完整的生产环境至少需要9个Mongod实例，一个Mongos实例，10台机器才能组成，而由于可以对部分实例采用合并在一台机器上进行部署的操作（对资源消耗较低，或对资源消耗的类型不同），能够得到典型的部署结构如下：

![](/assets/典型的mongodb集群部署架构.jpg)

以上部署方案使得每个shard（复制集）中的节点完全分开到不同的服务器上，并将config服务分开，使得其中任何一台服务器宕机，集群都可正常工作，当然我认为，最少只需要三台服务器即可构成稳定的集群（3坏1可正常工作），然而却非典型的架构了。

## 分片工作机制

MongoDB的分片是基于集合（表）来进行的，要对一个集合分片，就要像使其所在的数据库支持分片。

### 集合分片
MongoDB的分片是基于返回的，即任何一个文档一定位于指定片键的某个范围内，一单片键选择好以后，chunks就会按照片键将一部分documents从逻辑上组合在一起
如：对users集合选择city作为片键来分片，如果city字段中存在 'beijing','guangzhou','changsha'，初始时随机的向集群中插入文档，由于初始时chunks比较小，没有达到阈值64MB或10000个文档，不需要分片，随着继续插入，超过chunk阈值后chunk被分割成两个，最终分步可能如下所示：



 开始键值 -- 结束键值 --所在分片
开始键值 结束键值 所在分片

| 开始键值 | 结束键值 |所在分片|
| :---: | :--- |:---|
| -∞| beijing |rs0|
| beijing| changsha|rs1|
| changsha| guangzhou|rs0|
| guangzhou| ∞|rs1|

需要注意的是：chunks所包含的文档并非物理上包含，而是一种逻辑包含，指标是带有片键的文档会落在那个范围内。


使集合支持分片，必须先使数据库支持分片：


```
sh.enableSharding('cms')

```

对于已有的集合如果要分片，那么选定片键后，需要在片键上面首先创建索引，如果集合初始状态为空，则自动创建索引
查看rs.status()可以看到，如下信息片段：


```
 databases:
       {  "_id" : "cms",  "primary" : "rs0",  "partitioned" : true }
       {  "_id" : "test",  "primary" : "rs0",  "partitioned" : false }

```
证明 cms已经支持分片


在已存在的集合中分片
先创建索引：


```
db.user.ensureIndex({city:1})
sh.shardCollection("cms.users",{city:1})
```
注意：由于我本地是单台PC，做分片后看不到效果，但如果有两个分片，分别为rs0,与rs1，往users中插入数据，最后的结果会是


![](/assets/Image.png)

通过


```
 db.changelog.find()
```
可以看到具体的块分割过程，当存储的chunk小于64MB时，不分割，当大于64MB时，开始分割成两部分，-∞~beijing,beijing~∞,随着数据的进一步增大，beijing~∞被继续分割成beijing~guangzhou,guangzhou~∞，这样，rs0就有三个chunk了，随后，-∞到beijing的chunk开始发生转移，从rs0迁移到rs1上面去，依次类推，MongDB就是这样实现了海量数据的分布式存储的。















### 集群平衡器

在上一步的操作中最后讲到了块chunk从rs0迁移到rs1的操作，这个动作是由 平衡器的后台进程自动完成的。

当一个被分片的集合的所有chunk在集群中分布不均衡时（如200个chunk在rs0,50个在rs1上），平衡器就会将chunk从最多的片上迁移到最少的片上，直到chunk数量基本相等为止。平衡器会根据chunk数量的不同，有不同的规则触发块迁移，默认配置为chunk<20，阈值为2，20<chunk<79阈值为4，chunk>80，阈值为8，这也就是为什么我们在刚才要在chunk为3时才能看到迁移过程的变化。
基本的chunk迁移流程如下所示：

![](/assets/Image2.png)

本示例中源片指的是rs0,目标片指的是rs1



### 集群的写、读

### 片键选择策略

## 参考链接

[](https://docs.mongodb.com/manual/core/sharded-cluster-components/)