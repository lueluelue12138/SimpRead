# 2、MongoDB快速入门

## 2.1、MongoDB简介

![img](https://cdn.nlark.com/yuque/0/2022/svg/27683667/1663075345721-5773cbfa-8294-4c79-a869-7efb121c6b92.svg)

- MongoDB是一个基于分布式文件存储的数据库，由C++语言编写。
- MongoDB是一个介于关系数据库和非关系数据库之间的产品，是非关系数据库当中功能最丰富，最像关系数据库的，它支持的数据结构非常松散，是类似json的bson格式，因此可以存储比较复杂的数据类型。
- MongoDB的查询语言非常强大，其语法有点类似于面向对象的查询语言，几乎可以实现了类似关系数据库单表查询的绝大部分功能（可以通过聚合的方式实现多表查询），而且还支持对数据建立索引。

官网：

https://www.mongodb.com

## 2.2、部署安装

推荐使用Docker部署安装MongoDB，在101机器中已经完成了安装。

```shell
docker run -d \
--name mongodb \
-p 27017:27017 \
--restart=always \
-v mongodb:/data/db \
-e MONGO_INITDB_ROOT_USERNAME=sl \
-e MONGO_INITDB_ROOT_PASSWORD=123321 \
mongo:4.4

#进入容器进行设置
docker exec -it mongodb /bin/bash
#进行认证
mongo -u "sl" -p "123321" --authenticationDatabase "admin"

#测试命令，查看已有数据库
> show dbs
admin   0.000GB
config  0.000GB
local   0.000GB
```

## 2.3、基本概念

为了更好的理解，下面与SQL中的概念进行对比：

| **SQL术语/概念** | **MongoDB术语/概念** | **解释/说明**                       |
| ---------------- | -------------------- | ----------------------------------- |
| database         | database             | 数据库                              |
| table            | collection           | 数据库表/集合                       |
| row              | document             | 数据记录行/文档                     |
| column           | field                | 数据字段/域                         |
| index            | index                | 索引                                |
| table joins      |                      | 表连接,MongoDB不支持                |
| primary key      | primary key          | 主键,MongoDB自动将_id字段设置为主键 |

![img](https://cdn.nlark.com/yuque/0/2022/png/27683667/1663122725512-37729247-c1a4-4a3f-a5df-e48a6c7b4993.png?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_24%2Ctext_6buR6ams56iL5bqP5ZGYwrfnoJTnqbbpmaI%3D%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10)

## 2.4、MongoDB基本操作

### 2.4.1、数据库以及表的操作

```shell
#查看所有的数据库
> show dbs
admin   0.000GB
config  0.000GB
local   0.000GB

#通过use关键字切换数据库
> use admin
switched to db admin

#创建数据库
#说明：在MongoDB中，数据库是自动创建的，通过use切换到新数据库中，进行插入数据即可自动创建数据库
> use testdb
switched to db testdb
> show dbs #并没有创建数据库
admin   0.000GB
config  0.000GB
local   0.000GB
> db.user.insert({id:1,name:'zhangsan'})  #插入数据
WriteResult({ "nInserted" : 1 })
> show dbs
admin   0.000GB
config  0.000GB
local   0.000GB
testdb  0.000GB #数据库自动创建

#查看表
> show tables
user
> show collections
user
> 

#删除集合（表）
> db.user.drop()
true  #如果成功删除选定集合，则 drop() 方法返回 true，否则返回 false。

#删除数据库
> use testdb #先切换到要删除的数据库中
switched to db testdb
> db.dropDatabase()  #删除数据库
{ "dropped" : "testdb", "ok" : 1 }
> show dbs
admin   0.000GB
config  0.000GB
local   0.000GB
```

### 2.4.2、新增数据

在MongoDB中，存储的文档结构是一种类似于json的结构，称之为bson（全称为：Binary JSON）。

```shell
#插入数据

#语法：db.COLLECTION_NAME.insert(document)
> db.user.insert({id:1,username:'zhangsan',age:20})
WriteResult({ "nInserted" : 1 })

> db.user.save({id:2,username:'lisi',age:25})
WriteResult({ "nInserted" : 1 })

> db.user.find()  #查询数据
{ "_id" : ObjectId("5c08c0024b318926e0c1f6dc"), "id" : 1, "username" : "zhangsan", "age" : 20 }
{ "_id" : ObjectId("5c08c0134b318926e0c1f6dd"), "id" : 2, "username" : "lisi", "age" : 25 }
```

- _id 是集合中文档的主键，用于区分文档（记录），_id自动编入索引。
- 默认情况下，_id 字段的类型为 ObjectID，是 MongoDB 的 BSON 类型之一，如果需要，用户还可以将 _id 覆盖为 ObjectID 以外的其他内容。
- ObjectID 长度为 12 字节，由几个 2-4 字节的链组成。每个链代表并指定文档身份的具体内容。以下的值构成了完整的 12 字节组合：

- 一个 4 字节的值，表示自 Unix 纪元以来的秒数
- 一个 3 字节的机器标识符
- 一个 2 字节的进程 ID
- 一个 3字节的计数器，以随机值开始

### 2.4.3、更新数据

update() 方法用于更新已存在的文档。语法格式如下：

```shell
db.collection.update(
   <query>,
   <update>,
   [
     upsert: <boolean>,
     multi: <boolean>,
     writeConcern: <document>
   ]
)
```

**参数说明：**

- **query** : update的查询条件，类似sql update查询内where后面的。
- **update** : update的对象和一些更新的操作符（如![img](https://g.yuque.com/gr/latex?%2C)inc...）等，也可以理解为sql update查询内set后面的
- **upsert** : 可选，这个参数的意思是，如果不存在update的记录，是否插入objNew,true为插入，默认是false，不插入。
- **multi** : 可选，mongodb 默认是false,只更新找到的第一条记录，如果这个参数为true,就把按条件查出来多条记录全部更新。
- **writeConcern** :可选，抛出异常的级别。

```shell
> db.user.find()
{ "_id" : ObjectId("5c08c0024b318926e0c1f6dc"), "id" : 1, "username" : "zhangsan", "age" : 20 }
{ "_id" : ObjectId("5c08c0134b318926e0c1f6dd"), "id" : 2, "username" : "lisi", "age" : 25 }

> db.user.update({id:1},{$set:{age:22}}) #更新数据

WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })

> db.user.find()
{ "_id" : ObjectId("5c08c0024b318926e0c1f6dc"), "id" : 1, "username" : "zhangsan", "age" : 22 }
{ "_id" : ObjectId("5c08c0134b318926e0c1f6dd"), "id" : 2, "username" : "lisi", "age" : 25 }

#注意：如果这样写，会删除掉其他的字段
> db.user.update({id:1},{age:25})
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
> db.user.find()
{ "_id" : ObjectId("5c08c0024b318926e0c1f6dc"), "age" : 25 }
{ "_id" : ObjectId("5c08c0134b318926e0c1f6dd"), "id" : 2, "username" : "lisi", "age" : 25 }

#更新不存在的字段，会新增字段
> db.user.update({id:2},{$set:{sex:1}}) #更新数据
> db.user.find()
{ "_id" : ObjectId("5c08c0024b318926e0c1f6dc"), "age" : 25 }
{ "_id" : ObjectId("5c08c0134b318926e0c1f6dd"), "id" : 2, "username" : "lisi", "age" : 25, "sex" : 1 }

#更新不存在的数据，默认不会新增数据
> db.user.update({id:3},{$set:{sex:1}})
WriteResult({ "nMatched" : 0, "nUpserted" : 0, "nModified" : 0 })
> db.user.find()
{ "_id" : ObjectId("5c08c0024b318926e0c1f6dc"), "age" : 25 }
{ "_id" : ObjectId("5c08c0134b318926e0c1f6dd"), "id" : 2, "username" : "lisi", "age" : 25, "sex" : 1 }

#如果设置第一个参数为true，就是新增数据
> db.user.update({id:3},{$set:{sex:1}},true)
WriteResult({
	"nMatched" : 0,
	"nUpserted" : 1,
	"nModified" : 0,
	"_id" : ObjectId("5c08cb281418d073246bc642")
})
> db.user.find()
{ "_id" : ObjectId("5c08c0024b318926e0c1f6dc"), "age" : 25 }
{ "_id" : ObjectId("5c08c0134b318926e0c1f6dd"), "id" : 2, "username" : "lisi", "age" : 25, "sex" : 1 }
{ "_id" : ObjectId("5c08cb281418d073246bc642"), "id" : 3, "sex" : 1 }
```

### 2.4.4、删除数据

通过remove()方法进行删除数据，语法如下：

```shell
db.collection.remove(
   <query>,
   {
     justOne: <boolean>,
     writeConcern: <document>
   }
)
```

**参数说明：**

- **query** :（可选）删除的文档的条件。
- **justOne** : （可选）如果设为 true 或 1，则只删除一个文档，如果不设置该参数，或使用默认值 false，则删除所有匹配条件的文档。
- **writeConcern** :（可选）抛出异常的级别。

```shell
> db.user.remove({age:25})
WriteResult({ "nRemoved" : 2 })  #删除了2条数据

#插入4条测试数据
db.user.insert({id:1,username:'zhangsan',age:20})
db.user.insert({id:2,username:'lisi',age:21})
db.user.insert({id:3,username:'wangwu',age:22})
db.user.insert({id:4,username:'zhaoliu',age:22})

> db.user.remove({age:22},true)
WriteResult({ "nRemoved" : 1 })  #删除了1条数据

#删除所有数据
> db.user.remove({})

#说明：为了简化操作，官方推荐使用deleteOne()与deleteMany()进行删除数据操作。
db.user.deleteOne({id:1})
db.user.deleteMany({})  #删除所有数据
```

### 2.4.5、查询数据

MongoDB 查询数据的语法格式为：`db.user.find([query],[fields])`

- **query** ：可选，使用查询操作符指定查询条件
- **fields** ：可选，使用投影操作符指定返回的键。查询时返回文档中所有键值， 只需省略该参数即可（默认省略）。

如果你需要以易读的方式来读取数据，可以使用 pretty() 方法，语法格式为：`db.col.find().pretty()`

条件查询：

| **操作**   | **格式**               | **范例**                                  | **RDBMS中的类似语句**   |
| ---------- | ---------------------- | ----------------------------------------- | ----------------------- |
| 等于       | {<key>:<value>}        | db.col.find({"by":"黑马程序员"}).pretty() | where by = '黑马程序员' |
| 小于       | {<key>:{$lt:<value>}}  | db.col.find({"likes":{$lt:50}}).pretty()  | where likes < 50        |
| 小于或等于 | {<key>:{$lte:<value>}} | db.col.find({"likes":{$lte:50}}).pretty() | where likes <= 50       |
| 大于       | {<key>:{$gt:<value>}}  | db.col.find({"likes":{$gt:50}}).pretty()  | where likes > 50        |
| 大于或等于 | {<key>:{$gte:<value>}} | db.col.find({"likes":{$gte:50}}).pretty() | where likes >= 50       |
| 不等于     | {<key>:{$ne:<value>}}  | db.col.find({"likes":{$ne:50}}).pretty()  | where likes != 50       |

```shell
#插入测试数据
db.user.insert({id:1,username:'zhangsan',age:20})
db.user.insert({id:2,username:'lisi',age:21})
db.user.insert({id:3,username:'wangwu',age:22})
db.user.insert({id:4,username:'zhaoliu',age:22})

db.user.find()  #查询全部数据
db.user.find({},{id:1,username:1})  #只查询id与username字段
db.user.find().count()  #查询数据条数
db.user.find({id:1}) #查询id为1的数据
db.user.find({age:{$lte:21}}) #查询小于等于21的数据
db.user.find({age:{$lte:21}, id:{$gte:2}}) #and查询，age小于等于21并且id大于等于2
db.user.find({$or:[{id:1},{id:2}]}) #查询id=1 or id=2

#分页查询：Skip()跳过几条，limit()查询条数
db.user.find().limit(2).skip(1)  #跳过1条数据，查询2条数据
db.user.find().sort({id:-1}) #按照age倒序排序，-1为倒序，1为正序
```

## 2.5、索引

索引通常能够极大的提高查询的效率，如果没有索引，MongoDB在读取数据时必须扫描集合中的每个文件并选取那些符合查询条件的记录。

MongoDB支持的索引类型有：

- **单字段索引（Single Field）**

- 支持所有数据类型中的单个字段索引

- **复合索引（Compound Index）**

- 基于多个字段的索引，创建复合索引时要注意字段顺序与索引方向

- **多键索引（Multikey indexes）**

- 针对属性包含数组数据的情况，MongoDB支持针对数组中每一个element创建索引。

- **全文索引（Text Index）**

- 支持任意属性值为string或string数组元素的索引查询。
- 注意：一个集合仅支持最多一个Text Index，中文分词不理想，推荐Elasticsearch。

- **地理空间索引（Geospatial Index）**

- 2dsphere索引，用于存储和查找球面上的点
- 2d索引，用于存储和查找平面上的点

- **哈希索引（Hashed Index）**

- 针对属性的哈希值进行索引查询，当要使用Hashed index时，MongoDB能够自动的计算hash值，无需程序计算hash值。
- hash index仅支持等于查询，不支持范围查询。

我们重点需要掌握的是【单字段索引】、【2dsphere索引】。

```shell
#单字段索引，1表示升序创建索引，-1表示降序创建索引
db.集合名.createIndex({"字段名":排序方式})

#2dsphere索引
db.集合名.createIndex({"字段名":"2dsphere"})

#示例，创建user集合，其中username和loc字段设置索引
db.user.createIndex({"username":1})
db.user.createIndex({"loc":"2dsphere"})

db.user.insert({id:1,username:'zhangsan',age:20,loc:{type:"Point",coordinates:[116.343847,40.060539]}})
db.user.insert({id:2,username:'lisi',age:22,loc:{type:"Point",coordinates:[121.612112,31.034633]}})

#查看索引
db.user.getIndexes()
#查看索引大小，单位：字节
db.user.totalIndexSize()

#删除索引
db.user.dropIndex("loc_2dsphere")
#或者，删除除了_id之外的索引
db.user.dropIndexes()
```

地理空间索引的type可以是下列的类型：

- **Point**（坐标点），coordinates必须是单个位置
- **MultiPoint**（多个点），coordinates必须是位置数组
- **LineString**（线形），coordinates必须是两个或多个位置的数组
- **MultiLineString**（多行线形），coordinates必须是LineString坐标数组的数组
- **Polygon**（多边形），coordinates成员必须是 LinearRing 坐标数组的数组，必须是闭环，也就是第一个和最后一个坐标点要相同。
- **MultiPolygon**（多个多边形），coordinates成员必须是 Polygon 坐标数组的数组。
- **GeometryCollection**（几何集合）,geometries是任何一个对象的集合。

## 2.6、UI客户端工具

前面我们是通过命令行操作MongoDB，这样不太方便，可以通过可视化工具操作MongoDB，在这里推荐使用Studio 3T。

官网：https://studio3t.com/

![img](https://cdn.nlark.com/yuque/0/2022/png/27683667/1663150840273-d1af66b5-5d12-4b90-a758-aed609a5ba82.png?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_55%2Ctext_6buR6ams56iL5bqP5ZGYwrfnoJTnqbbpmaI%3D%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10)

下载地址：https://studio3t.com/download/ （或使用课程资料中提供的安装包进行安装）

Studio 3T提供了30天的试用期，时期用到后可以永久使用免费版，免费版比收费版功能要少一些，对我们而言免费版也够用了。

Studio 3T需要注册账号，自习注册即可。

![img](https://cdn.nlark.com/yuque/0/2022/jpeg/27683667/1663151373366-a8bdc5d5-460b-4173-87d4-d8c1dd528429.jpeg?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_26%2Ctext_6buR6ams56iL5bqP5ZGYwrfnoJTnqbbpmaI%3D%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10)

![img](https://cdn.nlark.com/yuque/0/2022/png/27683667/1663151303212-fb974ec2-bb23-42cf-9bc8-5912666b9d8f.png?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_32%2Ctext_6buR6ams56iL5bqP5ZGYwrfnoJTnqbbpmaI%3D%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10)

新建链接，输入链接字符串：`mongodb://sl:123321@192.168.150.101:27017/admin`

链接字符串的格式：

```
mongodb://[username:password@]host1[:port1][,...hostN[:portN]][/[defaultauthdb][?options]]
```

![img](https://cdn.nlark.com/yuque/0/2022/png/27683667/1663151510545-fb3e6af5-cc08-4c8c-8fe8-47f449086210.png?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_20%2Ctext_6buR6ams56iL5bqP5ZGYwrfnoJTnqbbpmaI%3D%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10)

![img](https://cdn.nlark.com/yuque/0/2022/png/27683667/1663151564305-cc1b4099-81d2-4408-93d0-618a7c699672.png?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_27%2Ctext_6buR6ams56iL5bqP5ZGYwrfnoJTnqbbpmaI%3D%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10)

查看数据以及索引：

![img](https://cdn.nlark.com/yuque/0/2022/png/27683667/1663151606075-bf237799-d615-45d2-8a84-8c3114ea941d.png?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_50%2Ctext_6buR6ams56iL5bqP5ZGYwrfnoJTnqbbpmaI%3D%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10)

查看json结构数据：

![img](https://cdn.nlark.com/yuque/0/2022/png/27683667/1663151764882-11096f41-d664-462d-9ab9-bcceaa217dfa.png?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_41%2Ctext_6buR6ams56iL5bqP5ZGYwrfnoJTnqbbpmaI%3D%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10)

![img](https://cdn.nlark.com/yuque/0/2022/png/27683667/1663151651622-f9d5ec11-d1e9-4ada-adb6-4df9d44422ec.png?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_28%2Ctext_6buR6ams56iL5bqP5ZGYwrfnoJTnqbbpmaI%3D%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10)

其他功能后续使用中逐步熟悉。

# 3、Spring Data MongoDB

spring-data对MongoDB做了支持，使用spring-data-mongodb可以简化MongoDB的操作。

https://spring.io/projects/spring-data-mongodb

## 3.1、创建工程

创建`sl-express-mongodb`工程对Spring Data MongoDB的使用做基本的学习。导入依赖：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>com.sl-express</groupId>
        <artifactId>sl-express-parent</artifactId>
        <version>1.4</version>
    </parent>

    <groupId>com.sl-express.mongodb</groupId>
    <artifactId>sl-express-mongodb</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <maven.compiler.source>11</maven.compiler.source>
        <maven.compiler.target>11</maven.compiler.target>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-mongodb</artifactId>
        </dependency>
    </dependencies>

</project>
```

## 3.2、配置

```yaml
spring:
  application:
    name: sl-express-mongodb
  data:
    mongodb:
      host: 192.168.150.101
      port: 27017
      database: sl
      authentication-database: admin #认证数据库
      username: sl
      password: "123321"
      auto-index-creation: true #自动创建索引
```

## 3.3、Entity

```java
package com.sl.mongo.entity;

import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Data;
import lombok.NoArgsConstructor;
import org.bson.types.ObjectId;
import org.springframework.data.annotation.Id;
import org.springframework.data.mongodb.core.geo.GeoJsonPoint;
import org.springframework.data.mongodb.core.index.GeoSpatialIndexType;
import org.springframework.data.mongodb.core.index.GeoSpatialIndexed;
import org.springframework.data.mongodb.core.index.Indexed;
import org.springframework.data.mongodb.core.mapping.Document;

@Data
@Builder
@AllArgsConstructor
@NoArgsConstructor
@Document("sl_person") //指定表名
public class Person {

    @Id // 标识为主键
    private ObjectId id;
    @Indexed //标识索引字段
    private String name;
    private int age;
    /**
     * 用户位置
     * x: 经度，y：纬度
     */
    @GeoSpatialIndexed(type = GeoSpatialIndexType.GEO_2DSPHERE)
    private GeoJsonPoint location;
    //存储嵌套对象数据
    private Address address;
}
package com.sl.mongo.entity;

import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Data;
import lombok.NoArgsConstructor;
import org.springframework.data.mongodb.core.mapping.Document;

@Data
@Builder
@AllArgsConstructor
@NoArgsConstructor
@Document("sl_address") //指定表名
public class Address {
    private String street;
    private String city;
    private String zip;
}
```

## 3.4、Service

```java
package com.sl.mongo.service;

import com.sl.mongo.entity.Person;

import java.util.List;

public interface PersonService {

    /**
     * 新增数据
     *
     * @param person 数据
     */
    void savePerson(Person person);

    /**
     * 更新数据
     *
     * @param person 数据
     */
    void update(Person person);

    /**
     * 根据名字查询用户列表
     *
     * @param name 用户名字
     * @return 用户列表
     */
    List<Person> queryPersonListByName(String name);

    /**
     * 分页查询用户列表
     *
     * @param page     页数
     * @param pageSize 页面大小
     * @return 用户列表
     */
    List<Person> queryPersonPageList(int page, int pageSize);

    /**
     * 根据id删除用户
     *
     * @param id 主键
     */
    void deleteById(String id);
}
```

接口实现类：

```java
package com.sl.mongo.service.impl;

import com.sl.mongo.entity.Person;
import com.sl.mongo.service.PersonService;
import org.bson.types.ObjectId;
import org.springframework.data.domain.PageRequest;
import org.springframework.data.mongodb.core.MongoTemplate;
import org.springframework.data.mongodb.core.query.Criteria;
import org.springframework.data.mongodb.core.query.Query;
import org.springframework.data.mongodb.core.query.Update;
import org.springframework.stereotype.Service;

import javax.annotation.Resource;
import java.util.List;

@Service
public class PersonServiceImpl implements PersonService {

    @Resource
    private MongoTemplate mongoTemplate;

    @Override
    public void savePerson(Person person) {
        this.mongoTemplate.save(person);
    }

    @Override
    public void update(Person person) {
        //条件
        Query query = Query.query(Criteria.where("id").is(person.getId()));

        //更新的数据
        Update update = Update.update("age", person.getAge())
                .set("name", person.getName())
                .set("location", person.getLocation())
                .set("address", person.getAddress());

        //更新数据
        this.mongoTemplate.updateFirst(query, update, Person.class);
    }

    @Override
    public List<Person> queryPersonListByName(String name) {
        Query query = Query.query(Criteria.where("name").is(name)); //构造查询条件
        return this.mongoTemplate.find(query, Person.class);
    }

    @Override
    public List<Person> queryPersonPageList(int page, int pageSize) {
        PageRequest pageRequest = PageRequest.of(page - 1, pageSize);
        Query query = new Query().with(pageRequest);
        return this.mongoTemplate.find(query, Person.class);
    }

    @Override
    public void deleteById(String id) {
        Query query = Query.query(Criteria.where("id").is(new ObjectId(id)));
        this.mongoTemplate.remove(query, Person.class);
    }
}
```

## 3.5、启动类

```java
package com.sl.mongo;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class MongoApplication {

    public static void main(String[] args) {
        SpringApplication.run(MongoApplication.class, args);
    }

}
```

## 3.6、测试

```java
package com.sl.mongo.service;

import com.sl.mongo.entity.Address;
import com.sl.mongo.entity.Person;
import org.bson.types.ObjectId;
import org.junit.jupiter.api.Test;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.data.mongodb.core.geo.GeoJsonPoint;

import javax.annotation.Resource;

import java.util.List;

import static org.junit.jupiter.api.Assertions.*;

@SpringBootTest
class PersonServiceTest {

    @Resource
    PersonService personService;

    @Test
    void savePerson() {
        Person person = Person.builder()
                .id(ObjectId.get())
                .name("张三")
                .age(20)
                .location(new GeoJsonPoint(116.343847, 40.060539))
                .address(new Address("人民路", "上海市", "666666")).build();
        this.personService.savePerson(person);
    }

    @Test
    void update() {
        Person person = Person.builder()
                .id(new ObjectId("632283c08139e535c2bd7579"))
                .name("张三")
                .age(22) //修改数据
                .location(new GeoJsonPoint(116.343847, 40.060539))
                .address(new Address("人民路", "上海市", "666666")).build();
        this.personService.update(person);
    }

    @Test
    void queryPersonListByName() {
        List<Person> personList = this.personService.queryPersonListByName("张三");
        personList.forEach(System.out::println);
    }

    @Test
    void queryPersonPageList() {
        List<Person> personList = this.personService.queryPersonPageList(1, 10);
        personList.forEach(System.out::println);
    }

    @Test
    void deleteById() {
        this.personService.deleteById("632283c08139e535c2bd7579");
    }
}
```

数据：

![img](https://cdn.nlark.com/yuque/0/2022/png/27683667/1663208737781-8f0eae5b-8a6f-4567-8e93-c99719612e88.png?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_23%2Ctext_6buR6ams56iL5bqP5ZGYwrfnoJTnqbbpmaI%3D%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10)

