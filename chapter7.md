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

### 数据同步

### 故障转移

### 写关注

### 读参考

## 参考

> https://docs.mongodb.com/manual/replication/\#edge-cases-2-primaries



