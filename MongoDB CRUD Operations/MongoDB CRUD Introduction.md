[목록](https://github.com/yuby/mongodb-ko)


#MongoDB CRUD Introduction

몽고디비는 데이터를 JSON같이 필드와 값으로 이뤄진 형태인 documents형식으로 저장을 합니다.  Documents는 키와 값으로 아뤄진 구조인데 이는 dictionaries, hashes, maps, and associative arrays 같은 우리에게 익숙한 구조입니다.
정식적으로는 몽고디비의 documents는 [BSON](http://docs.mongodb.org/manual/reference/glossary/#term-bson) documents입니다. BSON은 [JSON](http://docs.mongodb.org/manual/reference/glossary/#term-json)을 추가적인 정보와 같이 바이너리 형태로 표현하는 방식입니다. documents에서는 BSON이 허용하는 어떠한 데이터 타입도 값으러 저장이 될 수 있습니다. 다른 documents는 물론 배열, documents의 배열이 있습니다. 더 자세한 정보는 [이곳](http://docs.mongodb.org/manual/core/document/)에서 확인하세요.

![A MongoDB document.](http://docs.mongodb.org/manual/_images/crud-annotated-document.png)


몽고디비는 모든 documents를 collections에 저장을 합니다. collections는 서로연관된 공통된 인덱스를 공유하는 documents의 집단입니다. 기존의 RDBMS 에서는 테이블의 개념으로 collections를 이해하고 해당 테이블의 row를 documents로 이해하면 됩니다.

![A collection of MongoDB documents.](http://docs.mongodb.org/manual/_images/crud-annotated-collection.png)



##Database Operations

###Query

몽고디비에서의 쿼리는 특정 collections의 documents를 대상으로 이뤄집니다. 쿼리에는 documents를 구부분짓는  특정 기준, 또는 조건을 명시하고 몽고디비는 이 결과를 클라이언트에 전달해줍니다.

In the following diagram, the query process specifies a query criteria and a sort modifier:
아래의 다이어그램은 쿼리에 기준을 명시하고 정렬를 진행하는 진행 방식을 나태나고 있습니다.

![The stages of a MongoDB query with a query criteria and a sort modifier.](http://docs.mongodb.org/manual/_images/crud-query-stages.png)

이 동작에 대한 더많은 정보는 [이곳](http://docs.mongodb.org/manual/core/read-operations-introduction/)에서 확인 하시면 됩니다.


###Data Modification
데이터를 수정하는 작업은 생성, 수정, 삭제로 구분할수 있습니다. 몽고디비는 이러한 데이터의 수정 동작을 하나의 collection을 대상으로만 진행을 합니다. 수정과 삭제의 작업시에는 여러분은 특정 기준을 명시하여 documents를 선택하여 수정과 삭제의 작업을 진행하면 됩니다.

아래의 다이어그램은 uses collection에 새로운 document를 생성하는 작업을 나타냈습니다.

![The stages of a MongoDB insert operation.](http://docs.mongodb.org/manual/_images/crud-insert-stages.png)

더 자세한 정보는 [이곳](http://docs.mongodb.org/manual/core/write-operations-introduction/)에서 확인 하시면 됩니다.

##Related Features

###Indexes
일반적인 쿼리나 수정작업의 성능을 향상 시키기 위해 몽고 디비는 secondary indexes를 완전히 지원합니다. 이 indexs는 어플리케이션이 효과적으로 데이터를 저장할수 있도록 허용합니다. (*These indexes allow applications to store a view of a portion of the collection in an efficient data structure.*)
대부분의 indexes는 값의 필드나 그룹의 필드에 따라  순차적으로 저장되어 있습니다. indexes는 [위치정보 객체](http://docs.mongodb.org/manual/applications/geospatial-indexes/)의 저장과 [문자열 검색](http://docs.mongodb.org/manual/core/index-text/)을 용이하게 할수 있는  [고유성](http://docs.mongodb.org/manual/core/crud-introduction/)을 더욱 강화시킬수 있습니다.

###Replica Set Read Preference
리플리카셋과 리플리카 셋으로 구성된 샤드 클러스터트로 부터 어떻게 데이터를 사용자에게 바로 전달하는 지에 대한 [우선권](http://docs.mongodb.org/manual/core/read-preference/#replica-set-read-preference)에 대한 정의 입니다.

###Write Concern
어플리케이션은 write concern을 통해 생성하는 동작을 조절할 수 있습니다. 특히 리플리카셋에서 매우 유용한데, 클라이언트에게  데이터를 생성하는 작업이 제대로 성공되었는지를 알려주는 역활을 할 수있습니다.

###Aggregation
기본쿼리에 추가적으로 데이터를 추출할수 있는 기능을 몽고디비는 제공하고 있습니다. 예를들면 몽고디비는 쿼리와 일치하는 documents의 갯수를 전달할수 있고, 각 필드에 구분이 되는 값들의 갯수를 리턴하거나, 작업중인 collection의 documents에 각 단계별로 데이터를 다룰 수있는 pipeline 방식의 기능을 제공하고, map-reduce 작업도 가능하게 합니다.


[목록](https://github.com/yuby/mongodb-ko)