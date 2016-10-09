[목록](https://github.com/yuby/mongodb-ko)


#Bulk Write Operations

##Overview

몽고디비는 클라이언트가 쓰기작업을 큰 덩어리(bulk) 단위로 가능하도록 기능을 제공해주고 있습니다. Bulk에 쓰기 작업은 하나의 collection을 대상으로 이뤄집니다. 몽고디비는 bulk 단위의 쓰기 작에에 요구되는 허용수준을 확인할수가 있습니다.

새로운 [Bulk](http://docs.mongodb.org/manual/reference/method/Bulk/#Bulk) 메서드는 bulk 단위의 생성, 수정, 제거 작업이 간을 하도록 제공하고 있습니다. 몽고디비는  [db.collection.insert()](http://docs.mongodb.org/manual/reference/method/db.collection.insert/#db.collection.insert) 메서드에 배열로 데이터를 전달하여 다량의 정보를 생성하게 하는 것도 가능합니다.

2.6버전 이후 : 이전 버전에서는 [db.collection.insert()](http://docs.mongodb.org/v2.4/core/bulk-inserts/) 에 배열을 전달하는 방법뿐이았지만 이제는 [Bulk Insert](http://docs.mongodb.org/v2.4/core/bulk-inserts/)를 통해서 가능해졌습니다.


##Ordered vs Unordered Operations

bulk 쓰기 작업은 순차적 또는 비 순차적으로가 가능합니다. 순차적인 작업의 경우에는 순서대로 하나씩 동작을 합니다. 만약 쓰기작업 중간에 에러가 발생하면 몽고디비는 리스트에 남아있는 작업을 나두고 리턴을 합니다.

비 순차적인 작업의 경우에는 작업이 병렬로 진행됩니다. 만약에 작업중간에 에러가 발생해도 계속 남은 작업들을 진행합니다.

shared collection에서의 순차적인 작업은 비순차적인 작업에 비해 느린데 이유는 이전의 작업이 마치기를 기다려야 하기때문입니다.


##Bulk Methods

[Bulk()](http://docs.mongodb.org/manual/reference/method/Bulk/#Bulk) 메서드 사용하기

1. 작업의 리스트를 [db.collection.initializeUnorderedBulkOp()](http://docs.mongodb.org/manual/reference/method/db.collection.initializeUnorderedBulkOp/#db.collection.initializeUnorderedBulkOp) 나 [db.collection.initializeOrderedBulkOp()](http://docs.mongodb.org/manual/reference/method/db.collection.initializeOrderedBulkOp/#db.collection.initializeOrderedBulkOp)를 사용해서 초기화를 시킵니다.

2. 다음의 메서드를 사용해서 쓰기 작업들을 할수 있습니다.
[Bulk.insert()](http://docs.mongodb.org/manual/reference/method/Bulk.insert/#Bulk.insert)
[Bulk.find()](http://docs.mongodb.org/manual/reference/method/Bulk.find/#Bulk.find)
[Bulk.find.upsert()](http://docs.mongodb.org/manual/reference/method/Bulk.find.upsert/#Bulk.find.upsert)
[Bulk.find.update()](http://docs.mongodb.org/manual/reference/method/Bulk.find.update/#Bulk.find.update)
[Bulk.find.updateOne()](http://docs.mongodb.org/manual/reference/method/Bulk.find.updateOne/#Bulk.find.updateOne)
[Bulk.find.replaceOne()](http://docs.mongodb.org/manual/reference/method/Bulk.find.replaceOne/#Bulk.find.replaceOne)
[Bulk.find.remove()](http://docs.mongodb.org/manual/reference/method/Bulk.find.remove/#Bulk.find.remove)
[Bulk.find.removeOne()](http://docs.mongodb.org/manual/reference/method/Bulk.find.removeOne/#Bulk.find.removeOne)

3. 쓰기작업들을 수행하기 위해서 [Bulk.excute()](http://docs.mongodb.org/manual/reference/method/Bulk.execute/#Bulk.execute) 메서드를 사용합니다. write concern을 지정하여 [Bulk.excute()](http://docs.mongodb.org/manual/reference/method/Bulk.execute/#Bulk.execute) 메서드를 실행하는 것이 가능합니다.
한번 실행이 된 이후에는 초기화 작업이 없이 바로 재 실행하는 것이 불가능 합니다.

For example,
```
var bulk = db.items.initializeUnorderedBulkOp();
bulk.insert( { _id: 1, item: "abc123", status: "A", soldQty: 5000 } );
bulk.insert( { _id: 2, item: "abc456", status: "A", soldQty: 150 } );
bulk.insert( { _id: 3, item: "abc789", status: "P", soldQty: 0 } );
bulk.execute( { w: "majority", wtimeout: 5000 } );
```
Bulk Operation 메서드의 [참조 페이지](http://docs.mongodb.org/manual/reference/method/js-bulk/)를 확인하세요. 더 많은 정보와 [db.collection.insert()](http://docs.mongodb.org/manual/reference/method/db.collection.insert/#db.collection.insert)를 사용한 예는[ db.collection.insert()](http://docs.mongodb.org/manual/reference/method/db.collection.insert/#db.collection.insert)에서 확인하면 됩니다.


##Bulk Execution Mechanics

[순차적인](http://docs.mongodb.org/manual/reference/method/db.collection.initializeOrderedBulkOp/#db.collection.initializeOrderedBulkOp) 작업리스트를 실행하는 경우 [ operation type](http://docs.mongodb.org/manual/reference/method/Bulk.getOperations/#batchType)을 통해서 인접한 작업끼리 그룹을 맺습니다. 비순차적인 작업의 리스트를 실행하는 경우 몽고디비는 그룹화를 맺던지 재배열작업을 하여 성능을 향상시킬것입니다. 이처럼 [비순차적](http://docs.mongodb.org/manual/reference/method/db.collection.initializeUnorderedBulkOp/#db.collection.initializeUnorderedBulkOp)인 명령리스트의 경우에는 어플리케이션의 명령의 순서에는 의존적이지 않습니다.

각각의 작업 그룹은 최대 [1000개의 작업](http://docs.mongodb.org/manual/reference/limits/#Write-Command-Operation-Limit-Size)으로 구성이 되어있습니다. 만약 그룹이 이 [한계](http://docs.mongodb.org/manual/reference/limits/#Write-Command-Operation-Limit-Size)를 넘게 되면 몽고디비는 1000개나 그 이하의 크기로 줄입니다. 예를들면 bulk 작업리스트가 2000개로 구성되어있다면 몽고디비는 2개의 그룹으로 각각 1000개씩 나누게 됩니다.

사이즈와 그룹화의 매커니즘은 내부 성능의 세부사항이고 이후 버전에서 변경가능한 사항입니다.

어떻게 Bulk 작업에서 그룹화가 이뤄지는지는 실행이후에 [Bulk.getOperations()](http://docs.mongodb.org/manual/reference/method/Bulk.getOperations/#Bulk.getOperations)를 호출하면 됩니다.

더많은 정보는 [Bulk.execute()](http://docs.mongodb.org/manual/reference/method/Bulk.execute/#Bulk.execute)를 확인하면 됩니다.


##Strategies for Bulk Inserts to a Sharded Collection

큰 사이즈의 생성작업의 경우 최기 데이터의 생성이나 루틴데이터를 가져오는 일이 [shared 클러스터](http://docs.mongodb.org/manual/reference/glossary/#term-sharded-cluster)의 성능에 영향을 줍니다. Bulk 데이터의 생성에 있어서는 다음의 사항들을 고려해야 합니다.

##Pre-Split the Collection
만약에 shared collection이 비어있다면 collection은 하나의 shard에 위치한 오직 하나의 초기 [chunk](http://docs.mongodb.org/manual/reference/glossary/#term-chunk) 만 가집니다. 몽고디비는 반드시 데이터를 받아들이는데 시간을 가져야합니다. 데이터를 분할하고 분산시켜 사용가능한 shards에 분산하기 때문입니다. 이러한 성능상의 비용을 피하기 위해서는 [Split Chunks in a Sharded Cluster](http://docs.mongodb.org/manual/tutorial/split-chunks-in-sharded-cluster/)에 따라  미리 collecion을 분할시켜 둬야 합니다.

##Insert to Multiple mongos
병렬로 데이터를 가져오는 작업을 하는 경우 bulk insert나 생성작업으로 최소한 1번이상의 [monogs](http://docs.mongodb.org/manual/reference/program/mongos/#bin.mongos) 인스턴스로의 전달이 가능합니다. 비어있는 collecions에 대해서는 미리 분할하는 작업이 필요합니다. [Split Chunks in a Sharded Cluster.](http://docs.mongodb.org/manual/tutorial/split-chunks-in-sharded-cluster/)

##Avoid Monotonic Throttling
만약에 샤드키가 생성작업을 하는 동안 단조롭게만 생성이 된다면 생성된 모든 데이터는 collection의 제일 마지막 chunk에 위치하게 됩니다. 그러므로 클러스터에서의 생성작업의 양은 절대 단일 shard 의 생성량을 넘어설수가 없습니다.

만약에 생성할 양이 단일 샤드가 진행할수 있는 양보다 크고 단순히 증가하는 shard key를 피하고 샆다면 다음 수정 사항을 고려해보면 됩니다.

- shard key의 바이너리 비트를 뒤집습니다. 이러면 단순히 순차적을 증가하는 값을 비할수 있고 정보를 보존할수 있습니다.
-첫번째 마지막 16-비트 단어를 변경하여 생성합니다.


>EXAMPLE
The following example, in C++, swaps the leading and trailing 16-bit word of [BSON ObjectIds](http://docs.mongodb.org/manual/reference/glossary/#term-objectid) generated so they are no longer monotonically increasing.
C++에서 BSON ObjectId의 처음과 마지막 16빈트 단어를 바꿔서 더이상의 단순한 증가를 막는 작업을 했습니다.
```
using namespace mongo;
OID make_an_id() {
  OID x = OID::gen();
  const unsigned char *p = x.getData();
  swap( (unsigned short&) p[0], (unsigned short&) p[10] );
  return x;
}
void foo() {
  // create an object
  BSONObj o = BSON( "_id" << make_an_id() << "x" << 3 << "name" << "jane" );
  // now we may insert o into a sharded collection
}
```


[목록](https://github.com/yuby/mongodb-ko)