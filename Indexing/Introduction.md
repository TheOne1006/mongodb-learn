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
 * step1 创建一个 大集合 100 w 条数据
 */
> for (i =0;i<1000000; i++) {
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

索引的值是按照一定顺序排列的.因此,使用索引键进行排序非常快.  
然而, __只有首先使用索引键进行排序__ 时,索引才有用.  
复合索引就是一个建立在多个字段上的索引.  

例如:  
```js
> db.users.find().sort({"age":1, "username":1});
// 在排序中 "username" 上的索引没什么作用
```

这里先根据 "age" 排序在根据 "username"排序, 所有已这个发挥的作用并不大.  
为了优化这个排序,所以需要在 "age" 和 "username" 上简历索引  

```js
// 创建一条复合索引
> db.users.ensureIndex({"age":1,"username":1});
```

##### MongoDB 对这个索引的使用方式取决于查询的类型,下面是三种主要的方式

> `db.users.find({"age":21}).sort({"username":1})``

这是一个 __点查询(point query)__ , 用于查找单个值(尽管包含这个值的文档可能有多个).由于索引中的第二个字段,查询结果已经是有序的了  
这种查询是非常高效的: MongoDB 定位到了正确的位置,而且不需要对结果进行排序(只需要逆序就可以得到正确的顺序了)  

> `db.users.find({"age":{"$gte":21,"$lte":30}})`

这是一个 __多值查询(multi-value query)__ , 查找到多个值向匹配的文档.  
通常来说,如果 MongoDB 使用索引进行查询,那么查询的结果通常是按照索引序列排序的.  

> `db.users.find({"age":{"$gte":21,"$lte":30}}).sort({"username":1})`

这是一个多值查询,与上一个类似, 只是需要对查询结果进行排序, 跟之前一样, Mongodb 使用索引匹配查询结果.  
然而,这个索引得到的结果集中 username 是无序的,所以 Mongodb 需要先在内存中对结果进行排序,然后才返回.  
所以还不如上一个高效.  

当然,查询速度还取决于查询条件匹配文档的数量:  
- 如果结果集只有少量文档, 排序不需要耗费多少时间
- 如果文档较多,查询速度就会比较慢,甚至根本不能用:
    - 如果结果集合大小超过 __32MB__ ,mongodb 就会出错,拒绝如果此多的数据排序

> `db.users.find({"username":1, "age":1})`

使用了另一个索引, 但是顺序调换了,  

这样,在内存中不需要大量对数据进行排序.  
但是, Mongodb 不得不扫描整个索引匹配所有的文档.  

如果对查询结果的范围作了限制, 那么 mongodb 在几次匹配之后,就可以不再扫描索引.  

案例:  

```js
> db.users.find({"age": {"$gte":21, "$lte":30}}).sort({"username":1}).explain('executionStats');
{
    ...
    "executionTimeMillisEstimate" : 2600,
    ...
    // 使用索引 {"age":1,"username":1}
    "indexName" : "age_1_username_1",
    ...
}

/**
 * 使用  hint 强制使用特定索引
 */
> db.users.find({"age": {"$gte":21, "$lte":30}}).sort({"username":1}).hint({"username":1,"age":1}).explain('executionStats');
... ??? 报错

/**
 * 限制每次查询数量
 */
> db.users.find({"age":{"$gte":21, "$lte":30}}).sort({"username":1}).limit(1000).hint({"age":1,"username":1}).explain('executionStats');

> db.users.find({"age":{"$gte":21, "$lte":30}}).sort({"username":1}).limit(1000).hint({"username":1,"age":1}).explain('executionStats');
... ???
```

### 使用复合索引

在多个键上简历的索引就是 _复合索引_,  
复合索引比单键索引要复杂一些, 但是也更强大.  

> 1. 选择键的方向  

只有基于多键排序时, 方向才变得重要

> 2. 使用覆盖索引

如果你的查询只需要查找到索引中包含的字段, 那就根本没有必要获取实际的文档.   
当一个索引包含用户所请求的所有字段, 可以认为这个引用覆盖了本次查询.  

实际中, 应该优先使用覆盖索引, 从而不是去获取实际文档, 这样可以保持 __工作集比较小__  

> 3. 隐式查询

如果有一个 `{age:1,username:1}` , 也可以当做`{age:1}`索引.  
但是无法当做`{username:1}`  

### $操作符号如何使用索引

1. 低效率的操作符
    - 有一些查询完全无法使用索引
        - 不如`$where查询` 和 `{"key":{"$exists":true}}`
    - `$not` 有时候能够使用索引
        - 能够对基本的返回(如$lt, $gte) 和正则表达式进行翻转.
        - 大多数使用`$not`查询都会退化为进行全表扫描
    - `$nin` 总是全表扫描
2. 范围
    - 精确匹配的字段,放在索引前面,范围匹配字段放在后面.
3. OR 查询
    - `$or` 可以每个子句都使用索引,因为 `$or` 实际是执行两次,或者多次,然后将结果合并.  
    - 因此尽可能使用 `$in`
    - 如果不得不使用 `$or` ,MongoDB 需要检查每次查询的结果并从中移除重复文档


### 索引对象和数组

MongoDB 允许深入文档内部, __对嵌套字段和数组建立索引__  
可以与复合索引中的顶级字段一起使用.  
比较特殊, __但大多数情况与正常索引一样__  

> 1.索引嵌套文档

比如:  
```js
{
    "username": "sid",
    "loc": {
        "ip": "1.2.3.4",
        "city": "SpringField", // 建立索引
        "state":"NY"
    }
}

> db.users.ensureIndex({"loc.city": 1});
/**
 * 对 loc 建立索引,与 loc.city 建立索引是不一样的
 */
```

> 2. 索引数组

也可以对数组建立索引, 可以高效的搜索数组中的特定元素.  
与嵌套文档不同,无法将整个数组作为一个实体建立索引.  

> 3. 多键索引

对于某个索引的键,如果这个键在某个文档是一个数组,那么这个索引就会被标记为 多键索引  
多件索引可能会被非多键索引慢一些.  

### 索引基数(cardinality)

基数 就是集合某个字段拥有不同值的数量.  
比如: "gender" 可能就两个值, 这种键的基数非常低, "username" 非常的高

通常, 一个字段的基数越高,这个键上的索引就越有用.因为能够迅速的缩小搜寻范围.  
