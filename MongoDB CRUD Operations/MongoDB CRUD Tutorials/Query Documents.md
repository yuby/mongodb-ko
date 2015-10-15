[목록](https://github.com/yuby/mongodb-ko)



#Query Documents

몽고디비에서 [db.collection.find()](http://docs.mongodb.org/manual/reference/method/db.collection.find/#db.collection.find) 메서드는 collection으로 부터 documents를 검색할수 있습니다. [db.collection.find()](http://docs.mongodb.org/manual/reference/method/db.collection.find/#db.collection.find) 메서드는 검색된 documents의 [cursor](http://docs.mongodb.org/manual/core/cursors/)를 리턴합니다.

이 튜토리어얼은 [몽고](http://docs.mongodb.org/manual/reference/program/mongo/#bin.mongo)쉘에서  [db.collection.find()](http://docs.mongodb.org/manual/reference/method/db.collection.find/#db.collection.find)를 사용한 읽기작업 예제를 제공합니다. 이 예제들에서는 검색된 documents의 모든 필드를 리턴받거나 또는 몇개 필요한 필드에 제한을 둬서 리턴을 받는 방법을 확인할수 있습니다.  더 자세한 정보는 [Limit Fields to Return from a Query](http://docs.mongodb.org/manual/tutorial/project-fields-from-query-results/)에서 확인 하시면 됩니다.

[1]  [db.collection.findOne()](http://docs.mongodb.org/manual/reference/method/db.collection.findOne/#db.collection.findOne)메서드도 읽기작업을 통해 하나의 document를 리턴합니다. [db.collection.findOne()](http://docs.mongodb.org/manual/reference/method/db.collection.findOne/#db.collection.findOne) 메서드 대신에 [db.collection.find()](http://docs.mongodb.org/manual/reference/method/db.collection.find/#db.collection.find) 메서드에 limit 1 해서도 동일한 결과를 만들어 낼수 있습니다.


##Select All Documents in a Collection

({}) 비어있는 쿼리는 collection으로 부터 모든 documents를 읽어오겠다는 말입니다.
```
db.inventory.find( {} )
```

특정 쿼리를 정하지 않고 오직[find()](http://docs.mongodb.org/manual/reference/method/db.collection.find/#db.collection.find)만을 사용한 것은 ({})  비어있는 쿼리와 동일합니다. 그러므로 이는 위의 쿼리와 동일합니다.


'''
db.inventory.find()
'''

##Specify Equality Condition

특정 조건을 지정하기 위해서는 쿼리의 document에 { <field>: <value> }를 사용해서 모든 documents에서 <field>의 특정  <value>를 가진 documents를 추출해낼수 있습니다.

다음의 예제는 inventory collection의 모든 documents로 부터 type필드의 값이 snacks인 documents를 검색하는 방법입니다.

```
db.inventory.find( { type: "snacks" } )
```

##Specify Conditions Using Query Operators

쿼리 document는 특정조건에 [쿼리 연산자](http://docs.mongodb.org/manual/reference/operator/query/#query-selectors)를 사용해서 쿼리의 조건을 정할수 있습니다.

다음의 예제는 inventory collection의 모든 documents로 부터 type필드의 값이 food 나 snacks인 documents를 검색하는 방법입니다.

```
db.inventory.find( { type: { $in: [ 'food', 'snacks' ] } } )
```
우린 [$or](http://docs.mongodb.org/manual/reference/operator/query/or/#op._S_or) 연산자를 사용해서 표현할수 있지만 동일한 필드를 대상으로 확인을 한다면 [$in](http://docs.mongodb.org/manual/reference/operator/query/in/#op._S_in) 연산자가 여기에서 더욱 적합해 보입니다.

[Query and Projection Operators](http://docs.mongodb.org/manual/reference/operator/query/) 를 참고해서 쿼리연산자에 대한 정보를 확인해보세요.


##Specify AND Conditions

쿼리에 조건들을 조합하여 collection의 documents의 하나이상의 필드를 대상으로 조건을 정할수 있습니다. AND 의 경우에는 모든 조건들을 결합하여 모든 조건에 일치하는 documents가 대상이 됩니다.

다음의 예제는 inventory collection의 모든 documents로 부터 type필드와 [$lt](http://docs.mongodb.org/manual/reference/operator/query/lt/#op._S_lt)를 사용해서 price 필드의 값이 더 작은 졍우를 대상으로 검색하는 방법입니다.

```
db.inventory.find( { type: 'food', price: { $lt: 9.95 } } )
```
이 쿼리는 type이 food인 경우에 pricerk 9.95 이하인 모든 documents를 검색합니다. 더 많은 비교 연산자들은 [comparison operators](http://docs.mongodb.org/manual/reference/operator/query-comparison/#query-selectors-comparison)에서 확인하면 됩니다.


##Specify OR Conditions

[$or](http://docs.mongodb.org/manual/reference/operator/query/or/#op._S_or) 연산자를 사용하면 쿼리의 조건에 최소한 하나만 일치하는 경우가 있는 documents를 대상으로 동작을 합니다.

다음의 예제는 inventory collection의 모든 documents로 부터 qty필드의 크기가 100[보다 크거나](http://docs.mongodb.org/manual/reference/operator/query/gt/#op._S_gt)  price가 9.95 [보다 작은](http://docs.mongodb.org/manual/reference/operator/query/lt/#op._S_lt) 모든 경우를 대상으로 합니다.

```
db.inventory.find(
   {
     $or: [ { qty: { $gt: 100 } }, { price: { $lt: 9.95 } } ]
   }
)
```

##Specify AND as well as OR Conditions

추가적인 조건들을 통해서 정확한 조건에 일치하는 documents를 검색할수 있습니다.

다음의 예제는 inventory collection에서 type이 food 이고 qty가 100[보다 크거](http://docs.mongodb.org/manual/reference/operator/query/gt/#op._S_gt)나 price가 9.95[보다 작은 ](http://docs.mongodb.org/manual/reference/operator/query/lt/#op._S_lt)인경우를 모두 검색합니다.

```
db.inventory.find(
   {
     type: 'food',
     $or: [ { qty: { $gt: 100 } }, { price: { $lt: 9.95 } } ]
   }
)
```

##Embedded Documents

필드가 내부에서 document를 가지는 경우에는 [dot](http://docs.mongodb.org/manual/reference/glossary/#term-dot-notation) 을 통해서 해당 필드의 내부 document에 접근해서 동일한 방식으로 검색을 하면 됩니다.


###Exact Match on the Embedded Document
모든 내부document를 대상으로 검색을 하는 경우 쿼리 document의 { <field>: <value> }에서 value가 내부 document를 의미 합니다.  내부 document에서 검색작업을 하기 위해서는 우선 일치하는 특정 value값을 얻어내야 합니다.

다음의 예제는 inventory collection의 documents의 producer필드를 통해 내부 document를 얻을수 있습니다. 그리고 그 중에 company는 ABC123 이고 address가 123 Street인 경우를 모두 검색합니다.

```
db.inventory.find(
    {
      producer:
        {
          company: 'ABC123',
          address: '123 Street'
        }
    }
)
```

###Equality Match on Fields within an Embedded Document
[dot](http://docs.mongodb.org/manual/reference/glossary/#term-dot-notation) 을 통해 특정필드의 내부 document에 접근할수가 있습니다. 내부 document의 특정필드를 바탕으로 검색을 하면 그와 일치하는  collectiion상의 documents를  리턴받을수 있습니다. 내부 document는 추가적인 필드정보를 가질수가 있습니다.

다음의 예제는 [dot](http://docs.mongodb.org/manual/reference/glossary/#term-dot-notation) 을 통해 내부 document의 필드인 company의 값이 ABC123인 경우를 검색하여 일치하는 collection상의 모든 documents를 검색하고 있습니다.

```
db.inventory.find( { 'producer.company': 'ABC123' } )
```


##Arrays

필드가 배열을 값으로 가질때 쿼리가 배열의 특정 값과 일치한다면 해당 document를 추출할수 있습니다. 그리고 만약에 배열에 내부 documents를 가진다면 [dot](http://docs.mongodb.org/manual/reference/glossary/#term-dot-notation) 을 통해 세부 필드에 값과 일치하는 documents를 추출해 낼수 있습니다.

만약 [$elemMatch](http://docs.mongodb.org/manual/reference/operator/query/elemMatch/#op._S_elemMatch)를 통해 다중 조건을 정의한다면, 배열은 최소 모든 조건중에 하나라도 일치하는 것이 있어야 합니다. 더 자세한 정보는 S[ingle Element Satisfies the Criteria](http://docs.mongodb.org/manual/tutorial/query-documents/#single-element-satisfies-criteria)에서 확인하시기 바랍니다.

[$elemMatch](http://docs.mongodb.org/manual/reference/operator/query/elemMatch/#op._S_elemMatch) 사용하지 않고 다중 조건을 정의 하는 경우에는 반드시 모든 조건을 만족해야 합니다.즉, 배열의 다른 요소는 조건의 다른 부분을 만족시킬 수있습니다. 더 자세한 정보를 [Combination of Elements Satisfies the Criteria](http://docs.mongodb.org/manual/tutorial/query-documents/#combination-of-elements-satisfies-criteria) 에서 확인해보세요.

inventory collection에  다음과 같이 데이터를 가지고 있다고 가정해 보겠습니다.

```
{ _id: 5, type: "food", item: "aaa", ratings: [ 5, 8, 9 ] }
{ _id: 6, type: "food", item: "bbb", ratings: [ 5, 9 ] }
{ _id: 7, type: "food", item: "ccc", ratings: [ 9, 5, 8 ] }
```

###Exact Match on an Array
쿼리 document { <field>: <value> }의 <value>를 사용해서 배열에서 일치하는 요소를 찾을수 있습니다. 일치하는 요소를 찾기 위해서는 배열을 값과 비교의 대상이 배열의 순서까지 정확하게 일치해야 합니다.
To specify equality match on an array, use the query document { <field>: <value> } where <value> is the array to match. Equality matches on the array require that the array field match exactly the specified <value>, including the element order.

다음 쿼리 예제는 모든 documents에서 정확하게 5,8,9 세개의 엘리먼트가 일치하는 경우를 찾습니다.
```
db.inventory.find( { ratings: [ 5, 8, 9 ] } )
```

결과를 다음과 같이 리턴합니다.

```
{ "_id" : 5, "type" : "food", "item" : "aaa", "ratings" : [ 5, 8, 9 ] }
```

###Match an Array Element
배열의 하나의 요소만을 대상으로 검색이 가능합니다. 이 방법은 배열이 최소한 하나의 엘리먼트가 값과 일치를 하면 됩니다.

다음 쿼리 예제는 ratings 컬럼이 5를 가지는 경우를 검색을 합니다.
```
db.inventory.find( { ratings: 5 } )
```

결과를 다음과 같이 리턴합니다.
```
{ "_id" : 5, "type" : "food", "item" : "aaa", "ratings" : [ 5, 8, 9 ] }
{ "_id" : 6, "type" : "food", "item" : "bbb", "ratings" : [ 5, 9 ] }
{ "_id" : 7, "type" : "food", "item" : "ccc", "ratings" : [ 9, 5, 8 ] }
```


###Match a Specific Element of an Array

배열의 특정 위치나 인덱스의 값을 [dot](http://docs.mongodb.org/manual/reference/glossary/#term-dot-notation)을 통해 검색하는 것도 가능합니다.

다음예제는 첫번째 엘리먼트가 5를 가진 경우를 검색합니다.
```
db.inventory.find( { 'ratings.0': 5 } )
```

결과를 다음과 같이 리턴합니다.
```
{ "_id" : 5, "type" : "food", "item" : "aaa", "ratings" : [ 5, 8, 9 ] }
{ "_id" : 6, "type" : "food", "item" : "bbb", "ratings" : [ 5, 9 ] }
```

###Specify Multiple Criteria for Array Elements
####Single Element Satisfies the Criteria
[$elemMatch연산자](http://docs.mongodb.org/manual/reference/operator/query/elemMatch/#op._S_elemMatch)를 사용해서 다중의 제약사항으로 검색을 하는 경우 최소한 배열중에 하나라도  모든 조건에 일치해야 됩니다.

다음예제는 엘리먼트중에 하나가  5보다 [크고](http://docs.mongodb.org/manual/reference/operator/query/gt/#op._S_gt) 9보다 [작은](http://docs.mongodb.org/manual/reference/operator/query/lt/#op._S_lt) 요소를 가진 대상을 검색을 합니다.
```
db.inventory.find( { ratings: { $elemMatch: { $gt: 5, $lt: 9 } } } )
```

위 쿼리의 결과는 5보다 크고 9보다 작은 값인  8을 가진 documents들을 검색하여 리턴해줍니다.
```
{ "_id" : 5, "type" : "food", "item" : "aaa", "ratings" : [ 5, 8, 9 ] }
{ "_id" : 7, "type" : "food", "item" : "ccc", "ratings" : [ 9, 5, 8 ] }
```

####Combination of Elements Satisfies the Criteria
다음의 쿼리는 ratings 배열이 쿼리의 조건을 만족하는 경우인데 하나의 요소가 5보다 크고 다른 엘리먼트가 9보다 작은 경우거나 하나의 엘리먼트가 두가지를 모두 만족하는 경우입니다.
```
db.inventory.find( { ratings: { $gt: 5, $lt: 9 } } )
```

위 쿼리는 다음의 결과를 리턴합니다.
```
{ "_id" : 5, "type" : "food", "item" : "aaa", "ratings" : [ 5, 8, 9 ] }
{ "_id" : 6, "type" : "food", "item" : "bbb", "ratings" : [ 5, 9 ] }
{ "_id" : 7, "type" : "food", "item" : "ccc", "ratings" : [ 9, 5, 8 ] }
```
"ratings" : [ 5, 9 ]의 경우에는 9보다 작은 경우를 5가 만족하고 5보다 큰경우에 9가 만족하게 됩니다.


###Array of Embedded Documents
inventory collection이 다음의 documents를 가진다고 가정하겠습니다.

{
  _id: 100,
  type: "food",
  item: "xyz",
  qty: 25,
  price: 2.5,
  ratings: [ 5, 8, 9 ],
  memos: [ { memo: "on time", by: "shipping" }, { memo: "approved", by: "billing" } ]
}

{
  _id: 101,
  type: "fruit",
  item: "jkl",
  qty: 10,
  price: 4.25,
  ratings: [ 5, 9 ],
  memos: [ { memo: "on time", by: "payment" }, { memo: "delayed", by: "shipping" } ]
}

####Match a Field in the Embedded Document Using the Array Index
만약에 내부 document의 배열의 인덱스 정보를 알고 있다면 [dot](http://docs.mongodb.org/manual/reference/glossary/#term-dot-notation)을 통해 특정 조건을 지정할수 있습니다.

다음의 예제는 memos 배열의 첫번째 document의 by컬럼이 'shipping'인 경우를 검색합니다.
```
db.inventory.find( { 'memos.0.by': 'shipping' } )
```
위 쿼리는 다음과 같은 결과를 리턴합니다.

```
{
   _id: 100,
   type: "food",
   item: "xyz",
   qty: 25,
   price: 2.5,
   ratings: [ 5, 8, 9 ],
   memos: [ { memo: "on time", by: "shipping" }, { memo: "approved", by: "billing" } ]
}
```

####Match a Field Without Specifying Array Index
만약에 배열의 document의 인덱스 정보를 모른다면 dot을 통해 배열에 존재하는 필드이름을 연결하여 검색을 진행합니다.

다음예제는 memos배열에 최소하나의 document가 by필드를 가지고 있고 그 값이 'shipping'인 경우를 검색합니다.

```
db.inventory.find( { 'memos.by': 'shipping' } )
```

위 쿼리는 다음과 같은 결과를 리턴합니다.
```
{
  _id: 100,
  type: "food",
  item: "xyz",
  qty: 25,
  price: 2.5,
  ratings: [ 5, 8, 9 ],
  memos: [ { memo: "on time", by: "shipping" }, { memo: "approved", by: "billing" } ]
}
{
  _id: 101,
  type: "fruit",
  item: "jkl",
  qty: 10,
  price: 4.25,
  ratings: [ 5, 9 ],
  memos: [ { memo: "on time", by: "payment" }, { memo: "delayed", by: "shipping" } ]
}
```

###Specify Multiple Criteria for Array of Documents
####Single Element Satisfies the Criteria
[$elemMatch](http://docs.mongodb.org/manual/reference/operator/query/elemMatch/#op._S_elemMatch) 연산자를 통해 다중의 조건을 배열내의 내부 documents를 상대로 검색을 하여 최소 하나의 엘리먼트가 모든 조건을 만족하는 지를 검색합니다.

다음의 예제는 memos 배열의  내부 documents중에 최소 하나라도 두가지 필드를 가지고 값을 비교합니다.
The following example queries for documents where the memos array has at least one embedded document that contains both the field memo equal to 'on time' and the field by equal to 'shipping':

```
db.inventory.find(
   {
     memos:
       {
          $elemMatch:
            {
               memo: 'on time',
               by: 'shipping'
            }
       }
    }
)
```

위 쿼리는 다음과 같은 결과를 리턴합니다.
```
{
   _id: 100,
   type: "food",
   item: "xyz",
   qty: 25,
   price: 2.5,
   ratings: [ 5, 8, 9 ],
   memos: [ { memo: "on time", by: "shipping" }, { memo: "approved", by: "billing" } ]
}
```

####Combination of Elements Satisfies the Criteria
다음의 쿼리는 memos 배열의 엘리먼트가 다음의 조건을 만적하는 경우입니다. 하나의 엘리먼트가 memo필드의 값이 'on time'이고 다른 하나의 엘리먼트는 by 필드가 'shipping'을 만족하는 경우이거나 아니면 하나의 필드가 두가지의 경우를 모두 만족하는 경우입니다.

```
db.inventory.find(
  {
    'memos.memo': 'on time',
    'memos.by': 'shipping'
  }
)
```

위 쿼리는 다음과 같은 결과를 리턴합니다.
'''
{
  _id: 100,
  type: "food",
  item: "xyz",
  qty: 25,
  price: 2.5,
  ratings: [ 5, 8, 9 ],
  memos: [ { memo: "on time", by: "shipping" }, { memo: "approved", by: "billing" } ]
}
{
  _id: 101,
  type: "fruit",
  item: "jkl",
  qty: 10,
  price: 4.25,
  ratings: [ 5, 9 ],
  memos: [ { memo: "on time", by: "payment" }, { memo: "delayed", by: "shipping" } ]

}
'''

SEE ALSO
[Limit Fields to Return from a Query](http://docs.mongodb.org/manual/tutorial/project-fields-from-query-results/)




[목록](https://github.com/yuby/mongodb-ko)