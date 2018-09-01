---
title: Mongodb Replica Set 集群配置
date: 2016-10-11 16:22:15
tags: mongodb 
---

> Replica Set  为 mongodb 3.0 以上版本官方推荐的群集方法
Mongodb 3.0 以上版本使用 Replica Set 搭建 Mongodb 集群服务，原 master-slave 则在3.0以上版本为不推荐使用
https://docs.mongodb.com/manual/core/replica-set-architectures/
Replica Set （副本集）支持一台写，多台读, 分片 (sharding) 则可以支持每个分片放置副本集 https://docs.mongodb.com/manual/sharding/


## 防火墙设定
<!-- more -->
``` bash
ufw allow 27017/tcp # 允许 27017端口通过
```

## 所有服务器的 mongod.conf (一般位于 /etc/mongod.conf)
``` conf
net:
  port: 27017
  bindIp: 192.168.0.104,192.168.0.107 # 增加其他服务器IP
   
replication:
  replSetName: mongo1 # 副本集名称 所有服务器相同
  oplogSizeMB: 5120 # 大幅性能 https://docs.mongodb.com/manual/reference/configuration-options/#replication-options
   
```

``` bash
# 重新启动所有服务器 mongod 服务
service mongod restart
```

## 主 Mongo 设定

进入 Mongo Shell
``` conf
config = {
    _id: 'mongo1', 
    members: [
        {_id: 0, host: '192.168.0.104:27017', priority:1}, #主mongo
        {_id: 1, host: '192.168.0.107:27017'} #其他 mongo
        ]
}
rs.initiate(config)
rs.conf()
rs.status()
 
# 返回结果
 
{
    "set" : "mongo1",
    "date" : ISODate("2016-10-11T08:14:16.365Z"),
    "myState" : 1,
    "members" : [
        {
            "_id" : 0,
            "name" : "192.168.0.104:27017",
            "health" : 1,
            "state" : 1,
            "stateStr" : "PRIMARY",
            "uptime" : 1713,
            "optime" : Timestamp(1476173146, 1),
            "optimeDate" : ISODate("2016-10-11T08:05:46Z"),
            "electionTime" : Timestamp(1476172001, 1),
            "electionDate" : ISODate("2016-10-11T07:46:41Z"),
            "configVersion" : 84486,
            "self" : true
        },
        {
            "_id" : 1,
            "name" : "192.168.0.107:27017",
            "health" : 1,
            "state" : 2,
            "stateStr" : "SECONDARY",
            "uptime" : 1656,
            "optime" : Timestamp(1476173146, 1),
            "optimeDate" : ISODate("2016-10-11T08:05:46Z"),
            "lastHeartbeat" : ISODate("2016-10-11T08:14:15.569Z"),
            "lastHeartbeatRecv" : ISODate("2016-10-11T08:14:14.929Z"),
            "pingMs" : 0,
            "syncingTo" : "192.168.0.104:27017",
            "configVersion" : 84486
        }
    ],
    "ok" : 1
}
 
# 如果配置有误  health:0 显示为0
# 重新配置加 force:true
rs.reconfig(config, {force: true})
```

## 增加更多 Mongo

```bash
rs.add("192.168.0.108:27017")
```

## 删除不使用的 Mongo

``` bash
rs.remove("192.168.0.108:27017")
```

## 其他 Mongo 设定
进入 Mongo Shell

```bash
rs.slaveOk() #即可完成数据同步
```

>只有主 Mongo 允许写数据，所以测试时进入主 Mongo，创建新 collection ，回到所有 其他  Mongo 通过以下查看同步过来的 collection

``` bash
show collections
```

> 无法写入错误
副本集至少需要有2个以上 Mongod 服务，如果只剩下最后一个 Mongod 服务那就会出现无法写入情况
