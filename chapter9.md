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

由于本章节主要涉及到的是部分理论以及运维实践，暂不涉及具体代码的开发实现（具体代码将以Java为例在后面章节中介绍）。

上传一个大于16MB的文件试一试：



```
D:\MongoDB\Server\3.2\bin>mongofiles.exe put E:\synch.rar
2017-03-06T14:33:11.028+0800    connected to: localhost
added file: E:\synch.rar

D:\MongoDB\Server\3.2\bin>mongofiles.exe list
2017-03-06T14:33:15.265+0800    connected to: localhost
E:\deliveryTask.doc     2971
E:\synch.rar    24183487
```
通过mongos命令查看文件存储情况：




```
> db.fs.files.find()
{ "_id" : ObjectId("58bcf683afa0fa20bc854a2b"), "chunkSize" : 261120, "uploadDate" : ISODate("2017-03-06T05:41:23.604Z"), "length" : 2971, "md5" : "5434b803306299fff57c8a54d3adf78b", "filename" : "E:\\deliveryTask.doc" }
{ "_id" : ObjectId("58bd02a7afa0fa21d4a14b2c"), "chunkSize" : 261120, "uploadDate" : ISODate("2017-03-06T06:33:12.013Z"), "length" : 24183487, "md5" : "bbfe4d8579372aa0729726185997e908", "filename" : "E:\\synch.rar" }
```
也成功了，

查看chunks:


```
> db.fs.chunks.find({},{data:0})
{ "_id" : ObjectId("58bcf683afa0fa20bc854a2c"), "files_id" : ObjectId("58bcf683afa0fa20bc854a2b"), "n" : 0 }
{ "_id" : ObjectId("58bd02a7afa0fa21d4a14b2d"), "files_id" : ObjectId("58bd02a7afa0fa21d4a14b2c"), "n" : 0 }
{ "_id" : ObjectId("58bd02a7afa0fa21d4a14b2e"), "files_id" : ObjectId("58bd02a7afa0fa21d4a14b2c"), "n" : 1 }
{ "_id" : ObjectId("58bd02a7afa0fa21d4a14b2f"), "files_id" : ObjectId("58bd02a7afa0fa21d4a14b2c"), "n" : 2 }
{ "_id" : ObjectId("58bd02a7afa0fa21d4a14b30"), "files_id" : ObjectId("58bd02a7afa0fa21d4a14b2c"), "n" : 3 }
{ "_id" : ObjectId("58bd02a7afa0fa21d4a14b31"), "files_id" : ObjectId("58bd02a7afa0fa21d4a14b2c"), "n" : 4 }
{ "_id" : ObjectId("58bd02a7afa0fa21d4a14b32"), "files_id" : ObjectId("58bd02a7afa0fa21d4a14b2c"), "n" : 5 }
{ "_id" : ObjectId("58bd02a7afa0fa21d4a14b33"), "files_id" : ObjectId("58bd02a7afa0fa21d4a14b2c"), "n" : 6 }
{ "_id" : ObjectId("58bd02a7afa0fa21d4a14b34"), "files_id" : ObjectId("58bd02a7afa0fa21d4a14b2c"), "n" : 7 }
{ "_id" : ObjectId("58bd02a7afa0fa21d4a14b35"), "files_id" : ObjectId("58bd02a7afa0fa21d4a14b2c"), "n" : 8 }
{ "_id" : ObjectId("58bd02a7afa0fa21d4a14b36"), "files_id" : ObjectId("58bd02a7afa0fa21d4a14b2c"), "n" : 9 }
{ "_id" : ObjectId("58bd02a7afa0fa21d4a14b37"), "files_id" : ObjectId("58bd02a7afa0fa21d4a14b2c"), "n" : 10 }
{ "_id" : ObjectId("58bd02a7afa0fa21d4a14b38"), "files_id" : ObjectId("58bd02a7afa0fa21d4a14b2c"), "n" : 11 }
{ "_id" : ObjectId("58bd02a7afa0fa21d4a14b39"), "files_id" : ObjectId("58bd02a7afa0fa21d4a14b2c"), "n" : 12 }
{ "_id" : ObjectId("58bd02a7afa0fa21d4a14b3a"), "files_id" : ObjectId("58bd02a7afa0fa21d4a14b2c"), "n" : 13 }
{ "_id" : ObjectId("58bd02a7afa0fa21d4a14b3c"), "files_id" : ObjectId("58bd02a7afa0fa21d4a14b2c"), "n" : 15 }
{ "_id" : ObjectId("58bd02a7afa0fa21d4a14b3b"), "files_id" : ObjectId("58bd02a7afa0fa21d4a14b2c"), "n" : 14 }
{ "_id" : ObjectId("58bd02a7afa0fa21d4a14b3e"), "files_id" : ObjectId("58bd02a7afa0fa21d4a14b2c"), "n" : 17 }
{ "_id" : ObjectId("58bd02a7afa0fa21d4a14b3d"), "files_id" : ObjectId("58bd02a7afa0fa21d4a14b2c"), "n" : 16 }
{ "_id" : ObjectId("58bd02a7afa0fa21d4a14b3f"), "files_id" : ObjectId("58bd02a7afa0fa21d4a14b2c"), "n" : 18 }
Type "it" for more
```

可以看到大文件被分成了好多个chunk，



## GridFS文件存储
