# **5 MongoDB整合SpringBoot**

## **5.1 环境准备** 

### **1 引入依赖** 

```
<!-- spring data mongodb --＞ 
<dependency> 
	<groupId>org.sprintframework.boot</groupId> 
	<artifactId>spring-boot-starter-data-mongodb</artifactId>
</dependency> 
```


### **2．配置yml**

```
spring: 
	data: 
		mongodb: 
			uri: mongodb://jacob:jacob@192.168.65.174:27017/test?authsource=admin 
			# uri 等同于下面的配置 
			# database:test 
			# host:192.168.65.174 
			# port: 27017 
			# username:jacob 
			# password: jacob 
			# authentication-database: admin 
```

连接配置参考文档：[https://docs.mongodb.com/manual/reference/connection-string](https://docs.mongodb.com/manual/reference/connection-string)

### **3．使用时注入mongolemplate** 

```
@Autowired 
MongoTemplate mongoTemplate;
```


## **5.2 集合操作**

```
@Test 
public void testCollection(){ 

	boolean exists=mongoTemplate.collectionExists("emp"); 
	if(exists){ 
		// 删除集合
		mongoTemlate.dropCollection("emp"); 
		}  
		// 创建集合	
	mongoTemplate.createCollection("emp");
}
```



## **5.3 文档操作**

### **相关注解** 

* `@Document`
	* **修饰范围**：用在类上 
	* 作用：用来映射这个类的一个对象为mongo中一条文档数据。 
	* 属性：`(value、collection）`用来指定操作的集合名称 
	
* `@Id` 
	* **修饰范围：用在成员变量、方法上** 
	* 作用：用来将成员变量的值映射为文档的`_id`的值 
* `@Field` 
	* 修饰范围：用在成员变量、方法上 
	* 作用：用来将成员变量及其值映射为文档中一个`key:value`对。
	* 属性：(`name,value`）用来指定在文档中key的名称，默认为成员变量名 

* `@Transient`
	* 修饰范围：用在成员变量、方法上 
	* 作用：用来指定此成员变量不参与文档的序列化 


**创建实体**

```
@Document("emp") // 对应emp集合中的个文档 
@Data 
@AllArgsConstructor 
@NoArgsConstructor 
public class Employee{ 

	@Id //映射文档中的 _Id 
	private Integer id; 
	@Field("usernarne") 
	private String name; 
	@Field 
	private int age; 
	@Field 
	private Double salary; 
	@Field 
	private Date birthday; 
} 
```

### **添加文档** 

Insert方法返回值是新增的`Document`对象，里面包含了新增后id的值。如果集合不存在会自动创建集合。

通过`Spring Data MongoDB`还会给集合中多加一个`class`的属性，存储新增时`Document`对应`Java`中类的全限定路径。这么做为了查询时能把`Document`转换为`Java`类型

```
@Test
public void testInsert(){
	Employee employee = new Ernployee(1, "小区"，30, 10000.00, new Date());
	// 添加文档 
	// sava: _id 记存在时更新数据 
	// mongoTemplate.save (employee); 
	// insert: _id：抛出异常	支持批量操作 

mongoTemplate.insert(employee)；

List<Employee> list= Arrays.asList(
	Employee(2,"张三", 21,5000.00,  new Date()),
	Employee(3,"李四", 26,8000.00,  new Date()),
	Employee(4,"王五", 22,5000.00,  new Date()),
	Employee(5,"张龙", 28,6000.00,  new Date()),
	Employee(6,"赵虎", 24,7000.00,  new Date()),
	Employee(7, "赵六', 28,12000.00,  new Date());
   // 插入多条数据
   mongoTemplate.insert(list,Employee.class)；
}
```

### **查询文档**

**Criteral 是标准查询的接口，可以引用静态的`Criteria.where`的把多个条件组合在一起，就可以轻松地将多个方法标准和查询连接起来，方便我们操作查询语句**。

![Alt Image Text](../images/mon1_5_1.png "Body image")

```
@Test public void testFind(){ 
	System.out.printin("========查询文档========");
	//查询所有文档
	List<Employee> list = mongoTemplate.findAll(Employee.class);
	list.forEach("System.out::println");
	
	System.out.printin("========根据_id文档========");
	// 根据_id查询
	Employees=mongoTemplate.findById(1, Employee.class); 
	System.out.printin(e); 
	
	System.out.printin("========findOne返回第一个文档========");
	//如果查询结果是多个，返回其中第一个文档对象
	Employee one=mongoTemplate.findOne(new Query(), Employee.class); 
 	System.out.println(one); 
	
	System.out.printin("========条件查询========");
	//new Query()表示没有条件
	//查询薪资大于等于8000的员工 
	// Query query=new Query(Criteria.where("salary").gte(8000));
	//查询薪资大于等于4000小于10000的员工 
	// Query query=new Query(Criteria.where("salary").gt(4000).le(10000));
	//正则查询（模糊查询）java中正则不需要有// 
	//Query query=new Query(Criteria.where("name").regex("张")); 
	
	//and  or多条件查询 
	Criteria criteria=new Criteria(); 
	//and 查询年龄大于25&.薪资小于8000的员工 
	//criteria.andoperator(Criteria.where("age").gt(25),Criteria.where("salary").gt(8000)); 
	//or 查询姓名是张三或者薪资大于8000的员工
	criteria.orOperator(Criteria.where("name").is("张三“),Criteria.where("salary").gt(8000)); 
	Query query=new Query(criteria);
	
	//sort排序 
	//query.with(Sort.by(Sort.Order.desc("salary'')));
	
	//skip limit分页	skip用于指定跳过记录数，limit则用于限定返回结果数量。
	 query.with(Sort.by(Sort.Order.desc("salary"))) 
	 	.skip(0) //指定跳过记录数 
	 	.limit(4); //每页显示记录数 
	
	//查询结果 
	List<Employee> employees = mongoTemplate.find( 
		query, Employee.class);
	employees.forEach(System.out::println()); 
} 
```

### **4.3 更新文档**

在Mongodb中无论是使用客户端API还是使用SpringData，更新返回结果一定是受行数影响。如果更新后的结果和更新前的结果是相同，返回0 

* **`updateFirst()`只更新满足条件的第一条记录**
* **`updateMulti()`更新所有满足条件的记录** 
* **`upsert()没`有符合条件的记录则插入数据**

```
@Test
public void testUpdate(){ 
	//query设置查询条件

	Query query=new Query(Criteria.where("salary").gte(15000)) 
	
	System.out.println("========更新前========");
	List <Employee>employees = mongoTemplate.find(query, Employee.class); 
	employees.fo:Each(System.out::println);  
	
	Update update=new Update() 
	// 设置更新属性 
	update.set("salary", 13000); 
	
	//updateFirst()  只更新满足条件的第一条记录 
	//UpdateResult updateResult=mongoTemplate.updateFirst(query, update, Employee.class); 
更新所有下两走条件的记求 
	//updateMulti() // 更新所有满足条件的记录
	//UpdateResult updateResult=mongoTemplate.updateMulti(query, update, Employee.class); 
	
	//upsert(）没有符合条件的记录则插入数据 
	//update.setOnInsert("id",11);  //指定_id
	// UpdateResult updateResult=mongoTemplate.upsert(query, update, Employee.class); 
	
	// 返回修改的记录数 
	System.out.println(updateResult.getModifiedCount()); 

	System.out.printin("========更新后========");
	employees = mongoTemplate.find(query, Employee.class)
	employees.Foreach(System.out::println);
} 
```



## **4.4 删除文档**

```
@Test 

public void testDelete(){ 
	
	//删除所有文档
	//mongoTemplate.remove(new Query(), Empioye.class);
	
	//删除条件
    Query query=new Query(Criteria.where("salary").Employee.gte(10000)); 
	mongoTemplate.remove(query, Employee.class));
} 
```


