[목록](https://github.com/yuby/mongodb-ko)


#Create an Auto-Incrementing Sequence Field

##Synopsis

몽고디비는 _id필드를 모든 documents의 primary key로 최고 레벨로 설정해 두고 있습니다. _id 는 반드시 유일한 값입니다. 그리고 항상 인덱스를 가지고 유일함을 유지합니다. 하지만  [고유한 제한조건(unique constraint)](https://docs.mongodb.org/manual/core/index-unique/#index-type-unique)을 제외하고는 collection의 어떠한 값도 _id필드의 값으로 사용이 가능합니다. 이번에는 순차적으로 증가하는 _id를 만드는 두가지 방법을 확인해보겠습니다.

- [Use Counters Collection](https://docs.mongodb.org/manual/tutorial/create-an-auto-incrementing-field/#auto-increment-counters-collection)
- [Optimistic Loop](https://docs.mongodb.org/manual/tutorial/create-an-auto-incrementing-field/#auto-increment-optimistic-loop)


##Considerations

보통 몽고디비는 자동으로 증가하는 방법으로 _id필드나 다른 필드의  값을 지정하지 않습니다. 왜냐하면 많은 수의 documents를 확장할수가 없기 때문입니다. 그래서 기본적으로 생성이 되는 [ObjectId](https://docs.mongodb.org/manual/reference/glossary/#term-objectid)값이 _id필드의 값으로 가장 이상적입니다.

##Procedures

###Use Counters Collection

####Counter Collection Implementation
counters collection을 따로 분리 시켜 마지막에 들어간 값을 계속 주시합니다. _id필드는 사용될 필드의 이름을 지정하고 seq 필드에는 순차적으로 증가한 마지막 값을 지정합니다.

1. counters collection에 새로운 정보를 생성하고 기본 값을 설정합니다.
```
db.counters.insert(
   {
      _id: "userid",
      seq: 0
   }
)
```

2. getNextSequence 함수를 생성하고 생성된 값으로 사용될 필드의 이름을 인자로 전달합니다. 함수는 findAndModify()메서드를 사용해서 순차적으로 증가시킨 값을 새로운 값으로 설정합니다.
```
function getNextSequence(name) {
   var ret = db.counters.findAndModify(
          {
            query: { _id: name },
            update: { $inc: { seq: 1 } },
            new: true
          }
   );

   return ret.seq;
}
```

3. getNextSequence() 메서드를 사용해서 insert() 메서드가 실행하는 동안 새로운 값을 생성합니다.
```
db.users.insert(
   {
     _id: getNextSequence("userid"),
     name: "Sarah C."
   }
)

db.users.insert(
   {
     _id: getNextSequence("userid"),
     name: "Bob D."
   }
)
```
그리고 find()메서드를 사용해서 결과를 확인합니다.
```
db.users.find()
The _id fields contain incrementing sequence values:
{
  _id : 1,
  name : "Sarah C."
}
{
  _id : 2,
  name : "Bob D."
}
```

####findAndModify Behavior
[findAndModify()](https://docs.mongodb.org/manual/reference/method/db.collection.findAndModify/#db.collection.findAndModify) 메서드가 upsert: true옵션을 포합하고 쿼리의 필드가 고유한 인덱싱이 되어 있지 않다면 메서드는 특정상황에서 여러번의 documents의 생성이 일어날수 있습니다. 예를들어 만약에 다수의 클라이언트가 동일한 쿼리로 메서드를 호출을 한다면 이 메서드는 검색을 수정단계 이전에 완료를 하게 됩니다. 이는 메서드가 동일한 document를 생성하게 되는 원인이 됩니다.

counters collection 예제에서는 쿼리필드는 _id필드로 항상 유일한 인덱스를 가지고 있습니다. upsert: true을 사용한 [findAndModify()](https://docs.mongodb.org/manual/reference/method/db.collection.findAndModify/#db.collection.findAndModify)의 예제를 확인해보겠습니다.
```
function getNextSequence(name) {
   var ret = db.counters.findAndModify(
          {
            query: { _id: name },
            update: { $inc: { seq: 1 } },
            new: true,
            upsert: true
          }
   );

   return ret.seq;
}
```
다수의 클라이언트가 getNextSequence()메서드를 동시에 동일한 파라메터로 실행을 하면 메서드는 다음과 같은 행동을 하게 됩니다.
- 정확하게 [findAndModify()](https://docs.mongodb.org/manual/reference/method/db.collection.findAndModify/#db.collection.findAndModify) 메서드를 한번한 실행해 하나의 새로운 document를 생성하게 됩니다.
- 0 또는 더 많은 [findAndModify()](https://docs.mongodb.org/manual/reference/method/db.collection.findAndModify/#db.collection.findAndModify) 메서드가 새로운 document 를 생성하게 됩니다.
- 0 또는 더 많은 [findAndModify()](https://docs.mongodb.org/manual/reference/method/db.collection.findAndModify/#db.collection.findAndModify) 메서드가 중복으로 생성작업을 시도하면서 작업이 실패할것입니다.

만약에 메서드가 실패를 한다면 이는 유니크한 인덱스의 제한에 위배되기 때문일것입니다.


####Optimistic Loop
이러한 패턴에서 Optimistic Loop는 증가된 _id 값을 계산하고 그 계산된 값으로 새로운 document의 _id 값으로 설정을 합니다. 만약에 이 생성작업이 성공적이라면 반복문은 끝이 나지만 그렇지 않은 경우에는 가능한 _id 값을 가질지고 생성이 성공할때 까지 이 반복문은 계속됩니다.


1. insertDocument으로 함수이름을 만들어 생성작업을 하는 기능을 구현합니다. 함수는 [insert()]() 메서드를 감싸고 doc과 targetCollection 인자를 전달 받습니다.

>2.6 버전 이후
[db.collection.insert()](https://docs.mongodb.org/manual/reference/method/db.collection.insert/#db.collection.insert) 메서드는 [WriteResult](https://docs.mongodb.org/manual/reference/method/db.collection.insert/#writeresults-insert) 작업에 대한 상태값을 담아 객체를 리턴합니다. 이전 버전에서는 [db.getLastErrorObj()](https://docs.mongodb.org/manual/reference/method/db.getLastErrorObj/#db.getLastErrorObj) 메서드를 호출했어야 했습니다.

```
function insertDocument(doc, targetCollection) {

    while (1) {

        var cursor = targetCollection.find( {}, { _id: 1 } ).sort( { _id: -1 } ).limit(1);

        var seq = cursor.hasNext() ? cursor.next()._id + 1 : 1;

        doc._id = seq;

        var results = targetCollection.insert(doc);

        if( results.hasWriteError() ) {
            if( results.writeError.code == 11000 /* dup key */ )
                continue;
            else
                print( "unexpected error inserting data: " + tojson( results ) );
        }

        break;
    }
}
```

while (1) 루프는 다음과 같은 동작을 합니다.
- targetCollection 으로 부터 쿼리를 통해 document와  최대 _id값을 구합니다.
- 얻어진 document로 부터 _id 값을 위한 작업을 합니다
  - 구해진 _id 값에 1을 더합니다.
  - 아니면 리턴받은 document가 없었다면 1로 설정합니다.
- doc의 _id필드의 값에 seq 값을 설정합니다.
- targetCollection에 doc을 생성합니다.
- 만약에 생성작업 키의 중복으로 오류가 발생했다면 위의 작업을 반복합니다. 반면에 작업이 다른 문제로 인해 에러가 발생하거나 작업이 성공을 한다면 루프는 종료가 됩니다.

2. insertDocument() 메서드를 document 생성할때 사용을 합니다.

```
var myCollection = db.users2;

insertDocument(
   {
     name: "Grace H."
   },
   myCollection
);

insertDocument(
   {
     name: "Ted R."
   },
   myCollection
)
```
작업의 결과를 [find()](https://docs.mongodb.org/manual/reference/method/db.collection.find/#db.collection.find) 메서드를 통해 확인합니다.
```
db.users2.find()
```

_id 필드의 값이 순차적으로 증가했음을 확인했습니다.
```
{
  _id: 1,
  name: "Grace H."
}
{
  _id : 2,
  "name" : "Ted R."
}
```
대량의 생성작업이 필요하는 경우 이 while루프의 반복은 매우 많아 질수 있습니다.



[목록](https://github.com/yuby/mongodb-ko)