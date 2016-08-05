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


- - -
