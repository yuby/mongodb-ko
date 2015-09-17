#Read Operations Overview

Read operations, or queries, retrieve data stored in the database. In MongoDB, queries select documents from a single collection.
읽기작업, 또는 쿼리는 저장된 데이터를 데이터베이스로 부터 검색하는 작업입니다. 몽고디비에서의 셀렉트 쿼리는 하나의 collection의 documents를 대상으로 동작을 합니다.

Queries specify criteria, or conditions, that identify the documents that MongoDB returns to the clients. A query may include a projection that specifies the fields from the matching documents to return. The projection limits the amount of data that MongoDB returns to the client over the network.
쿼리는 기준과 저건을 명시하여 documents를 구분짓고 몽고디비는 그 결과를 클라이언트에 전달합니다. 쿼리는 매칭된  documents의 특정필드를 전달하기 위한 projection을 포함하고 있을 수 있습니다. projection은 몽고디비로부터 잔달되는 데이터의 양에 따라 제한이 될수도 있습니다.


##Query Interface

For query operations, MongoDB provides a db.collection.find() method. The method accepts both the query criteria and projections and returns a cursor to the matching documents. You can optionally modify the query to impose limits, skips, and sort orders.

쿼리작업을 위해 몽고디비는 [db.collection.find()](http://docs.mongodb.org/manual/reference/method/db.collection.find/#db.collection.find) 메서드를 제공합니다. 이 메서드는 결과를 위한 기준과  projections와 일치하는 documents의 [cursor](http://docs.mongodb.org/manual/core/cursors/)를 정의 할 수있습니다.
선택적으로 전달되는 결과의 limits, skips, sort 를 추가 할 수 있습니다.

아래의 다이어그램은 몽고디비의 쿼리를 컴퍼넌트별로 강조하여 보여주고 있습니다.

![The components of a MongoDB find operation.](http://docs.mongodb.org/manual/_images/crud-annotated-mongodb-find.png)

아래의 다이어그램은 동일한 작업을 보통의 SQL문으로 표현한 것입니다.

[The components of a SQL SELECT statement.](http://docs.mongodb.org/manual/_images/crud-annotated-sql-select.png)

EXAMPLE
'''
db.users.find( { age: { $gt: 18 } }, { name: 1, address: 1 } ).limit(5)
'''

이 쿼리는 users collection에서 18세 이상의 정보를 불러오는 쿼리입니다. '더 큰' 이라는 조건이 명시되어 쿼리의 셀렉트 작업이 수행됩니다. 쿼리는 최대 5개의 일치하는 조건의 결과를 리턴합니다. 그리고 오직 _id, name, address  정보만을 전달합니다.
[Projections](http://docs.mongodb.org/manual/core/read-operations-introduction/#projections) 에 대한 상세정보를 확인하세요.

###SEE:
[SQL 을 몽고디비와 비교한 표입니다.](http://docs.mongodb.org/manual/reference/sql-comparison/)


##Query Behavior

몽고디비 쿼리는 아래와 같은 특징을 나타냅니다.

- 몽고디비에서의 모든 쿼리는 오직 하나의 collection을 대상으로 합니다.
- 쿼리의 결과에 limits , skips, sort orders를 추가 할 수 있습니다.
- sort()로 명시하지 않는 이상 리턴되는 documents는 순서가 없습니다.
- [이미 존재하는 documents에 대한 작업](http://docs.mongodb.org/manual/tutorial/modify-documents/)의 경우에는 select를 위한 쿼리를 그대로 update에 사용하면 됩니다.
- [aggregation](http://docs.mongodb.org/manual/core/aggregation/) 파이프라인의 [$match](http://docs.mongodb.org/manual/reference/operator/aggregation/match/#pipe._S_match) 파이프라인 단계에서는 몽고디비 쿼리에 접근할수 있습니다.

몽고디비는 오직 하나의 document만을 제공하는 경우를 위해  db.collection.findOne()  메서드를 제공하고 있습니다.


##Query Statements

아래의 다이어그램은 쿼리의 조건들에 따라 진행되는 과정을 나타낸 그림입니다.

![The stages of a MongoDB query with a query criteria and a sort modifier.](http://docs.mongodb.org/manual/_images/crud-query-stages.png)

다이어 그램은 users collection에서 document를 조건을 통해서 일치하는 documents를 select하고 있습니다. 나이가 18세 이상의 사용자를 일단 추려내고 그다음 sort() 를 통해 이전 단계의 결과를 나이에따라 오름차순으로 정렬하고 있습니다.

[The stages of a MongoDB query with a query criteria and a sort modifier.](http://docs.mongodb.org/manual/_images/crud-query-stages.png)

더 많은 정보는 [이곳](http://docs.mongodb.org/manual/tutorial/query-documents/)에서 확인하세요.


##Projections

몽고디비의 쿼리는 기본적으로 일치하는 documents의 필드정보를 리턴합니다. 그래서 쿼리에 [projection](http://docs.mongodb.org/manual/reference/glossary/#term-projection)을 추가해서 전달되는 데이터의 양을 제한합니다. projecting을 통해서 결과로 전달되는 데이터의 양을 조절하여 네트워크나 프로세싱에 대한 부담을 감소시킵니다.

Projections, which are the second argument to the find() method, may either specify a list of fields to return or list fields to exclude in the result documents.
Projections은 find() 메서드의 두번째 전달인자로 필요한 필드 리스트 혹은 제외하고 싶은 리스트를 넘깁니다.

> IMPORTANT
_id 필드의 경우에는 필드를 제외할때 다른 필드와 방식이 다릅니다. 일반 필드의 경우에는 projection에 추가하지 않으면 제외의 대상이지만 _id필드의 경우에는 _id:0 을 통해서 제외가 됩니다.

아래의 다이어그램의 경우 쿼리가 조건을 처리하고 projection을 처리하는 과정을 보여줍니다.

[The stages of a MongoDB query with a query criteria and projection. MongoDB only transmits the projected data to the clients.](http://docs.mongodb.org/manual/_images/crud-query-w-projection-stages.png)

다이어그램에서는 users collection에 select 작업을 진행합니다. 조건과 일치하는 documents를 가지고 projection에 특정 필드 이름을 더해 결과를 리턴합니다.


Projection Examples
Exclude One Field From a Result Set
db.records.find( { "user_id": { $lt: 42 } }, { "history": 0 } )
This query selects documents in the records collection that match the condition { "user_id": { $lt: 42 } }, and uses the projection { "history": 0 } to exclude the history field from the documents in the result set.

Return Two fields and the _id Field
db.records.find( { "user_id": { $lt: 42 } }, { "name": 1, "email": 1 } )
This query selects documents in the records collection that match the query { "user_id": { $lt: 42 } } and uses the projection { "name": 1, "email": 1 } to return just the _id field (implicitly included), name field, and the email field in the documents in the result set.

Return Two Fields and Exclude _id
db.records.find( { "user_id": { $lt: 42} }, { "_id": 0, "name": 1 , "email": 1 } )
This query selects documents in the records collection that match the query { "user_id": { $lt: 42} }, and only returns the name and email fields in the documents in the result set.

SEE
Limit Fields to Return from a Query for more examples of queries with projection statements.
Projection Behavior
MongoDB projections have the following properties:

By default, the _id field is included in the results. To suppress the _id field from the result set, specify _id: 0 in the projection document.
For fields that contain arrays, MongoDB provides the following projection operators: $elemMatch, $slice, and $.
For related projection functionality in the aggregation framework pipeline, use the $project pipeline stage.