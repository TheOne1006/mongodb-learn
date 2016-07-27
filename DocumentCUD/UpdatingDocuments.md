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

##### 数组修改器 ??

数组是常用且非常有用的数据结构:  

- 它不仅是可通过索引进行引用的列表
- 还可以作为数据集( set ) 来用

##### 添加元素 `$push`

如果数组存在, `$push` 向已有的数组末尾加入一个元素, 没有则创建一个新的数组.  

- 配合 `$each` 子操作符,可以通过一次 `$push` 添加多个值.  
- 配合 `$slice` 限制数组只包含最后10个元素. `$slice` 的值必须是 __负整数__
    - 用于 `$push` 时,必须 使用 `$each` 配合
- 配合 `$sort` 可以对数组所有对象进行排序
    - 用于 `$push` 时,必须 使用 `$each` 配合

案例1:  

```js
// 初始化
> db.blog.post.insert({"title":"blog post", "content":"content"});
WriteResult({ "nInserted" : 1 })
> db.blog.post.find();
{ "_id" : ObjectId("579636da298bd870559eed57"), "title" : "blog post", "content" : "content" }

// 创建一个 comments 字段, 并创建一条评论, comments 是一组数组
> db.blog.post.update({"_id":ObjectId("579636da298bd870559eed57")}, {"$push": {
    comments: {"content":"nice","name":"joe"}
}});
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
> db.blog.post.findOne();
{
	"_id" : ObjectId("579636da298bd870559eed57"),
	"title" : "blog post",
	"content" : "content",
	"comments" : [
		{
			"content" : "nice",
			"name" : "joe"
		}
	]
}

// 再添加一个评论
> db.blog.post.update({"_id":ObjectId("579636da298bd870559eed57")}, {"$push": {
    comments: {"content":"very good","name":"make"}
}});
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
> db.blog.post.findOne();
{
	"_id" : ObjectId("579636da298bd870559eed57"),
	"title" : "blog post",
	"content" : "content",
	"comments" : [
		{
			"content" : "nice",
			"name" : "joe"
		},
		{
			"content" : "very good",
			"name" : "make"
		}
	]
}

// 一次添加多个评论
> var comments = [{"content":"hello","name":"lilei"},{"content":"world","name":"hanmeimei"}];
> db.blog.post.update({"_id":ObjectId("579636da298bd870559eed57")}, {"$push": {
    comments: {"$each": comments}
}});

> db.blog.post.findOne();
{
	"_id" : ObjectId("579636da298bd870559eed57"),
	"title" : "blog post",
	"content" : "content",
	"comments" : [
		{
			"content" : "nice",
			"name" : "joe"
		},
		{
			"content" : "very good",
			"name" : "make"
		},
		{
			"content" : "hello",
			"name" : "lilei"
		},
		{
			"content" : "world",
			"name" : "hanmeimei"
		}
	]
}
```


案例2:  
```js
// 初始化
>  db.bar.insert({"foo":['a','b','c','d','e','f','g']});
WriteResult({ "nInserted" : 1 })
> db.bar.findOne();
{
	"_id" : ObjectId("57963afa298bd870559eed58"),
	"foo" : [
		"a",
		"b",
		"c",
		"d",
		"e",
		"f",
		"g"
	]
}
// 测试$slice 保留最后5位
> db.bar.update({"_id" : ObjectId("57963afa298bd870559eed58")},
    {"$push": {"foo": {
        "$each": ["h","l","i"],
        "$slice": -5
    }}}
);
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
> db.bar.findOne();
{
	"_id" : ObjectId("57963afa298bd870559eed58"),
	"foo" : [
		"f",
		"g",
		"h",
		"l",
		"i"
	]
}

// 测试排序
> db.bar.update({"_id" : ObjectId("57963afa298bd870559eed58")},
    {"$push": {"foo": {
        "$each": ["x","z","y"],
        "$slice": -3,
        "$sort": 1
    }}}
);

```

##### 将数组当做数据集操作 `$addToSet`

- `$addToSet`: 添加数组操作,能够避免重复

> 其他简易案例

```js
// 使用 $addToSet 避免重复
db.users.update({id: ObjectId("57963afa298bd870559eed58")},
{"$addToSet":{ "email":"jie@email.com"}});

// 使用 $addToSet & $each 一次添加多个
db.users.update({id: ObjectId("57963afa298bd870559eed58")},
{"$addToSet":{ "email": {"$each": ["jie@email.com", "jie1@email.com"]}}});

```

##### 删除数组元素 `$pop`,`$pull`

若把元素看成队列, 可以使用 `$pop`,可以从数组的两端删除一个元素.  

- `{"$pop":{"key": 1}}`: 从数组尾删除一个元素
- `{"$pop":{"key": -1}}`: 从数组头部删除一个元素

使用 `$pull` 根据特定条件删除 指定元素的位置  

- `{"$pull":{"todo":"remove"}}` ,从数组字段 "todo" 中删除 "remove" 元素

```js
> db.lists.insert({"todo":["dishes","laundry","dry cleaning"]});
> db.lists.update({},{"$pull":{"todo":"laundry"}});
> db.lists.findOne();
{
	"_id" : ObjectId("5798ac6ef79535cca3c4572a"),
	"todo" : [
		"dishes",
		"dry cleaning"
	]
}
```

##### 修改器速度
有的修改器运行比较快, 如 `$inc`,因为不需要改变文档大小.  
而数组修改器可能会改变文档大小,就会慢一些.  

原因:  
1. 将文档插入到 MongoDB 中,依次插入文档在磁盘的位置是相邻的.
2. 因此文档变大,原先的位置放不下,这个文档就会被移动到集合的另一个位置

案例:
```js
db.coll.insert({"x":"a"});
db.coll.insert({"x":"b"});
db.coll.insert({"x":"c"});
```

### upsert

```js
// 第三个参数 表示 upsert
update({query},{修改文档}, true);
```

upsert 是一种特殊的更新. 要是没有找到符合更新条件的文档,就会以这条件和更新文件为基础创建一个新的文档.  

`$setOnInsert` 指定字段只有在创建时生效.

### 更新多文档

```js
update({query},{modify}, isInsert, 多文档更新 boolean);
```

### 返回被更新的文档

调用 `getLastError` 仅能获得关于更新的有限信息, 并不能返回被更新的文档.  
可以通过 `findAndModify` 命令得到被更新的文档.  
这对于操作队列以及执行其他需要进行原子性取值和赋值的操作来说,十分方面.  

```js
db.runCommand({
    "findAndModify": "prcesses", // 字符串, 集合名
    "query": {"status": "READY"}, // 查询文档
    "sort": {"priority": -1}, // 排序结果
    "remove": true,  // 是否删除文档
    // "update": boolean , 与 remove 必须指定一个
    "new": boolean, // true 返回更新后的文档, 默认 false 更新前的文档
    "fields": {object}, // 文档中需要返回的字段
    "upsert": boolean,
})
```









- - -
