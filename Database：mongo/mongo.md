一、开篇

## 1. 介绍

非关系型数据库，non-relational，Not Only SQL

### 优势

- **易扩展**：Nosql数据库种类繁多，但是共同点都是去掉关系型数据库的关系性特性。数据之间无关系，这样就非常容易扩展
- **大数据量，高性能**：NoSQL数据库都有非常高的读写性能，尤其在大数据量下，同样表现优秀。这得益于他的无关系性，数据库结构简单
- **灵活的数据模型**：NoSQL无需事先为要存储的数据建立字段，随时可以存储自定义的格式数据。而在关系数据库中，需要增加字段。

### 启动mongo：

```shell
mongod --config /usr/local/mongodb/etc/mongod.conf
mongod
mongo
```

### 备份与恢复





# 二、MongoDB 操作

## 1. 数据库的操作

### 1.1 增

```shell
use database_name
```

如果数据库不存在,则创建数据库，否则切换到指定数据库。

创建的数据库并不会马上出现在数据库列表中，只有当数据库中有数据时，才会显示。

```shell
> use demo
switched to db demo
> db.demo.insert({'name':'an'})
WriteResult({ "nInserted" : 1 })

> show dbs
admin   0.000GB
config  0.000GB
demo    0.000GB
local   0.000GB
```

MongoDB中的默认数据库为test，如果没有创建新的数据库，集合将放到test数据库中(一进入mongodb，敲db查看所在数据库，即为test)

### 1.2 删

```shell
db.dropDatabase()
```

db.dropDatabases() 此处的db相当于python中的self，use 了哪个数据库，db就代表谁，

### 1.3 查



```shell
# 如果你想查看所有数据库：
> show dbs
admin   0.000GB
config  0.000GB
local   0.000GB

> show databases
admin   0.000GB
config  0.000GB
local   0.000GB

# 查看你所在的数据库
> db
demo
```



## 2. 集合的操作

### 2.0 创建集合

向不存在的集合中第一次增加数据时，集合会被创造出来

```shell
> show tables
demo

> db.non_exist_col.insert({'name':'an'})
WriteResult({ "nInserted" : 1 })

> show tables
demo
non_exist_col
```

或者

手动创建集合

```shell
db.createCollection(name,options)
db.createCollection('firstcol')

db.createCollection('firstcol',{capped:true, size:10})
```

> | 字段        | 类型 | 描述                                                         |
> | :---------- | :--- | :----------------------------------------------------------- |
> | capped      | 布尔 | （可选）如果为 true，则创建固定集合。固定集合是指有着固定大小的集合，当达到最大值时，它会自动覆盖最早的文档。 **当该值为 true 时，必须指定 size 参数。** |
> | autoIndexId | 布尔 | （可选）如为 true，自动在 _id 字段创建索引。默认为 false。   |
> | size        | 数值 | （可选）为固定集合指定一个最大值，即字节数。 **如果 capped 为 true，也需要指定该字段。** |
> | max         | 数值 | （可选）指定固定集合中包含文档的最大数量。                   |



### 2.1 增

```shell
db.collection_name.insert(document)
db.collection_name.insert({name:'an',gender:'male'})
db.collection_name.insert({_id:'19991102',name:'an',gender:'male'})
# 插入文档时，不指定 _id 参数，MongoDB会为文档分配一个唯一的ObjectId

# 插入多个
db.collection_name.insert([{_id:'19991102',name:'an',gender:'male'},{_id:'1999',name:'an1',gender:'male'}])
```



### 2.2 删除

1、删除集合

```shell
db.collection_name.drop()
```

2、remove 删除文档

```shell
db.collection_name.remove(<query>,{justOne:<boolean>})
db.collection.remove(
   <query>,
   {
     justOne: <boolean>,
     writeConcern: <document>
   }
)
```

> **query** :（可选）删除的文档的条件。
>
> **justOne** : （可选）如果设为 true 或 1，则只删除一个文档，如果不设置该参数，或使用默认值 false，则删除所有匹配条件的文档。
>
> **writeConcern** :（可选）抛出异常的级别。

3、delete：删除文档

```shell
db.collect_name.deleteMany({}) # 如删除集合下全部文档
db.collect_name.deleteMany({ status : "A" }) # 删除 status 等于 A 的全部文档
db.collect_name.deleteOne( { status: "D" } ) # 删除 status 等于 D 的一个文档
```



### 2.3 改

1、**save()**：如果文档存在则修改，如果文档_id不存在则添加

```shell
db.collection_name.save(document)
```

2、**update()**：更新内容

```shell
db.collection.update(
   <query>,
   <update>,
   {
     upsert: <boolean>,
     multi: <boolean>,
     writeConcern: <document>
   }
)
```

> **query** : update的查询条件，类似sql update查询内where后面的。
>
> **update** : update的对象和一些更新的操作符（如$,$inc...）等，也可以理解为sql update查询内set后面的
>
> **upsert** : 可选，这个参数的意思是，如果不存在update的记录，是否插入objNew,true为插入，默认是false，不插入。
>
> **multi** : 可选，mongodb 默认是false,只更新找到的第一条记录，如果这个参数为true,就把按条件查出来多条记录全部更新。
>
> **writeConcern** :可选，抛出异常的级别。

```shell
db.collection.update({name:'an'},{name:'anwenxuan'}) # 更新一条，使用<update>的内容整个替换掉<query>的结果
db.collection.update({name:'an'},{$set:{name:'anwenxuan'}}) # 更新一条
db.collection.update({},{$set:{gender:'male'}},{multi:true}) # 更新全部，multi操作必须配合 $
```



### 2.4 查看

**1、集合**

```shell
# 查看所有集合
> show collections
demo
runoob

> show tables
demo
runoob

# 查看集合详细信息
> db.demo.find()
{ "_id" : ObjectId("617ff058330c5f0214377b02"), "name" : "an" }
```



**2、数据**

```shell
db.collection_name_find(query, projection)    # 查询所有
db.collection_name_findOne(query, projection)    # 查询一条
db.collection_name_find(query, projection).pretty()    # 查询所有并格式化输出
```

> **query** ：可选，使用查询操作符指定查询条件
>
> **projection** ：可选，使用[投影](#touying)操作符指定返回的键。查询时返回文档中所有键值， 只需省略该参数即可（默认省略）。



**3、范围运算符**

- (>) 大于 - $gt		(greater than)
- (<) 小于 - $lt		 (less than)
- (>=) 大于等于 - $gte	(greater than equal)
- (<= ) 小于等于 - $lte	
- ( != ) 不等于 - $ne

```shell
# 在stu集合中查找age小于等于18的
db.stu.find({age:{$gte:18}})
```



**4、逻辑运算符**

- and：在json中写多个条件即可

- or：使用$or，值为数组，数组中每个元素为json

```shell
# 查询年龄大于18，且性别为man的
db.stu.find({age:{$gte:18},gender:'man'})

# 查询年龄大于18，性别为man的学生
db.stu.find(
	{
		$or:[{age:{$gt:18}},{gender:'man'}]
	}
)

# 查询年龄大于18 或 性别为man，并且姓名是‘an’
db.stu.find(
	{
		$or:[{age:{$gt:18}},{gender:'man'}],name:'an'
	}
)
```

**5、使用正则表达式**

使用// 或 $regex编写正则表达式

- https://www.runoob.com/mongodb/mongodb-regular-expression.html

  

**6、limit 和 skip**

- limit()：用于读取指定数量的文档
- skip()：用于跳过指定数量的文档

```shell
# 查询两条学生信息
db.stu.find().limit(2)

# 查询第二条文档
db.col.find().limit(1).skip(1)
```



**7、统计个数**

count()：计算统计结果中文档条数

```shell
db.collect_name.find(content).count()
db.
```



**8、去重**

distinct()    

db.collect_name.distinct('去重字段',{条件})

```shell
# 年龄大于18的人的家乡会在哪
db.collect_name.distinct('hometown',{age:{$gt:18}})
```



**9、排序**

sort()：对集合排序

db.collect_name.find(content).sort({字段:1,...})

> 参数 1 为升序排列
>
> 参数 -1 为降序排列

```shell
# 根据性别降序，再根据年龄升序
db.stu.find.sort({gender:-1,age:1})
```





# 三、数据类型、组成

文档的数据结构和 JSON 基本一样

> **Object ID**：文档ID
>
> **String**：必须是有效的UTF-8
>
> **Boolean**：存储一个布尔值，true或者false
>
> **Integer**：整数可以是32火或者64位，取决于服务器
>
> **Double**：存储浮点值
>
> **Arrays**：数组或者列表，多个值存储到一个键
>
> **Object**：用于嵌入式的文档，一个值即为一个文档
>
> **Timestamp**：时间戳，表示从1970-1-1到现在的总秒数
>
> **Date**：存储当前日期或者时间的UNIX时间格式

- 创建日期代码

```shell
> new Date('2021-11-01')
ISODate("2021-11-01T00:00:00Z")
```

- 每个文档都有一个属性，为 **_id**，保证每个文档的唯一性
- 可以自己设置 _id 插入文档，如果没有提供 ，那么MongoDB会为每个文档提供一个独特的 _id，类型为objectID
- objectID是一个12字节的十六进制数
  - 前 4 个字节为当前时间戳
  - 接下来 3 个为机器的id
  - 接下来的 2 个为MongoDB的服务进程id
  - 最后 3 个是简单的增量值



# 四、聚合查询

## 1、<a id="touying">投影</a>

在查询到的返回结果中，只选择必要的字段

```shell
db.collection_name.find(
	{}, # 匹配所有数据
	{字段名称,1}
)

db.stu.find({},{_id:0,name:1,gender:1})
```

参数为字段值与值，值为1表示显示，不写则默认不显示，除了_id

> 特殊：对于 _id 列默认是显示的，如果不显示需要明确设置为0



## 2、聚合表达式以及常用管道

1、集合表达式

|  表达式   | 描述                                                         |
| :-------: | :----------------------------------------------------------- |
|   $sum    | 计算总和。    {$group:{_id:'$gender',counter:{**$sum:1**}}}   即对前面$group 匹配到的进行计算 |
|   $avg    | 计算平均值                                                   |
|   $min    | 获取集合中所有文档对应值得最小值。                           |
|   $max    | 获取集合中所有文档对应值得最大值。                           |
|   $push   | 将值加入一个数组中，不会判断是否有重复的值。                 |
| $addToSet | 将值加入一个数组中，会判断是否有重复的值，若相同的值在数组中已经存在了，则不加入。 |
|  $first   | 根据资源文档的排序获取第一个文档数据。                       |
|   $last   | 根据资源文档的排序获取最后一个文档数据                       |

2、常用管道

管道在Unix和Linux中一般用于将当前命令的输出结果作为下一个命令的参数。

MongoDB的聚合管道将MongoDB文档在一个管道处理完毕后将结果传递给下一个管道处理。管道操作是可以重复的。

表达式：处理输入文档并输出。表达式是无状态的，只能用于计算当前聚合管道的文档，不能处理其它的文档。

- $project：修改输入文档的结构。可以用来重命名、增加或删除域，也可以用于创建计算结果以及嵌套文档。
- $match：用于过滤数据，只输出符合条件的文档。$match使用MongoDB的标准查询操作。
- $limit：用来限制MongoDB聚合管道返回的文档数。
- $skip：在聚合管道中跳过指定数量的文档，并返回余下的文档。
- $unwind：将文档中的某一个数组类型字段拆分成多条，每条包含数组中的一个值。
- $group：将集合中的文档分组，可用于统计结果。
- $sort：将输入文档排序后输出。
- $geoNear：输出接近某一地理位置的有序文档。



## 3、聚合查询

**aggregate() ：**

MongoDB中聚合的方法使用aggregate()。

```
>db.COLLECTION_NAME.aggregate(AGGREGATE_OPERATION)
```



- $group

  ```shell
  # 统计男生女生的总人数
  db.stu.aggregate(
  	{
  		$group:
  			{
  				_id:'$gender',
  				counter:{$sum:1}
  			}	
  	}
  )
  
  # Group by null 将集合中所有文档分为一组
  # 求学生总人数、平均年龄
  db.stu.aggregate(
  	{
  		$group:
  			{
  				_id:null,
  				counter:{$sum:1},
  				avgAge:{$avg:'$age'} 
  				# 此处的counter和avgAge均为一个变量值，将计算后的结果复制给变量值
  				# $avg 为计算平均值的聚合表达式
  			}	
  	}
  )
  
  # 同时给多个键进行分组
  db.demo.find()
  { "_id" : ObjectId("61813e86083eecfa69ddbf72"), "age" : 22, "gender" : "male", "color" : "yellow", "country" : "China" }
  { "_id" : ObjectId("61813e93083eecfa69ddbf73"), "age" : 24, "gender" : "male", "color" : "yellow", "country" : "China" }
  { "_id" : ObjectId("61813e9d083eecfa69ddbf74"), "age" : 24, "gender" : "male", "color" : "white", "country" : "US" }
  { "_id" : ObjectId("61813ea0083eecfa69ddbf75"), "age" : 24, "gender" : "male", "color" : "white", "country" : "UK" }
  { "_id" : ObjectId("61813ea8083eecfa69ddbf76"), "age" : 24, "gender" : "male", "color" : "black", "country" : "US" }
  { "_id" : ObjectId("61813eb0083eecfa69ddbf77"), "age" : 24, "gender" : "female", "color" : "black", "country" : "US" }
  
  > db.demo.aggregate({$group:{_id:{country:"$country",color:"$color"}}} )
  { "_id" : { "country" : "UK", "color" : "white" } }
  { "_id" : { "country" : "US", "color" : "white" } }
  { "_id" : { "country" : "US", "color" : "black" } }
  { "_id" : { "country" : "China", "color" : "yellow" } }
  ```

  

  $group 对应的字典中有几个键，结果就有几个键

  分组需要放到 _id 后面去

  取不同的字段的值时要使用$，'$gender'，'$age'

  取字典嵌套的字典中的值的时候要使用 . $_id.xxxx

  

- $project

  ```shell
  # 查询学生的姓名年龄
  db.stu.aggregate(
  	{$project:{_id:0,name:1,age:1}}
  )
  
  # 查询男生、女生人数，输出人数
  db.stu.aggregate(
  	{$group:{_id:'$gender',counter:{$sum:1}}},
  	{{$project:{_id:0,counter:1}}}
  )
  ```

  > 为什么能修改输入?
  >
  > ​	如果$project语句下还有其他的语句，其他语句的输入即为$project 的输出



- $match

  ```shell
  # 查询年龄大于20的学生
  db.stu.aggregate(
  	{$match:{age:{$gt:20}}}	
  )
  
  # 查询年龄大于20的男生、女生人数
  db.stu.aggregate(
  	{$match:{age:{$gt:20}}},
      {$match:{_id:'$gender',count:{$sum:1}}}
      {$project:{_id:0,gender:"$_id",count:1}}
  )
  
  /**
  	选择年龄大于20的学生 -> 
  	以$gender进行分组，计算各个组的人数 -> 
  	输出时 _id 为0 不显示。增加变量 gender ，值为$_id(上一步的_id的结果)，count为1 ，输出时显示
  */
  ```

  

# mongo 与 Python 开发

1、mongo返回的数据包含 ObjectId，fastapi无法处理

2、mongo 插入会插入到最后



motor  异步库



$mongo 命令执行后可以看做是一个shell
