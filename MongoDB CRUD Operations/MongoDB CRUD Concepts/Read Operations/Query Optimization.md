[목록](https://github.com/yuby/mongodb-ko)

#Query Optimization

Indexes는 읽기 작업을 할 때 전체적으로 동작하게 될 쿼리의 양을 줄여서 효과적으로 데이터를 불러올수 있도록 합니다. 이 간단화 작업은 몽고디비 내에서 쿼리의 동작과 연관이 있습니다.

##Create an Index to Support Read Operations

만약에 어플리케이션의 쿼리가 collection의 특정 필드나 필드의 셋을 대상으로 실행이 되는 경우네 인덱스는 필드 나 [인덱스의 집합(compound index)](http://docs.mongodb.org/manual/core/index-compound/)을 대상으로 지정이 되어 쿼리가 collection의 전체 데이터를 대상으로 스캔을 하는 작업을 예방합니다.
indexes에 대한 더 많은 정보는 [아곳](http://docs.mongodb.org/manual/core/indexes/)에서 확인하세요.

 >EXAMPLE
inventory collection의 type필드에 사용자가 지정하는 값으로 검색을 하는 작업입니다.
```
var typeValue = <someUserInput>;
db.inventory.find( { type: typeValue } );
```
이 쿼리의 성능을 향상시키기 위해서 내림차순이나 오름차순 indexes를 inventory collection의 type필드에 적용해 줍니다. [몽고](http://docs.mongodb.org/manual/reference/program/mongo/#bin.mongo)쉘에서 [db.collection.createIndex()](http://docs.mongodb.org/manual/reference/method/db.collection.createIndex/#db.collection.createIndex) 메서드를 사용하여 index를 생성할수 있습니다.
```
db.inventory.createIndex( { type: 1 } )
```
이 index를 통해서 결과를 리턴하기 위해 전체 collection을 스캔하는 작업을 예방할수 있습니다.

index의 성능을 확인하기 위한 정보는 [이곳](http://docs.mongodb.org/manual/tutorial/analyze-query-plan/)에서 확인하시면 됩니다.


추가적으로 읽기작업을 최적화 시키기 위해서 indexes는 정렬작업, 효과적인 저장공간의 사용을 위해서도 사용이 가능합니다.
[db.collection.createIndex()](http://docs.mongodb.org/manual/reference/method/db.collection.createIndex/#db.collection.createIndex) 와 이 [튜토리얼](http://docs.mongodb.org/manual/administration/indexes/)을 통해서 index에 대한 더 많은 정보를 얻으실수 있습니다.


##Query Selectivity

쿼리의 선별력은 collection으로 부터 documents를 얼마나 잘 예측하여 효과적으로 추출해 낼수 있을 지를 의미합니다. 쿼리가 효율적으로  index를 사용할지 아니면 전혀 사용하지 않는지를 통해 결정이 됩니다.

쿼리의 선별력이 높다는 말은 더 적은 퍼센테이지의 documents를 매칭시킨다는 말입니다. 예를들면, 유니크한 _id 필드를 사용한다면 오직 하나의 document가 일치하게 될것이고 이는 높은 선별력을 의미합니다.

선별력이 낮음은 큰 퍼센테이지의 documents를 매칭한다는 말입니다. 낮은 선별력을 가진 쿼리는 indexes를 효과적으로 사용하지 못하거나 전혀 사용하지 않은 경우입니다.
예를들면 부등연산인 [$nin](http://docs.mongodb.org/manual/reference/operator/query/nin/#op._S_nin) 과 [$ne](http://docs.mongodb.org/manual/reference/operator/query/ne/#op._S_ne) 는 매우 선택력이 낮습니다. 그래서 이들은 종종 큰범위의 인덱스를 매칭합니다. 그 결과 [$nin](http://docs.mongodb.org/manual/reference/operator/query/nin/#op._S_nin)과 [$ne](http://docs.mongodb.org/manual/reference/operator/query/ne/#op._S_ne) 가 index와 함께 사용된다면 그냥 인덱스를 사용하지 않고 전체 documents를 대상으로 [$nin](http://docs.mongodb.org/manual/reference/operator/query/nin/#op._S_nin)과 [$ne](http://docs.mongodb.org/manual/reference/operator/query/ne/#op._S_ne)가 동작하는것이 훨씬 성능이 좋을 수 있습니다.

[정규식](http://docs.mongodb.org/manual/reference/operator/query/regex/#op._S_regex)에 대한 선택력은 정규식 그 자체에 달려있는데 자세한 정보는 [이곳](http://docs.mongodb.org/manual/reference/operator/query/regex/#regex-index-use)에서 확인하시면 됩니다.


##Covered Query

index가 쿼리를 커버하는 두가지 경우가 있습니다.
- [쿼리](http://docs.mongodb.org/manual/tutorial/query-documents/#read-operations-query-document)의 모든 필드가 인덱스의 일부인 경우
- 리턴되는 결과의 필드가 모두 동일한 index인 경우

예를들면 inventory collection이 type 과 item필드에 인덱스를 가지고 있다고 하겠습니다.
```
db.inventory.createIndex( { type: 1, item: 1 } )
```
이 인덱스는 아래의 쿼리처럼 type과 item필드를 대상으로 동작을 하고 리턴되는 결과도 오직 item필드뿐인 경우입니다.
```
db.inventory.find(
   { type: "food", item:/^c/ },
   { item: 1, _id: 0 }
)
```
index지정은 쿼리를 커버하고 document에 projection에서 반드시 인덱스가 없는 _id필드를 제거해줘야합니다.

###Performance

index는 쿼리가 필요로하는 모든 필드를 포함하고 있기 때문에 몽고디비는 쿼리의 조건은 물론 리턴되는 결과물에 index를 사용해야 합니다.

Querying only the index can be much faster than querying documents outside of the index. Index keys are typically smaller than the documents they catalog, and indexes are typically available in RAM or located sequentially on disk.

###Limitations

####Restrictions on Indexed Fields
인덱스가 쿼리를 보완하지 못하는 경우도 있습니다.

- 어떤 인덱스 필드가 collection의 documents가 배열 행태로 존재하는 경우입니다. 만약에 인덱스필드가 배열형태라면 인덱스는 [다중-키 인덱스](http://docs.mongodb.org/manual/core/index-multikey/#index-type-multikey)가 되고 쿼리를 커버하지 못하게됩니다.
- any of the indexed field in the query predicate or returned in the projection are fields in embedded documents.  예를 들어 collection의 documents의 형태가
```
{ _id: 1, user: { login: "tester" } }
```
이와 같습니다. 그리고 이 collection의 경우에
```
{ "user.login": 1 }
```
이와 같은 인덱스를 가지고 있습니다. 이경우에는 { "user.login": 1 } 인덱스는 다음의 쿼리를 전혀 커버해주지 못합니다.
```
db.users.find( { "user.login": "tester" }, { "user.login": 1, _id: 0 } )
```

하지만 쿼리는 { "user.login": 1 } 인덱스를 일치하는 documents를 찾는데는 사용가능합니다.


####Restrictions on Sharded Collection

mongos에서 실행하는 동안 인덱스가 [sharded](http://docs.mongodb.org/manual/reference/glossary/#term-shard) collection들 간에 shard key에 포함되지 않았다면 다음의  _id인덱스를 제외하고는 쿼리를 커버하지 못합니다.
만약에 쿼리가 sharded된 collection에서 _id필드로만 조건이 걸리고 리턴받은 필드가 오직 _id인경우 _id 인덱스는 shard key가 아니더라도 [mongos](http://docs.mongodb.org/manual/reference/program/mongos/#bin.mongos) 상에서는 쿼리를 커버합니다.

3.0버전 이후 : 이전 버전에서는 mongos동작시 인덱스가 sharded collection에서는 쿼리를 커버하지 못했습니다.

###explain
쿼리를 커버하는지 하지 않는지에 대한 정의는 [db.collection.explain()](http://docs.mongodb.org/manual/reference/method/db.collection.explain/#db.collection.explain) 나  [explain()](http://docs.mongodb.org/manual/reference/method/cursor.explain/#cursor.explain)메서드를 사용해서 [결과](http://docs.mongodb.org/manual/reference/explain-results/#explain-output-covered-queries)를 확인해보면 됩니다.

[db.collection.explain()](http://docs.mongodb.org/manual/reference/method/db.collection.explain/#db.collection.explain)는 [db.collection.update()](http://docs.mongodb.org/manual/reference/method/db.collection.update/#db.collection.update)같은 다른 동작들의 실행에 대한 정보를 제공합니다. 더 많은 정보는 [이 곳](http://docs.mongodb.org/manual/reference/method/db.collection.explain/#db.collection.explain)에서 확인하시면 됩니다.

[Measure Index Use]()에 대한 더 많은 정보를 확인하세요.



[목록](https://github.com/yuby/mongodb-ko)