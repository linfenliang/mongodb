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

### 集群dump备份与恢复

## 监控

### 数据库监控命令

### 操作系统监控命令

### WEB控制台监控
