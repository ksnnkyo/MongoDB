# windows下Mongodb数据库扩容方案

启动mongodb时，需要指定data目录，在windows下这个目录必须位于某个分区上，而分区的大小是固定的，也就是说，mongodb数据入库时，当数据量达到磁盘大小时就会出现存储瓶颈，此时，分区容量已确定，增加新的磁盘虽然可以扩展整个系统可用的存储空间，但不能扩大mongodb的data目录可以用的存储空间，又或者选择更大的磁盘，又或者使用专门的存储设备，但是无论哪种解决办法，都必须迁移数据，有时间成本，成本大小视数据量大小而定。下面使用一种使用windows的mklink命令可以解决这个问题，同时，再次也深入理解一下 --directoryperdb参数的巧妙用处。
    
## 配置mongodb的启动环境


```
storage:
    dbPath: "e:/7.DBData/MongoDB/sl/data"
    directoryPerDB: true
systemLog:
    destination: file
    path: "e:/7.DBData/MongoDB/sl/log/log.log"
net:
    port: 27017
```


## 分配实际的存储路径

在开启数据库之前，实际设置好，在要实际存储某个db（如test）的分区上存储该db的文件夹 E:/7.DBData/MongoDB/test2/ ,然后，使用mklink命令在data目录下为test数据库创建一个软连接，指向真实存储该数据库的目录。
    
```
mklink /J E:/7.DBData/MongoDB/sl/data/test E:/7.DBData/MongoDB/test
```

## 启动mongodb数据库

```
mongod -f "D:/Program/MongoDB/Server/3.4/bin/mongodb.conf"
```

## 连接mongodb数据库

连接mongodb数据库，执行一下命令：
```
>use test
>db.createCollection('test')
>db.test.insert({a:1})
```

注意，数据库的名字应当与创建的软连接的名字一致。
命令均执行成功，此时数据实际被存储到了软连接指定的目录上。
 
## 查看data目录

使用dir命令查看data目录，会发现，test文件夹的类型是 **JUNCTION**，而不是 **DIR**
    
## 总结

- 该方式对于已经在运行中的mongodb也有效，需要暂停mongodb服务，然后按照上面的步骤创建新库，或者将旧库迁移到其他目录，然后使用mklink创建连接，然后再启动mongodb。
- 使用参数directoryperdb可以更有效的管理数据，同时也利于数据的扩展。
- linux下可以直接通过软连接的形式实现，效果一致。