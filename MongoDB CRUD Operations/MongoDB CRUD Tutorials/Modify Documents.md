[목록](https://github.com/yuby/mongodb-ko)



#Modify Documents

몽고디비는 [update()](http://docs.mongodb.org/manual/reference/method/db.collection.update/#db.collection.update)메서드를 제공하여 collection의 documents를 수정하는 기능을 제공합니다. 메서드는 파라메터를 받습니다.

- 조건에 일치하는 documents를 업데이트 합니다.
- 세부적인 수정 작업을 할수 있습니다.
- 옵션을 통해 수정을 작업을 할수 있습니다.

세부적인 업데이트 조건은 쿼리조건을 지정할때와 동일한 구조와 신택스를 사용합니다

기본적으로 [update()](http://docs.mongodb.org/manual/reference/method/db.collection.update/#db.collection.update)는 하나의 document를 대상으로 진행이 되고 [multi](http://docs.mongodb.org/manual/reference/method/db.collection.update/#multi-parameter) option을 사용해서 다수의 documents를 업데이트 할수 있습니다.


##Update Specific Fields in a Document

필드의 값을 [수정](https://docs.mongodb.org/manual/reference/operator/update/)할때  [$set](http://docs.mongodb.org/manual/reference/operator/update/set/#up._S_set) 같은 연산자를 통해서 값을 수정합니다.

[$set](http://docs.mongodb.org/manual/reference/operator/update/set/#up._S_set) 같은 업데이트 연산자의 경우에는 필드가 존재하지 않는경우에는 필드를 생성하고 값을 추가합니다.  추가적인 [업데이트 연산자](https://docs.mongodb.org/manual/reference/operator/update/)를 확인하시기 바랍니다.

### 1 Use update operators to change field values.
document의 item필드의 값이 "MNO2" 인 경우에는 [$set](http://docs.mongodb.org/manual/reference/operator/update/set/#up._S_set) 연산자를 사용해서 category 필드와 detail 필드의 세부적인 값을 수정하고 [$currentDate](http://docs.mongodb.org/manual/reference/operator/update/currentDate/#up._S_currentDate) 연산자를 통해서 lastModified필드의 값을 현재 시간으로 수정합니다.

```
db.inventory.update(
    { item: "MNO2" },
    {
      $set: {
        category: "apparel",
        details: { model: "14Q3", manufacturer: "XYZ Company" }
      },
      $currentDate: { lastModified: true }
    }
)
```
수정작업은 [WriteResult](http://docs.mongodb.org/manual/reference/method/WriteResult/#WriteResult) 에 작업의 상태를 담아 리턴합니다. 성공적으로 작업이 마쳤다면

```
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
```
[nMatched](http://docs.mongodb.org/manual/reference/method/WriteResult/#WriteResult.nMatched)필드는 조건과 일치된 갯수를 [nModified](http://docs.mongodb.org/manual/reference/method/WriteResult/#WriteResult.nModified)는 수정된 documents의 갯수를 나타냅니다.

### 2 Update an embedded field.
내부 document를 수정하기 위해서는 [dot](http://docs.mongodb.org/manual/reference/glossary/#term-dot-notation)을 사용해서 실제 원하는 필드까지 접근하면 됩니다.

다음은 내부 document의 detail이 model 필드의 값을 변경하는 방법입니다.

```
db.inventory.update(
  { item: "ABC1" },
  { $set: { "details.model": "14Q2" } }
)
```
수정작업의 결과로 [WriteResult](http://docs.mongodb.org/manual/reference/method/WriteResult/#WriteResult) 객체를 작업의 상태값과 함께 리턴합니다. 성공적인 수정작업의 결과로 다음과 같은 형태의 document를 리턴합니다.
```
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
```

### 3 Update multiple documents.
기본적으로 [update()](http://docs.mongodb.org/manual/reference/method/db.collection.update/#db.collection.update)메서드는 하나의 document를 대상으로 합니다. 다수의 documents를 수정하기 위해서 multi 옵션을 사용해야 합니다.

category필드의 값을 apparel로 수정을 하고 lastModified필드의 값을 현재 날짜로 수정하는 작업을 category필드가 clothing모든 documents를 대상으로 합니다.

```
db.inventory.update(
   { category: "clothing" },
   {
     $set: { category: "apparel" },
     $currentDate: { lastModified: true }
   },
   { multi: true }
)
```
수정작업의 결과로 [WriteResult](http://docs.mongodb.org/manual/reference/method/WriteResult/#WriteResult) 객체를 작업의 상태값과 함께 리턴합니다. 성공적인 수정작업의 결과로 다음과 같은 형태의 document를 리턴합니다.
```
WriteResult({ "nMatched" : 3, "nUpserted" : 0, "nModified" : 3 })
```

##Replace the Document
_id필드를 제외한 document의 모든 컨텐츠를 수정할때는 전체 정보를 담은 새로운 document를 [update()](http://docs.mongodb.org/manual/reference/method/db.collection.update/#db.collection.update) 메서드의 두번째 인자로 전달하면 됩니다.

새로운 document로 대체하는 경우에는 원래의 필드의 정보와 다른 필드가 추가 될수도 있습니다. _id필드의 경우에는 불변의 필드이기 때문에 생략가능합니다. 만약 _id필드를 포함시킨다면 반드시 기존에 존재하는 값과 동일한 값이 어야합니다.

###1 Replace a document.
다음의 작업은 item이 BE10 값을 가지는 document를 대체하는 작업을 합니다. 새롭게 대체될 document의 경우에는 원래 가진 _id필드와 새로운 document로 대체될것입니다.

```
db.inventory.update(
   { item: "BE10" },
   {
     item: "BE05",
     stock: [ { size: "S", qty: 20 }, { size: "M", qty: 5 } ],
     category: "apparel"
   }
)
```
수정작업의 결과로 [WriteResult](http://docs.mongodb.org/manual/reference/method/WriteResult/#WriteResult) 객체를 작업의 상태값과 함께 리턴합니다. 성공적인 수정작업의 결과로 다음과 같은 형태의 document를 리턴합니다.
```
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
```


##upsert Option
기본적으로 [update()](http://docs.mongodb.org/manual/reference/method/db.collection.update/#db.collection.update) 메서드는 일치하는 doucments가 없으면 아무것도 하지 않습니다.

하지만 [upsertL true](http://docs.mongodb.org/manual/reference/method/db.collection.update/#upsert-parameter) 를 지정하면 [update()](http://docs.mongodb.org/manual/reference/method/db.collection.update/#db.collection.update)메서드는 일치하는 document에 대해서는 수정작업을 그리고 존재하지 않는 경우에는 새롭게 doucment를 생성하는 작업을 동시에 합니다.

### 1 Specify upsert: true for the update replacement operation.
upsert: true를 사용해서 document를 대체하거나 일치하는 document가 존재하지 않는 경우에는 몽고디비는 두번째 인자로 전달된 document와 동일한 정보의 document를 생성하거나 _id필드만을 제외하고 이 정보로 정보를 대체합니다.

```
db.inventory.update(
   { item: "TBD1" },
   {
     item: "TBD1",
     details: { "model" : "14Q4", "manufacturer" : "ABC Company" },
     stock: [ { "size" : "S", "qty" : 25 } ],
     category: "houseware"
   },
   { upsert: true }
)
```
수정작업의 결과로 [WriteResult](http://docs.mongodb.org/manual/reference/method/WriteResult/#WriteResult) 객체를 작업의 상태값과 수정되거나 생성된 doucment의 정보를 함께 리턴합니다. 성공적인 수정작업의 결과로 다음과 같은 형태의 document를 리턴합니다.
```
WriteResult({
    "nMatched" : 0,
    "nUpserted" : 1,
    "nModified" : 0,
    "_id" : ObjectId("53dbd684babeaec6342ed6c7")
})
```
[nMatched](http://docs.mongodb.org/manual/reference/method/WriteResult/#WriteResult.nMatched)필드는 일치한 documents의 갯수를 보여 줍니다.
[nUpserted](http://docs.mongodb.org/manual/reference/method/WriteResult/#WriteResult.nUpserted)는 새롭게 추가된 document의 갯수를 보여 줍니다.
[nModified](http://docs.mongodb.org/manual/reference/method/WriteResult/#WriteResult.nModified) 가 0이라는 말은 업데이트를 위해 일치하는 document가 존재하지 않는 다는 말입니다.
_id필드의 경우에는 추가된 document의 정보입니다.

### 2 Specify upsert: true for the update specific fields operation.
upsert: true을 정의 해서 특정 필드의 값을 수정하거나 일치하는  document가 존재하지 않는 경우 몽고디비는 두번째 인자로 전달된 document 와 동일한 정보의 document를 생성하거나 해당 정보로 정보를 업데이트 합니다.

```
db.inventory.update(
   { item: "TBD2" },
   {
     $set: {
        details: { "model" : "14Q3", "manufacturer" : "IJK Co." },
        category: "houseware"
     }
   },
   { upsert: true }
)
```
수정작업의 결과로 [WriteResult](http://docs.mongodb.org/manual/reference/method/WriteResult/#WriteResult) 객체를 작업의 상태값과 수정되거나 생성된 doucment의 정보를 함께 리턴합니다. 성공적인 수정작업의 결과로 다음과 같은 형태의 document를 리턴합니다.
```
WriteResult({
    "nMatched" : 0,
    "nUpserted" : 1,
    "nModified" : 0,
    "_id" : ObjectId("53dbd7c8babeaec6342ed6c8")
})
```
[nMatched](http://docs.mongodb.org/manual/reference/method/WriteResult/#WriteResult.nMatched)는 일치해서 작업을 진행한 갯수를 알려줍니다.
[nUpserted](http://docs.mongodb.org/manual/reference/method/WriteResult/#WriteResult.nUpserted)가 1이라는 말은 수정으로 추가된 document의 갯수를 말합니다.
[nModified](http://docs.mongodb.org/manual/reference/method/WriteResult/#WriteResult.nModified)가 0이라는 말은 일치하는 document가 업어서 업데이트된 document가 없다는 말입니다.
_id필드는 추가된 doicument의 정보입니다.

##Additional Examples and Methods

더 많은 정보는  [db.collection.update()](http://docs.mongodb.org/manual/reference/method/db.collection.update/#db.collection.update) 정보 페이지의 [Update examples](http://docs.mongodb.org/manual/reference/method/db.collection.update/#update-method-examples)을 확인 하시면 됩니다.

[db.collection.findAndModify()](http://docs.mongodb.org/manual/reference/method/db.collection.findAndModify/#db.collection.findAndModify) 와 [db.collection.save()](http://docs.mongodb.org/manual/reference/method/db.collection.save/#db.collection.save) 메서드도 존재하는 document에 대해서는 수정을 그렇지 않은 경우에는 새로운것을 생성합니다. 각각의 정보 페이지에서 더 많은 정보를 확인하세요.



[목록](https://github.com/yuby/mongodb-ko)

