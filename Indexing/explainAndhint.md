## 使用explain() 和 hint()


### explain

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
        ...
	},
	"executionStats" : {
		"executionSuccess" : true,
		"nReturned" : 1,
        // 该query的整体查询时间
		"executionTimeMillis" : 104,
		"totalKeysExamined" : 0,
		"totalDocsExamined" : 100000,
		"executionStages" : {
			"stage" : "COLLSCAN",
			"filter" : {
				"username" : {
					"$eq" : "user99"
				}
			},
			"nReturned" : 1,
			"executionTimeMillisEstimate" : 80,
			"works" : 100002,
			"advanced" : 1,
			"needTime" : 100000,
			"needYield" : 0,
			"saveState" : 782,
			"restoreState" : 782,
			"isEOF" : 1,
			"invalidates" : 0,
			"direction" : "forward",
			"docsExamined" : 100000
		}
	},
	"serverInfo" : {
        ...
	},
	"ok" : 1
}
```


- - -
