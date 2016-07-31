## 游标

数据库使用游标返回 find 执行结果. 客户对游标的实现能够对最终结果进行有效的控制.  
可以限制结果的数量,滤过部分结果,根据任意键顺序的组合对结果进行各种排列,或者执行其他一些强大的操作.  

想要从 shell 中创建一个游标,首先要对集合填充一些文档,然后对其执行查询,
并且将结果分配给一个局部变量.  

> 好处:  

可以一次查看一条记过,如果将结果放到 全局变量 或者就没放在变量中, Mongodb shell 会自动迭代, 自动显示最开始的若干文档.  

- 调用 find 时, shell 并不立即查询数据库,而是等待真正开始要求 _获得结果时才放查询_
- 这样在执行之前可以给查询附加额外的选项.  
- 几乎游标对象的每个方法都会返回游标本身, 这样就可以按任意顺序组成方法链.  
- `cursor.hasNext()` 查询立刻发往服务器. shell 立刻获取前 100 条结果或者前 4MB 数据(两者最少)
    - 这样下次 next 或者 hasNext 就没必要再次连接服务器获取结果了.
    - 客户端用完第一组结果, shell 会再一次连接数据库,
    - 使用 `getMore` 请求提取更多结果.
    - getMore 请求包含一个查询标识符, 向数据库询问是否还有更多结果,如果有,则返回下一批结果.知道游标耗尽


案例:  

```js
/**
 * 创建一个简单的集合
 */
> for(i = 0; i < 100; i++) {
    db.collection.insert({x:i});
}
/**
 * 使用 cursor 变量保存结果
 */
> var cursor = db.collection.find();

/**
 * 要迭代结果,可以使用游标的 next() 方法.
 * 也可以使用 hasNext() 来查看游标中是否有其他结果.
 */
> while(cursor.hasNext()) {
    obj = cursor.next();
    print(obj.x);
}
/**
 * 0
 * 1
 * 2
 * 3
 */
```

方法链案例:  

```js
/**
 * 以下几种表达是等价的
 */
> var cursor = db.foo.find().sort({"x":1}).limit(1).skip(10);
> var cursor = db.foo.find().limit(1).sort({"x":1}).skip(10);
> var cursor = db.foo.find().skip(10).limit(1).sort({"x":1});
// 此时还没有真正的查询, 这些函数都只是构造查询.
// 假设我么执行如下操作
> cursor.hasNext(); // 这时查询被发往服务器.
```


### limit, skip 和 sort

最常用的查询选项就是 __限制返回结果数量(limit)__ , __忽略一定数量的结果(skip)__ , __排序__  
所有这些选项一定要在查询被 __发送到服务器结果前指定__ .  

> limit(number)

限制返回数量

> skip(number)

忽略 number 个文档,返回余下的文档.   

__略过过多会导致性能问题__

> sort(object)

sort 接受一个对象作为参数, 这个对象是一组 键值对, 键对应文档的键名, 值代表排序方向.  
排序方向 1(升序) 或者 -1(降序).  
如果指定多个键, 则按照这些键指定的顺序逐个排序.  

如:  

```js
// username 升序 age 降序
> db.c.find().sort({username:1,age:-1});
```

##### 比较顺序

MongoDB 处理不同类型的数据是有一定顺序的.  
有时一个键可能有多种类型的, 例如 整型,布尔,字符串,null.  
如果对这种混合类型排序,其排序顺序是预先定义好的.优先级从小到大,其顺序如下:  

1. 最小值
2. null
3. 数字(整型,长整型,双精度)
4. 字符串
5. 对象/文档
6. 数组
7. 二进制数据
8. 对象 ID
9. 布尔型
10. 日期型
11. 时间戳
12. 正则表达式
13. 最大值

### 避免使用 skip 略过大量结果

使用 skip 略过 少量文档还是不错的. 但是要略过数量非常多的话, skip 就会变得很慢,
因为 __要先找到被忽略的数据__, 然后在抛弃这些数据. 大多数数据库都会在索引中保存更多的元数据,用于处理 skip, 但是 MongoDB 目前还不支持, 所以要尽量避免过多的数据.  

> 1. 不用 skip 对结果分页

最简单的分页方法就是用 limit 返回结果的第一页,然后将后续页面作为相对于开始的偏移量返回.

```js
// 不要这么做
var page1 = db.foo.find(criteria).limit(100);
var page2 = db.foo.find(criteria).skip(100).limit(100);
var page3 = db.foo.find(criteria).skip(200).limit(100);

/**
 * 一般来讲可以找到一种方法不适用 skip 的情况下实现分页, 这取决于查询本身.例如 按照 "date" 降序文档列表
 */
var page1 = db.foo.find().sort({"date":-1}).limit(100);
// 根据最后一个文档中的 date 的值作为查询条件, 用来查询下一页
// 显示第一页
var latest = null;
while (page1.hasNext()) {
    latest = page1.next();
    display(latest);
}

// 获取下一页
var page2 = db.foo.find({"date": {"$gt": latest.date}}).sort({"date":-1}).limit(100);
```

问题:  

1. date 相等怎么办?

> 2. 随机选取文档

从集合随机挑选一个文档是常见问题.  

最笨方法(低效):  
```js
> var total = db.foo.count();
> var randon = Math.floor(Math.random() * total);
> db.foo.find().skip(random).limit(1);
```

只要在创建数据时,写入一个 random 随机数

```js
> db.people.insert({"name":"joe","random":Math.random()});
> db.people.insert({"name":"john","random":Math.random()});
> db.people.insert({"name":"jim","random":Math.random()});
// 使用时
> var random = Math.random();
> result = db.foo.findOne({"random":{"$gt":random}});
// 如果随机数比较极端,没有找到, 可以换个方向
> if( result == null) {
    result = db.foo.findOne({"random":{"$lt":random}});
}
```

### 高级查询选项

有两种类型的查询:  

- 简单查询( plain query)
    - 如 `var cursor = db.foo.find({"foo":"bar"})`
- 封装查询(wrapped query)
    - 如 `var cursor = db.foo.find({"foo":"bar"}).sort({"x":1})`
    - 实际不是将`{"foo":"bar"}` 作为查询直接发送给数据库,
    - 而是将查询封装在一个更大的文档中.
    - 封装成`{"$query":{"foo":"bar"},"$orderby":{"x":1}}`

绝大多数驱动程序提供了辅助函数,用于查询中添加各种选项. 下面列举了其他一些有用的选项.  

- `$maxscan`: integer
    - 指定本次中扫描文档数量的上限
    - 如: `db.foo.find(criteria)._addSpecial("$maxscan",20)`
- `$min`: document
    - 查询的开始条件, 在这样的查询中,文档必须与索引的键完全匹配. 查询中会前置使用给定索引.  
    - 内部使用 ??
- `$max`: document
    - 查询结束条件.
    - 内部
- `$showDiskLoc`: true
    - 查询结果中添加一个 `$diskLoc` 字段,用于显示该条结果在磁盘上的位置.
    - 如: `> db.foo.find()._addSpecial('$showDiskLoc',true);`
    - result: `{ "_id" : ObjectId("579cd0771461c98666dccc75"), "apple" : 1, "banana" : 6, "peach" : 3, "$recordId" : NumberLong(10) }`


### 获取一致结果

##### snapshot 快照

使用快照,查询就在索引桑遍历执行,确保每个文档只被返回一次.  
快照会使得查询变慢.  

所有返回单批结果查询都被有效的进行了快照. 当游标正在等待下批结果时,如果集合发生变化,数据才可能出现不一致.  


```js
> db.foo.find().snapshot();
```


### 游标的声明周期

看待游标有两种角度:  

- 客户端游标
- 客户端游标代表的数据库游标.

服务器端,游标消耗内存和资源. 游标遍历尽了结果以后,或者客户端发来消息要求终止,数据库将会释放这些资源.  

游标如何清理:  

1. 游标匹配结果的迭代时, 它会清理自身
2. 客户端的游标已经不再作用域内, 驱动程序会想服务器发送一条特别的消息,让其销毁.
3. 几遍用户没有迭代完成, 如果一个游标在 10 分组内没有使用, 也会自动销毁.  
4. 多数驱动程序实现了一个 `immortal` 的函数,或者类似机制. 告知数据库不要让游标超时.  
