## 使用 GridFS 存储文件

GridFS 是 mongodb 的一种存储机制,用来存储大型二进制文件.  
下面列出使用 GridFS 作为文件存储的理由:  

1. __替代存储工具__ :使用 GridFS 能够简化你的栈. 可以使用 GridFS 代替独立的文件存储工具.  
2. __故障扩展方便__ :GridFS 会自动平衡已有的复制或者为 MongoDB 设置的自动分片, 所以文件存储做故障转移或者横向扩展更容易.  
3. __解决文件系统可能会遇到的问题__ : 例如, 在同一个目录下存储大量文件
4. __GridFS存储集中度会比较高__ : MongoDB 以 2GB 为单位来分配数据文件

缺点:  

1. 性能低: MongoDB 访问文件不如从文件系统访问文件速度快
2. 如果需要修改, 只能先删除,然后重新保存.
3. 将文件当做多个文档存储, 所以无法再统一事件对文件中的所有块加锁.  

通常用于不经经常改变但是经常需要连续访问的大文件  

### GridFS 入门

...