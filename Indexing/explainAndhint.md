## 使用explain() 和 hint()

### explain 2.0

explain 能够提供大量与查询相关的信息.对于速度比较慢的查询来说,这是最重要的诊断之一.  



### explain 3.0

MongoDB 3.0之后，explain的返回与使用方法与之前版本有了不少变化.  
现版本explain有三种模式，分别如下:  

- queryPlanner
    - explain() 的默认参数 , `db.foo.find({query}).explain('queryPlanner')`;
    - queryPlanner是现版本explain的 __默认模式__，queryPlanner模式下并不会去真正进行query语句查询，而是针对query语句进行执行计划分析并选出winning plan(命中计划?)
- executionStats
    - `db.foo.find({query}).explain('executionStats')`;
- allPlansExecution


##### queryPlanner

```js
> db.users.find({username: "user99"}).explain();
{
	"queryPlanner" : {
		"plannerVersion" : 1,
        // namespace 所查询的表
		"namespace" : "test.users",
        // 针对该query是否有indexfilter
		"indexFilterSet" : false,
		"parsedQuery" : {
			"username" : {
				"$eq" : "user99"
			}
		},
        // winningPlan 查询优化器针对该query所返回的最优执行计划的详细内容
		"winningPlan" : {
            // 最优执行计划的stage ??
			"stage" : "COLLSCAN",
			"filter" : {
				"username" : {
					"$eq" : "user99"
				}
			},
            // 此query的查询顺序，此处是forward
			"direction" : "forward"
		},
        // 其他执行计划（非最优而被查询优化器reject的）的详细返回
		"rejectedPlans" : [ ]
	},
	"serverInfo" : {
		"host" : "af3175d32152",
		"port" : 27017,
		"version" : "3.2.8",
		"gitVersion" : "ed70e33130c977bda0024c125b56d159573dbaf0"
	},
	"ok" : 1
}
```


### executionStats

```js
> db.users.find({username: "user99"}).explain('executionStats');
{
	"queryPlanner" : {
		"plannerVersion" : 1,
		"namespace" : "test.users",
		"indexFilterSet" : false,
		"parsedQuery" : {
			"username" : {
				"$eq" : "user99"
			}
		},
		"winningPlan" : {
			"stage" : "FETCH",
			"inputStage" : {
				"stage" : "IXSCAN",
				"keyPattern" : {
					"username" : 1
				},
				"indexName" : "username_1",
				"isMultiKey" : false,
				"isUnique" : false,
				"isSparse" : false,
				"isPartial" : false,
				"indexVersion" : 1,
				"direction" : "forward",
				"indexBounds" : {
					"username" : [
						"[\"user99\", \"user99\"]"
					]
				}
			}
		},
		"rejectedPlans" : [ ]
	},
	"executionStats" : {
		"executionSuccess" : true,
		"nReturned" : 1,
		"executionTimeMillis" : 0,
		"totalKeysExamined" : 1,
		"totalDocsExamined" : 1,
		"executionStages" : {
			"stage" : "FETCH",
			"nReturned" : 1,
			"executionTimeMillisEstimate" : 0,
			"works" : 2,
			"advanced" : 1,
			"needTime" : 0,
			"needYield" : 0,
			"saveState" : 0,
			"restoreState" : 0,
			"isEOF" : 1,
			"invalidates" : 0,
			"docsExamined" : 1,
			"alreadyHasObj" : 0,
			"inputStage" : {
				"stage" : "IXSCAN",
				"nReturned" : 1,
				"executionTimeMillisEstimate" : 0,
				"works" : 2,
				"advanced" : 1,
				"needTime" : 0,
				"needYield" : 0,
				"saveState" : 0,
				"restoreState" : 0,
				"isEOF" : 1,
				"invalidates" : 0,
				"keyPattern" : {
					"username" : 1
				},
				"indexName" : "username_1",
				"isMultiKey" : false,
				"isUnique" : false,
				"isSparse" : false,
				"isPartial" : false,
				"indexVersion" : 1,
				"direction" : "forward",
                /**
                 * 描述索引使用情况,给出索引的遍历范围,
                 */
				"indexBounds" : {
					"username" : [
						"[\"user99\", \"user99\"]"
					]
				},
				"keysExamined" : 1,
				"dupsTested" : 0,
				"dupsDropped" : 0,
				"seenInvalidated" : 0
			}
		}
	},
	"serverInfo" : {
		"host" : "af3175d32152",
		"port" : 27017,
		"version" : "3.2.8",
		"gitVersion" : "ed70e33130c977bda0024c125b56d159573dbaf0"
	},
	"ok" : 1
}
```

### 查询优化器

MongoDB 的查询优化器与其他数据库稍有不同.  
基本来说, 如果一个索引能够精确匹配到一个查询, 那么查询优化器就会使用这个索引.  
不然的话, 可能会有几个索引都适合你的查询.mongodb 会从这些可能的索引子集中为每次查询计划选择一个,
这个查询是并行执行的, 最早返回 100 个结果就是胜者, 其他的查询计划就会被终止.  

这个查询计划会被缓存, 这个查询接下来都会用它, 知道集合数据发生了比较大的变化.
