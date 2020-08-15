# MongoDB


<!--more-->

## 应用场景
MongoDB采用No-Schema的方式，免去您变更表结构的痛苦，非常适用于初创型的业务需求。您可以将模式固定的结构化数据存储在RDS（Relational Database Service）中，模式灵活的业务存储在MongoDB中，高热数据存储在云数据库Memcache或云数据库Redis中，实现对业务数据高效存取，降低存储数据的投入成本。




## 数据去重

![pop](/images/202003/mongodb1.png "image1")

mongo中有如图所示的重复数据, 使用如下命令可去重:
```text
use dbname

# 使用组合字段列表作为判重依据, 可添加多个字段
db.集合.aggregate([
    {
        $group: { _id: {字段1: '$字段1',字段2: '$字段2'},count: {$sum: 1},dups: {$addToSet: '$_id'}}
    },
    {
        $match: {count: {$gt: 1}}
    }
],{
allowDiskUse: true
}).forEach(function(doc){
    doc.dups.shift();
    db.集合.remove({_id: {$in: doc.dups}});
})
```

数据过多时, 可能导致内存溢出, ```allowDiskUse: true``` 选项可允许数据转存到硬盘;

**注意: 此方法效率不高, 数据过多时不可使用此法, 而应该从插入数据的源头治理.**

[Reference](https://www.cnblogs.com/nicolegxt/p/24b3653947991ebe73e5d70609ab2943.html)

---


## 集合导入与导出
mongoDB中的mongoexport工具可以把一个collection导出成JSON格式或CSV格式的文件.

### 导出
```shell script
mongoexport -d dbname -c collectionname -o filename --type json/csv -f field

# 示例
sudo mongoexport -d mongotest -c users -o /home/python/Desktop/mongoDB/users.json --type json -f "_id,user_id,user_name,age,status" 

```

参数说明:
* -d ：数据库名
* -c ：collection名
* -o ：输出文件名
* --type ： 输出格式，默认为json
* -f ：输出字段，如果-type为csv，则需要加上-f "字段名"

### 导入

```shell script
mongoimport -d dbname -c collectionname --file filename --headerline --type json/csv -f field

# 示例
sudo mongoimport -d mongotest -c users --file /home/mongodump/articles.json --type json
```

参数说明：
* -d ：数据库名
* -c ：collection名
* --type ：导入格式, 默认json
* -f ：导入字段名
* --headerline ：如果导入的格式是csv，则可以使用第一行的标题作为导入的字段
* --file ：要导入的文件

## 数据库备份与恢复

### 备份

```shell script
mongodump -h dbhost -d dbname -o dbdirectory

# 示例
sudo rm -rf /home/momgodump/
sudo mkdir -p /home/momgodump
sudo mongodump -h 192.168.17.129:27017 -d itcast -o /home/mongodump/
```

参数说明：
* -h: MongDB所在服务器地址，例如: 127.0.0.1，当然也可以指定端口号: 127.0.0.1:27017
* -d: 需要备份的数据库实例，例如: test
* -o: 备份文件存放目录，例如: /home/mongodump/，当然该目录需要提前建立，这个目录里面存放该数据库实例的备份数据。


### 恢复

```shell script
mongorestore -h dbhost -d dbname --dir dbdirectory

# 示例
mongorestore -h 192.168.17.129:27017 -d itcast_restore --dir /home/mongodump/itcast/
```

参数说明：
* -h: MongDB所在服务器地址，例如: 127.0.0.1，当然也可以指定端口号: 127.0.0.1:27017
* -d: 需要恢复的数据库实例，例如：test，当然这个名称也可以和备份时候的不一样，比如test2
* --dir: 备份数据所在位置，例如：/home/mongodump/itcast/
* --drop: 恢复的时候，先删除当前数据，然后恢复备份的数据。慎用！


* [Reference](https://blog.csdn.net/qq_16313365/article/details/56494522)
* [Reference](https://blog.csdn.net/gaomengwang/article/details/78354927)(更详细选项)
---

## 可视化管理工具--studio3t

[Release the full power of MongoDB](https://studio3t.com/)
