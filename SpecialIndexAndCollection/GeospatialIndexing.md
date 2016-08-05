## 地理空间索引

MongoDB 支持集中地理空间索引.  

- 最常用的是 `2dsphere` 索引(用于地球表面类型的地图)
- 2d 索引 平面地图和事件连续的数据.  

2dsphere 允许 GeoJSON 格式, 指定 点,线,多边形.  
点可以用形如 \[longitude, latitude\] `([经度, 维度])`

案例:  

```js
// 点
{
    "name": "New York City",
    "loc": {
        "type": "Point",
        "coordinates": [50, 2]
    }
}
// 线
{
    "name": "Hudson River",
    "loc": {
        "type": "Line",
        "coordinates": [[0,1], [0,2], [0,3]]
    }
}
// 多边形
{
    "name": "New England",
    "loc": {
        "type": "Polygon",
        "coordinates": [[0,1], [0,2], [1,2]]
    }
}
```

在 ensureIndex 使用 "2dsphere" 选项可以创建一个地理空间索引:  

```bash
> db.world.ensureIndex({"loc":"2dsphere"});
```

### 地理空间查询的类型

可以使用多种不同类型的地理空间查询:   

- 交集(intersection)
- 包括(within)
- 接近(nearness)

查询是, 需要将希望查找到的内容指定为形如 `{"$geometry": geoJsonDesc}` 的 GeoJSON 对象.  

例如,可以使用 `$geoIntersects` 操作符 找出与查询位置相交的文档:  

```js
// 首尾连接, [[]]
> db.world.insert({
    "name":"land3",
    "loc": {
        type: "Polygon",
        coordinates: [[
            [0, 0],
            [0, 60],
            [60, 60],
            [60, 0],
            [0, 0],
        ]]
    }
});

> var leftTop = {
    "type": "Polygon",
    "coordinates": [[
        [0,0],
        [30,0],
        [30,30],
        [0,30],
        [0, 0]
    ]]
};
> db.world.find({"loc":{"$geoIntersects":{"$geometry": leftTop}}})
// 找到所有与 leftTop 交集的文档
{ "_id" : ObjectId("57a4ae2021fe32b44dc59ce8"), "name" : "land2", "loc" : { "type" : "Polygon", "coordinates" : [ [ [ 0, 0 ], [ 0, 60 ], [ 60, 60 ], [ 60, 0 ], [ 0, 0 ] ] ] } }

> var bigScope = {
    "type": "Polygon",
    "coordinates": [[
        [-1,-1],
        [70,-1],
        [70,70],
        [-1,70],
        [-1, -1]
    ]]
};
> db.world.find({"loc":{"$within":{"$geometry": bigScope}}});
// 查找在 完全包含在  bigScope 中的文档
{ "_id" : ObjectId("57a4ae2021fe32b44dc59ce8"), "name" : "land2", "loc" : { "type" : "Polygon", "coordinates" : [ [ [ 0, 0 ], [ 0, 60 ], [ 60, 60 ], [ 60, 0 ], [ 0, 0 ] ] ] } }

> var nearScope = {
    "type": "Point",
    "coordinates": [
        -2, -2
    ]
};

> db.world.find({"loc":{"$near":{"$geometry": nearScope}}});
// 附近的位置
// $near 是唯一一个 会对查询进行排序的地理操作符号, $near 结果 按照由近及远
{ "_id" : ObjectId("57a4ae2021fe32b44dc59ce8"), "name" : "land2", "loc" : { "type" : "Polygon", "coordinates" : [ [ [ 0, 0 ], [ 0, 60 ], [ 60, 60 ], [ 60, 0 ], [ 0, 0 ] ] ] } }
{ "_id" : ObjectId("57a4a9d621fe32b44dc59ce2"), "name" : "land", "loc" : { "type" : "Point", "coordinates" : [ 122.53233, 52.968872 ] } }
```

### 复合地理空间索引

如果有其他类型的索引, 可以将地理空间索引 与 其他字段组合在一起使用, 以便对更复杂的查询进行优化.  


### 2d 索引

对于非球面地图(游戏地图, 时间连续的手等) ,可以使用 `2d` 索引代替 `2dsphere`  

`2d` 索引用于扁平表面, 而不是球体表面.  

文档中应该包含两个元素的数组表示 2d 索引字段.  

"2d" 索引 __只能对点进行索引__.  
可以保存一个由点组成的数组, 但是它只会被保存为由点组成的数组, 不会被当成线.  

默认情况下, 地理索引的值都介于 -180 ~ 180 . 可以根据需要在`ensureIndex` 设置最大值,最小值

```bash
# 创建一个 2000 * 2000 的空间索引
> db.map.ensureIndex({"foo":"2d"},{"min":-1000,"max":1000})
```

2d 查询比 2dsphere 简单许多. 可以直接使用 `$near` 或者 `$within` 而不必带有子对象 "$geometry"  

`$within` 可以查询出某个形状(矩形,圆形,多边形):  

- `$box:[[10,20], [15,30]]`: 第一个元素左下角, 第二个元素指定右上角
- `$center:[[12,25], 5]`: 第一个元素指定圆心, 第二个参数用于指定半径
- `$polygon:[[0,20],[10,0],[-10,0]]`: 数组组成多边形




```bash
# 海拉尔 大陆
> db.hyrule.ensureIndex({"tile": "2d"});
```
