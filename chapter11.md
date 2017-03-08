## 权限控制API

### 所有数据库的角色控制

首先在启用权限控制时，需要在启动MongoDB时指定启动，可以通过配置文件或启动命令添加：



```
D:\MongoDB\Server\3.2\bin>mongod --auth
```
再次连接到MongoDB，执行命令时显示无权限了：



```
> show dbs
2017-03-08T10:22:53.340+0800 E QUERY    [thread1] Error: listDatabases failed:{
        "ok" : 0,
        "errmsg" : "not authorized on admin to execute command { listDatabases: 1.0 }",
        "code" : 13
} :
```
然后停掉授权模式，重新启动mongod，添加用户，分配权限：




```
>  db.createUser({user:'linfenliang',pwd:'123456',roles:[{role:'root',db:'admin'}]})
Successfully added user: {
        "user" : "linfenliang",
        "roles" : [
                {
                        "role" : "root",
                        "db" : "admin"
                }
        ]
}
```

再次连接到Mongodb，查看：


```
D:\MongoDB\Server\3.2\bin>mongo.exe -u linfenliang -p 123456
2017-03-08T10:26:43.075+0800 I CONTROL  [main] Hotfix KB2731284 or later update is not installed, will zero-out data files
MongoDB shell version: 3.2.9
connecting to: test
> show dbs
admin         0.000GB
local         0.000GB
test          0.013GB
user_restore  0.000GB
```






### 单个数据库的角色控制


## 复制集与集群的权限控制
