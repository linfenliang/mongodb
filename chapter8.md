## 分片集群简介
在之前有说过关于MongoDB的复制集，复制集主要用来实现自动故障转移从而达到高可用的目的，然而，随着业务规模的增长和时间的推移，业务数据量会越来越大，当前业务数据可能只有几百GB不到，一台DB服务器足以搞定所有的工作，而一旦业务数据量扩充大几个TB几百个TB时，就会产生一台服务器无法存储的情况，此时，需要将数据按照一定的规则分配到不同的服务器进行存储、查询等，即为分片集群。分片集群要做到的事情就是数据分布式存储。

## 分片部署架构
先看一张图：

![](https://docs.mongodb.com/manual/_images/sharded-cluster-production-architecture.bakedsvg.svg)


## 分片工作机制

### 集合分片

### 集群平衡器

### 集群的写、读

### 片键选择策略

## 参考链接

[](https://docs.mongodb.com/manual/core/sharded-cluster-components/)