[목록](https://github.com/yuby/mongodb-ko)



#Limit Fields to Return from a Query

[projection](http://docs.mongodb.org/manual/reference/glossary/#term-projection) document 는 쿼리와 일치하는 document가 전달하는 필드를 제한합니다. projection document 는 필드의 추가와 제거를 설정하는 것이 가능합니다.


The specifications have the following forms:

| Syntax                | Description                           |
|-----------------------|---------------------------------------|
| <field>: <1 or true>  | Specify the inclusion of a field.     |
| <field>: <0 or false> | Specify the suppression of the field. |



>IMPORTANT
_id 필드는 기본적으로 결과값에 고정되어 있습니다. _id 필드를 제거하고 싶다면 _id : 0 으로  projection document에 설정하면 됩니다.

이 튜토리얼은 제한된 필드에 대한 다양한 쿼리 예제를 제공하고 있습니다. inventory collection을 대상으로 db.collection.find() 메서드를 몽고쉘을 통해서 구현을 합니다.[ db.collection.find() ](http://docs.mongodb.org/manual/reference/method/db.collection.find/#db.collection.find) 메서드는 검색된 documents를 [cursor](http://docs.mongodb.org/manual/core/cursors/)로 리턴합니다. 자세한 내용은 [Query Documents](http://docs.mongodb.org/manual/tutorial/query-documents/)을 확인하세요.


##Return All Fields in Matching Documents

만약 projection을 정의 하지 않는 다면 [find()](http://docs.mongodb.org/manual/reference/method/db.collection.find/#db.collection.find) 메서드는 모든 필드의 정보를 리턴합니다.
```
db.inventory.find( { type: 'food' } )
```
위 작업은 inventory collection으로 부터  type 이 food 인 경우를 검색합니다, 그리고 그 결과로 전체 컬럼의 정보가 리턴이 됩니다.


##Return the Specified Fields and the _id Field Only

projection은 몇몇 필드의 정보를 명시적으로 포함할수 있습니다. 다음 예제는 일치하는 모든 documents를 전달하는 작업을 합니다. 대신 결과로 오직 item, qty필드의 정보만을 받습니다. 기본적으로 _id 필드의 값은 가집니다.

```
db.inventory.find( { type: 'food' }, { item: 1, qty: 1 } )
```


##Return Specified Fields Only

_id플드의 정보를 결과값에서 제할수 있습니다.

```
db.inventory.find( { type: 'food' }, { item: 1, qty: 1, _id:0 } )
```
결과로 오직 item, qty필드의 정보만을 받습니다.


##Return All But the Excluded Field

만약에 모든 필드에서 하나의 필드나 필드 그룹을 결과에서 제하고 싶다면 다음과 같은 형태로 구현하면 됩니다.

```
db.inventory.find( { type: 'food' }, { type:0 } )
```
이 작업은 type필드의 정보만을 제외한 모든 필드의 정보를 리턴합니다. _id필드를 제외한 모든 필드의 정보는 이와 같은 방식으로 구현하는 것이 가능합니다.


##Return Specific Fields in Embedded Documents

[dot](http://docs.mongodb.org/manual/core/document/#document-dot-notation) 을 사용해서 내부 document의 특정 정보만을 리턴받을 수도 있습니다. collection에 다음과 같은 정보를 가진 document가 있습니다.
```
{
  "_id" : 3,
  "type" : "food",
  "item" : "aaa",
  "classification": { dept: "grocery", category: "chocolate"  }
}
```
다음 작업은 내부 classification document에 필드값을 설정하는 방법입니다.

```
db.inventory.find(
   { type: 'food', _id: 3 },
   { "classification.category": 1, _id: 0 }
)
```
이 작업의 결과로 다음의 document를 리턴합니다.

```
{ "classification" : { "category" : "chocolate" } }
```

##Suppress Specific Fields in Embedded Documents

[dot](http://docs.mongodb.org/manual/core/document/#document-dot-notation) 을 통해서 내부 document에 1대신 0을 설정해서 특정필드정보를 재한할수 있습니다. 예를들어 inventory collection아 다음과 같은 document를 가진다고 가정해보겠습니다.

```
{
   "_id" : 3,
   "type" : "food",
   "item" : "Super Dark Chocolate",
   "classification" : { "dept" : "grocery", "category" : "chocolate"},
   "vendor" : {
      "primary" : {
         "name" : "Marsupial Vending Co",
         "address" : "Wallaby Rd",
         "delivery" : ["M","W","F"]
      },
      "secondary":{
         "name" : "Intl. Chocolatiers",
         "address" : "Cocoa Plaza",
         "delivery" : ["Sa"]
      }
   }
}
```
다음 동작은 type이 food이고 _id필드가 3을 가지는 필드의 classification document의 category필드의 정보를 제한합니다. dept 필드는 그대로 남겨둡니다.
```
db.inventory.find(
   { type: 'food', _id: 3 },
   { "classification.category": 0}
)
```
이 작업은 다음과 같은 형태의 document를 리턴합니다.
```
{
   "_id" : 3,
   "type" : "food",
   "item" : "Super Dark Chocolate",
   "classification" : { "dept" : "grocery"},
   "vendor" : {
      "primary" : {
         "name" : "Bobs Vending",
         "address" : "Wallaby Rd",
         "delivery" : ["M","W","F"]
      },
      "secondary":{
         "name" : "Intl. Chocolatiers",
         "address" : "Cocoa Plaza",
         "delivery" : ["Sa"]
      }
   }
}
```
[dot](http://docs.mongodb.org/manual/core/document/#document-dot-notation)을 통한다면 어떤 깊이의 내부 documents에 접근해서 데이터를 제한할수 있습니다. 다음은 vendor의 secondary필드의 deliverydocument의 정보를 제한합니다.
```
db.inventory.find(
   { "type" : "food" },
   { "vendor.secondary.delivery" : 0 }
)
```
이 작업은 다음의 결과를 리턴합니다.
This returns all documents except the delivery array in the secondary document

{
   "_id" : 3,
   "type" : "food",
   "item" : "Super Dark Chocolate",
   "classification" : { "dept" : "grocery", "category" : "chocolate"},
   "vendor" : {
      "primary" : {
         "name" : "Bobs Vending",
         "address" : "Wallaby Rd",
         "delivery" : ["M","W","F"]
      },
      "secondary":{
         "name" : "Intl. Chocolatiers",
         "address" : "Cocoa Plaza"
      }
   }
}

##Projection for Array Fields

필드가 배열정보를 가지고 있다면 몽고디비는 [$elemMatch](http://docs.mongodb.org/manual/reference/operator/projection/elemMatch/#proj._S_elemMatch), [$slice](http://docs.mongodb.org/manual/reference/operator/projection/slice/#proj._S_slice), [$](http://docs.mongodb.org/manual/reference/operator/projection/positional/#proj._S_) 같은 연산자를 통해서 리턴받을 데이터를 조작하는 것이 가능합니다.

예를들어 inventory collection이 다음과 같은 형태의 document를 가지고 있다고 하겠습니다.
```
{ "_id" : 5, "type" : "food", "item" : "aaa", "ratings" : [ 5, 8, 9 ] }
```
그리고 다음은 $slice 연산자를 사용해 처음 2개의 배열정보만을 리턴받습니다.
```
db.inventory.find( { _id: 5 }, { ratings: { $slice: 2 } } )
```
[$elemMatch](http://docs.mongodb.org/manual/reference/operator/projection/elemMatch/#proj._S_elemMatch), [$slice](http://docs.mongodb.org/manual/reference/operator/projection/slice/#proj._S_slice), [$](http://docs.mongodb.org/manual/reference/operator/projection/positional/#proj._S_) 배열을 다룰수 있는 유일한 방법입니다.  예를들어 인덱스를 사용해서 배열의 정보를 추출하려고 할때  { "ratings.0": 1 } 이와 같은 방법으로는 원하는 결과를 얻어낼수 가 없습니다.

SEE ALSO
[Query Documents](http://docs.mongodb.org/manual/tutorial/query-documents/)



[목록](https://github.com/yuby/mongodb-ko)