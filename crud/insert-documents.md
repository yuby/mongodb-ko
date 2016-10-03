#Insert Documents

##Insert Methods
몽고디비는 다음의 메서드를 사용해 document의 삽입동작을 수행합니다.

- db.collection.insertOne()
- db.collection.insertMany()
- db.collection.insert()

##Inset Behavior
###Collection Creation
만약에 insert 동작을 수행하는 대상 collection이 존재하지 않는 다면 해당 collection을 생성합니다.

###_id Field

몽고디비에서 document의 _id 키를 유니크한 primary key로 간주하고 저장을 합니다. 만약에 document상에 _id 필드가 정의 되지 않은 경우에 몽고디비는 ObejectId를 _id 필드의 기본값으로 사용합니다. 즉, document 입력당시에 해당 document의 최상위단에 _id 필드가 존재하지 않은 경우 몽고디비 드라이버가 ObejctId를 기본값으로 가진 _id를 추가하게 되니다.

추가적은 만약에 mongod _id필드가 존재하지 않은 document를 insert작업을 하게 되는 경우에도 ObejctId를 기본값으로 가진 _id를 추가하게 되니다.


###Atomicity
모든 쓰기작업에 대해서는 document단위로 원자성을 보장합니다.


##db.collection.insertOne()
db.collection.insertOne() 는 하나의 document를 collection에 저장하는 메서드입니다.

다음의 예제는 새로운 유저정보를 users collection에 저장을 하면서 _id 필드를 지정하지 않는 경우입니다. 이경우 자동으로 몽고디비가 ObjectId를 가진 _id 필드를 추가하게 됩니다.
```
db.users.insertOne(
   {
      name: "sue",
      age: 19,
      status: "P"
   }
)
```
위의 작업을 성공적으로 수행하면 다음의 결과를 리턴합니다.

```
{
   "acknowledged" : true,
   "insertedId" : ObjectId("5742045ecacf0ba0c3fa82b0")
}
```
저장된 document를 _id 필드를 가지고 읽어오는 쿼리입니다.
```
db.users.find( { _id: ObjectId("5742045ecacf0ba0c3fa82b0") } )
```

##db.collection.insertMany()
db.collection.insertMany() 메서드는 다중의 document를 collection에 저장하는 메서드입니다.
다음의 예제는 users collection에 3명의 사용자 정보를 추가하는 경우입니다. 이경우에도 _id 필드를 지정하지 않는 경우입니다. 이경우 자동으로 몽고디비가 ObjectId를 가진 _id 필드를 추가하게 됩니다.
```
db.users.insertMany(
   [
     { name: "bob", age: 42, status: "A", },
     { name: "ahn", age: 22, status: "A", },
     { name: "xi", age: 34, status: "D", }
   ]
)
```
성공적으로 저장이 되면 다음과 같은 결과를 리턴합니다.
```
{
   "acknowledged" : true,
   "insertedIds" : [
      ObjectId("57420d48cacf0ba0c3fa82b1"),
      ObjectId("57420d48cacf0ba0c3fa82b2"),
      ObjectId("57420d48cacf0ba0c3fa82b3")
   ]
}
```
그리고 해당 결과를 $in 연산자를 사용해 한번에 읽어들일수 있습니다.
```
db.users.find(
   { _id:
      { $in:
         [
            ObjectId("57420d48cacf0ba0c3fa82b1"),
            ObjectId("57420d48cacf0ba0c3fa82b2"),
            ObjectId("57420d48cacf0ba0c3fa82b3")
         ]
      }
   }
)
```

##db.collection.insert()
db.collection.insert() 는 collection에 document를 하나 또는 여러개를 저장할수 있습니다. 하나를 저장하는 경우 해당 document를 인자로 전달하고 여러개를 저장하는 경우 배열에 담아 인자로 전달하면 됩니다.
```
db.users.insert(
   {
      name: "sue",
      age: 19,
      status: "P"
   }
)
```
이경우에는 동작의 결과로  WriteResult 객체를 리턴합니다. 성공적으로 동작이 수행된경우 다음의 결과를 확인할수 있습니다.

```
WriteResult({ "nInserted" : 1 })
```

nInserted 필드의 경우에는 저장된 document의 갯수를 나타냅니다. 만약에 동작이 실패하는 경우 실패한 정보를 WriteResult 상에 담아 리턴을 합니다.

```
db.users.insert(
   [
     { name: "bob", age: 42, status: "A", },
     { name: "ahn", age: 22, status: "A", },
     { name: "xi", age: 34, status: "D", }
   ]
)
```
디수의 document를 저장하는 경우에는 BulkWriteResult 객체를 전달합니다. BulkWriteResult  객체는 다음과 같습니다.
```
BulkWriteResult({
   "writeErrors" : [ ],
   "writeConcernErrors" : [ ],
   "nInserted" : 3,
   "nUpserted" : 0,
   "nMatched" : 0,
   "nModified" : 0,
   "nRemoved" : 0,
   "upserted" : [ ]
})
```

##Additional Methods
다음은 새로운 document를 추가하는 다른 메서드들입니다.

- db.collection.update() - upsert: true.
- db.collection.updateOne() - upsert: true.
- db.collection.updateMany() - upsert: true.
- db.collection.findAndModify() - upsert: true.
- db.collection.findOneAndUpdate() - upsert: true.
- db.collection.findOneAndReplace() - upsert: true.
- db.collection.save().
- db.collection.bulkWrite().

##Write Acknowledgement
쓰기작업시에 write concern을 함께 사용한다면 요청에 대한 구체적인 설정을 할수가 있습니다.





