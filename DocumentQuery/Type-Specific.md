## 特定类型的查询

MongoDB 文档可以使用多种类型的数据.其中在一些查询时会有特别的表现.  

### null

null 类型,能匹配自身.  
但是,不仅会匹配某个键值为`null` 的文档,还会匹配不包含这个键值的文档.  


如:  
```js
> db.c.find();
{ "_id" : ObjectId("579c533a223fb1fa6610983c"), "y" : null }
{ "_id" : ObjectId("579c533a223fb1fa6610983d"), "y" : 1 }
{ "_id" : ObjectId("579c533a223fb1fa6610983e"), "y" : 2 }
{ "_id" : ObjectId("579c5379223fb1fa6610983f"), "z" : null }
> db.c.find({z:null});
{ "_id" : ObjectId("579c533a223fb1fa6610983c"), "y" : null }
{ "_id" : ObjectId("579c533a223fb1fa6610983d"), "y" : 1 }
{ "_id" : ObjectId("579c533a223fb1fa6610983e"), "y" : 2 }
{ "_id" : ObjectId("579c5379223fb1fa6610983f"), "z" : null }
// 如果需要满足以上目的,应使用
// 没有 $eq 操作符,使用 $in 代替
> db.c.find({"z": {"$in":[null], "$exists":true}});
{ "_id" : ObjectId("579c5379223fb1fa6610983f"), "z" : null }
```


### 正则表达式

正则表达式能灵活有效的匹配字符串.  
MongoDB 为前缀正则表达式(如/^joe/) 查询创建索引,所以这种类型的查询会非常的高效.  


```js
> db.users.find({name:/joe/i})
```

### 查询数组

查询数组与查询标量值一样.  

```js
> db.food.find();
{ "_id" : ObjectId("579c5995223fb1fa66109840"), "fruit" : [ "apple", "banana", "peach" ] }
> db.food.find({fruit:'banana'});
{ "_id" : ObjectId("579c5995223fb1fa66109840"), "fruit" : [ "apple", "banana", "peach" ] }
```

> $all

如果需要通过多个元素来匹配数组, 就需要使用 `$all`  

`db.food.find({fruit:{'$all':'apple'}}) `等同于 `db.food.find({fruit:'apple'})`  


案例:  

```js
> db.food.find();
{ "_id" : ObjectId("579c5995223fb1fa66109840"), "fruit" : [ "apple", "banana", "peach" ] }
{ "_id" : ObjectId("579c5b5a223fb1fa66109841"), "fruit" : [ "apple", "kumquat", "orange" ] }
{ "_id" : ObjectId("579c5b8a223fb1fa66109842"), "fruit" : [ "cherry", "banana", "apple" ] }
// 查找即有 apple 又有 banana的文档
> db.food.find({"fruit": {"$all":["apple","banana"]}});
> db.food.find({"fruit": {"$all":["apple","banana"]}});
{ "_id" : ObjectId("579c5995223fb1fa66109840"), "fruit" : [ "apple", "banana", "peach" ] }
{ "_id" : ObjectId("579c5b8a223fb1fa66109842"), "fruit" : [ "cherry", "banana", "apple" ] }

// 精确匹配数组
> db.food.find({"fruit":["apple","banana"]});
> db.food.find({"fruit":["apple","banana","peach"]});
{ "_id" : ObjectId("579c5995223fb1fa66109840"), "fruit" : [ "apple", "banana", "peach" ] }

```

> `key.index`

想要查询数组特定位置的元素.

```js
// 匹配 下标为 2 的fruit 的相关字段
> db.food.find({"fruit.2":"peach"});
```

> $size

`$size`对于查询数组来说非常有用,用于查询特定长度的数组.  

- `$size` 并不能与其他查询条件($gt)组合使用
- 就是说没法查询, 大于某一`$size` 的文档

例如:  

```bash
# 查询 fruit key 的长度
> db.food.find({"fruit":{"$size":3}});
```

> $slice 操作符

- 可以返回某个键匹配的数组元素的子集.  
- 也可以指定偏移量,以及返回的元素数量.

```bash
# 查询前10条 评论
> db.blog.posts.findOne({query}, {"comments":{"$slice": 10}})
# 查询后10条 评论
> db.blog.posts.findOne({query}, {"comments":{"$slice": -10}})
# 返回第24 个 - 33个元素,如果数组不够 23 到最后的所有元素.
> db.blog.posts.findOne({query}, {"comments":{"$slice": [23, 10]}});
```

> 返回一个匹配的数组元素

```bash
> db.blog.posts.findOne();
{
	"_id" : ObjectId("579cb68e1461c98666dccc69"),
	"title" : "blog post",
	"content" : "content",
	"comments" : [
		{
			"name" : "bob",
			"email" : "bob@example.com",
			"content" : "good post"
		},
		{
			"name" : "bob",
			"email" : "bob@example.com",
			"content" : "good post2"
		},
		{
			"name" : "joe",
			"email" : "joe@example.com",
			"content" : "nice post"
		}
	]
}
# 1 === true
> db.blog.posts.findOne({"comments.name":"bob"}, {"comments.$":1});
{
	"_id" : ObjectId("579cb68e1461c98666dccc69"),
	"comments" : [
		{
			"name" : "bob",
			"email" : "bob@example.com",
			"content" : "good post"
		}
	]
}
```
> 数组和范围查询的相互作用

文档中的标量(非数组元素) 必须与查询条件中的每一条语句相匹配.  
如果字段是一个数组,可以匹配不同的数组元素.  
可以使用`$elemMatch`要求 mongodb 同时使用查询条件中的两个语句与一个数组元素进行比较.  
`min()`和`max()`
    - $elemMatch 将限定条件进行分组, 晋档需要一个内嵌文档多个键操作时才会用

案例:  

```js
> db.foo.find();
{ "_id" : ObjectId("579cba811461c98666dccc6e"), "x" : 5 }
{ "_id" : ObjectId("579cba811461c98666dccc6f"), "x" : 15 }
{ "_id" : ObjectId("579cba811461c98666dccc70"), "x" : 25 }
{ "_id" : ObjectId("579cba811461c98666dccc71"), "x" : [ 5, 25 ] }
> db.foo.find({"x":{"$gt":10, "$lt":20}});
# [5, 25] 匹配中一个元素 会返回整个文档, 使得查询查询范围对数组没做作用
{ "_id" : ObjectId("579cba811461c98666dccc6f"), "x" : 15 }
{ "_id" : ObjectId("579cba811461c98666dccc71"), "x" : [ 5, 25 ] }
# 对数组每一个元素查询
# 满足了 所有表达式的数组元素
# $elemMatch 只匹配数组元素
> db.foo.find({"x":{"$elemMatch":{"$gt":10, "$lt":20}}});
> db.foo.find({"x":{"$elemMatch":{"$gt":10, "$lt":26}}});
{ "_id" : ObjectId("579cba811461c98666dccc71"), "x" : [ 5, 25 ] }
```

### 查询内嵌文档

有两种方法可以查询内嵌文档:  

- 查询整个文档
    - 查询文档要完全等于文档的
    - 而且查询顺序必须相同
    - 如果再加入一个 middle name ,那么将不会再查询到
- 或者只针对其 键/值 对进行查询
    - `.`点 表示法也是待插入的文档键名不能包含 "." 的原因.

查询整个内嵌文档与普通查询.  

例如:  
```js
> db.people.find();
{ "_id" : ObjectId("579cbf731461c98666dccc72"), "name" : { "first" : "joe", "last" : "Schmoe" }, "age" : 32 }
// 搜索名为 joe Schmoe, 精确匹配
> db.people.findOne({"name":{"first":"joe","last":"Schmoe"}});
{
	"_id" : ObjectId("579cbf731461c98666dccc72"),
	"name" : {
		"first" : "joe",
		"last" : "Schmoe"
	},
	"age" : 32
}
// 针对键值匹配
> db.people.findOne({"name.first":"joe","name.last":"Schmoe"});
{
	"_id" : ObjectId("579cbf731461c98666dccc72"),
	"name" : {
		"first" : "joe",
		"last" : "Schmoe"
	},
	"age" : 32
}
```

注意:  
文档结构变得复杂之后需要些技巧,   
如:博客查找`joe`发表的5分以上的评论:  

```js
> db.blog.find();
{ "_id" : ObjectId("579cc8241461c98666dccc73"), "title" : "t", "content" : "c", "comments" : [ { "author" : "joe", "score" : 3, "commennt" : "nice post" }, { "author" : "mary", "score" : 6, "commennt" : "nice post2" } ] }
{ "_id" : ObjectId("579cc8561461c98666dccc74"), "title" : "t", "content" : "c", "comments" : [ { "author" : "joe", "score" : 5, "commennt" : "nice post" }, { "author" : "mary", "score" : 6, "commennt" : "nice post2" } ] }
// 使用精确查询, 必须完全匹配,所以没有结果
> db.blog.find({"comments":{"author":"joe","score":{"$gte":5}}});
// 使用 "." 查询, 返回两条结果, 因此 第一条满足 author, 第二条满足 score, 所以也可以满足条件
> db.blog.find({"comments.author":"joe","comments.score":{"$gte":5}});
{ "_id" : ObjectId("579cc8241461c98666dccc73"), "title" : "t", "content" : "c", "comments" : [ { "author" : "joe", "score" : 3, "commennt" : "nice post" }, { "author" : "mary", "score" : 6, "commennt" : "nice post2" } ] }
{ "_id" : ObjectId("579cc8561461c98666dccc74"), "title" : "t", "content" : "c", "comments" : [ { "author" : "joe", "score" : 5, "commennt" : "nice post" }, { "author" : "mary", "score" : 6, "commennt" : "nice post2" } ] }
// 使用 $elemMatch
// $elemMatch 将限定条件进行分组, 晋档需要一个内嵌文档多个键操作时才会用到
> db.blog.find({"comments":{"$elemMatch":{"author":"joe","score":{"$gte":5}}}});
{ "_id" : ObjectId("579cc8561461c98666dccc74"), "title" : "t", "content" : "c", "comments" : [ { "author" : "joe", "score" : 5, "commennt" : "nice post" }, { "author" : "mary", "score" : 6, "commennt" : "nice post2" } ] }
```
