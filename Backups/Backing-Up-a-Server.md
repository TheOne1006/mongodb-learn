## 对服务器进行备份

备份有很多方法. 无论采用哪种方法,都会增加系统的负担: 备份通常需要将所有数据读取到内存中.  

因此, 通常情况下,应对副本集的非主节点进行备份, 或者空闲时段对独立服务器备份.  


- 快照
- 复制数据文件

### 文件系统快照 (snapshot)

生成文件系统快照, 是 __最简单__ 的备份方法.  
然而,该方法的实现需要两点条件, 即文件系统本身支持快照技术, 以及在运行 mongod 时必须开启日志系统(journaling).  

恢复时, 确保 mongod 没有在运行. 从快照恢复数据的确切命令取决于不同的文件系统, 不过基本上就是恢复快照,
然后启动 mongod .  

如果对正在运行的系统生成快照, 那么快照的数据内容本质上相当于使用 `kill -9` 强制终止 mongod 后的数据内容.  

### 复制数据文件

另一种备份方式是复制数据目录中的所有文件. 没有文件系统的的支持, 我们就无法同时复制所有文件,因此在进行备份是必须方式数据文件发生变化. 可以使用 `fsynclock` 命令做到这一点:  

```bash
> db.fsyncLock();
```

该命令锁定(lock)数据库, 禁止任何写入, 并进行同步(fsync) , 即将所有脏页刷新至磁盘, 以确保数据目录中的文件是最新的, 且不会被改变.  

一旦运行着命令, mongod 会将之后的所有写入操作加入 队列等待, 且在解锁前不会对这些写入操作进行处理.  

```bash
$ cp -R /data/db/* /mnt/backup
```

确保复制了目录中的每一个文件和文件夹备份到的位置,漏掉文件或者文件夹可能会损坏备份.  

```bash
# 解锁数据库
> db.fsyncUnlock();
```

**注意**:  

身份验证和 fsncLock 命令存在一些锁定问题, 如果启用 身份验证, 则在调用 fsyncLock() 和 fsyncUnlock() 期间不要关闭 shell.
如果这期间断开了连接,则可能无法重新连接, 并不得不重启 mongod.  
fsyncLock 设定重启后不会保持生效, mongod 总是以非锁定模式启动.  

还可以关闭 mongod 然后复制, 效果也是一样.  

不要同时使用 fsyncLock 和 mongodump . 数据库被锁定也许会使得 mongodump 永远处于挂起状态.  

### mongodump

__缺点__:  
- 备份和恢复速度慢, 在处理副本集是存在一些问题.  
- 非快照, 无法确保实时性

__优点__:  
- 单独数据库,集合设置集合中的子集是, mongodump 是个很好的选择

##### 帮助

```bash
mongodump --help
```

##### 选项

- `mongodump -p`
- `mongodump --dbpath`
- `mongodump --oplog` 如果 mongod 启动时使用了 `--replSet`
    - 将存储过程中服务器进行的所有操作记录下来, 这样恢复备份是就会重新执行这些操作.


##### 操作

如果同一台机器上运行 mongod 和 mongodump ,只需指定 mongod 运行时监听端口即可:  

```bash
mongodump -p 31000
```

mongodump 会在当前目录建立一个 _转储(dump)_ 目录, 其中包含 一份所有数据倾卸.  
转储目录中的目录和子目录由数据库和集合构成.  
真正的数据存放在扩展名为 `.bson` 文件中, 其中以 BSON 格式一次存储了集合中的所有文档.  
可使用 Mongodb 自带的 bsondump 工具查看 .bson 文件

服务器无运行状态, 用 `--dbpath` 指定数据目录:  

```bash
mongodump --dbpath /data/db
```

##### 恢复 mongorestone

恢复 mongodump 产生的备份, 使用 mongorestore 工具

```bash
$ mongorestore -p 31000 --oplogReplay dump/
```

如果转储数据库 使用 `--oplog` 参数,运行 mongorestore 时必须使用 `--oplogReplay`

如果在运行的服务器上进行数据替换, 使用 `--drop` 选项,以在恢复一个集合前先删除它.  




- - -
