## 命令行启动

在命令行中运行 `mongod --help` 可列入这些选项.  

- `--dbpath`
    - 指定目录作为数据目录
    - 每个 mongod 实例拥有独立的目录
    - lock 文件,防止使用相同目录
- `--port`
    - 指定服务监听端口, 默认27017
- `--fork`
    - 调用 fork 创建子进程, 在后台运行 mongodb
    - 启动 `--fork` 必须同时启动 `--logpath`
- `--logpath`
    - 所有输出信息会被发送至指定文件,而非命令行上输出.
    - `--logappend`, 保留旧日志
- `--directoryperdb`
    - 启用该选项可将每个数据库放在单独的目录中,我们可由此按照不同的数据库挂在到不同的磁盘上.
- `--config`
    - 额外加载配置文件
    - 将命令行中指定选项使用配置文件中的参数. 确保每次重启选项相同

##### 使用配置文件

mongodb 支持从文件读取配置信息.  
使用 `-f` / `--config` 标记  

```bash
mongod --config ~/.mongodb.conf

# 配置文件
port = 5586
fork = true # --fork
logpath = /var/log/mongodb.log
logappend = true
```
