#Bulk Write Operations

##Overview

몽고디비는 대량의 쓰기작업을 지원합니다. 대량의 쓰기작업은 하나의 collection을 대상으로 이뤄집니다. 몽고디비는 어플리케이션이 대량의 쓰기작업에 필요한 허용수준을 확인 합니다.

db.collection.bulkWrite() 메서드는 생성, 수정, 삭제작업을 제공합니다. 물론 db.collection.insertMany() 메서드를 통해서 대량의 생성작업이 가능합니다.

##Ordered vs Unordered Operations
대량의 쓰기작업의 경우 순차적, 비순차적으로 데이터를 생성하는 것이 가능합니다.

순차적으로 생성하는 작업의 경우 몽고디비는 작업을 순차적으로 진행합니다. 작업중에 에러가 밸상하는 경우 몽고디비는 남아있는 데이터 생성작업을 진행하지 않습니다.

비순차적인 생성작업의 경우 몽고디비가 병렬로 생성작업을 진행합니다. 하지만 이 행동의 경우 작업의 성공을 보장하지 않습니다. 만약 작업중 오류가 발생해도 몽고디비는 남아있는 쓰기작업을 계속 진행합니다.

shard된 collection의 경우에는 순차적인 생성작업이 비순차적인 생성작업에 비해 느립니다. 왜냐하면 다음동작이 이전동작이 완료됨을 확인하고 다음을 진행해야합니다.

기본적으로, bulkWrite()는 순차적 동작을 합니다. 비순차적인 동작을 시키기 위해서는 ordered: false 옵션을 사용합니다.

##bulkWrite() Methods
bulkWrite() 메서드는 다음의 동작을 합니다.

- nsertOne
- updateOne
- updateMany
- replaceOne
- deleteOne
- deleteMany

bulkWrite() 파라메터로 다수의 document를 배열로 전달합니다.

```
{ "_id" : 1, "char" : "Brisbane", "class" : "monk", "lvl" : 4 },
{ "_id" : 2, "char" : "Eldon", "class" : "alchemist", "lvl" : 3 },
{ "_id" : 3, "char" : "Meldane", "class" : "ranger", "lvl" : 3 }
```
데이터를 삽입하는 코드는 다음과 같습니다.

```
try {
   db.characters.bulkWrite(
      [
         { insertOne :
            {
               "document" :
               {
                  "_id" : 4, "char" : "Dithras", "class" : "barbarian", "lvl" : 4
               }
            }
         },
         { insertOne :
            {
               "document" :
               {
                  "_id" : 5, "char" : "Taeln", "class" : "fighter", "lvl" : 3
               }
            }
         },
         { updateOne :
            {
               "filter" : { "char" : "Eldon" },
               "update" : { $set : { "status" : "Critical Injury" } }
            }
         },
         { deleteOne :
            { "filter" : { "char" : "Brisbane"} }
         },
         { replaceOne :
            {
               "filter" : { "char" : "Meldane" },
               "replacement" : { "char" : "Tanys", "class" : "oracle", "lvl" : 4 }
            }
         }
      ]
   );
}
catch (e) {
   print(e);
}
```

위코드를 실행하면 다음의 결과를 리턴합니다.

```
{
   "acknowledged" : true,
   "deletedCount" : 1,
   "insertedCount" : 2,
   "matchedCount" : 2,
   "upsertedCount" : 0,
   "insertedIds" : {
      "0" : 4,
      "1" : 5
   },
   "upsertedIds" : {

   }
}
```

##Strategies for Bulk Inserts to a Sharded Collection

최초로 데이터를 생성하든 반복적으로 데이터 불러와 대량으로 생성하는 작업은 sharded cluster의 성능에 영향을 미칩니다. 대량의 생성작업은 다음의 방식으로 이뤄집니다.

###Pre-Split the Collection
만약에 shared된 collection이 비어 있는 경우 collection은 최초의 하나의 chunk가 하나의 shard에 존재하게 됩니다. 몽고디비는 데이터를 받아들이는 시간이 반드시 필요하고 이를 분리하고 사용가능한 shard에 분산시키는 작업을 합니다. 이러한 동작의 부하를 피하기 위해서 미라 collection을 분리 시키는 작업을 할수 있습니다. 제세한 내영은 [이곳](https://docs.mongodb.com/manual/tutorial/split-chunks-in-sharded-cluster/) 에서 확인 하시면 됩니다.

###Unordered Writes to mongos
비순차적으로 shared cluster상에서 bulkWrite()동작의 성능을 향상시키기 위해서  mongos는 동시에 다수의 shard에 쓰기작업을 시도합니다. 빈collection의 경우에는 pre-splite를 우선진행합니다.

###Avoid Monotonic Throttling

만약에 생성을 하면서 shard key가 순차적으로 증가를 하면 해당 shard의 끝에 도달하게 되고, 결국 추가되는 모든 데이터들이 collection의 마지막 chunk로만 저장이 되게 됩니다. 그러므로 클러스터상에서의 생성은 결코 shard의 주어진 크기를 넘어서서 저장될수가 없습니다.

만역에 생성하려는 데이터의 크기가 주어진 shard의 크기보다 크고, 순차적으로 증가하는 shard key를 피할수 없는 경우 다음의 사항을 고려하기바랍니다.

- shard key의 binary 비트를 뒤집습니다. 이는 정보를 보호하고 순차적으로 증가하는 값의 중복을 피할수 있습니다.
- 처음과 마지막 16비트의 글자를 바꿉니다.

> EXAMPLE:
> 다음은 C++에서 처음과 마지막 16비트를 변경해서 새롭게 BSON ObjectIds를 생성하는 예제입니다.

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

