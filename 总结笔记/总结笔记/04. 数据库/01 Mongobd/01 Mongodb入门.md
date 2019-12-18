## 一、MongoDB概要

MongoDB是Nosql型数据库，即非关系型数据库。

MongoDB的优势：

1. 易扩展： NoSQL数据库种类繁多， 但是⼀个共同的特点都是去掉关系数据库的关系型特性。 数据之间⽆关系， 这样就⾮常容易扩展
2. ⼤数据量， ⾼性能： NoSQL数据库都具有⾮常⾼的读写性能， 尤其在⼤数据量下， 同样表现优秀。 这得益于它的⽆关系性， 数据库的结构简单
3. 灵活的数据模型： NoSQL⽆需事先为要存储的数据建⽴字段， 随时可以存储⾃定义的数据格式。 ⽽在关系数据库⾥， 增删字段是⼀件⾮常麻烦的事情。 如果是⾮常⼤数据量的表， 增加字段简直就是⼀个噩梦

## 二、MongoDB安装及配置

### 2.1 Ubuntu中安装

```shell
sudo apt-get install -y mongodb-org
```

### 2.2 服务器端配置及操作

- 查看帮助：mongod –help
- 启动：sudo service mongod start
- 停止：sudo service mongod stop
- 重启：sudo service mongod restart
- 查看是否启动成功：ps ajx|grep mongod

配置文件的位置：/etc/mongod.conf               默认端⼝：27017

日志的位置：/var/log/mongodb/mongod.log

### 2.3 客户端操作

- 启动本地客户端：mongo
- 查看帮助：mongo –help
- 退出：exit或者ctrl+c

## 三、MongoDB基本操作

### 3.1 基础命令

#### 3.1.1 数据库命令

- 查看当前的数据库：db
- 查看所有的数据库：show dbs   |   show databases
- 切换数据库：use db_name
- 删除当前的数据库：db.dropDatabase()

#### 3.1.2 集合命令

向不存在的集合中第一次加入数据时，集合会被创建出来。

- 手动创建集合：db.createCollection(name, options)

  - eg：db.createCollection("sub", {capped : true, size : 10})

    其中，参数capped默认值为false，表示不设置上限，值为true时需要制定size参数，表示上限大小（字节）。

- 查看集合：show collections

- 删除集合：db.collec_name.drop()

### 3.2 数据类型

MongoDB数据类型与JSON类似。

- Object ID： ⽂档ID，每个文档都有一个属性，为`_id`，保证每个文档唯一性。可以自己去设置`_id`插入文档，如果没有提供，nameMongoDB为每个文档提供一个独特的`_id`，类型为Object ID。
- String： 字符串， 最常⽤， 必须是有效的UTF-8
- Boolean： 存储⼀个布尔值， true或false（小写）
- Integer： 整数可以是32位或64位， 这取决于服务器
- Double： 存储浮点值
- Arrays： 数组或列表， 多个值存储到⼀个键
- Object： ⽤于嵌⼊式的⽂档， 即⼀个值为⼀个⽂档
- Null： 存储Null值
- Timestamp： 时间戳， 表示从1970-1-1到现在的总秒数
- Date： 存储当前⽇期或时间的UNIX时间格式，创建日期的语句为：`new Date('2018-11-12')`

> objectID是⼀个12字节的⼗六进制数：
>
> - 前4个字节为当前时间戳
> - 接下来3个字节的机器ID
> - 接下来的2个字节中MongoDB的服务进程id
> - 最后3个字节是简单的增量值

### 3.3 插入数据

- db.collect_name.insert(document)

  ```shell
  db.collect_name.insert({_id:"20170101",name:'gj',gender:1})
  ```

- db.集合collect_name.save(document)：通过`_id`判断具体操作。如果⽂档的`_id`已经存在则修改， 如果⽂档的`_id`不存在则添加

### 3.4 更新数据

- db.集合名称.update(<query> ,<update>,{multi: <boolean>})
  - query：查询条件
  - update：更新操作符
  - multi：可选， 默认是false，表示只更新找到的第⼀条记录， 值为true表示把满⾜条件的⽂档全部更新

```shell
db.collect_name.update({name:'hr'},{name:'mnc'})   // 更新一条数据
db.collect_name.update({name:'hr'},{$set:{name:'hys'}})  // 更新一条数据，只更新name
db.collect_name.update({},{$set:{gender:0}},{multi:true})   
// 更新全部数据，只更新gender，multi update only works with $ operators
```

### 3.5 删除数据

- db.集合名称.remove(<query>,{justOne: <boolean>})
  - query：可选，删除的⽂档的条件
  - justOne：可选， 如果设为true或1， 则只删除⼀条， 默认false， 表示删除多条

## 四、MongoDB数据查询

### 4.1 简单查询

- db.collect_name.find()
- db.collect_name.findOne()：查询，只返回第⼀个
- db.collect_name.pretty()： 将结果格式化

### 4.2 比较运算符

- 等于： 默认是等于判断， 没有运算符
- ⼩于：$lt （less than）
- ⼩于等于：$lte （less than equal）
- ⼤于：$gt （greater than）
- ⼤于等于：$gte
- 不等于：$ne

```
db.collect_name.find({age:{$gte:18}})
```

### 4.3 逻辑运算符

- and：在json中写多个条件即可

  ```
  db.collect_name.find({age:{$gte:18},gender:true})  // 查询年龄⼤于或等于18， 并且性别为true的学⽣
  ```

- or：使⽤$or， 值为数组， 数组中每个元素为json

  ```
  db.collect_name.find({$or:[{age:{$gt:18}},{gender:false}]})  // 查询年龄⼤于18， 或性别为false的学⽣
  ```

### 4.4 范围运算符

- 使⽤"$in"， "$nin" 判断是否在某个范围内

  ```
  db.collect_name.find({age:{$in:[18,28]}})  // 查询年龄为18、 28的学⽣
  ```

### 4.5 使用正则表达式

- 使⽤//或$regex编写正则表达式

  ```
  db.collect_name.find({name:/^⻩/})  // 查询姓⻩的学⽣
  db.collect_name.find({name:{$regex:'^⻩'}})  // 查询姓⻩的学⽣
  ```

### 4.6 分组和跳过

- db.collect_name.find().limit(num)： ⽤于读取指定数量的⽂档

  ```
  db.stu.find().limit(2)  // 查询2条学⽣信息
  ```

- db.集合collect_name.find().skip(num)： ⽤于跳过指定数量的⽂档

  ```
  db.stu.find().skip(2)  
  ```

- 同时使用：`db.stu.find().limit(4).skip(5)`  或  `db.stu.find().skip(5).limit(4)`

### 4.7 自定义查询

- 使⽤$where后⾯写⼀个函数， 返回满⾜条件的数据

  ```
  // 查询年龄⼤于30的学⽣
  db.stu.find({
  	$where:function() {
  		return this.age>30;  
  	}
  })
  ```

### 4.8 投影

- db.collect_name.find({},{ 字段名称:1 ,...})：在查询到的返回结果中， 只选择必要的字段

  - 参数为字段与值， 值为1表示显示， 不写表示不显示；对于_id列默认是显示的， 如果不显示需要明确设置为0

  ```
  db.collect_name.find({},{_id:0,name:1,gender:1})
  ```

### 4.9 排序

- db.collect_name.find().sort({字段:1,...})：⽅法sort()， ⽤于对集进⾏排序

  - 参数1为升序排列
  - 参数-1为降序排列

  ```
  db.collect_name.find().sort({gender:-1,age:1})  // 根据性别降序， 再根据年龄升序
  ```

### 4.10 统计个数

- db.collect_name.find({条件}).count()：⽅法count()⽤于统计结果集中⽂档条数

  ```
  db.stu.find({gender:true}).count()
  ```

- db.collect_name.count({条件})

  ```
  db.stu.count({age:{$gt:20},gender:true})
  ```

### 4.11 消除重复

- db.collect_name.distinct('去重字段',{条件})

  ```
  db.stu.distinct('hometown',{age:{$gt:18}})
  ```

## 五、MongoDB聚合

聚合(aggregate)是基于数据处理的聚合管道，每个文档通过一个由多个阶段（stage）组成的管道，可以对每个阶段的管道进行分组、过滤等功能，然后经过一系列的处理，输出相应的结果。 

- db.collect_name.aggregate({管道:{表达式}})

### 5.1 常用管道

在mongodb中，⽂档处理完毕后， 通过管道进⾏下⼀次处理

常用管道如下：

- $group： 将集合中的⽂档分组， 可⽤于统计结果
- $match： 过滤数据， 只输出符合条件的⽂档
- $project： 修改输⼊⽂档的结构， 如重命名、 增加、 删除字段、 创建计算结果
- $sort： 将输⼊⽂档排序后输出
- $limit： 限制聚合管道返回的⽂档数
- $skip： 跳过指定数量的⽂档， 并返回余下的⽂档
- $unwind： 将数组类型的字段进⾏拆分

### 5.2 常用表达式

- 表达式:'$列名'：处理输⼊⽂档并输出

常⽤表达式:

- $sum： 计算总和， $sum:1 表示以⼀倍计数
- $avg： 计算平均值
- $min： 获取最⼩值
- $max： 获取最⼤值
- $push： 在结果⽂档中插⼊值到⼀个数组中
- $first： 根据资源⽂档的排序获取第⼀个⽂档数据
- $last： 根据资源⽂档的排序获取最后⼀个⽂档数据

### 5.3 $group使用

$group：将集合中的文档分组，可用于统计结果；`_id`表示分组的依据，使用某个字段的格式为'$字段'

```shell
// 统计男生、女生的总人数
db.stu.aggregate({
	$group:{
		_id:'$gender',
		counter:{
			$sum:1
		}
	}
})

// 求学生总人数、平均年龄
db.stu.aggregate({
	$gorup:{
		_id:null,
		counter:{$sum:1},
		avgAge:{$avg:'$age'}
	}
})

// 统计不同性别的学生姓名，使用$$ROOT可以将文档内容加入到结果集的数据中
db.stu.aggregate({
	$group:{
		_id:'$gender',
		name:{'$push':'$$ROOT'}
	}
})
```

### 5.4 $match使用

$match用于过滤数据，只输出符合条件的文档

```
// 查询年龄大于20的男生、女生人数
db.stu.aggregate(
	{$match:{age:{$gt:20}}},
	{$group:{_id:'$gender', counter:{$sum:1}}}
)
```

### 5.5 $project使用

$project用来修改输入文档的结构，如重命名、增加、删除字段、创建计算结果。

```
// 查询男生、女生人数，输出人数
db.stu.aggregate(
	{$group:{_id:'$gender', counter:{$sum:1}}},
	{$project:{_id:0, counter:1}}
)
```

### 5.6 $sort使用

$sort将输入文档排序后输出

```
// 查询男生、女生人数，按人数排序
db.stu.aggregate(
	{$group:{_id:'$gender', counter:{$sum:1}}},
	{$sort:{counter:-1}}
)
```

### 5.7 $limit和$skip

$limit：限制聚合管道返回的文档数；$skip：跳过指定数量的文档，并返回余下的文档。

```
// 统计男生、女生人数，按人数升序，取第二条数据
db.stu.aggregate(
	{$group:{_id:'$gender', counter:{$sum:1}}},
	{$sort:{counter:1}},
	{$skip:1},
	{$limit:1}
)
```

### 5.8 $unwind

$unwind：将文档中的某一个数组类型字段拆分成多条，每条包含数据中的一个值。

```
db.t2.insert({_id:1,item:'t-shirt',size:['S','M','L']})
db.t2.aggregate({$unwind:'$size'})  // 拆分size
```

```
db.t3.aggregate({$unwind:{path:'$字段名称', preserveNullAndEmptyArrays:<boolean>}})
// 属性preserveNullAndEmptyArrays的值为false时丢弃属性值为空的文档，为true时保留属性值为空的文档。
```

## 六、索引和备份

建立索引可以极大地提升查询速度。

- db.t1.ensureIndex({"name":1})：在name字段上建立索引，1为升序，-1为降序
- db.t1.ensureIndex({"name":1},{"unique":true})：创建唯一索引
- db.t1.ensureIndex({"name":1},{"unique":true,"dropDups":true}) ：创建唯一索引并消除重复
- db.t1.ensureIndex({name:1,age:1})：建立联合索引
- db.t1.getIndexes()：查看当前集合的所有索引
- db.t1.dropIndex('索引名称')：删除索引

## 七、数据的备份和恢复

- mongodump -h dbhost -d dbname -o dbdirectory：备份数据库

  - -h： 服务器地址， 也可以指定端⼝号
  - -d： 需要备份的数据库名称
  - -o： 备份的数据存放位置， 此⽬录中存放着备份出来的数据

  ```
  mongodump -h 192.168.196.128:27017 -d test1 -o ~/Desktop/test1bak
  ```

- mongorestore -h dbhost -d dbname --dir dbdirectory

  - -h： 服务器地址
  - -d： 需要恢复的数据库实例
  - --dir： 备份数据所在位置

  ```
  mongorestore -h 192.168.196.128:27017 -d test2 --dir ~/Desktop/test1bak/test1
  ```

## 八、MongoDB与Python交互

```python
from pymongo import MongoClient

client = MongoClient(host='10.81.1.122', port=27017)
collection = client['test']['t251']

# 插入一条数据，返回值的ID
ret1 = collection.insert({'name':"xiaowangaaaa", "age":10})

# 插入多条数据
data_list = [{"name":"test{}".format(i)} for i in range(10)]
collection.insert_many(data_list)

# 查询一个记录
ret2 = collection.find_one({"name":"xiaowangaaaa"})
print(ret2)

# 查询多个记录，只能读取一次
ret3 = collection.find({"name":"xiaowangaaaa"})
for i in ret3:
	print(i)

# 更新记录
collection.update_one({"name":"test0"}, {"$set":{"name":"new_test0"}})
collection.update_many({"name":"xiaowangaaaa"}, {"$set":{"name":"new_xiaowangaaaa"}})

# 删除记录
collection.delete_one({"name":"new_test0"})
collection.delete_many({"name":"new_xiaowangaaaa"})
```



