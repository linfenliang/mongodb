## 简介

MongoDB的管理与监控是可以参考关系型数据库的各种管理思想的，如常用的数据导入导出、备份、监控等，本节主要介绍相关的内容是如何在MongoDB中实现的，MongoDB管理的DBA需要重点关注。

## 数据的导入与导出
数据导入与导出可以借助mongoexport与mongoimport实现

数据导出：
先查看数据：



```
> use test
switched to db test
> show collections
fs.chunks
fs.files
restaurants
user
> db.user.find()
{ "_id" : ObjectId("5895950a4aed6ad4d4a3fe5c"), "username" : "linfenliang", "password" : "123456" }
```

导出命令：



```
D:\MongoDB\Server\3.2\bin>mongoexport.exe --db test --collection user --out E:\user.json
2017-03-06T15:53:32.916+0800    connected to: localhost
2017-03-06T15:53:32.927+0800    exported 1 record
```

导出后的user.json数据：


```
{"_id":{"$oid":"5895950a4aed6ad4d4a3fe5c"},"username":"linfenliang","password":"123456"}
```

导出CSV格式命令：



```
D:\MongoDB\Server\3.2\bin>mongoexport.exe --db test --collection user --type=csv --fields username,password --out E:\user.csv
2017-03-06T16:04:00.231+0800    connected to: localhost
2017-03-06T16:04:00.242+0800    exported 1 record
```

查看csv格式数据：


```
username	password
linfenliang	123456

```
**注意：这种方式由于会强制数据库读取所有的要导出的数据到内存，会对正在运行的Mongod示例性能产生影响。**

数据导入：

原始数据：



```
{"username":"lisi","password":"654321"}
```

导入命令：

```
D:\MongoDB\Server\3.2\bin>mongoimport.exe --db test --collection user --file E:\user.json --type json --upsert
2017-03-06T16:47:27.328+0800    connected to: localhost
2017-03-06T16:47:27.356+0800    imported 1 document
```

导入结果：


```
> db.user.find()
{ "_id" : ObjectId("5895950a4aed6ad4d4a3fe5c"), "username" : "linfenliang", "password" : "123456" }
{ "_id" : ObjectId("58bd221fd6d41608a1a3f750"), "username" : "lisi", "password" : "654321" }
```

其中 --upsert表示如果有相同记录，则更新（默认的依据是_id相同）可以通过--upsertFields指定判断记录是否相同的字段。

导入csv文件时，格式基本一致，但是需要指定一个参数：--headerline，此参数表示文件的第一行为字段名，不导入。

MongoDB的这两个工具，适合用来将关系型数据库中的数据与MongoDB数据做互相转换，并不适合读数据库全量备份。








## 备份与恢复



### 单节点dump备份与恢复

备份命令：



```
D:\MongoDB\Server\3.2\bin>mongodump.exe --db test --out E:\test_bak
2017-03-06T18:20:05.565+0800    writing test.restaurants to
2017-03-06T18:20:05.577+0800    writing test.user to
2017-03-06T18:20:05.578+0800    writing test.fs.chunks to
2017-03-06T18:20:05.579+0800    writing test.fs.files to
2017-03-06T18:20:05.580+0800    done dumping test.restaurants (2 documents)
2017-03-06T18:20:05.581+0800    done dumping test.user (2 documents)
2017-03-06T18:20:05.582+0800    done dumping test.fs.chunks (1 document)
2017-03-06T18:20:05.582+0800    done dumping test.fs.files (1 document)
```
这是生成后的文件信息：

![](/assets/dump_image.png)

常用的命令参数还有：

--collection 指定具体的集合
--query 按照查询条件备份

恢复命令：



```
D:\MongoDB\Server\3.2\bin>mongorestore.exe --db user_restore E:\test_bak\test
2017-03-06T18:43:04.835+0800    building a list of collections to restore from E:\test_bak\test dir
2017-03-06T18:43:04.849+0800    reading metadata for user_restore.fs.chunks from E:\test_bak\test\fs.chunks.metadata.json
2017-03-06T18:43:04.960+0800    restoring user_restore.fs.chunks from E:\test_bak\test\fs.chunks.bson
2017-03-06T18:43:04.961+0800    reading metadata for user_restore.restaurants from E:\test_bak\test\restaurants.metadata.json
2017-03-06T18:43:04.967+0800    reading metadata for user_restore.fs.files from E:\test_bak\test\fs.files.metadata.json
2017-03-06T18:43:04.969+0800    reading metadata for user_restore.user from E:\test_bak\test\user.metadata.json
2017-03-06T18:43:05.019+0800    restoring user_restore.restaurants from E:\test_bak\test\restaurants.bson
2017-03-06T18:43:05.021+0800    restoring indexes for collection user_restore.fs.chunks from metadata
2017-03-06T18:43:05.082+0800    restoring user_restore.fs.files from E:\test_bak\test\fs.files.bson
2017-03-06T18:43:05.133+0800    restoring user_restore.user from E:\test_bak\test\user.bson
2017-03-06T18:43:05.135+0800    restoring indexes for collection user_restore.restaurants from metadata
2017-03-06T18:43:05.171+0800    finished restoring user_restore.restaurants (2 documents)
2017-03-06T18:43:05.172+0800    restoring indexes for collection user_restore.fs.files from metadata
2017-03-06T18:43:05.174+0800    restoring indexes for collection user_restore.user from metadata
2017-03-06T18:43:05.175+0800    finished restoring user_restore.fs.chunks (1 document)
2017-03-06T18:43:05.177+0800    finished restoring user_restore.fs.files (1 document)
2017-03-06T18:43:05.178+0800    finished restoring user_restore.user (2 documents)
2017-03-06T18:43:05.180+0800    done
```

查看恢复情况：



```
> use user_restore
switched to db user_restore
> show collections
> show collections
fs.chunks
fs.files
restaurants
user
```
常用的命令参数还有：

--collection 指定具体的集合
--drop 恢复数据前删除此数据库下的所有集合





### 集群dump备份与恢复

在MongoDB集群上备份，实质上是对MongoDB的复制集进行备份操作，具体流程如下：

1. 禁用平衡器：sh.stopBalancer()
因为分片集群上会有一个balancer进程在后台维护各个分片上的数据块数量的均衡，如果不禁用平衡器就可能导致设备护具的重复或缺失。
2. 停止每个片上的某一个secondary节点，利用此节点进行备份，停止其中某一个配置服务器，保证备份时配置服务器上的源数据不会改变，备份secondary节点数据与及备份完全相同
3. 重启所有停掉的复制集成员（他会自动从primary节点同步恢复数据，达到最终数据一致性）
4. 重启分片集群上的平衡器
命令为：


```
use config
sh.startBalancer()


```

集群数据恢复流程：

1. 停止集群中所有的mongod实例以及mongos实例
2. 利用dump文件依次回复每一个片中的每个复制集
3. 恢复配置服务器
4. 重启所有mongod实例以及Mongos实例
5. 通过mongo连接Mongos，执行命令 db.printShardingStatus()确保集群可用






## 监控

### 数据库监控命令
1. mongostat命令：


```
D:\MongoDB\Server\3.2\bin>mongostat.exe
insert query update delete getmore command % dirty % used flushes vsize   res qr|qw ar|aw netIn netOut conn                      time
    *0    *0     *0     *0       0     1|0     0.0    0.0       0  169M 45.0M   0|0   0|0   79b  19.3k    1 2017-03-07T15:59:29+08:00
    *0    *0     *0     *0       0     1|0     0.0    0.0       0  169M 45.0M   0|0   0|0   79b  19.3k    1 2017-03-07T15:59:30+08:00
    *0    *0     *0     *0       0     1|0     0.0    0.0       0  169M 45.0M   0|0   0|0   79b  19.3k    1 2017-03-07T15:59:31+08:00
    *0    *0     *0     *0       0     1|0     0.0    0.0       0  169M 45.0M   0|0   0|0   79b  19.3k    1 2017-03-07T15:59:32+08:00
    *0    *0     *0     *0       0     1|0     0.0    0.0       0  169M 45.0M   0|0   0|0   79b  19.3k    1 2017-03-07T15:59:33+08:00
    *0    *0     *0     *0       0     1|0     0.0    0.0       0  169M 45.0M   0|0   0|0   79b  19.3k    1 2017-03-07T15:59:34+08:00
```

上面这个命令一看就懂，基本不需要多说。
2. 另一个命令：mongotop



```
D:\MongoDB\Server\3.2\bin>mongotop.exe
2017-03-07T16:01:04.063+0800    connected to: 127.0.0.1

                    ns    total    read    write    2017-03-07T16:01:05+08:00
    admin.system.roles      0ms     0ms      0ms
  admin.system.version      0ms     0ms      0ms
     local.startup_log      0ms     0ms      0ms
  local.system.replset      0ms     0ms      0ms
        test.fs.chunks      0ms     0ms      0ms
         test.fs.files      0ms     0ms      0ms
      test.restaurants      0ms     0ms      0ms
             test.user      0ms     0ms      0ms
user_restore.fs.chunks      0ms     0ms      0ms
 user_restore.fs.files      0ms     0ms      0ms
```

3. serverStatus命令：



```
D:\MongoDB\Server\3.2\bin>mongo.exe
2017-03-07T16:02:49.399+0800 I CONTROL  [main] Hotfix KB2731284 or later update is not installed, will zero-out data files
MongoDB shell version: 3.2.9
connecting to: test
> db.serverStatus()
```

信息量打印太多，不再贴出来了。

4. stats命令：



```
D:\MongoDB\Server\3.2\bin>mongo.exe
2017-03-07T16:02:49.399+0800 I CONTROL  [main] Hotfix KB2731284 or later update is not installed, will zero-out data files
MongoDB shell version: 3.2.9
connecting to: test
> db.stats()
{
        "db" : "test",
        "collections" : 4,
        "objects" : 6,
        "avgObjSize" : 663.6666666666666,
        "dataSize" : 3982,
        "storageSize" : 14036992,
        "numExtents" : 0,
        "indexes" : 5,
        "indexSize" : 176128,
        "ok" : 1
}
```
影响数据库性能的几个主要参数：

锁，可以通过serverStatus中globalLock数据查看currentQueue中的total值，该值反应了是否存在并发问题严重程度

内存，可通过serverStatus查看


```
"mem" : { "bits" : 64,
    "resident" : 45,
    "virtual" : 169,
    "supported" : true,
    "mapped" : 0,
    "mappedWithJournal" : 0 },
```

注意 mapped如果过大，会引起缺页错误，resident过大，表示系统内存过小

缺页错误，page_faults



```
"extra_info" : { "note" : "fields vary by platform",
    "page_faults" : 51143,
    "usagePageFileMB" : 72,
    "totalPageFileMB" : 16133,
    "availPageFileMB" : 10188,
    "ramMB" : 8067 },
```

这个会导致大量的内存置换，解决的办法是部署分片集群

连接数，activeClients 表示连接数，


```
 "globalLock" : { "totalTime" : { "$numberLong" : "1561525000" },
    "currentQueue" : { "total" : 0,
      "readers" : 0,
      "writers" : 0 },
    "activeClients" : { "total" : 9,
      "readers" : 0,
      "writers" : 0 } },
```
对于读操作大的应用，可以增加复制集成员数，将读操作分发到secondary节点上，对写操作大的应用，通过部署分片集群分发写操作。



### 操作系统监控命令

top
free
iostat

基本的常用命令，非mongod独有，不再赘述。


### WEB控制台监控

启动mongod时，添加参数：>mongod --httpinterface --rest
前一个参数表示打开Web控制台，后一个参数能够获取更为详细的数据，不建议在生产环境打开。