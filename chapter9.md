## 概述
对于MongoDB的存储基本单元BSON文档对象，字段值可以是二进制类型，基于此特点，我们可以直接在MongoDB中存储文件，但是有一个限制，由于MongoDB中单个BSON对象不能大于16MB，故而如果需要存储更大的文件，就需要GridFS了。

## 小文件存储系统
我们先看下MongoDB存储小文件系统的例子：


先采用MongoDB的mongofiles执行文件上传：
```
D:\MongoDB\Server\3.2\bin>mongofiles.exe list
2017-03-06T13:41:03.283+0800    connected to: localhost

D:\MongoDB\Server\3.2\bin>mongofiles.exe put E:\deliveryTask.doc
2017-03-06T13:41:23.535+0800    connected to: localhost
added file: E:\deliveryTask.doc

D:\MongoDB\Server\3.2\bin>mongofiles.exe list
2017-03-06T13:41:30.114+0800    connected to: localhost
E:\deliveryTask.doc     2971
```
通过mongos命令查看文件存储情况：


```
> use test
switched to db test
> show collections
fs.chunks
fs.files
restaurants
user
> db.fs.files.find()
{ "_id" : ObjectId("58bcf683afa0fa20bc854a2b"), "chunkSize" : 261120, "uploadDat
e" : ISODate("2017-03-06T05:41:23.604Z"), "length" : 2971, "md5" : "5434b8033062
99fff57c8a54d3adf78b", "filename" : "E:\\deliveryTask.doc" }
```

可以看到文件上传成功了



## GridFS文件存储
