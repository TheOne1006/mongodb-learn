## 删除文档

```js
> db.foo.remove();
```

上述命令会删除 foo 集合的所有文档. 但是不会删除集合本身,也不会删除集合的元信息.  
remove 函数接受一个查询文档作为参数.  
删除数据是永久性的,不能撤销也不能恢复  

### 删除速度

删除文档通常很快,但是要清空整个集合, 那么使用 `drop` 直接删除集合更快(然后在空集合上重建各种索引).  

例如:  
```js
> for(var i = 0; i < 100000; i++) {
    db.tester.insert({"foo":"bar","baz":i, "z": 10 - i});
}
// 使用 remove 删除
> db.tester.remove({})
// 使用 drop() 删除整个文档
> db.tester.drop();
true
```

db.tester.drop() 更快  
