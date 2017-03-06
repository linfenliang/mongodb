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






## 备份与恢复

### 单节点dump备份与恢复

### 集群dump备份与恢复

## 监控

### 数据库监控命令

### 操作系统监控命令

### WEB控制台监控
