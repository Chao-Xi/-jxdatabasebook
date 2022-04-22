# **8．视图** 

MongoDB视图是一个可查询的对象，它的内容由其他集合或视图上的聚合管道定义。MongoDB不会将视图内容持久化到磁盘。当客户端查询视图时视图的内容按计算。

MongoDB可以要求客户端具有查询视图的权限。MongoDB不支持对视图进行写操作。 

作用： 

* 数据油象 
* 保护敏感数据的一种方法 
* 将敏感数据投影到视图之外 
* 只读 
* 结合基于角色的授权，可按角色访问信息 

**数据准备**

`srars.js`

```
var orders = new Array();
var shipping = new Array();
var addresses = ["NewYork City","Hoboken","Jerrsey City","Brea", "Huston", "Teaxs", "LA","Brooklyn"];

for (var i = 10000; i < 20000; i++) {
        var orderNo = i + Math.random().toString().substr(2, 5);
        orders[i] = { orderNo: orderNo, userId: i, price: Math.round(Math.random() * 10000) / 100, qty: Math.floor(Math.random() * 10) + 1, orderTime: new Date(new Date().setSeconds(Math.floor(Math.random() * 10000))) };

        var address = addresses[Math.floor(Math.random() * 10)];
        shipping[i] = {orderNo: orderNo, address: address, recipienter: "Wilson", province: address.substr(0, 3), city: address.substr(3,3) }
}
db.order.insert(orders);
db.shipping.insert(shipping);

```

```
./mongo localhost:27017 -u fox -p fox --authenticationDatabase=admin
> show dbs

> use viewdemo
switched to db viewdemo

> load("stats.js")
true
```

### **8.1 创建视图**

基本语法格式 

```
db.createView( 
	"<viewName>",
  "<source>", 
  [<pipeline＞],
   { 
   "collation" : {<collation>} 
	}
)
```

* `viewName` 必须，视图名称 
* `source` 必须，数据源，集合／视图 
* `[<pipeline>]`：可选，一组管道 
* `Collation`:  可选，排序规则 

**单个集合创建视图**

**假设现在查看当天最高的10笔订单视图，例如需要实时显示金额最高的订单** 

```
db.createView(
	”orderInfo", //视图名称
	”order",     //数据源
	[
	  // 筛选符合条件的订单，大于当天，这里要注意时区 
	  {$match: {"orderTime": 	{ $gte:ISODate("2022-01-26T00:00:00.000Z“ }}},
	// 按金额倒序 
	{$sort: {"price"：-1}},
	// 限制10个文档
	{ $limit: 10 },
	// 选择要显示的字段
	// O: 安排除字段，若字段上使用（_id除外）就不能有其他包含字段
	// 1: 包含字段
	 { $project: { _id:0,  orderNo: 1, price:1, orderTime: 1  } }
 ]
)
```

视图创建成功后可以直接使用视图查询数据 


**多个集合创建视图** 

跟单个是集合是一样，只是多了＄$Iookup连接操作符，视图根据管道最终结果显示，所以可以关联多个集合 

```
db.orderDetail.drop() 
db.createView(
	"orderDetail",
	"order",
	[
		{$lookup:{ from:"shipping", localField: "orderNo", foreignField:"orderNo", as:"shipping" }},
		{$project: { "orderNo":1, "price":1, "shipping.address":1 }} 
	]
)
```


### **6.2 修改视图** 

```
db.runCommand[{ 
	collMod: "ordeInfo", 
	viewOn："order" 
	pipeline:[
		{$match: {"ordeTime": {$gte：ISODate("2020-04-13T16:00:00.000z")}}}，
		{$sort:{"price"：-1}}, 
		{$limit:10}, 
		// 增加qty 
		{$project:{ _id:0, orderNo:1, price:1, qty:1, orderTime:1}} 
]
}) 
```

### **6.2 修改视图** 

```
db.orderInfo.drop()
```
