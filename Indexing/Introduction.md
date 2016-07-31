## 索引简介

不使用索引的查询被称为 __全表扫描(来自关系型数据库)__,对于大集合来说,全表扫描效率非常低.  

使用索引的代价:
- 对于添加的每一个索引,每次写操作 都将耗费更多的时间.   
- 这是因为,数据变动时, __不仅要更新文档,还要更新集合上的索引__.  
- 因此, Mongodb 限制每个集合最多只能有 64个索引.  
    - 通常,一个特定的集合上,不应该拥有两个以上的索引

> Mongodb 的索引几乎与传统关系型数据库索引一模一样.  




案例:  

```js
/**
 * step1 创建一个 大集合 10 w 条数据
 */
> for (i =0;i<100000; i++) {
    db.users.insert({"i":i,"username":"user"+i,"age":Math.floor(Math.random()*120),"created": new Date()});
}
/**
 * 使用 explain() 函数查看 MongoDB 在执行查询的过程中所做的事情
 */
> db.users.find({username: "user99"}).explain();
{
	"queryPlanner" : {
        ...
	},
	"executionStats" : {
		"executionSuccess" : true,
		"nReturned" : 1,
		"executionTimeMillis" : 43,
		"totalKeysExamined" : 0,
		"totalDocsExamined" : 100000,
		"executionStages" : {
			...
		},
		"allPlansExecution" : [ ]
	},
	"serverInfo" : {
        ...
	},
	"ok" : 1
}
```

尝试创建 username 索引:  
```js
> db.users.ensureIndex({"username": 1});
{
	"createdCollectionAutomatically" : false,
	"numIndexesBefore" : 1,
	"numIndexesAfter" : 2,
	"ok" : 1
}
> db.users.find({username: "user99"}).explain('executionStats');
{
	"queryPlanner" : {
        ...
	},
	"executionStats" : {
		"executionSuccess" : true,
		"nReturned" : 1,
		"executionTimeMillis" : 0,
		"totalKeysExamined" : 1,
		"totalDocsExamined" : 1,
		"executionStages" : {
            ...
		}
	},
	"serverInfo" : {
        ...
	},
	"ok" : 1
}
```

### 复合索引简介

索引的值是按照一定顺序排列的.  
