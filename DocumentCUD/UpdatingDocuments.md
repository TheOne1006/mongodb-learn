## 更新文档

update 有两个参数:  
- 第一个是查询文档
- 第二个是修改器( modifier )文档,说明找到的文档需要进行哪些修改.

更新操作是不可分割的:  
如果两个更新同时发生,先到达服务器的先执行,接着执行另外一个.  
所以,两个需要同时进行的更新会迅速接连完成, 此过程不会破坏文档: 最新的更新会胜利.  

### 文档替换

方法:  
- 最简单的更新就是用一个 __新文档完全替换匹配的文档__ .
    - 这适用于大规模模式迁移的情况,或 改变文档结构
    - 更新单条数据,使用 `_id` 作为查询文档, 在执行更新是保证速度,以及避免字段相同,更新 `_id` 重复.


```js
> db.users.insert({name:'joe',friends:32,enemies:2});
WriteResult({ "nInserted" : 1 })
> var joe = db.users.findOne({name:'joe'});
> joe;
{
	"_id" : ObjectId("57961c3e298bd870559eed53"),
	"name" : "joe",
	"friends" : 32,
	"enemies" : 2
}
> var newJoe = {username: joe.name, relationships:{ friends: joe.friends, enemies: joe.enemies}}
> newJoe;
{
	"username" : "joe",
	"relationships" : {
		"friends" : 32,
		"enemies" : 2
	}
}
> db.users.update({name:'joe'}, newJoe);
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
> db.users.findOne({username:'joe'});
{
	"_id" : ObjectId("57961c3e298bd870559eed53"),
	"username" : "joe",
	"relationships" : {
		"friends" : 32,
		"enemies" : 2
	}
}
```


### 使用修改器

通常文档只会有一部分需要更新. 可以使用原子性的 __更新修改器( update modifier)__ ,
指定对文档中的某些字段进行更新. 更新修改器是一种特殊的键, 用来指定复杂的更新操作,  
比如: 修改,增加或者删除键,还有可能是操作数组或者内嵌文档.  

例子: 网站分析数据:  

```js
> db.analytics.insert({url:'www.theone.io',pageviews:51});
WriteResult({ "nInserted" : 1 })
> db.analytics.update({url:'www.theone.io'},{'$inc':{pageviews:1}});
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
> db.analytics.findOne({'url':'www.theone.io'});
{
	"_id" : ObjectId("579620a0298bd870559eed54"),
	"url" : "www.theone.io",
	"pageviews" : 52
}
```

使用修改器时, `_id` 的值不能改变.(而使用文档替换时可以改变 `_id`),
而其他唯一索引的键,都是可以更改的.  

> 修改方式:  

1. `$set` 用来指定一个字段
2. `$unset` 移除文档的一个字段
3. `$inc` 自增或自减

##### `$set` 修改器入门

- `$set` 用来指定一个字段的值,如果这个字段不存在,则创建它.
    - 这对更新模式或者增加用户定义的键来说非常方便.  
    - 还可以修改 键的类型.  
    - 也可以用来修改内嵌文档
- `$unset` 可以删除指定键.  


例如:  

```js
> db.users.insert({name:'joe',age:30,sex:'male'});
WriteResult({ "nInserted" : 1 })
> db.users.findOne();
{
	"_id" : ObjectId("579623e8298bd870559eed55"),
	"name" : "joe",
	"age" : 30,
	"sex" : "male"
}
// 设置 favorite 字段
> db.users.update({"_id":ObjectId("579623e8298bd870559eed55")},{$set:{"favorite":"football"}});
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
> db.users.findOne();
{
	"_id" : ObjectId("579623e8298bd870559eed55"),
	"name" : "joe",
	"age" : 30,
	"sex" : "male",
	"favorite" : "football"
}
// 修改 favorite 字段值
> db.users.update({"_id":ObjectId("579623e8298bd870559eed55")},{$set:{"favorite":"basketball"}});
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
> db.users.findOne();
{
	"_id" : ObjectId("579623e8298bd870559eed55"),
	"name" : "joe",
	"age" : 30,
	"sex" : "male",
	"favorite" : "basketball"
}

// 修改 favorite 字段类型
> db.users.update({"_id":ObjectId("579623e8298bd870559eed55")},{$set:{"favorite":["basketball","football"]}});
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
> db.users.findOne();
{
	"_id" : ObjectId("579623e8298bd870559eed55"),
	"name" : "joe",
	"age" : 30,
	"sex" : "male",
	"favorite" : [
		"basketball",
		"football"
	]
}

// 删除 favorite 字段
> db.users.update({"_id":ObjectId("579623e8298bd870559eed55")},{$unset:{"favorite":1}});
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
> db.users.findOne();
{
	"_id" : ObjectId("579623e8298bd870559eed55"),
	"name" : "joe",
	"age" : 30,
	"sex" : "male"
}

// 设置 attr 字段
> db.users.update({"_id":ObjectId("579623e8298bd870559eed55")}, {$set: {"attr": {"foo":1,"bar":"baz"}}});
> db.users.findOne();
{
	"_id" : ObjectId("579623e8298bd870559eed55"),
	"name" : "joe",
	"age" : 30,
	"sex" : "male",
	"attr" : {
		"foo" : 1,
		"bar" : "baz"
	}
}
// 更新 attr.foo 属性
> db.users.update({"_id":ObjectId("579623e8298bd870559eed55")}, {$set: {"attr.foo":2}});
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
> db.users.findOne();
{
	"_id" : ObjectId("579623e8298bd870559eed55"),
	"name" : "joe",
	"age" : 30,
	"sex" : "male",
	"attr" : {
		"foo" : 2,
		"bar" : "baz"
	}
}

```

##### `$inc` 增加和减少

`$inc` 修改器用来增加已有键的值,如果不存在则创建.  
    - 对于更新分析数据,因果关系,投票或者其他数值的地方

> 注意:

- `$inc` 与 `$set` 类似,是专门用来 增减 数字的.  
- `$inc` 只能用于 整型,长整型,双精度浮点值. 其他类型的数据将会失败.
    - 如果 是 null, boolean, 数字构成的字符串, 将会自动将它们转换为数值类型,再计算

案例:  

```js
// 初始数据
> db.games.insert({"game":"pinball","user":"joe"});
WriteResult({ "nInserted" : 1 })
> db.games.findOne();
{
	"_id" : ObjectId("57963341298bd870559eed56"),
	"game" : "pinball",
	"user" : "joe"
}
// 增加 分值 字段,并赋予 50
> db.games.update({"_id":ObjectId("57963341298bd870559eed56")}, {"$inc":{"score":50}});
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
> db.games.findOne();
{
	"_id" : ObjectId("57963341298bd870559eed56"),
	"game" : "pinball",
	"user" : "joe",
	"score" : 50
}

// 增加积分属性
> db.games.update({"_id":ObjectId("57963341298bd870559eed56")}, {"$inc":{"score":1000}});
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
> db.games.findOne();
{
	"_id" : ObjectId("57963341298bd870559eed56"),
	"game" : "pinball",
	"user" : "joe",
	"score" : 1050
}

```




- - -