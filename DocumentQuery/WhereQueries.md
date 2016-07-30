## $where 查询

键/值表达力非常好,但是依然有些需求无法表达.   
这时候就需要 `$where` 登场了.  

用 `$where` 可以在查询中执行任意 JavaScript. 这样就能在查询中做(几乎)任何事情.  
为了安全起见,应该 __严格限制__ 或者 __消除__ `$where` 语句使用.  
应该禁止终端用户使用任意的 `$where` 语句.  

`$where`最常见的应用就是比较稳当的两个键值是否相等.

- 非必要时,避免使用 `$where`
- 因为速度上比常规查询慢很多.
    - 因为每个文档要从 `BSON` 转换成 JavaScript 对象,然后再通过 `$where` 表达式来运行.


案例:  

```js
> db.foo.find();
{ "_id" : ObjectId("579cd0771461c98666dccc75"), "apple" : 1, "banana" : 6, "peach" : 3 }
{ "_id" : ObjectId("579cd07c1461c98666dccc76"), "apple" : 1, "banana" : 3, "peach" : 3 }
/**
 * 希望返回两个键 具有相同同值的文档
 */
> db.foo.find({"$where": function(){
    for(var current in this) {
        for(var other in this) {
            if(current != other && this[current] == this[other]) return true;
        }
    }
    return false;
}});
// result:
{ "_id" : ObjectId("579cd07c1461c98666dccc76"), "apple" : 1, "banana" : 3, "peach" : 3 }
```


### 服务器端脚本

在服务器上执行 JavaScript 时必须注意安全性,如果使用不当,服务器端 JavaScript 很容易受到诸如攻击, 与关系型数据库的注入攻击类似.  
不过,只要接受输入时遵从一些规则,就可以安全的使用 JavaScript.  
也可以在运行 mongod 时指定 `--noscripting` 选项, 完全关闭 JavaScript 的执行.  

避免 JavaScript 还可以使用作用域来传递 变量 的值.  

> shell 没有包含作用域代码类型,所以作用域 只能在字符串或者 JavaScript 函数中使用
