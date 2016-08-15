## 日志

mongod 默认将日志发送至 stdout (标准输出, 通常为终端).  
大多数初始设置会使用 `--logpath` 将日志发送至日志文件.  

**注意**: 确保每个实例单独的的文件, 确保有文件的访问权限.  

MongoDB 会输出大量日志信息, 但不要使用 `--quiet` (隐藏部分日志消息).  
建议保持日志默认级别,日志占用空间并不大.  

首先, 重启 MongoDB 时, 可以在参数中附加数据更多的 "v"(`-v`, ....., `-vvvvv`)  
或者, 通过`set Parameter`命令,完成日志级别( log level)的更改.  

```bash
> db.adminCommand({ "setPararmeter": 1, "logLevel": 3 });
```

将日志级别重设为 0, 记录内容最少, 级别为 5 是, 几乎包含所有操作.  

### 慢查询

mongodb 默认记录耗时超过 100 毫秒的查询信息,如果 100 不适用,可以通过 `setProfilingLevel` 修改阈值:  

```bash
# 只记录耗时超过 500 毫秒的查询操作
> db.setProfilingLevel(1, 500)
{"was":0, "slowns":100, "ok":1 }
# 关闭分析器,
> db.setProfilingLevel(0)
{"was":1, "slowns":500, "ok":1 }
```

也可以使用 `--slowns` 重启 MongoDB 更改这一阈值.  

设置一个计划任务便每天或者每周分割( rotate ) 日志文件, 如果使用 `--logpath` 启动, 向进程发送一个 `SIGUSR1` 信号即使对日志进行分割, 也可以使用 `logRotate` 命令以达到相同目的:  

```bash
> db.adminCommand({"logRotate": 1});
```

如果不是通过 `--logpath` 选项启动, 则不能对日志进行分割.  
