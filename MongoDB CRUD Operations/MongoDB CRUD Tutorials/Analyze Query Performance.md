[목록](https://github.com/yuby/mongodb-ko)


#Analyze Query Performance

[cursor.explain("executionStats")](https://docs.mongodb.org/manual/reference/method/cursor.explain/#cursor.explain) 와 [db.collection.explain("executionStats")](https://docs.mongodb.org/manual/reference/method/db.collection.explain/#db.collection.explain) 메서드는 쿼리 성능의 통계정보를 제공해 줍니다. 이는 인덱스를 어떤식으로 사용할것인지를 측정하는데 매우 유용합니다.

[db.collection.explain()](https://docs.mongodb.org/manual/reference/method/db.collection.explain/#db.collection.explain) 는 [db.collection.update()](https://docs.mongodb.org/manual/reference/method/db.collection.update/#db.collection.update)같은 작업의 정보를 제공합니다. 더 많은 정보를 [db.collection.explain()](https://docs.mongodb.org/manual/reference/method/db.collection.explain/#db.collection.explain)에서 확인하세요.


##Evaluate the Performance of a Query

inventory에 존재하는 documents입니다.
```
{ "_id" : 1, "item" : "f1", type: "food", quantity: 500 }
{ "_id" : 2, "item" : "f2", type: "food", quantity: 100 }
{ "_id" : 3, "item" : "p1", type: "paper", quantity: 200 }
{ "_id" : 4, "item" : "p2", type: "paper", quantity: 150 }
{ "_id" : 5, "item" : "f3", type: "food", quantity: 300 }
{ "_id" : 6, "item" : "t1", type: "toys", quantity: 500 }
{ "_id" : 7, "item" : "a1", type: "apparel", quantity: 250 }
{ "_id" : 8, "item" : "a2", type: "apparel", quantity: 400 }
{ "_id" : 9, "item" : "t2", type: "toys", quantity: 50 }
{ "_id" : 10, "item" : "f4", type: "food", quantity: 75 }
```

###Query with No Index
다음 쿼리는 quantity필드의 값이 100보다 크고 200보다 작은 정보를 검색합니다.
```
db.inventory.find( { quantity: { $gte: 100, $lte: 200 } } )
```
결과로 다음을 전달합니다.
```
{ "_id" : 2, "item" : "f2", "type" : "food", "quantity" : 100 }
{ "_id" : 3, "item" : "p1", "type" : "paper", "quantity" : 200 }
{ "_id" : 4, "item" : "p2", "type" : "paper", "quantity" : 150 }
```
explain("executionStats") 를 다음과 같이 사용합니다.
```
db.inventory.find(
   { quantity: { $gte: 100, $lte: 200 } }
).explain("executionStats")
```

결과로 다음과 같이 리턴합니다.
```
{
   "queryPlanner" : {
         "plannerVersion" : 1,
         ...
         "winningPlan" : {
            "stage" : "COLLSCAN",
            ...
         }
   },
   "executionStats" : {
      "executionSuccess" : true,
      "nReturned" : 3,
      "executionTimeMillis" : 0,
      "totalKeysExamined" : 0,
      "totalDocsExamined" : 10,
      "executionStages" : {
         "stage" : "COLLSCAN",
         ...
      },
      ...
   },
   ...
}
```

- [queryPlanner.winningPlan.stage](https://docs.mongodb.org/manual/reference/explain-results/#explain.queryPlanner.winningPlan.stage) 는  COLLSCAN 은 collection의 스캔을 표시합니다.
- [executionStats.nReturned](https://docs.mongodb.org/manual/reference/explain-results/#explain.executionStats.nReturned) 는 3을 표시합니다. 이는 쿼리를 통해 일치하는 3개의 documents가 있음을 의미합니다.
- [executionStats.totalDocsExamined](https://docs.mongodb.org/manual/reference/explain-results/#explain.executionStats.totalDocsExamined) 는 10을 표시하는데 이는 몽고디비가 10개의 document를 검색해서 3개의 일치하는 documents를 찾았다는 말입니다.

일차하는 documents의 갯수와 실제 검색을 진행한 documents의 갯수가 다릅니다. 이는 성능을 향상시킬수 있음을 의미하고 이는 인덱스를 사용하면 가능합니다.

###Query with Index
quantity 필드에 인덱스를 추가해보겠습니다.
```
db.inventory.createIndex( { quantity: 1 } )
```
explain("executionStats")메서드르 사용해서 쿼리정보를 확인해보겠습니다.

```
db.inventory.find(
   { quantity: { $gte: 100, $lte: 200 } }
).explain("executionStats")
```

```
{
   "queryPlanner" : {
         "plannerVersion" : 1,
         ...
         "winningPlan" : {
               "stage" : "FETCH",
               "inputStage" : {
                  "stage" : "IXSCAN",
                  "keyPattern" : {
                     "quantity" : 1
                  },
                  ...
               }
         },
         "rejectedPlans" : [ ]
   },
   "executionStats" : {
         "executionSuccess" : true,
         "nReturned" : 3,
         "executionTimeMillis" : 0,
         "totalKeysExamined" : 3,
         "totalDocsExamined" : 3,
         "executionStages" : {
            ...
         },
         ...
   },
   ...
}
```

- [queryPlanner.winningPlan.inputStage.stage](https://docs.mongodb.org/manual/reference/explain-results/#explain.queryPlanner.winningPlan.inputStage) 는 IXSCAN로 인덱스를 사용했음을 알수 있습니다.
- [executionStats.nReturned](https://docs.mongodb.org/manual/reference/explain-results/#explain.executionStats.nReturned)  가 3이라는 말은 일치하는 documents의 갯수가 3을 의미합니다.
- [executionStats.totalKeysExamined](https://docs.mongodb.org/manual/reference/explain-results/#explain.executionStats.totalKeysExamined) 가 3이라는 말은 사용한 몽고디비가 인덱스 수입니다.
- [executionStats.totalDocsExamined](https://docs.mongodb.org/manual/reference/explain-results/#explain.executionStats.totalDocsExamined)가 3이라는 말은 검색한 documents의 갯수가 3개라는 말입니다.

인덱스를 사용하면 쿼리는 3개의 인덱스와 3개의 documents를 검색해서 일치하는 3개의 documents를 리턴합니다. 인덱스를 사용하지 않는 경우에는 일치하는 3개의 documents를 리턴하기 위해 collection의 전체 documents를 검색했습니다.

##Compare Performance of Indexes

하나 이상의 인덱스를 적용하여 성능의 차이를 하나하나 분석하기 위해선느 [hint()](https://docs.mongodb.org/manual/reference/method/cursor.hint/#cursor.hint) 메서드를 [explain()](https://docs.mongodb.org/manual/reference/method/cursor.explain/#cursor.explain) 메서드와 결합하여 사용하면 됩니다.
```
db.inventory.find( { quantity: { $gte: 100, $lte: 300 }, type: "food" } )
```
```
{ "_id" : 2, "item" : "f2", "type" : "food", "quantity" : 100 }
{ "_id" : 5, "item" : "f3", "type" : "food", "quantity" : 300 }
```

쿼리를 지원하기 위해서 [복합인덱스(compound index)](https://docs.mongodb.org/manual/core/index-compound/) 를 추가할수 있습니다. 이때 필드의 순서는 복한 인덱스에서 중요한 부분입니다.

예를 들어 다음과 같이 복한 인덱스를 추가 해보겠습니다. 첫번째 인덱스는 quantity, type 순서의 인덱스 입니다. 그리고 다음은 type, quantity순서입니다.

````
db.inventory.createIndex( { quantity: 1, type: 1 } )
db.inventory.createIndex( { type: 1, quantity: 1 } )
```

첫번째 설정에 따른 쿼리를 실행해 보겠습니다.

```
db.inventory.find(
   { quantity: { $gte: 100, $lte: 300 }, type: "food" }
).hint({ quantity: 1, type: 1 }).explain("executionStats")
```
explain() 메서드는 다음과 같은 결과를 리턴합니다.
```
{
   "queryPlanner" : {
      ...
      "winningPlan" : {
         "stage" : "FETCH",
         "inputStage" : {
            "stage" : "IXSCAN",
            "keyPattern" : {
               "quantity" : 1,
               "type" : 1
            },
            ...
            }
         }
      },
      "rejectedPlans" : [ ]
   },
   "executionStats" : {
      "executionSuccess" : true,
      "nReturned" : 2,
      "executionTimeMillis" : 0,
      "totalKeysExamined" : 5,
      "totalDocsExamined" : 2,
      "executionStages" : {
      ...
      }
   },
   ...
}
```
몽고디비는 5개의 인덱스 키를 스캔하고 2개의 일치하는 documents를 리턴합니다.

다음 두번째 설정으로 쿼리를 실행해 보겠습니다.
```
db.inventory.find(
   { quantity: { $gte: 100, $lte: 300 }, type: "food" }
).hint({ type: 1, quantity: 1 }).explain("executionStats")
```
[explain()](https://docs.mongodb.org/manual/reference/method/cursor.explain/#cursor.explain) 메서드는 다음과 같은 결과를 리턴합니다.

```
{
   "queryPlanner" : {
      ...
      "winningPlan" : {
         "stage" : "FETCH",
         "inputStage" : {
            "stage" : "IXSCAN",
            "keyPattern" : {
               "type" : 1,
               "quantity" : 1
            },
            ...
         }
      },
      "rejectedPlans" : [ ]
   },
   "executionStats" : {
      "executionSuccess" : true,
      "nReturned" : 2,
      "executionTimeMillis" : 0,
      "totalKeysExamined" : 2,
      "totalDocsExamined" : 2,
      "executionStages" : {
         ...
      }
   },
   ...
}
```
몽고디비는 2개의 인덱스 키를 스캔하고 2개의 일치하는 documents를 리턴합니다.

위의 예제는 type, quantity 순서의 복합인덱스가 quantity,type 순서의 복합인덱스 보다 훨씬 효과적으로 나타납니다.



[목록](https://github.com/yuby/mongodb-ko)