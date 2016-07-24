## 插入并保存文档

插入是向 MongoDB 中添加数据的基本方法. 可以使用 `insert` 方法向目标集合插入一个文档:  

```js
db.foo.insert({"bar":"baz"});
WriteResult({ "nInserted" : 1 })
```

如果文档会自动添加一个 `_id` 键(如果原来没有),然后将其保存到 MongoDB 中.

### batchInsert 批量插入

> batchInsert mongodb新版的v3.2.3中，batchInsert已经被废弃掉了

使用批量插入,可以将一组文档传递给数据库.  

在 shell 中,可以使用 insert 函数实现批量插入,
只是它接受的是一个文档数组作为参数:

demo:  
```js
> arr = [{"foo":"1"},{"foo":"2"},{"foo":"3"}];
[ { "foo" : "1" }, { "foo" : "2" }, { "foo" : "3" } ]
> db.baz.insert(arr);
BulkWriteResult({
	"writeErrors" : [ ],
	"writeConcernErrors" : [ ],
	"nInserted" : 3,
	"nUpserted" : 0,
	"nMatched" : 0,
	"nModified" : 0,
	"nRemoved" : 0,
	"upserted" : [ ]
})
> db.baz.find();
{ "_id" : ObjectId("5794d2cd5f99be330d49da38"), "foo" : "1" }
{ "_id" : ObjectId("5794d2cd5f99be330d49da39"), "foo" : "2" }
{ "_id" : ObjectId("5794d2cd5f99be330d49da3a"), "foo" : "3" }
```

当前版本MongoDb 最大接受消息是 ??  

如果执行批量插入过程,有一个文档插入失败,那么这个文档之前的所有文档会成功插入到集合,
而之后的文档都全部插入失败.  

希望忽略错误并且继续执行 ??

### 插入校验

- 插入数据时, MongoDB 只对数据进行 __最基本的检查__
- 检查文档的基本结构, `_id` 字段, 如果没有就添加.
- 检查大小, 所有文档小于 16M (未来可能增加)

由于 MongoDB 只进行基本的检查,所以插入非法数据很容易.  
因此,应该只允许新人的源连接数据库.  
主流语言的所有驱动程序,都会在数据插入到数据库之前做大量的数据校验(如 文档是否过大, 是否包含非 UTF-8 字符串, 是否使用不可识别类型)



- - -
