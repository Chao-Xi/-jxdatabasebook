# **10 索引属性**

## **唯一索引（Unique Indexes)**

在现实场景中，唯一性是很常见的一种索引约束需求，重复的数据记录会带来许多处理上的麻烦，比如订单的编号、用户的登录名等。通过建立唯一性索引可以保证集合中文档的指定字段拥有唯一值。

```
＃创建唯一索引 
db.values.createIndex({title:1}, {unique:true}} 

＃复合索引支持唯一性约束 
db.values.createIndex({title:1.type:1}, {unique:true}) 

＃多键索引支唯一索引性约束 
db.inventory.createIndex({ratings:1},{unique:true}) 
```


* 唯一性索引对于文档中缺失的字段，会使用null值代替，因此不允许存在多个文档缺失索引字段的情况。 
* 对于分片的集合，唯一性约束必须匹配分片规则。换句话说，为了保证全局的唯一性，分片键必须作为唯一性索引的前缀字段。 

```
> db.values.drop()

> db.values.insert({x:1})
WriteResult({ "nInserted" : 1 })

> db.values.insert({x:1})
WriteResult({ "nInserted" : 1 })

> db.values.drop()
true

> db.values.insert({x:1})
WriteResult({ "nInserted" : 1 })

> db.values.createIndex({x:1},{unique:true})
{
	"createdCollectionAutomatically" : false,
	"numIndexesBefore" : 1,
	"numIndexesAfter" : 2,
	"ok" : 1
}

> db.values.insert({x:1})
WriteResult({
	"nInserted" : 0,
	"writeError" : {
		"code" : 11000,
		"errmsg" : "E11000 duplicate key error collection: appdb.values index: x_1 dup key: { x: 1.0 }"
	}
})


> db.values.drop()
true

> db.values.insert({x:[1,2,3]})
WriteResult({ "nInserted" : 1 })

> db.values.createIndex({x:1},{unique:true})
{
	"createdCollectionAutomatically" : false,
	"numIndexesBefore" : 1,
	"numIndexesAfter" : 2,
	"ok" : 1
}

> db.values.insert({x:[1,2,3]})
WriteResult({
	"nInserted" : 0,
	"writeError" : {
		"code" : 11000,
		"errmsg" : "E11000 duplicate key error collection: appdb.values index: x_1 dup key: { x: 1.0 }"
	}
})

> db.values.insert({x:[1]})
WriteResult({
	"nInserted" : 0,
	"writeError" : {
		"code" : 11000,
		"errmsg" : "E11000 duplicate key error collection: appdb.values index: x_1 dup key: { x: 1.0 }"
	}
})

> db.values.drop()
true

> db.values.insert([{x:4},{y:1}])
BulkWriteResult({
	"writeErrors" : [ ],
	"writeConcernErrors" : [ ],
	"nInserted" : 2,
	"nUpserted" : 0,
	"nMatched" : 0,
	"nModified" : 0,
	"nRemoved" : 0,
	"upserted" : [ ]
})

> db.values.find()
{ "_id" : ObjectId("626363b461515830838dcb7c"), "x" : 4 }
{ "_id" : ObjectId("626363b461515830838dcb7d"), "y" : 1 }


> db.values.createIndex({x:1},{unique:true})
{
	"createdCollectionAutomatically" : false,
	"numIndexesBefore" : 1,
	"numIndexesAfter" : 2,
	"ok" : 1
}

> db.values.insert({y:1})
WriteResult({
	"nInserted" : 0,
	"writeError" : {
		"code" : 11000,
		"errmsg" : "E11000 duplicate key error collection: appdb.values index: x_1 dup key: { x: null }"
	}
})
```


## **部分索引（Partial Indexes)**

**部分索引仅对满足指定过滤器表达式的文档进行索引**。**通过在一个集合中为文档的一个子集建立索，部分索引具有更低的存储需求和更低的索引创建和维护的性能成本。 3.2新版功能**。

部分索引提供了稀疏索引功能的超集，应该优先于稀疏索引。 

```
db.restaurants.createIndex(
	{cuisine: 1, name: 1},
	{partialFilterExpression: { rating: {$gt:5 }}}
)
```

partialFilterExpression选项接受指定过滤条件的文档：

* 等式表达式（例如：`field:Value`或使用`＄eq`操作符） 
* `$exists:true` 
* `$gt`,`$gte`,`$It`,`$lte` 
* `$type` 
* 顶层的`＄and` 


```
＃符合条件，使用索引 
db.restaurants.find({cuisine:"Italian",rating:{$gte:8}}) 

＃不符合条件，不能使用索引
db.restaurants.find({cuisine:"Italian"}) 
```

**案例**

```
db.restaurants.insert({ 
	"_id" : ObjectId("5641f6a7522545bc535b5dc9"), 
	"address" : { 
		"building" : "1007", 
		"coord" : 
			[ -73.856077, 
			40.8484471 
			], 
		"street" : "Morris Park Ave", 
		"zipcode" : "10462" 
}, 
	"borough" : "Bronx", 
	"cuisine" : "Bakery", 
	"rating" : { "date" : ISODate("2014-03-03T00:00:00Z"), 
						"grade" : "A", 
						"score" : 2 
					}, 
	"name" : "Morris Park Bake Shop", 
	"restaurant_id" : "30075445" 
}) 
```

```
WriteResult({ "nInserted" : 1 })
```


**创建索引**

```
db.restaurants.createIndex( 
	{borough:1, cuisine: 1}, 
 	{partialFilterExpression:{'rating.grade': {$eq: "A" }}}
) 
```

```
{
	"createdCollectionAutomatically" : false,
	"numIndexesBefore" : 1,
	"numIndexesAfter" : 2,
	"ok" : 1
}
```

**测试**

```
db.restaurants.find({borough: "Bronx" , 'rating.grade': "A"})

db.restaurants.find({borough: "Bronx", cuisine: "Bakery"  })
```

```
> db.restaurants.find({borough: "Bronx" , 'rating.grade': "A"})
{ "_id" : ObjectId("5641f6a7522545bc535b5dc9"), "address" : { "building" : "1007", "coord" : [ -73.856077, 40.8484471 ], "street" : "Morris Park Ave", "zipcode" : "10462" }, "borough" : "Bronx", "cuisine" : "Bakery", "rating" : { "date" : ISODate("2014-03-03T00:00:00Z"), "grade" : "A", "score" : 2 }, "name" : "Morris Park Bake Shop", "restaurant_id" : "30075445" }
```

```
> db.restaurants.find({borough: "Bronx", cuisine: "Bakery"  })
{ "_id" : ObjectId("5641f6a7522545bc535b5dc9"), "address" : { "building" : "1007", "coord" : [ -73.856077, 40.8484471 ], "street" : "Morris Park Ave", "zipcode" : "10462" }, "borough" : "Bronx", "cuisine" : "Bakery", "rating" : { "date" : ISODate("2014-03-03T00:00:00Z"), "grade" : "A", "score" : 2 }, "name" : "Morris Park Bake Shop", "restaurant_id" : "30075445" }
```

 
**唯一约束结合部分索引使用导致唯一约束失效的问题** 

注意：如果同时指定了`partialFilterExpression`和唯一约束，那么唯一约束只适用于满足筛选器表达式的文档。如果文档不满足筛选条件，那么带有唯一约束的部分索引不会阻止插入不满足惟一约束的文档。 


**案例2 users集合数据准备**

```
db.users.insertMany([
	{username: "david", age:29 },
	{username: "amanda", age:35 },
	{username: "rajiv", age:57 },
])
```
```
{
	"acknowledged" : true,
	"insertedIds" : [
		ObjectId("6263c08c61515830838dcb7f"),
		ObjectId("6263c08c61515830838dcb80"),
		ObjectId("6263c08c61515830838dcb81")
	]
}
```


**测试** 

索引防止了以下文档的插入，因为文档已经存在，且指定的用户名和年龄字段大于21 

```
db.users.insertMany([
	{username: "david", age:29 },
	{username: "amanda", age:35 },
	{username: "rajiv", age:57 },
])
```

创建索引指定username字段和部分过滤器表达式`age: {$gte: 21｝` 的唯一约束。 

```
db.users.createIndex( 
	{ username: 1 }, 
	{ unique: true , partialFilterExpression: { age: { $gte:21 } } } 
)
```

```
{
	"createdCollectionAutomatically" : true,
	"numIndexesBefore" : 1,
	"numIndexesAfter" : 2,
	"ok" : 1
}
```


**测试**

索引防止了以下文档的插入，因为文档已经存在，且指定的用户名和年龄字段大于21 

```
db.users.insertMany([
	{username: "david", age:29 },
	{username: "amanda", age:35 },
	{username: "rajiv", age:57 },
])
```

```
{
	"acknowledged" : true,
	"insertedIds" : [
		ObjectId("6263cbe77e25e70e549f7a34"),
		ObjectId("6263cbe77e25e70e549f7a35"),
		ObjectId("6263cbe77e25e70e549f7a36")
	]
}
```


但是，以下具有重复用户名的文档是允许的，因为唯一约束只适用于年龄大于或等于21岁的文档。 

```
db.users.insertMany([
	{username: "david", age:29 },
	{username: "amanda" },
	{username: "rajiv", age:null },
])
```

```
> db.users.insertMany([
... {username: "david", age:29 },
... {username: "amanda" },
... {username: "rajiv", age:null },
... ])
uncaught exception: BulkWriteError({
	"writeErrors" : [
		{
			"index" : 0,
			"code" : 11000,
			"errmsg" : "E11000 duplicate key error collection: test.users index: username_1 dup key: { username: \"david\" }",
			"op" : {
				"_id" : ObjectId("6263cc0b7e25e70e549f7a37"),
				"username" : "david",
				"age" : 29
			}
		}
	],
	"writeConcernErrors" : [ ],
	"nInserted" : 0,
	"nUpserted" : 0,
	"nMatched" : 0,
	"nModified" : 0,
	"nRemoved" : 0,
	"upserted" : [ ]
}) :
BulkWriteError({
	"writeErrors" : [
		{
			"index" : 0,
			"code" : 11000,
			"errmsg" : "E11000 duplicate key error collection: test.users index: username_1 dup key: { username: \"david\" }",
			"op" : {
				"_id" : ObjectId("6263cc0b7e25e70e549f7a37"),
				"username" : "david",
				"age" : 29
			}
		}
	],
	"writeConcernErrors" : [ ],
	"nInserted" : 0,
	"nUpserted" : 0,
	"nMatched" : 0,
	"nModified" : 0,
	"nRemoved" : 0,
	"upserted" : [ ]
})
BulkWriteError@src/mongo/shell/bulk_api.js:367:48
BulkWriteResult/this.toError@src/mongo/shell/bulk_api.js:332:24
Bulk/this.execute@src/mongo/shell/bulk_api.js:1186:23
DBCollection.prototype.insertMany@src/mongo/shell/crud_api.js:326:5
@(shell):1:1

```

## **稀疏索引（Sparse Indexes)** 

索引的稀疏属性确保索引只包含具有索引字段的文档的条目，**索引将跳过没有索引字段的文档**。 


特性：只对存在字段的文档进行索引（包括字段值为null的文档） 

```
＃索引不包含xmpp_id字段的文档 
db.addresses.createIndex({"xmpp_id":1},{sparse:true}) 
```

如果稀疏索引会导致查询和排序操作的结果集不完整，MongoDB将不会使用该索引除非`hint()`明确指定索引。 

**案例1**

数据准备 

```
db.scores.insertMany([ 
	{"userid": "newbie"}, 
	{"userid":"abby",  "score": 82}, 
	{"userid": "nina", "score": 90} 
]) 
```

```
> db.scores.insertMany([
... {"userid": "newbie"},
... {"userid":"abby",  "score": 82},
... {"userid": "nina", "score": 90}
... ])
{
	"acknowledged" : true,
	"insertedIds" : [
		ObjectId("6263cf897e25e70e549f7a3a"),
		ObjectId("6263cf897e25e70e549f7a3b"),
		ObjectId("6263cf897e25e70e549f7a3c")
	]
}
```

**创建稀疏索引** 

```
db.scores.createIndex({score: 1},{sparse: true}) 
```

```
> db.scores.createIndex({score: 1},{sparse: true})
{
	"createdCollectionAutomatically" : false,
	"numIndexesBefore" : 1,
	"numIndexesAfter" : 2,
	"ok" : 1
}
```

```
> db.scores.getIndexes()
[
	{
		"v" : 2,
		"key" : {
			"_id" : 1
		},
		"name" : "_id_"
	},
	{
		"v" : 2,
		"key" : {
			"score" : 1
		},
		"name" : "score_1",
		"sparse" : true
	}
]
```

**测试** 


```
# 使用稀疏索引 
db.scores.find({score:{$lt:90}}).explan()
```

```
> db.scores.find({score:{$lt:90}})
{ "_id" : ObjectId("6263cf897e25e70e549f7a3b"), "userid" : "abby", "score" : 82 }
```

```
> db.scores.find({score:{$lt:90}}).explain()
{
	"queryPlanner" : {
		"plannerVersion" : 1,
		"namespace" : "test.scores",
		"indexFilterSet" : false,
		"parsedQuery" : {
			"score" : {
				"$lt" : 90
			}
		},
		"queryHash" : "36A7AA2F",
		"planCacheKey" : "84D0C5E1",
		"winningPlan" : {
			"stage" : "FETCH",
			"inputStage" : {
				"stage" : "IXSCAN",
```

```
＃即使排序址通过索引字段，MongoDB也不会选徉稀疏索引来完成查询，以返回完招的结果 
db.scores.find().sort({sore:-1}) 
```

```
＃要使用稀疏索引，使用hint()显示指定索引 
db.scores.find().sort({score:-1}).hint({score: 1}) 
```

同时具有稀疏性和唯一性的索引可以防止集台中存在字段值重复的文档，但允许不包含此索引字段的文档插入。 

```
> db.scores.find()
{ "_id" : ObjectId("6263cf897e25e70e549f7a3a"), "userid" : "newbie" }
{ "_id" : ObjectId("6263cf897e25e70e549f7a3b"), "userid" : "abby", "score" : 82 }
{ "_id" : ObjectId("6263cf897e25e70e549f7a3c"), "userid" : "nina", "score" : 90 }
```

```
> db.scores.find().sort({score:-1})
{ "_id" : ObjectId("6263cf897e25e70e549f7a3c"), "userid" : "nina", "score" : 90 }
{ "_id" : ObjectId("6263cf897e25e70e549f7a3b"), "userid" : "abby", "score" : 82 }
{ "_id" : ObjectId("6263cf897e25e70e549f7a3a"), "userid" : "newbie" }
```

```
> db.scores.find().sort({score:-1}).hint({score: 1})
{ "_id" : ObjectId("6263cf897e25e70e549f7a3c"), "userid" : "nina", "score" : 90 }
{ "_id" : ObjectId("6263cf897e25e70e549f7a3b"), "userid" : "abby", "score" : 82 }
```

```
db.scores.dropIndex("sccore_1")

> db.scores.dropIndex("score_1")
{ "nIndexesWas" : 2, "ok" : 1 }
```

**案例2** 

```
＃创建具有唯一约束的稀疏索引 
db.scores.createIndex({score:1}, {sparse:true, unique:true}) 
```
```
{
	"createdCollectionAutomatically" : false,
	"numIndexesBefore" : 1,
	"numIndexesAfter" : 2,
	"ok" : 1
}
```

测试 

这个索引将允许许插入具有唯一的分数字段值或不包含分数字段的文档。因此， 给定`scores`集合中的现有文档，索引允许以下插入操作：

``` 
db.scores.insertMany([
	{ "userid": "AAAAAAA", "score":43 },
	{ "userid": "BBBBBBB", "score":34 },
	{ "userid": "ccccccc" },
	{ "userid": "ccccccc" },
])
```

```
{
	"acknowledged" : true,
	"insertedIds" : [
		ObjectId("6263e8677e25e70e549f7a3d"),
		ObjectId("6263e8677e25e70e549f7a3e"),
		ObjectId("6263e8677e25e70e549f7a3f"),
		ObjectId("6263e8677e25e70e549f7a40")
	]
}
```

索引不允许添加下列文件，因为已经存在评分为82和90的文件 

**选择语言** 

```
db.scores.insertMany([
	{ "userid": "AAAAAAA", "score":82 },
	{ "userid": "BBBBBBB", "score":90 }
])
```

```
uncaught exception: BulkWriteError({
	"writeErrors" : [
		{
			"index" : 0,
			"code" : 11000,
			"errmsg" : "E11000 duplicate key error collection: test.scores index: score_1 dup key: { score: 82.0 }",
			"op" : {
				"_id" : ObjectId("6263e89b7e25e70e549f7a41"),
				"userid" : "AAAAAAA",
				"score" : 82
			}
		}
	],
	"writeConcernErrors" : [ ],
	"nInserted" : 0,
	"nUpserted" : 0,
	"nMatched" : 0,
	"nModified" : 0,
	"nRemoved" : 0,
	"upserted" : [ ]
}) :
BulkWriteError({
	"writeErrors" : [
		{
			"index" : 0,
			"code" : 11000,
			"errmsg" : "E11000 duplicate key error collection: test.scores index: score_1 dup key: { score: 82.0 }",
			"op" : {
				"_id" : ObjectId("6263e89b7e25e70e549f7a41"),
				"userid" : "AAAAAAA",
				"score" : 82
			}
		}
	],
	"writeConcernErrors" : [ ],
	"nInserted" : 0,
	"nUpserted" : 0,
	"nMatched" : 0,
	"nModified" : 0,
	"nRemoved" : 0,
	"upserted" : [ ]
})
```


## **TTL索引（TTL Indexes)**

在一般的应用系统中，并非所有的数据都需要永久存储。例如一些系统事件、用户消息等，这些数据随看时间的推移，其重要程度逐渐降低。更重要的是，存储这些大量的历史数据需要花费较高的成本，因此项目中通常会对过期且不再使用的数据进行老化处理。 


通常的做法如下： 

* **方案一：为每个数据记录一个时间戳，应用侧开启一个定时器，按时间戳定期删除过期的数据**。
* **方案二：数据按日期进行分表，同一氏的数据归档到同一张表，同样使用定时器删除过期的表**。 

对于数据老化，MongoDB提供了一种更加便捷的做法：TTL(Tome to Live)索引。TTL索引需要声明在一个日期类型的字段中，TTL索引是特殊的单字段索引MongoDB可以使用它在一定时间或特定时钟时间后自动从集合中删除文档。 

```
＃创建TTL索引,TTL值为3600秒
db.eventlog.CreateIndex({"lastModifiedDate":1}, {expireAfterSeconds:3600}) 

```

对集合创建TTL索引之后，MongoDB会在周期性运行的后台线程中对该集合进行检查及数据清理工作。除了数据老化功能，TTL索引具有普通索引的功能，同样可以用于加速数据的查询。 

**TTL索引不保证过期数据会在过期后立即被删除。**文档过期和MongoDB从数据库中删除文档的时间之间可能存在延迟。删除过期文档的 后台任务每60秒运行一次。因此，在文档到期和后台任务运行之间的时间段内，文档可能会保留在集合中。 

**案例**

数据准备 

```
db.log_events.insertOne({ 
	"CreatedAt": new Date(),
	"logEvent":2, 
	"logMessage":"Success!" 
})
```
```
{
	"acknowledged" : true,
	"insertedId" : ObjectId("6263ee9120b8b386588f0dec")
}
```
	
**创建TTL索引**

```
db.log_events.createIndex({"CreatedAt":1},{expireAfterSeconds:20}) 
```

```
{
	"createdCollectionAutomatically" : false,
	"numIndexesBefore" : 1,
	"numIndexesAfter" : 2,
	"ok" : 1
}
```

```
> db.log_events.getIndexes()
[
	{
		"v" : 2,
		"key" : {
			"_id" : 1
		},
		"name" : "_id_"
	},
	{
		"v" : 2,
		"key" : {
			"CreatedAt" : 1
		},
		"name" : "CreatedAt_1",
		"expireAfterSeconds" : 20
	}
]
```

```
> db.log_events.find()
{ "_id" : ObjectId("6263efe920b8b386588f0ded"), "CreatedAt" : ISODate("2022-04-23T12:24:09.085Z"), "logEvent" : 2, "logMessage" : "Success!" }

# 20s past

> db.log_events.find()
```

TTL索引在创建之后，仍然可以对过期时间进行修改。这需要使用collMod命令对索引的定义进行变更 

```
db.runCommand({collMod:"log_events",index:{keyPattern:{createdAt:1},expireAfterSeconds:600}}) 
```

```
{
	"ok" : 0,
	"errmsg" : "cannot find index { createdAt: 1.0 } for ns test.log_events",
	"code" : 27,
	"codeName" : "IndexNotFound"
}
```


### **使用约束**

TTL索引索引的确可以减少开发的工作量，而且通过数据库自动清理的方式会更加高效、可靠，但是在使用TTL索引时需要注意以下的限制： 

* TTL索引只能支持单个字段，并且必须是非`_Id`字段。 
* TTL索引不能用于固定集合。
* TTL索引无法保证及时的数据老化，MongoDB会通过后台的TTLMonitor定时器来清理老化数据，默认的间隔时间是1分钟。
* 当然如果在数据库负载过高的情况下，TTL的行为则会进一步受到影响。 
* TTL索引对于数据的清理仅仅使用了remove命令，这种方式并不是很高效。因此TTL Monitor在运行期间对系统CPU、磁盘都会造成一定的压力。**相比之下，按日期分表的方式操作会更加高效**。 


## **隐藏索引（Hidden Indexes)** 

隐藏索引对查询规划器不可见，不能用于支持查询。

通过对规划器隐藏索引，用户可以在不实际删除索引的情况下评估删除索引的潜在影响。**如果影响是负面的，用户可以取消隐藏索引，而不必重新创建已删除的索引。4.4新版功能。**

```
创建隐藏索引
db.restaurants.createIndex({borough:1},{hidden:true}); 


# 隐藏现有索引 
db.restaurants.hideIndex({borough:1}); 
db.restaurants.hideIndex({"索引名称"}); 


＃取消隐藏索引 
db.restaurants.unhideIndex({borough:1}); 
db.restaurants.unhideIndex({"索引名称"}); 
```


```
db.scores.insertMany([ 
	{"userid": "newbie"}, 
	{"userid": "abby", "score" :82},
	{"userid": "nina", "score" :90}
]) 

```

```
{
	"acknowledged" : true,
	"insertedIds" : [
		ObjectId("6263f70b20b8b386588f0df1"),
		ObjectId("6263f70b20b8b386588f0df2"),
		ObjectId("6263f70b20b8b386588f0df3")
	]
}
```

创建隐藏索引 

```
db.scores.createIndex( 
	{userid: 1}, 
	{hidden:true} 
) 
```
```
{
	"createdCollectionAutomatically" : false,
	"numIndexesBefore" : 1,
	"numIndexesAfter" : 2,
	"ok" : 1
}
```

查看索引信息

```
db.scores.getIndexes() 
```

**索引属性hidden只在值为true时返回**

```
> db.scores.getIndexes()
[
	{
		"v" : 2,
		"key" : {
			"_id" : 1
		},
		"name" : "_id_"
	},
	{
		"v" : 2,
		"hidden" : true,
		"key" : {
			"userid" : 1
		},
		"name" : "userid_1"
	}
]
```

**测试**

```
# 不用索引
db.scores.find({ userid: "abby" }).explain()


> db.scores.find({ userid: "abby" }).explain()
{
	"queryPlanner" : {
		"plannerVersion" : 1,
		"namespace" : "test.scores",
		"indexFilterSet" : false,
		"parsedQuery" : {
			"userid" : {
				"$eq" : "abby"
			}
		},
		"queryHash" : "37A12FC3",
		"planCacheKey" : "7FDF74EC",
		"winningPlan" : {
			"stage" : "COLLSCAN",
···

# 取消影藏索引
> db.scores.unhideIndex({ userid: 1 })

 { "hidden_old" : true, "hidden_new" : false, "ok" : 1 }
 
# 用索引
> db.scores.find({ userid: "abby" }).explain()
...
"winningPlan" : {
			"stage" : "FETCH",
			"inputStage" : {
				"stage" : "IXSCAN",
				"keyPattern" : {
```


