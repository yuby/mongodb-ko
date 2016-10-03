#Query Documents

##Query Method
db.collection.find() 메서드는 해당 collection에서 일치하는 document 정보를 cursor로 전달을 합니다.
```
db.collection.find( <query filter>, <projection> )
```
db.collection.find() 메서드에 특정한 필드 옵션을 추가 할수 있습니다.

- query filter 를 통한 조건 추가
- 전달받은 결과 document에 projection 설정을 통해 전달되는 필드 종류 설정. projection은 몽고디비가 제한하는 양만큼의 데이터까지만 설정가능합니다.

이외에도  limits, skips, sort 메서드를 사용해서 전달된 cursor를 조작할수 있습니다. 전달받은 document의 경우에는 sort() 메서드를 사용하기전까지는 정렬방식이 지정되지 않습니다.

##Example Collection
mongo shell에서 db.collection.find()를 사용하는 예제입니다. mongo shell에서 별도의 리턴받은 cursor에 변수를 지정하지 않은면 리턴된 결과의 처음 20개의 document를 반복적으로 리턴하게 됩니다.

아래의 코드를 mongo shell 에서 실행하시기 바랍니다.
>NOTE:
>만약 users collection이 존재한다면 해당 collection을 drop 시키고 아래 코드를 실행하시기 바랍니다.

```
db.users.insertMany(
  [
     {
       _id: 1,
       name: "sue",
       age: 19,
       type: 1,
       status: "P",
       favorites: { artist: "Picasso", food: "pizza" },
       finished: [ 17, 3 ],
       badges: [ "blue", "black" ],
       points: [
          { points: 85, bonus: 20 },
          { points: 85, bonus: 10 }
       ]
     },
     {
       _id: 2,
       name: "bob",
       age: 42,
       type: 1,
       status: "A",
       favorites: { artist: "Miro", food: "meringue" },
       finished: [ 11, 25 ],
       badges: [ "green" ],
       points: [
          { points: 85, bonus: 20 },
          { points: 64, bonus: 12 }
       ]
     },
     {
       _id: 3,
       name: "ahn",
       age: 22,
       type: 2,
       status: "A",
       favorites: { artist: "Cassatt", food: "cake" },
       finished: [ 6 ],
       badges: [ "blue", "red" ],
       points: [
          { points: 81, bonus: 8 },
          { points: 55, bonus: 20 }
       ]
     },
     {
       _id: 4,
       name: "xi",
       age: 34,
       type: 2,
       status: "D",
       favorites: { artist: "Chagall", food: "chocolate" },
       finished: [ 5, 11 ],
       badges: [ "red", "black" ],
       points: [
          { points: 53, bonus: 15 },
          { points: 51, bonus: 15 }
       ]
     },
     {
       _id: 5,
       name: "xyz",
       age: 23,
       type: 2,
       status: "D",
       favorites: { artist: "Noguchi", food: "nougat" },
       finished: [ 14, 6 ],
       badges: [ "orange" ],
       points: [
          { points: 71, bonus: 20 }
       ]
     },
     {
       _id: 6,
       name: "abc",
       age: 43,
       type: 1,
       status: "A",
       favorites: { food: "pizza", artist: "Picasso" },
       finished: [ 18, 12 ],
       badges: [ "black", "blue" ],
       points: [
          { points: 78, bonus: 8 },
          { points: 57, bonus: 7 }
       ]
     }
  ]
)
```

##Select All Documents in a Collection
빈 query filter 는 collection의 모든 document를 리턴하게 됩니다.
```
db.users.find( {} )
```
query filter를 작성하지 않고 db.users.find() 로만 작성을 해도 db.users.find( {} )와 동일합니다.
```
db.users.find()
```

##Specify Query Filter Conditions

###Specify Equality Condition
query filter document의 경우  <field>:<value> 의 방식으로 조건을 설정합니다.
```
{ <field1>: <value1>, ... }
```

다음의 예제는 status가 'A'인 사람의 정보를 확인하는 코드입니다.
```
db.users.find( { status: "A" } )
```

###Specify Conditions Using Query Operators
query filter에 query 연산자를 사용할수 있습니다.
```
{ <field1>: { <operator1>: <value1> }, ... }
```
다음은 status가 'P' , 'D'를 가지는 경우를 찾는 코드입니다.
```
db.users.find( { status: { $in: [ "P", "D" ] } } )
```
위의 코드의 경우 $or 연산자를 사용하여 표현하는 것도 가능하지만 위와 같이 동일한 필드의 값을 확인 하는 경우에는 $in 연산자를 사용하는 것이 성능상 더 뛰어납니다.

###Specify AND Conditions
하나 이상의 조건을 지정할수 가 있습니다. 모든 조건을 만족하는 collection의 document를 선택하기 위해서 AND 연산을 수행해야합니다. 다음은 users의 status가 'A' 이고(and) age가 30 이하($lt)인 경우를 찾는 코드입니다.

```
db.users.find( { status: "A", age: { $lt: 30 } } )
```

###Specify OR Conditions
$or 연산의 경우 최소한 하나의 조건에 일치하는 경우의 document를 찾는 경우입니다. 다음은 users의 status가 'A' 이거나(or) age가 30 이하($lt)인 경우를 찾는 코드입니다.

```
db.users.find(
   {
     $or: [ { status: "A" }, { age: { $lt: 30 } } ]
   }
)
```

###Specify AND as well as OR Conditions
조금더 정교한 조건을 설장하는 것이 가능합니다.

다음의 예는 status는 'A'이고 age가 30이하이거나 type이 1인 경우를 찾는 코드입니다.
```
db.users.find(
   {
     status: "A",
     $or: [ { age: { $lt: 30 } }, { type: 1 } ]
   }
)
```

##Query on Embedded Documents
필드가 내부 document 형태로 데이터를 가지는 경우가 있고, 내부 document의 특정 필드와 일치하는 경우나 내우 document 자체를 비교하는 것이 (.) dot notation을 사용해서 접근이 가능합니다.

###Exact Match on the Embedded Document
{ <field>: <value> } 의 value가 내부 document인 경우 해당 document 자체와 일치하는 특정 데이터를 추출하는 것이가능합니다.

다음의 예제는 favorites 필드가 가지는 내부 document를 비교하는 쿼리입니다.

```
db.users.find( { favorites: { artist: "Picasso", food: "pizza" } } )
```

###Equality Match on Fields within an Embedded Document
(.) dot notation을 사용하면 내부 document의 필드에 접근할수 있습니다. 내부 document의 특정필드와 값이 일치하는 경우 해당 document의 정보를 읽을수가 있습니다. 내부document의 경우에는 추가적인 필드정보를 가질수가있습니다.

다음 예제는 (.) 을 사용해서 favorites 내부 document의 artist 필드의 값이 Picasso인 경우를 선택하는 쿼리입니다.

```
db.users.find( { "favorites.artist": "Picasso" } )
```

##Query on Arrays

필드가 array 값을 가지는 경우, array의 특정 값을 조건으로 document를 추출하룻 있습니다. 만약에 array의 내부에 document를 가지는 경우 해당 필드에 대해서도 (.)을 통해 필드에 접근할수 있습니다.

만약에 array에 다수의 값이 존재하는지 다중 조건이 필요한 경우에는 $elemMatch 연산자를 사용합니다. 이 경우에는 최소 하나의 조건이 일치하면 해당 document를 추출할수 있습니다.

만약에 $elemMatch 연산자없이 다중의 조건을 조합한다면, 반드시 모든 조건을 만족해야 해당 document가 추출이 됩니다. 즉, array의 다른 엘리먼트가 존재하는 다른 조건들에도 만족되어야 합니다.

##Exact Match on an Array
document { <field>: <value> } 의 value가 array를 가지는 경우 동일한 array를 가지는 document를 추출할수 있습니다.
```
db.users.find( { badges: [ "blue", "black" ] } )
```

위의 쿼리와 일치하는 document의 형태입니다.
```
{
   "_id" : 1,
   "name" : "sue",
   "age" : 19,
   "type" : 1,
   "status" : "P",
   "favorites" : { "artist" : "Picasso", "food" : "pizza" },
   "finished" : [ 17, 3 ]
   "badges" : [ "blue", "black" ],
   "points" : [ { "points" : 85, "bonus" : 20 }, { "points" : 85, "bonus" : 10 } ]
}
```

###Match an Array Element
array의 특정 엘리먼트와 일치하는 경우, 해당 array에 하나만 일치하는 값이 있어도 해당 document를 추출할수 있습니다.
```
{
   "_id" : 1,
   "name" : "sue",
   "age" : 19,
   "type" : 1,
   "status" : "P",
   "favorites" : { "artist" : "Picasso", "food" : "pizza" },
   "finished" : [ 17, 3 ]
   "badges" : [ "blue", "black" ],
   "points" : [ { "points" : 85, "bonus" : 20 }, { "points" : 85, "bonus" : 10 } ]
}
{
   "_id" : 4,
   "name" : "xi",
   "age" : 34,
   "type" : 2,
   "status" : "D",
   "favorites" : { "artist" : "Chagall", "food" : "chocolate" },
   "finished" : [ 5, 11 ],
   "badges" : [ "red", "black" ],
   "points" : [ { "points" : 53, "bonus" : 15 }, { "points" : 51, "bonus" : 15 } ]
}
{
   "_id" : 6,
   "name" : "abc",
   "age" : 43,
   "type" : 1,
   "status" : "A",
   "favorites" : { "food" : "pizza", "artist" : "Picasso" },
   "finished" : [ 18, 12 ],
   "badges" : [ "black", "blue" ],
   "points" : [ { "points" : 78, "bonus" : 8 }, { "points" : 57, "bonus" : 7 } ]
}
```

###Match a Specific Element of an Array
(.)dot notation을 사용하면 array의 엘리먼트를 특정 index나 위치값을 가지고 해당 document를 추출하는 것이 가능합니다.
```
db.users.find( { "badges.0": "black" } )
```

결과는 다음과 같습니다.

```
{
   "_id" : 6,
   "name" : "abc",
   "age" : 43,
   "type" : 1,
   "status" : "A",
   "favorites" : { "food" : "pizza", "artist" : "Picasso" },
   "finished" : [ 18, 12 ],
   "badges" : [ "black", "blue" ],
   "points" : [ { "points" : 78, "bonus" : 8 }, { "points" : 57, "bonus" : 7 } ]
}
```

##Specify Multiple Criteria for Array Elements

###Single Element Satisfies the Criteria

$elemMatch를 사용하면 array의 엘리먼트가 하나의 조건만 만족하더라도 해당 document를 추출할수 있습니다.

```
db.users.find( { finished: { $elemMatch: { $gt: 15, $lt: 20 } } } )
```

다음의 둘중 하나의 조건에 일치하는 array를 가진 document의 형태입니다.

```
{
   "_id" : 1,
   "name" : "sue",
   "age" : 19,
   "type" : 1,
   "status" : "P",
   "favorites" : { "artist" : "Picasso", "food" : "pizza" },
   "finished" : [ 17, 3 ]
   "badges" : [ "blue", "black" ],
   "points" : [ { "points" : 85, "bonus" : 20 }, { "points" : 85, "bonus" : 10 } ]
}
{
   "_id" : 6,
   "name" : "abc",
   "age" : 43,
   "type" : 1,
   "status" : "A",
   "favorites" : { "food" : "pizza", "artist" : "Picasso" },
   "finished" : [ 18, 12 ],
   "badges" : [ "black", "blue" ],
   "points" : [ { "points" : 78, "bonus" : 8 }, { "points" : 57, "bonus" : 7 } ]
}
```


###Combination of Elements Satisfies the Criteria
다음은 finished 의 array가 다음 쿼리의 조건에 일치하는 경우입니다. 예를들면 하나의 엘리먼트가 15보다 크고 다른 엘리먼트가 20보다 작은 경우거나 아니면 하나의 엘리먼트가 두조건 모두에 일치하는 경우입니다.

```
db.users.find( { finished: { $gt: 15, $lt: 20 } } )
```

> 위의 경우 배열중에 엘리먼트 하나가 15보다 큰게 있고 다른 엘리먼트가 20보다 작은 경우거나 하나의 엘리먼트가 15와 20사이의 숫자인 경우입니다.

위의 코드에 만족하는 array의 형태입니다.
```
{
   "_id" : 1,
   "name" : "sue",
   "age" : 19,
   "type" : 1,
   "status" : "P",
   "favorites" : { "artist" : "Picasso", "food" : "pizza" },
   "finished" : [ 17, 3 ]
   "badges" : [ "blue", "black" ],
   "points" : [ { "points" : 85, "bonus" : 20 }, { "points" : 85, "bonus" : 10 } ]
}
{
   "_id" : 2,
   "name" : "bob",
   "age" : 42,
   "type" : 1,
   "status" : "A",
   "favorites" : { "artist" : "Miro", "food" : "meringue" },
   "finished" : [ 11, 20 ],
   "badges" : [ "green" ],
   "points" : [ { "points" : 85, "bonus" : 20 }, { "points" : 64, "bonus" : 12 } ]
}
{
   "_id" : 6,
   "name" : "abc",
   "age" : 43,
   "type" : 1,
   "status" : "A",
   "favorites" : { "food" : "pizza", "artist" : "Picasso" },
   "finished" : [ 18, 12 ],
   "badges" : [ "black", "blue" ],
   "points" : [ { "points" : 78, "bonus" : 8 }, { "points" : 57, "bonus" : 7 } ]
}
```

##Array of Embedded Documents

###Match a Field in the Embedded Document Using the Array Index
만약에 array의 특정인덱스가 document를 가지고 있다는 사실을 알고 있다면 dot notation(.)을 통해 접근할수 있습니다.

다음은 points가 array형태의 데이터를 가지고 0번 인덱스에 document가 있고 그 document의 points필드에 접근해 조건을 설정하는 코드입니다.
```
db.users.find( { 'points.0.points': { $lte: 55 } } )
```

결과는 다음과 같은 형태입니다.

```
{
   "_id" : 4,
   "name" : "xi",
   "age" : 34,
   "type" : 2,
   "status" : "D",
   "favorites" : { "artist" : "Chagall", "food" : "chocolate" },
   "finished" : [ 5, 11 ],
   "badges" : [ "red", "black" ],
   "points" : [ { "points" : 53, "bonus" : 15 }, { "points" : 51, "bonus" : 15 } ]
}
```

###Match a Field Without Specifying Array Index
만약에 array에 document의 위치를 알지 못하는 경우 위치정보를 생략하고 dot notation(.)으로 바로 필드에 접근할수 있습니다.

다음은 points array에 최소 하나의 document가 points필드를 가지는 경우를 나타내는 코드입니다.

```
db.users.find( { 'points.points': { $lte: 55 } } )
```

결과의 형태입니다.

```
{
   "_id" : 3,
   "name" : "ahn",
   "age" : 22,
   "type" : 2,
   "status" : "A",
   "favorites" : { "artist" : "Cassatt", "food" : "cake" },
   "finished" : [ 6 ],
   "badges" : [ "blue", "red" ],
   "points" : [ { "points" : 81, "bonus" : 8 }, { "points" : 55, "bonus" : 20 } ]
}
{
   "_id" : 4,
   "name" : "xi",
   "age" : 34,
   "type" : 2,
   "status" : "D",
   "favorites" : { "artist" : "Chagall", "food" : "chocolate" },
   "finished" : [ 5, 11 ],
   "badges" : [ "red", "black" ],
   "points" : [ { "points" : 53, "bonus" : 15 }, { "points" : 51, "bonus" : 15 } ]
}
```


##Specify Multiple Criteria for Array of Documents

###Single Element Satisfies the Criteria
$elemMatch 연산자를 사용한다면 array내부의 document을 다중의 조건으로 검색하는 것이 가능합니다. 조건중 하나만 일치하는 document가 존재하면 해당 document를 추출할수있습니다.

다음은 points array의 document중 points가 70과 같거나 작고, bouns가 20인 경우를 찾는 코드입니다.

```
db.users.find( { points: { $elemMatch: { points: { $lte: 70 }, bonus: 20 } } } )
```
결과는 다음과 같습니다.

```
{
   "_id" : 3,
   "name" : "ahn",
   "age" : 22,
   "type" : 2,
   "status" : "A",
   "favorites" : { "artist" : "Cassatt", "food" : "cake" },
   "finished" : [ 6 ],
   "badges" : [ "blue", "red" ],
   "points" : [ { "points" : 81, "bonus" : 8 }, { "points" : 55, "bonus" : 20 } ]
}
```

###Combination of Elements Satisfies the Criteria
다음의 예는 points array의 document 엘리먼트가 여러 조건의 조합에 만족하도록하는 예제입니다. 예를들면 points가 70보다 작거나 같고 다른 엘리먼트인 bonus가 20인 경우이거나, 하나의 엘리먼트가 두개의 조건 모두에 만족하는 경우입니다.
```
db.users.find( { "points.points": { $lte: 70 }, "points.bonus": 20 } )
```
결과로 다음과 같습니다.

```

{
   "_id" : 2,
   "name" : "bob",
   "age" : 42,
   "type" : 1,
   "status" : "A",
   "favorites" : { "artist" : "Miro", "food" : "meringue" },
   "finished" : [ 11, 20 ],
   "badges" : [ "green" ],
   "points" : [ { "points" : 85, "bonus" : 20 }, { "points" : 64, "bonus" : 12 } ]
}
{
   "_id" : 3,
   "name" : "ahn",
   "age" : 22,
   "type" : 2,
   "status" : "A",
   "favorites" : { "artist" : "Cassatt", "food" : "cake" },
   "finished" : [ 6 ],
   "badges" : [ "blue", "red" ],
   "points" : [ { "points" : 81, "bonus" : 8 }, { "points" : 55, "bonus" : 20 } ]
}
```

##Additional Methods

- db.collection.findOne
- aggregation pipeline의 $match
```
{ $match: { <query> } }
```


##Read Isolation

replica set과 replica set shards의 데이터를 읽는 경우 클라이언트가 읽기작업을 하는데 고립의 단계를 설정할수가 잇습니다. [Read Concern](https://docs.mongodb.com/manual/reference/read-concern/)에 대한 더 자세한 내영을 확인해보기시 바랍니다.



