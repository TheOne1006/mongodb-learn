## 索引管理

对于一个集合,每个索引只需要创建一次. 如果重复创建相同索引, 是没有任何作用的.  

所有的数据库索引信息都存在 `system.indexes` 集合中. 这是一个 __保留集合__ ,不能再其中插入或者删除  
只能通过 `ensureIndex` 或者 `dropIndexes` 对其进行操作.  

可以使用 `db.collectionName.getIndexes()` 查看指定集合上的索引信息.  

```js
> db.foo.getIndexes();
[
	{
		"v" : 1,
		"key" : {
			"_id" : 1
		},
		"name" : "_id_",
		"ns" : "test.foo"
	},
	{
		"v" : 1,
		"unique" : true,
		"key" : {
			"x" : 1
		},
		"name" : "x_1",
		"ns" : "test.foo",
		"sparse" : true
	}
]
```


### 标识索引

集合中的每一个索引都有一个名称,用于唯一标识这个索引,也可以用于服务器端来删除或者操作索引.  
索引默认形式是 keyname1_dir1_keyname2_dir2..., `keyname`是索引的键, `dirX`是索引的方向.  

如果多个键以上, 可以在 `ensureIndex` 中指定索引名称:  

```js
// 索引名称长度是有限制的,所以建立复杂做一年可能需要自定义索引名称
> db.foo.ensureIndex({"a":1,"b":1,"c":1},{"name":"alphabet"});
```

### 修改索引

随着应用的变化, 可以使用 `dropIndex` 删除不需要的索引.  

新建索引是一件费时又费资源的事情, 默认情况下, mongodb 会尽快创建索引, 阻塞所有对数据库的读请求和写请求,
一直到创建完成.  
如果希望数据库在创建索引同时依然能够处理读写请求, 可以创建索引时指定 `background` 选项.
