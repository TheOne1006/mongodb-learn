## Time-To-Live Indexes

对于固定集合内容何时覆盖,只拥有非常有限的控制权限.  
如果需要更灵活的老化移出系统(age-out system), 可以使用 TTL 索引.  

这种索引允许:  

允许为每个文档设置一个超时时间, 当一个文档到达老化程度之后就会被删除.  

在 `ensureIndex` 中指定 `expireAfterSecs` 选项建立 TTL 索引:  

```bash
# 设置超时时间为 24 小时
> db.foo.ensureIndex({"lastUpdated":1},{"expireAfterSecs": 60 * 60 * 24 });
```

在 "lastUpdated" 创建一个 TTL 索引. 如果一个文档的 "lastUpdated" 字段存在并且他的值是日期类型,
当服务器时间比文档的 "lastUpdated" 字段的时间晚 expireAfterSecs 秒, 文档就会被删除.  

Mongodb 每分钟对 TTL 索引进行一次清理,所以精确度在分钟.  
可以使用 `collMod` 命令修改 `expireAfterSecs` 的值:  

```bash
> db.runCommand({"collMod":"someapp.cache","expireAfterSecs": 3600})
```

在一个给定集合上可以有多个 TTL 索引, TTL 索引不能是复合索引, 但是可以向 普通索引一样用来优化排序和查询.  
