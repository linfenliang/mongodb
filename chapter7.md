## 复制集功能概述

复制集（replica set）是MongoDB用来保持相同的数据集合的一个MongoD进程组，复制集提供了所有生产部署的基础：数据冗余以及高可用。MongoDB的高可用靠的是自动故障转移来实现的，本节就是介绍MongoDB的该部分实现的。

## 复制集工作原理

虽然Journaling日志功能提供了数据恢复的功能，但是他通常针对的是单个节点来说的，而复制集则针对的是一组进程，通常是多个节点组成，在每个节点上有Journaling日志保证数据完整性，在整个复制集中实现自动故障转移，从而保证了数据库的高可用性。  
在生产环境中，一个复制集应该最少包含三个节点，一个仲裁节点（arbiter），唯一一个数据主节点（primary），一个或多个数据次节点（secondary）。  
主节点用来接收所有的写操作，一个复制集有且仅有一个primary能够进行写关注（写关注将在后面介绍），主节点在他的操作日志oplog中将所有的修改记录到数据集data sets中。典型的结构如下所示：

![](https://docs.mongodb.com/manual/_images/replica-set-read-write-operations-primary.png)

secondary节点备份primary节点上的数据，secondary节点可以有多个，一旦primary节点不可用，abiter将从secondary节点中选取一个作为primary节点，secondary节点的作用如下：

![](https://docs.mongodb.com/manual/_images/replica-set-primary-with-two-secondaries.png)

现在除了primary，secondary节点外，可以新增一个mongod实例副本集作为arbiter，arbiter不能维护数据集。arbiter的主要作用是维持与复制集中所有的其他节点的心跳以保证选举需要的节点数，因为arbiter不是一个数据存储集，arbiter可以提供一个比全功能副本集更廉价的方法来获取法定人数。如果复制集中是偶数个节点，可以通过添加arbiter节点使得primary可以获取到大多数的投票。arbiter不需要专门的硬件支持。arbiter的作用如下：

![](https://docs.mongodb.com/manual/_images/replica-set-primary-with-secondary-and-arbiter.png)

相对于primary与secondary节点可能在一次选举中（主节点失效触发）互换角色，arbiter仲裁者永远都是arbiter。

故障转移流程如下所示：

![https://docs.mongodb.com/manual/replication/#edge-cases-2-primaries](https://docs.mongodb.com/manual/_images/replica-set-trigger-election.png)

复制集的创建：

在D:\MongoDB\Server\3.2\bin文件夹中运行CMD命令，启动mongod三个进程，
分别是：


```
mongod --dbpath=D:\MongoDB\Server\3.2\data\rs0_0 --logpath=D:\MongoDB\Server\3.2\logs\rs0_0.log --port=40000 --replSet=rs0
mongod --dbpath=D:\MongoDB\Server\3.2\data\rs0_1 --logpath=D:\MongoDB\Server\3.2\logs\rs0_1.log --port=40000 --replSet=rs0
mongod --dbpath=D:\MongoDB\Server\3.2\data\rs0_2 --logpath=D:\MongoDB\Server\3.2\logs\rs0_2.log --port=40000 --replSet=rs0
```



注意：需要创建data文件夹下的子文件夹为rs0_0,rs0_1,rs0_2

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


### 故障转移

### 写关注

### 读参考

## 参考

> https://docs.mongodb.com/manual/replication/\#edge-cases-2-primaries



