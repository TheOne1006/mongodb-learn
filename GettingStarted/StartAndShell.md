## Start Mongodb (启动 Mongodb)

通常, Mongodb 作为网络服务器来运行, 客户端连接到该服务器并执行操作.

### with docker

这里使用 docker 启动  

```bash
# daocloud.io/mongo 使用 daocloud 镜像
$ docker run --name some-mongo -d daocloud.io/mongo

# 存储位置
$ docker run --name some-mongo -v /my/own/datadir:/data/db -d daocloud.io/mongo:tag

# 使用 mongo 命令
$ docker run -it --link some-mongo:mongo --rm daocloud.io/mongo sh -c 'exec mongo "$MONGO_PORT_27017_TCP_ADDR:$MONGO_PORT_27017_TCP_PORT/test"'
```

## Shell

### 运行 shell

> 成功连接后

```
MongoDB shell version: 3.2.8
connecting to: 172.17.0.4:27017/test
Welcome to the MongoDB shell.
...
```

shell 是一个功能完备的 JavaScript 解释器,可以运行任意 JavaScript 程序.  

> 注意!  
    可使用多行命令, shell 会检测输入的 JavaScript 语句是否完整,如果没有完整可以在下行接着写.
    在某行连续三次摁下回车键可取消未输入完成的命令, 并退回到提示符

### Mongodb客户端

shell 的强大之处在于,它是一个独立的 MongoDB 客户端.  
启动时, shell 会连接到 MongoDB 服务器的 test 数据库, 并将数据库 __连接赋值给全局变量 db__.  

通过 `db` 命令查看执行那个数据库  

> 语法糖

为方便习惯使用 SQL shell 用户, shell 还包含了一些非 JavaScript 语法的扩展.
这些扩展不提供额外的功能, 而是一些 _语法糖_.   

例如重要的操作之一为选择数据库:

```bash
> use foo
switched to db foo
# 再次通过 db 查看 db 变量的指向
> db
foo
> db.baz
foo.baz
```
因为这是一个 JavaScript Shell , 一个变量会转化成字符串(即数据库名) 并打印出来.  

通过 db 变量,可访问其中的集合. 例如,通过 db.baz 可以返回当前数据库的 baz 集合.
因为通过 shell 可以访问集合, 这意味,几乎所有的数据库操作都可以通过对 shell 完成

### shell 中的基本操作

- create
- read
- update
- delete

#### 创建

insert 函数可将一个文档添加到集合中.

例如:  
```
#  mongo version: 3.2.8
> person = { "firstName": "king", "secondName":"theone", "date":new Date()};
{
	"firstName" : "king",
	"secondName" : "theone",
	"date" : ISODate("2016-07-23T04:25:59.125Z")
}
# 在 当前数据库的 company 集合中插入 person
> db.company.insert(person);
WriteResult({ "nInserted" : 1 })
# 查询当前数据 company 结合中的数据
> db.company.find();
{
    "_id" : ObjectId("5792f23d6cca423c020b693a"),
    "firstName" : "king",
    "secondName" : "theone",
    "date" : ISODate("2016-07-23T04:25:59.125Z")
}

```

#### 读取

find() 和 findOne() 方法可以用于查询集合里的文档, 若只查看一个文档可用 findOne():

```js
> db.company.findOne();
{
	"_id" : ObjectId("5792f23d6cca423c020b693a"),
	"firstName" : "king",
	"secondName" : "theone",
	"date" : ISODate("2016-07-23T04:25:59.125Z")
}
```
find 和 findOne 可以接受一个 _查询文档_ 作为限定条件.
这样就可以查询符合一定的条件的文档.  

> 注意 shell 中的 find, 最多显示最多20个匹配的文档

#### 更新

使用 update 更新数据, update __至少__ 接收两个参数:  

- 第一个是限定条件(用于匹配待更新文档)
- 第二个是新文档,假设我们我们要为先前写的文章增加评论功能, 就需要增加一个新的键,用于保存/更新.

例如:  
```js
// person 为之前代码使用的变量
> person.age = 18;
18
> db.company.update({firstName: "king"}, person);
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
> db.company.findOne();
{
	"_id" : ObjectId("5792f23d6cca423c020b693a"),
	"firstName" : "king",
	"secondName" : "theone",
	"date" : ISODate("2016-07-23T11:04:20.601Z"),
	"age" : 18
}
```

#### 删除

使用 remove 方法将文档从数据库中永久删除,   
如果没有任何参数,它会将集合内的所有文档全部删除, 它可以接受一个座位限定条件的文档作为参数.
