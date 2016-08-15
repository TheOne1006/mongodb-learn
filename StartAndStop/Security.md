## 安全性

不要讲 MongoDB 直接暴露在外网.  
应尽可能限制外部访问.

最好的方式是设置防火墙, 只允许内部网络地址对 MongoDB 访问.  
除了防火墙外,也可在配置文件中加入以下选项来增强安全性.  

- `--bind_ip`
    - 指定 MongoDB 监听接口.  
    - 通常指定为内部的 IP 地址,从而保证服务器集群访问, 同时拒绝外网访问.  
    - 如果与应用在同一机器上,可以指定为 localhost
- `--nohttpinterface` no http interface
    - MongoDB 启动时, 默认在端口 1000 启动一个微型 HTTP 服务器,
    - 该服务器可提供系统信息,但是可以在其他地方找到.
    - 除非正在开发,否则应该关闭此项
- `--nounixsocket` no unix socket
    - 如果不打算使用 UNIX socket 来进行连接, 则可以禁用此项.  
    - 只有本地,才能使用 socket
- `--noscripting` no scrpting
    - 完全禁止服务器端 JavaScript 脚本运行, 尤其是 `sh.status()`.
    - 在一台禁止 JavaScript 服务器上运行这些辅助函数, 会出现错误提示.  

不要启用 REST 界面, 该界面默认禁用,开启后可在服务器上执行很多命令, 但并非为生产环境设计.  


### 数据加密

mongodb 还未提供内置数据加密机制, 如果需要对数据加密, 可以使用加密文件系统.  

### SSL 安全连接

连接至 MongoDB 传输数据默认不被加密, 然而, mongodb 支持 SSL 连接.  
默认版本未包含 SSL , 可以从 <http://www.10gen.com> 下载一个支持 SSL 版本.  




- - -
