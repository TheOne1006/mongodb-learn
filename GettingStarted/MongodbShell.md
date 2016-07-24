## 使用 MongoDb Shell

- 如何将 shell 作为命令行工具的一部分
- 如何对 shell 进行定制
- 一些 shell 的高级功能

可以通过 shell 连接到任何 MongoDB 实例.

```bash
# host 远程连接地址
# 30000 mongodb port
# myDB 指定 myDB 数据库
$ mongo host:30000/myDb

## --nodb 参数 启动 shell, 启动时就不会连接任何数据库
```

在通过 `--nodb` 启动 mongodb 后, 需要运行 new Mongo(hostname),在连接到 想要的 mongod:  

```js
> conn = new Mongo("some-host:30000");
connection to some-host:30000
> db = conn.getDB("myDB");
myDB
```

### shell 小贴士

mongo 是一个简化的 JavaScript Shell 可以通过查看 JavaScript 在线文档得到大量帮助,
对 Mongodb 特有功能, shell 内置了帮助文档,可以使用 help 命令查看.

```js
> help
	db.help()                    help on db methods
	db.mycoll.help()             help on collection methods
	sh.help()                    sharding helpers
	rs.help()                    replica set helpers
	help admin                   administrative help
	help connect                 connecting to a db help
	help keys                    key shortcuts
	help misc                    misc things to know
	help mr                      mapreduce

    show dbs                     show database names
    show collections             show collections in current database
    show users                   show users in current database
    show profile                 show most recent system.profile entries with time >= 1ms
    show logs                    show the accessible logger names
    show log [name]              prints out the last segment of log in memory, 'global' is default
    use <db_name>                set current database
    db.foo.find()                list objects in collection foo
    db.foo.find( { a : 1 } )     list objects in foo where a == 1
    it                           result of the last line evaluated; use to further iterate
    DBQuery.shellBatchSize = x   set default number of items to display on shell
    exit                         quit the mongo shell
```

通过 db.help() 查看 数据库级别帮助, 使用 db.foo.help() 查看集合级别的帮助

如果想知道函数是做什么用的, 可以直接在 shell 命令中输入函数名,可以看到相关的 函数 JavaScript 实现代码:

```js
show logs                    show the accessible logger names
show log [name]              prints out the last segment of log in memory, 'global' is default
use <db_name>                set current database
db.foo.find()                list objects in collection foo
db.foo.find( { a : 1 } )     list objects in foo where a == 1
it                           result of the last line evaluated; use to further iterate
DBQuery.shellBatchSize = x   set default number of items to display on shell
exit                         quit the mongo shell
...
```


### 使用 shell 执行脚本

mongo shell 会一次执行传入脚本.  

在脚本中可以访问 db 变量,以及其他全局变量. 然而 在, __shell 辅助函数(比如"use db" 和 "show collections")__ 不可以在文件中使用.  

这些辅助函数都有对应的 JavaScript 函数

1. use foo
    - db.getSisterDB("foo")
2. show dbs
    - db.getMongo().getDBs();
3. show collections
    - db.getCollectionNames()

> 脚本的使用

可以将变量注入 shell 例如:  
1. 简化脚本初始化的辅助函数
2. 脚本还将通用的任务和管理活动自动化
3. 默认情况 shell 会从当前目录中查找脚本 (可以使用 `run("pwd")`)
4. 使用 load('/path/to/file') 加载文件
    - 可以使用绝对路径
    - 不会解析 `~`

### 创建 `.mongorc.js` 文件

如果某些脚本被频繁加载, 可以将他们 添加到 .mongorc.js 文件中. 这个文件会在 shell 启动时自动运行.  
在用户主目录下创建  

实用功能:  
1. 创建一些自己需要的全局变量,
2. 为太长的名字创建一个简短的别名
3. 重写内置代码.
    - 最常见用途之一,移除哪些比较危险的 shell 辅助函数, 比如 `dropDatabase` , `detleteIndexs`
4. 在启动 shell 时 指定 `--norc` 参数,就可以进制加载 .mongorc.js



重写 __危险__ 方法:  
- 改变数据库函数时,要确保 db 变量和 DB 原型进行改变.
- 如果只改变一个, 那么 db 变量可能没有改变, 或者 DB 原型没有改变,那么切换 database 之后也不会修改
- 这种方式不能保证恶意用户攻击, 只能预防自己 "手贱"

```js
var no = function () {
    print("Not on my watch");
}

// 禁止删除数据库
db.dropDatabase = DB.protptype.dropDatabase = no;

// 进制删除集合
DBCollection.protptype.drop = no;

// 进制删除索引
DBCollection.prototype.dropIndex = no;
```

### 定制 shell 提示

将 `prompt` 变量设定为一个字符串或者函数, 这样就可以重写默认 shell 提示.  

```js
> prompt = function() {
... return (new Date()) + "> ";
... }

Sun Jul 24 2016 12:31:12 GMT+0000 (UTC)>
```

> 注意,提示函数应该 返回字符串, 而且英爱小新谨慎处理异常: 如果出现异常会对用户造成困扰!

通常来说,提示函数中应该包含对 `getLastError` 的调用. 这样就可以捕获 数据库错误,
而且可以在 shell 断开时自动重新连接

### 编辑符合变量

shell 的多行支持是非常有限的: 不可以编辑之前的行.  
为方便调试编辑器, 可以在 shell 中设置 EDITOR 变量(也可以在变量环境中设置)

```js
> EDITOR="/usr/bin/emasc"

// 如果要编辑一个变量,可以使用 "editor 变量名" ,这个 命令

> var wap = {foo: 1}
> edit wap

// 可以在 .mongorc.js 中添加一行 编辑器内容
```

### 集合命名注意事项

- 可以使用 `db.collectionName` 获取一个集合的内容
    - 但是集合名中包含 保留字或者 无效的 JavaScript 属性名称,db.collectionName 就不能正常工作.  
- 如果要访问 `version` 集合, 不能使用 `db.version`
    - 必须要使用 `getCollection()` 函数: `db.getCollection('version')`
    - 如果 集合名称包括无效 JavaScript 属性名, 也可以通过这个函数来访问相应的结合
