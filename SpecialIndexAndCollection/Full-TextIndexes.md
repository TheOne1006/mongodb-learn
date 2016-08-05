## 全文本索引

Mongodb 有一种特殊类型的索引用于 __在文档中搜索文本__.  

使用精确匹配和正则式来查询字符串, 但是这些技术有一些限制:  
正则搜索大块文本的速度非常慢, 而且无法处理语言的理解问题(??)

创建任何一种索引的开销都比较大, 而创建全文本索引成本更高.  

全文本索引也会导致比"普通"索引更严重的性能问题,
因为 __所有字符串都需要被分解,分词,并保存到一些地方__.  
因此 __全文本索引的集合写入速度要差__ .


3.0 如何设置 ??  