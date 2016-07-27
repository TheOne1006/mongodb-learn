## 查询条件

查询不能像前面说的那样精确匹配,还能匹配更加复杂的条件,比如 范围, OR 子句和取反.

### 查询条件

- 比较操作符:`$lt`, `$lte`, `$gt`, `$gte`
    - 对应 `<`, `<=`, `>`, `>=`
    - 组合 db.users.find({"age":{"$gte": 18, "$lte": 30}}) // age > 18 and age < 30
- `$ne` 表示不等于 `!=`
    - 能用于所有类型的数据


### OR 查询

MongoDB 有两种方式进行 OR 查询:  

1. `$in` 可以用来查询一个键的多个值
    - `$nin` 匹配不满足条件的文档
2. `$or` 更通用一些,可以在多个键值查询任何给定值
    - `$or` 接受一个包含所有可能条件的数组作为参数

`$in`案例:  
```js
// 找出号码为 725, 542, 390
> db.raffle.find({"ticket_no": {"$in": [725, 542, 390]}});

// 以下两条效果一样
> db.raffle.find({"ticket_no": {"$in": [390]}});
// 等同于
> db.raffle.find({"ticket_no": 390});
```

`$or`案例:  
```js
// 字段 winner 为 true, 或者 tocket_no 为 735 的文档
> db.raffle.find({"$or":[{"tocket_no":725}, {"winner":true}]})
```

### `$not`

`$not` 是元条件句, 即可以在任何其他条件之上.  

```js
// 所有 num != 10 的文档
> db.users.find({"num":{"$not": 10}});
```

### 条件语义




- - -
