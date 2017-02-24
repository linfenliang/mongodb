## 复制集功能概述

复制集（replica set）是MongoDB用来保持相同的数据集合的一个MongoD进程组，复制集提供了所有生产部署的基础：数据冗余以及高可用。MongoDB的高可用靠的是自动故障转移来实现的，本节就是介绍MongoDB的该部分实现的。

## 复制集工作原理

虽然Journaling日志功能提供了数据恢复的功能，但是他通常针对的是单个节点来说的，而复制集则针对的是一组进程，通常是多个节点组成，在每个节点上有Journaling日志保证数据完整性，在整个复制集中实现自动故障转移，从而保证了数据库的高可用性。  
在生产环境中，一个复制集应该最少包含三个节点，一个仲裁节点（arbiter），唯一一个数据主节点（primary），一个或多个数据次节点（secondary）。  
主节点用来接收所有的写操作，一个复制集有且仅有一个primary能够进行写关注（写关注将在后面介绍），主节点在他的操作日志oplog中将所有的修改记录到数据集data sets中。典型的结构如下所示：

![](https://docs.mongodb.com/manual/_images/replica-set-read-write-operations-primary.bakedsvg.svg)

secondary节点备份primary节点上的数据，secondary节点可以有多个，一旦primary节点不可用，abiter将从secondary节点中选取一个作为primary节点，secondary节点的作用如下：

![](https://docs.mongodb.com/manual/_images/replica-set-primary-with-two-secondaries.bakedsvg.svg)

现在除了primary，secondary节点外，可以新增一个mongod实例副本集作为arbiter，arbiter不能维护数据集。arbiter的主要作用是维持与复制集中所有的其他节点的心跳以保证选举需要的节点数，因为arbiter不是一个数据存储集，arbiter可以提供一个比全功能副本集更廉价的方法来获取法定人数。如果复制集中是偶数个节点，可以通过添加arbiter节点使得primary可以获取到大多数的投票。arbiter不需要专门的硬件支持。arbiter的作用如下：

![](https://docs.mongodb.com/manual/_images/replica-set-primary-with-secondary-and-arbiter.bakedsvg.svg)

相对于primary与secondary节点可能在一次选举中（主节点失效触发）互换角色，arbiter仲裁者永远都是arbiter。

故障转移流程如下所示：

![https://docs.mongodb.com/manual/replication/#edge-cases-2-primaries](https://docs.mongodb.com/manual/_images/replica-set-trigger-election.bakedsvg.svg)

复制集的创建：

在D:\MongoDB\Server\3.2\bin文件夹中运行CMD命令，启动mongod三个进程，  
分别是：

```
mongod --dbpath=D:\MongoDB\Server\3.2\data\rs0_0 --logpath=D:\MongoDB\Server\3.2\logs\rs0_0.log --port=40000 --replSet=rs0
mongod --dbpath=D:\MongoDB\Server\3.2\data\rs0_1 --logpath=D:\MongoDB\Server\3.2\logs\rs0_1.log --port=40001 --replSet=rs0
mongod --dbpath=D:\MongoDB\Server\3.2\data\rs0_2 --logpath=D:\MongoDB\Server\3.2\logs\rs0_2.log --port=40002 --replSet=rs0
```

注意：需要创建data文件夹下的子文件夹为rs0\_0,rs0\_1,rs0\_2

然后执行启动一个客户端，mongo，执行：

```
mongo --port 40000
```

执行初始化复制集命令：

```
> rs.initiate()
```

显示结果如下：

```
{
        "info2" : "no configuration specified. Using a default configuration for
 the set",
        "me" : "linfl-PC:40000",
        "ok" : 1
}
```

查看下：

```
rs0:OTHER> rs.conf()
```

```
{
        "_id" : "rs0",
        "version" : 1,
        "protocolVersion" : NumberLong(1),
        "members" : [
                {
                        "_id" : 0,
                        "host" : "linfl-PC:40000",
                        "arbiterOnly" : false,
                        "buildIndexes" : true,
                        "hidden" : false,
                        "priority" : 1,
                        "tags" : {

                        },
                        "slaveDelay" : NumberLong(0),
                        "votes" : 1
                }
        ],
        "settings" : {
                "chainingAllowed" : true,
                "heartbeatIntervalMillis" : 2000,
                "heartbeatTimeoutSecs" : 10,
                "electionTimeoutMillis" : 10000,
                "getLastErrorModes" : {

                },
                "getLastErrorDefaults" : {
                        "w" : 1,
                        "wtimeout" : 0
                },
                "replicaSetId" : ObjectId("58a2e2f5c2e580f7b1c85b18")
        }
}
```

将两个节点加入进来：

```
rs0:PRIMARY> rs.add("linfl-PC:40001")
{ "ok" : 1 }
rs0:PRIMARY> rs.addArb("linfl-PC:40002")
{ "ok" : 1 }
```

注意：此时命令行的前缀已经变了：rs0:PRIMARY

观察复制集的状态信息：

```
rs0:PRIMARY> rs.status()
```

发现有如下输出：

```
{
        "set" : "rs0",//复制集名称
        "date" : ISODate("2017-02-14T11:00:36.634Z"),
        "myState" : 1,//1：primary；2：secondary；
        "term" : NumberLong(1),
        "heartbeatIntervalMillis" : NumberLong(2000),
        "members" : [
                {
                        "_id" : 0,
                        "name" : "linfl-PC:40000",
                        "health" : 1,//1：运行；0：失败
                        "state" : 1,
                        "stateStr" : "PRIMARY",
                        "uptime" : 216,//成员在线时长（秒）
                        "optime" : {
                                "ts" : Timestamp(1487070006, 1),
                                "t" : NumberLong(1)
                        },
                        "optimeDate" : ISODate("2017-02-14T11:00:06Z"),
                        "infoMessage" : "could not find member to sync from",
                        "electionTime" : Timestamp(1487069941, 2),
                        "electionDate" : ISODate("2017-02-14T10:59:01Z"),
                        "configVersion" : 3,
                        "self" : true
                },
                {
                        "_id" : 1,
                        "name" : "linfl-PC:40001",
                        "health" : 1,
                        "state" : 2,
                        "stateStr" : "SECONDARY",
                        "uptime" : 40,
                        "optime" : {
                                "ts" : Timestamp(1487070006, 1),
                                "t" : NumberLong(1)
                        },
                        "optimeDate" : ISODate("2017-02-14T11:00:06Z"),
                        "lastHeartbeat" : ISODate("2017-02-14T11:00:36.075Z"),
                        "lastHeartbeatRecv" : ISODate("2017-02-14T11:00:35.082Z"
),
                        "pingMs" : NumberLong(0),//从远端成员到本实例间个路由包的来回时间
                        "syncingTo" : "linfl-PC:40000",//数据同步实例来源
                        "configVersion" : 3
                },
                               {
                        "_id" : 2,
                        "name" : "linfl-PC:40002",
                        "health" : 1,
                        "state" : 7,
                        "stateStr" : "ARBITER",
                        "uptime" : 5,
                        "lastHeartbeat" : ISODate("2017-02-15T02:01:11.170Z"),
                        "lastHeartbeatRecv" : ISODate("2017-02-15T02:01:10.172Z"
),
                        "pingMs" : NumberLong(0),
                        "syncingTo" : "linfl-PC:40001",
                        "configVersion" : 5
                }
        ],
        "ok" : 1
}
```

由于arbiter实例不同步数据，只是在主节点发生故障时在复制集剩下的secondary节点中选取一个新的primary，只是做仲裁，故而运行arbiter实例的机器不需要太多存储空间

### 数据同步

现在通过命令展示数据同步过程，并对数据同步过程做讲解。  
首先我们先查看下该复制集中的所有数据库：

```
rs0:PRIMARY> show dbs
local  0.000GB
```

只有一个local库，查看下local库中的集合：

```
rs0:PRIMARY> use local
switched to db local
rs0:PRIMARY> show collections
me
oplog.rs
replset.election
startup_log
system.replset
```

注意：MongoDB就是通过oplog.rs来实现复制集间数据同步的。  
我们通过往cms数据库中插入一条记录查看oplog.rs的变化：

```
rs0:PRIMARY> use cms
switched to db cms
rs0:PRIMARY> db.customers.insert({id:11,name:'lisi',orders:[{orders_id:1,create_time:'2017-02-06',products:[{product_name:'MiPad',price:'$100.00'},{product_name:'iphone',price:'$399.00'}]}],mobile:'13161020110',address:{city:'beijing',street:'taiyanggong'}})
WriteResult({ "nInserted" : 1 })
rs0:PRIMARY> db.customers.find()
{ "_id" : ObjectId("58a3bb2ca0bd576baa4763de"), "id" : 11, "name" : "lisi", "orders" : [ { "orders_id" : 1, "create_time" : "2017-02-06", "products" : [ { "product_name" : "MiPad", "price" : "$100.00" }, { "product_name" : "iphone", "price" : "$399.00" } ] } ], "mobile" : "13161020110", "address" : { "city" :"beijing", "street" : "taiyanggong" } }
rs0:PRIMARY> use local
switched to db local
rs0:PRIMARY> db.oplog.rs.find()
{ "ts" : Timestamp(1487069941, 1), "h" : NumberLong("-6355743292210446009"), "v" : 2, "op" : "n", "ns" : "", "o" : { "msg" : "initiating set" } }
{ "ts" : Timestamp(1487069942, 1), "t" : NumberLong(1), "h" : NumberLong("-1263029456710822127"), "v" : 2, "op" : "n", "ns" : "", "o" : { "msg" : "new primary"} }
{ "ts" : Timestamp(1487069995, 1), "t" : NumberLong(1), "h" : NumberLong("6502719191955655967"), "v" : 2, "op" : "n", "ns" : "", "o" : { "msg" : "Reconfig set","version" : 2 } }
{ "ts" : Timestamp(1487070006, 1), "t" : NumberLong(1), "h" : NumberLong("-2415405716599170931"), "v" : 2, "op" : "n", "ns" : "", "o" : { "msg" : "Reconfig set", "version" : 3 } }
{ "ts" : Timestamp(1487124022, 1), "t" : NumberLong(1), "h" : NumberLong("-478589502849657245"), "v" : 2, "op" : "n", "ns" : "", "o" : { "msg" : "Reconfig set", "version" : 4 } }
{ "ts" : Timestamp(1487124063, 1), "t" : NumberLong(1), "h" : NumberLong("653734030548343548"), "v" : 2, "op" : "n", "ns" : "", "o" : { "msg" : "Reconfig set","version" : 5 } }
{ "ts" : Timestamp(1487125292, 1), "t" : NumberLong(1), "h" : NumberLong("4089071333042150540"), "v" : 2, "op" : "c", "ns" : "cms.$cmd", "o" : { "create" :"customers" } }
{ "ts" : Timestamp(1487125292, 2), "t" : NumberLong(1), "h" : NumberLong("-682469243777763072"), "v" : 2, "op" : "i", "ns" : "cms.customers", "o" : { "_id" : ObjectId("58a3bb2ca0bd576baa4763de"), "id" : 11, "name" : "lisi", "orders" : [ { "orders_id" : 1, "create_time" : "2017-02-06", "products" : [ { "product_name" :"MiPad", "price" : "$100.00" }, { "product_name" : "iphone", "price" : "$399.00" } ] } ], "mobile" : "13161020110", "address" : { "city" : "beijing", "street" : "taiyanggong" } } }
rs0:PRIMARY>
```

发现oplog.rs已经有了一条我们刚刚创建的记录

```
{ "ts" : Timestamp(1487125292, 2), "t" : NumberLong(1), "h" : NumberLong("-682469243777763072"), "v" : 2, "op" : "i", "ns" : "cms.customers", "o" : { "_id" : ObjectId("58a3bb2ca0bd576baa4763de"), "id" : 11, "name" : "lisi", "orders" : [ { "orders_id" : 1, "create_time" : "2017-02-06", "products" : [ { "product_name" :"MiPad", "price" : "$100.00" }, { "product_name" : "iphone", "price" : "$399.00" } ] } ], "mobile" : "13161020110", "address" : { "city" : "beijing", "street" : "taiyanggong" } } }
```

其中op参数表示操作码：i表示insert操作；ns表示操作发生的命名空间，o为操作包含的对象。

当primary节点完成插入操作后，secondary节点为了保证数据的同步，也会完成一些动作，所有的secondary节点将检查自己的local数据库上oplog.rs是否有修改，找出最近一条记录的时间戳，然后secondary节点将此时间戳作为条件查询primary节点上的oplog.rs集合，并找出所有大雨此时间戳的记录，最后secondary节点将这些找到的记录差润到自己的oplog.rs集合，同时执行这些记录代表的操作，然后完成数据同步。

查看此时的secondary节点数据库信息：

```
D:\MongoDB\Server\3.2\bin>mongo --port 40001
2017-02-15T10:39:36.688+0800 I CONTROL  [main] Hotfix KB2731284 or later update is not installed, will zero
MongoDB shell version: 3.2.9
connecting to: 127.0.0.1:40001/test
rs0:SECONDARY> show dbs
2017-02-15T10:40:07.204+0800 E QUERY    [thread1] Error: listDatabases failed:{ "ok" : 0, "errmsg" : "not master and slaveOk=false", "code" : 13435 } :
_getErrorWithCode@src/mongo/shell/utils.js:25:13
Mongo.prototype.getDBs@src/mongo/shell/mongo.js:62:1
shellHelper.show@src/mongo/shell/utils.js:761:19
shellHelper@src/mongo/shell/utils.js:651:15
@(shellhelp2):1:1

rs0:SECONDARY> rs.slaveOk()//注意：正常情况下secondary不允许读写，这里做更改
rs0:SECONDARY> show dbs
cms    0.000GB
local  0.000GB
```

![](/assets/mongdb_repl.png)

还要注意：oplog.rs的大小是固定的。32位系统默认大小50MB，64位系统默认为空闲磁盘空间大小的5%，可以通过--oplogSize在启动时设置。

### 故障转移

MongoDB自动故障转移是依靠心跳包实现的就是在前文提到的（lastHeartbeat）字段。Mongod每隔两秒向其他成员发送一个心跳包并且通过rs.status\(\)返回的成员的"headth"来判断成员状态，如果出现复制集中primary节点不可用，则复制集中所有的secondary节点会触发一次选举操作，选举出新的primary节点，arbiter只是负责选举其他成员成primary节点，自己不会参与到选举中。如果secondary节点有多个则会选择拥有最新时间戳的oplog记录或较高权限的节点称为primary。而如果secondary节点失败，则不会发生重新选举primary过程。  
现在模拟两种情况下查看数据的处理过程，分别为secondary节点down，以及primary节点down

secondary节点down,查看故障转移情况：



```
rs0:PRIMARY> rs.status()
{
        "set" : "rs0",
        "date" : ISODate("2017-02-24T07:39:17.573Z"),
        "myState" : 1,
        "term" : NumberLong(2),
        "heartbeatIntervalMillis" : NumberLong(2000),
        "members" : [
                {
                        "_id" : 0,
                        "name" : "linfl-PC:40000",
                        "health" : 1,
                        "state" : 1,
                        "stateStr" : "PRIMARY",
                        "uptime" : 295,
                        "optime" : {
                                "ts" : Timestamp(1487921807, 1),
                                "t" : NumberLong(2)
                        },
                        "optimeDate" : ISODate("2017-02-24T07:36:47Z"),
                        "electionTime" : Timestamp(1487921806, 1),
                        "electionDate" : ISODate("2017-02-24T07:36:46Z"),
                        "configVersion" : 5,
                        "self" : true
                },
                {
                        "_id" : 1,
                        "name" : "linfl-PC:40001",
                        "health" : 0,
                        "state" : 8,
                        "stateStr" : "(not reachable/healthy)",
                        "uptime" : 0,
                        "optime" : {
                                "ts" : Timestamp(0, 0),
                                "t" : NumberLong(-1)
                        },
                        "optimeDate" : ISODate("1970-01-01T00:00:00Z"),
                        "lastHeartbeat" : ISODate("2017-02-24T07:39:15.017Z"),
                        "lastHeartbeatRecv" : ISODate("2017-02-24T07:38:33.501Z"
),
                        "pingMs" : NumberLong(0),
                        "lastHeartbeatMessage" : "Couldn't get a connection with
in the time limit",
                        "configVersion" : -1
                },
                {
                        "_id" : 2,
                        "name" : "linfl-PC:40002",
                        "health" : 1,
                        "state" : 7,
                        "stateStr" : "ARBITER",
                        "uptime" : 156,
                        "lastHeartbeat" : ISODate("2017-02-24T07:39:16.961Z"),
                        "lastHeartbeatRecv" : ISODate("2017-02-24T07:39:15.647Z"
),
                        "pingMs" : NumberLong(0),
                        "configVersion" : 5
                }
        ],
        "ok" : 1
}
```
可以看到 secondary节点state已经变为8（成员宕机状态了），同时，lastHeartbeatMessage显示：Couldn't get a connection within the time limit。

往primary节点插入一条记录并查看状态信息：


```
rs0:PRIMARY> use cms
switched to db cms
rs0:PRIMARY> db.customers.insert({id:12,name:'zhangsan'})
WriteResult({ "nInserted" : 1 })
rs0:PRIMARY> rs.status()
{
        "set" : "rs0",
        "date" : ISODate("2017-02-24T07:46:58.458Z"),
        "myState" : 1,
        "term" : NumberLong(2),
        "heartbeatIntervalMillis" : NumberLong(2000),
        "members" : [
                {
                        "_id" : 0,
                        "name" : "linfl-PC:40000",
                        "health" : 1,
                        "state" : 1,
                        "stateStr" : "PRIMARY",
                        "uptime" : 756,
                        "optime" : {
                                "ts" : Timestamp(1487922414, 1),
                                "t" : NumberLong(2)
                        },
                        "optimeDate" : ISODate("2017-02-24T07:46:54Z"),
                        "electionTime" : Timestamp(1487921806, 1),
                        "electionDate" : ISODate("2017-02-24T07:36:46Z"),
                        "configVersion" : 5,
                        "self" : true
                },
                {
                        "_id" : 1,
                        "name" : "linfl-PC:40001",
                        "health" : 0,
                        "state" : 8,
                        "stateStr" : "(not reachable/healthy)",
                        "uptime" : 0,
                        "optime" : {
                                "ts" : Timestamp(0, 0),
                                "t" : NumberLong(-1)
                        },
                        "optimeDate" : ISODate("1970-01-01T00:00:00Z"),
                        "lastHeartbeat" : ISODate("2017-02-24T07:46:57.445Z"),
                        "lastHeartbeatRecv" : ISODate("2017-02-24T07:38:33.501Z"
),
                        "pingMs" : NumberLong(0),
                        "lastHeartbeatMessage" : "����Ŀ�����������ܾ����޷����ӡ�",

                        "configVersion" : -1
                },
                {
                        "_id" : 2,
                        "name" : "linfl-PC:40002",
                        "health" : 1,
                        "state" : 7,
                        "stateStr" : "ARBITER",
                        "uptime" : 617,
                        "lastHeartbeat" : ISODate("2017-02-24T07:46:57.005Z"),
                        "lastHeartbeatRecv" : ISODate("2017-02-24T07:46:55.655Z"
),
                        "pingMs" : NumberLong(0),
                        "configVersion" : 5
                }
        ],
        "ok" : 1
}
rs0:PRIMARY>








```

检查optime信息发现已经发生了变化，重新启动secondary节点：
并再次查看：



```
rs0:PRIMARY> rs.status()
{
        "set" : "rs0",
        "date" : ISODate("2017-02-24T07:49:51.633Z"),
        "myState" : 1,
        "term" : NumberLong(2),
        "heartbeatIntervalMillis" : NumberLong(2000),
        "members" : [
                {
                        "_id" : 0,
                        "name" : "linfl-PC:40000",
                        "health" : 1,
                        "state" : 1,
                        "stateStr" : "PRIMARY",
                        "uptime" : 929,
                        "optime" : {
                                "ts" : Timestamp(1487922414, 1),
                                "t" : NumberLong(2)
                        },
                        "optimeDate" : ISODate("2017-02-24T07:46:54Z"),
                        "electionTime" : Timestamp(1487921806, 1),
                        "electionDate" : ISODate("2017-02-24T07:36:46Z"),
                        "configVersion" : 5,
                        "self" : true
                },
                {
                        "_id" : 1,
                        "name" : "linfl-PC:40001",
                        "health" : 1,
                        "state" : 2,
                        "stateStr" : "SECONDARY",
                        "uptime" : 6,
                        "optime" : {
                                "ts" : Timestamp(1487921807, 1),
                                "t" : NumberLong(2)
                        },
                        "optimeDate" : ISODate("2017-02-24T07:36:47Z"),
                        "lastHeartbeat" : ISODate("2017-02-24T07:49:51.570Z"),
                        "lastHeartbeatRecv" : ISODate("2017-02-24T07:49:47.386Z"
),
                        "pingMs" : NumberLong(0),
                        "configVersion" : 5
                },
                {
                        "_id" : 2,
                        "name" : "linfl-PC:40002",
                        "health" : 1,
                        "state" : 7,
                        "stateStr" : "ARBITER",
                        "uptime" : 790,
                        "lastHeartbeat" : ISODate("2017-02-24T07:49:51.016Z"),
                        "lastHeartbeatRecv" : ISODate("2017-02-24T07:49:50.656Z"
),
                        "pingMs" : NumberLong(0),
                        "configVersion" : 5
                }
        ],
        "ok" : 1
}
```

可以看到primary与secondary节点 optime 中t已经相同了（注意：由于误操作，刚才插入了两条记录）。

现在我们来实验primary节点失效，将primary节点关掉，查看复制集状态，


```
D:\MongoDB\Server\3.2\bin>mongo --port 40001
2017-02-24T15:57:09.654+0800 I CONTROL  [main] Hotfix KB2731284 or later update
is not installed, will zero-out data files
MongoDB shell version: 3.2.9
connecting to: 127.0.0.1:40001/test
rs0:PRIMARY> rs.status()
{
        "set" : "rs0",
        "date" : ISODate("2017-02-24T07:57:12.710Z"),
        "myState" : 1,
        "term" : NumberLong(3),
        "heartbeatIntervalMillis" : NumberLong(2000),
        "members" : [
                {
                        "_id" : 0,
                        "name" : "linfl-PC:40000",
                        "health" : 0,
                        "state" : 8,
                        "stateStr" : "(not reachable/healthy)",
                        "uptime" : 0,
                        "optime" : {
                                "ts" : Timestamp(0, 0),
                                "t" : NumberLong(-1)
                        },
                        "optimeDate" : ISODate("1970-01-01T00:00:00Z"),
                        "lastHeartbeat" : ISODate("2017-02-24T07:57:12.562Z"),
                        "lastHeartbeatRecv" : ISODate("2017-02-24T07:56:51.767Z"
),
                        "pingMs" : NumberLong(0),
                        "lastHeartbeatMessage" : "no response within election ti
meout period",
                        "configVersion" : -1
                },
                {
                        "_id" : 1,
                        "name" : "linfl-PC:40001",
                        "health" : 1,
                        "state" : 1,
                        "stateStr" : "PRIMARY",
                        "uptime" : 448,
                        "optime" : {
                                "ts" : Timestamp(1487923023, 1),
                                "t" : NumberLong(3)
                        },
                        "optimeDate" : ISODate("2017-02-24T07:57:03Z"),
                        "infoMessage" : "could not find member to sync from",
                        "electionTime" : Timestamp(1487923022, 1),
                        "electionDate" : ISODate("2017-02-24T07:57:02Z"),
                        "configVersion" : 5,
                        "self" : true
                },
                {
                        "_id" : 2,
                        "name" : "linfl-PC:40002",
                        "health" : 1,
                        "state" : 7,
                        "stateStr" : "ARBITER",
                        "uptime" : 445,
                        "lastHeartbeat" : ISODate("2017-02-24T07:57:12.565Z"),
                        "lastHeartbeatRecv" : ISODate("2017-02-24T07:57:10.743Z"
),
                        "pingMs" : NumberLong(0),
                        "configVersion" : 5
                }
        ],
        "ok" : 1
}
```
可以看到在arbiter的调整下，端口40000的节点已经变成了secondary，而端口40001的节点已经变为primary，此时，插入一条记录，并重启端口40000的节点查看复制集状态信息：



```
rs0:PRIMARY> use cms
switched to db cms
rs0:PRIMARY> db.customers.insert({id:13,name:'wangwu'})
WriteResult({ "nInserted" : 1 })
rs0:PRIMARY> db.status()
{
        "set" : "rs0",
        "date" : ISODate("2017-02-24T08:23:17.763Z"),
        "myState" : 1,
        "term" : NumberLong(3),
        "heartbeatIntervalMillis" : NumberLong(2000),
        "members" : [
                {
                        "_id" : 0,
                        "name" : "linfl-PC:40000",
                        "health" : 1,
                        "state" : 2,
                        "stateStr" : "SECONDARY",
                        "uptime" : 1085,
                        "optime" : {
                                "ts" : Timestamp(1487923321, 1),
                                "t" : NumberLong(3)
                        },
                        "optimeDate" : ISODate("2017-02-24T08:02:01Z"),
                        "lastHeartbeat" : ISODate("2017-02-24T08:23:16.122Z"),
                        "lastHeartbeatRecv" : ISODate("2017-02-24T08:23:16.122Z"
),
                        "pingMs" : NumberLong(0),
                        "syncingTo" : "linfl-PC:40001",
                        "configVersion" : 5
                },
                {
                        "_id" : 1,
                        "name" : "linfl-PC:40001",
                        "health" : 1,
                        "state" : 1,
                        "stateStr" : "PRIMARY",
                        "uptime" : 2013,
                        "optime" : {
                                "ts" : Timestamp(1487923321, 1),
                                "t" : NumberLong(3)
                        },
                        "optimeDate" : ISODate("2017-02-24T08:02:01Z"),
                        "electionTime" : Timestamp(1487923022, 1),
                        "electionDate" : ISODate("2017-02-24T07:57:02Z"),
                        "configVersion" : 5,
                        "self" : true
                },
                {
                        "_id" : 2,
                        "name" : "linfl-PC:40002",
                        "health" : 1,
                        "state" : 7,
                        "stateStr" : "ARBITER",
                        "uptime" : 2010,
                        "lastHeartbeat" : ISODate("2017-02-24T08:23:17.132Z"),
                        "lastHeartbeatRecv" : ISODate("2017-02-24T08:23:15.847Z"
),
                        "pingMs" : NumberLong(0),
                        "configVersion" : 5
                }
        ],
        "ok" : 1
}

```
此时，数据同步完成，复制集正常工作。




### 写关注

### 读参考

## 参考

> https://docs.mongodb.com/manual/replication/



