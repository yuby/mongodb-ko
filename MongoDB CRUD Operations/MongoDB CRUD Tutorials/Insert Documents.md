[목록](https://github.com/yuby/mongodb-ko)



#Insert Documents

몽고디비에서 [db.collection.insert()](http://docs.mongodb.org/manual/reference/method/db.collection.insert/#db.collection.insert)를 사용해서 새로운 documents를 collection에 추가합니다.

##Insert a Document

###1 Insert a document into a collection.
inventory라는 이름의 collection에 document를 추가합니다. 만약 해당 collection이 존재하지 않을 경우에는 새롭게 collection을 생성합니다.

```
db.inventory.insert(
   {
     item: "ABC1",
     details: {
        model: "14Q3",
        manufacturer: "XYZ Company"
     },
     stock: [ { size: "S", qty: 25 }, { size: "M", qty: 50 } ],
     category: "clothing"
   }
)
```

이 작업은 [WriteResult](http://docs.mongodb.org/manual/reference/method/WriteResult/#WriteResult) 객체와 작업의 상태를 함께 리턴합니다. 성공적으로 document를 생성하였다면 다음 객체를 리턴합니다.

```
WriteResult({ "nInserted" : 1 })
```
[nInserted](http://docs.mongodb.org/manual/reference/method/WriteResult/#WriteResult.nInserted)필드는 생성된 documents의 갯수를 의미합니다. 만약에 작업도중에 에러가 발생하는 경우 [WriteResult](http://docs.mongodb.org/manual/reference/method/WriteResult/#WriteResult)객체는 에러정보를 담아 리턴합니다.

###2 Review the inserted document.
생성작업이 성공적이라면 해당 결과를 collection에 쿼리를 통해 확인할수가 있습니다.

``
db.inventory.find()
``
생성되었던 document가 리턴됩니다.

```
{ "_id" : ObjectId("53d98f133bb604791249ca99"), "item" : "ABC1", "details" : { "model" : "14Q3", "manufacturer" : "XYZ Company" }, "stock" : [ { "size" : "S", "qty" : 25 }, { "size" : "M", "qty" : 50 } ], "category" : "clothing" }
```
리턴되는 document는 몽고디비가 추가한 _id필드가 추가된 document를 리턴합니다. 만약에 클라이언트가 임의로 _id필드를 추가하지 않는다면 몽고디비가 [ObjectId](https://docs.mongodb.org/manual/reference/object-id/)를 생성해서 _id값으로 설정을 합니다. document상의 [ObjectId](https://docs.mongodb.org/manual/reference/object-id/)값은 매번 다른 값일 것입니다.

##Insert an Array of Documents

documents를 배열의 형태로 전달하여 [db.collection.insert()](http://docs.mongodb.org/manual/reference/method/db.collection.insert/#db.collection.insert)를 통해 다수의 documents 생성이 가능합니다.

###1 Create an array of documents.
mydocuments 에 생성할 document를 배열형태로 저장을 합니다.

```
var mydocuments =
    [
      {
        item: "ABC2",
        details: { model: "14Q3", manufacturer: "M1 Corporation" },
        stock: [ { size: "M", qty: 50 } ],
        category: "clothing"
      },
      {
        item: "MNO2",
        details: { model: "14Q3", manufacturer: "ABC Company" },
        stock: [ { size: "S", qty: 5 }, { size: "M", qty: 5 }, { size: "L", qty: 1 } ],
        category: "clothing"
      },
      {
        item: "IJK2",
        details: { model: "14Q2", manufacturer: "M5 Corporation" },
        stock: [ { size: "S", qty: 5 }, { size: "L", qty: 1 } ],
        category: "houseware"
      }
    ];
  ```

###2 Insert the documents.
mydocuments을 [db.collection.insert()](http://docs.mongodb.org/manual/reference/method/db.collection.insert/#db.collection.insert) 인자로 전달하여 bulk 생성작업을 하도록 합니다.

```
db.inventory.insert( mydocuments );
```
[BulkWriteResult](http://docs.mongodb.org/manual/reference/method/BulkWriteResult/#BulkWriteResult)객체와 결과값을 리턴합니다. 성공적인 작업을 했다면 다음과 같은 형태의 객체를 확인할수 있습니다.
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
[nInserted](http://docs.mongodb.org/manual/reference/method/BulkWriteResult/#BulkWriteResult.nInserted)필드는 생성된 documents의 갯수입니다. 만약 에러가 발생한다면 [BulkWriteResult](http://docs.mongodb.org/manual/reference/method/BulkWriteResult/#BulkWriteResult)객체가 에러 정보를 담아 리턴합니다.

새롭게 생성되는 documents는 몽고디비에 의해 추가된 _id 필드를 가지고 있습니다.


##Insert Multiple Documents with Bulk

2.6버전 이후
New in version 2.6.

몽고디비는 [Bulk()](http://docs.mongodb.org/manual/reference/method/Bulk/#Bulk) api를 제공해서 다수의 쓰기작업을 지원하고 있습니다. 다음은 순차적으로 몽고디비에서 어떻게 [Bulk()](http://docs.mongodb.org/manual/reference/method/Bulk/#Bulk)  API를 사용해서 documents그룹을 새롭게 생성하는지에 대한 정보입니다.

###1 Initialize a Bulk operations builder.
inventory collection에 대한 Bulk작업의 초기화를 합니다.

```
var bulk = db.inventory.initializeUnorderedBulkOp();
``
이 작업은 비순차적인 작업리스트를 리턴합니다. 비순차적인 작업이란 몽고디비가 병렬적으로 작업들이 수행이 된다는 말입니다. 만약에 작업도중에 하나의 쓰기 작업에 에러가 발생하게 된다면 몽고디비는 남아있는 작업을 계속진행하게 됩니다.

물론 순차적으로 작업을 진행시킬수도 있습니다. [db.collection.initializeOrderedBulkOp()](http://docs.mongodb.org/manual/reference/method/db.collection.initializeOrderedBulkOp/#db.collection.initializeOrderedBulkOp) 에서 확인하시면 됩니다.

###2 Add insert operations to the bulk object.
[Bulk.insert()](http://docs.mongodb.org/manual/reference/method/Bulk.insert/#Bulk.insert) 메서드를 사용해서 두번의 생성작업을 진행합니다.

```
bulk.insert(
   {
     item: "BE10",
     details: { model: "14Q2", manufacturer: "XYZ Company" },
     stock: [ { size: "L", qty: 5 } ],
     category: "clothing"
   }
);
bulk.insert(
   {
     item: "ZYT1",
     details: { model: "14Q1", manufacturer: "ABC Company"  },
     stock: [ { size: "S", qty: 5 }, { size: "M", qty: 5 } ],
     category: "houseware"
   }
);
```

###3 Execute the bulk operation.
[execute()](http://docs.mongodb.org/manual/reference/method/Bulk.execute/#Bulk.execute) 메서드를 호출하여 리스트상의 작업을 실행합니다.

```
bulk.execute();
```
이 메서드는 [BulkWriteResult](http://docs.mongodb.org/manual/reference/method/BulkWriteResult/#BulkWriteResult)객체와 작업의 상태와 함께 넘깁니다. 성공적인 작업이라면 다음과 같은 객체를 확인할수 있습니다.
```
BulkWriteResult({
   "writeErrors" : [ ],
   "writeConcernErrors" : [ ],
   "nInserted" : 2,
   "nUpserted" : 0,
   "nMatched" : 0,
   "nModified" : 0,
   "nRemoved" : 0,
   "upserted" : [ ]
})
```
[nInserted](http://docs.mongodb.org/manual/reference/method/BulkWriteResult/#BulkWriteResult.nInserted) 필드는 생성된 document의 갯수를 의미합니다. 만약 작업중에 에러가 발생한다면 [BulkWriteResult](http://docs.mongodb.org/manual/reference/method/BulkWriteResult/#BulkWriteResult)객체는 에러에 대한 정보를 가지고 리턴됩니다.


##Additional Examples and Methods

[db.collection.insert()](http://docs.mongodb.org/manual/reference/method/db.collection.insert/#db.collection.insert)에 대한 더 많은 예를 확인하세요.

[db.collection.update()](http://docs.mongodb.org/manual/reference/method/db.collection.update/#db.collection.update) , [db.collection.findAndModify()](http://docs.mongodb.org/manual/reference/method/db.collection.findAndModify/#db.collection.findAndModify), 그리고  [db.collection.save()](http://docs.mongodb.org/manual/reference/method/db.collection.save/#db.collection.save) 메서드도 새로운 documents를 생성하는 메서드입니다. 각각의 페이지를 확인해서 더 많은 예제와 정보를 확인하세요.



[목록](https://github.com/yuby/mongodb-ko)