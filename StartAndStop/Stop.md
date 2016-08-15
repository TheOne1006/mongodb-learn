## 停止 MongoDB

安全停止与安全启动一样重要.  

关闭运行中的服务器,最简洁的方法是使用 `shutdown` 命令 -- `{"shutdown":1}`  

这是一个管理员命令, 必须运行在 admin 数据库上,
shell 提供了一个附属函数,用以简单的执行 shutown  

```bash
> use admin
> db.shutdownServer();
server should be down ....
```

在主节点( primary ) 运行 shutdown 时, 服务器在关闭前,会先等待备份节追赶主节点以保持同步.  
这将回滚的可能性降到最低, 但 shutdown 操作有失败的可能性. 如果几秒内没有备份节点成功同步,则 shutdown 操作失败.  

使用 force 选项,强制关闭主节点:

```bash
db.adminCommand({"shutdown":1, "force":true})
```
