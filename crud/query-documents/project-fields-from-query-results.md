#Project Fields to Return from Query

기본적으로 몽고디비는 일치하는 document의 모든 필드의 정보를 리턴합니다. 제한된 정보를 어플리케이션으로 전달하기 위해 projection를 포함시킬수가 있습니다.

##Projection Document
projection은 전달되는 필드에 제한을 주는 것입니다. projection 을 통해 특정 필드를 제하거나 추가할수 있습니다. 다음과 같은 형태를 가집니다.

```
{ field1: <value>, field2: <value> ... }
```

<value> 의 값은 다음과 같습니다.

- 1 이나 true 는 포함된 필드를 의미합니다.
- 0 이나 false는 제거된 필드를 의미합니다.
- [Projection Operation](https://docs.mongodb.com/manual/reference/operator/projection/)

>NOTE
>_id 필드의 경우에는 별도로 지정하지 않아도 _id : 0 으로 지정하지 않는 이상 항상 전달되는 정보입니다.

projection 의 경우 _id를 제외하고는 0,1 을 동시에 사용할수가 없습니다. 1을 선택한 경우에는 1만, 0을 사용하는 경우에는 0만 사용해야합니다.

##Example Collection
예제를 위해 아래의 코드를 mongo shell상에서 실행하시기 바랍니다. 
>NOTE
>users collection이 존재한다면 해당 collection을 삭제하고 하시기바랍니다.

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

##Return All Fields in Matching Documents
db.collection.find() 메서드에 아무런 projection을 추가하지 않는다면 모든 필드의 정보가 리턴됩니다.
```
db.users.find( { status: "A" } )
```
다음은 status가 'A'인 모든 정보를 리턴합니다.

```
{
   "_id" : 2,
   "name" : "bob",
   "age" : 42,
   "type" : 1,
   "status" : "A",
   "favorites" : { "artist" : "Miro", "food" : "meringue" },
   "finished" : [ 11, 25 ],
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

##Return the Specified Fields and the _id Field Only
projection에 몇몇 필드의 정보를 추가하겠습니다.

```
db.users.find( 
	{ status: "A" }, 
	{ name: 1, status: 1 } 
)

```

결과는 다음과 같습니다.

```
{ "_id" : 2, "name" : "bob", "status" : "A" }
{ "_id" : 3, "name" : "ahn", "status" : "A" }
{ "_id" : 6, "name" : "abc", "status" : "A" }
```

##Return Specified Fields Only
결과에서 _id  필드를 제거하겠습니다.

```
db.users.find( 
	{ status: "A" }, 
	{ name: 1, status: 1, _id: 0 } 
)
```

결과는 다음과 같습니다.
```
{ "name" : "bob", "status" : "A" }
{ "name" : "ahn", "status" : "A" }
{ "name" : "abc", "status" : "A" }
```

##Return All But the Excluded Field
특정필드를 제거하기 위해 필요한 필드를 모두 적는것보다는 제하고 싶은 필드를 설장할 수 있습니다.

```
db.users.find( 
	{ status: "A" }, 
	{ favorites: 0, points: 0 } 
)
```
결과는 다음과 같습니다.

```
{
   "_id" : 2,
   "name" : "bob",
   "age" : 42,
   "type" : 1,
   "status" : "A",
   "finished" : [ 11, 25 ],
   "badges" : [ "green" ]
}
{
   "_id" : 3,
   "name" : "ahn",
   "age" : 22,
   "type" : 2,
   "status" : "A",
   "finished" : [ 6 ],
   "badges" : [ "blue", "red" ]
}
{
   "_id" : 6,
   "name" : "abc",
   "age" : 43,
   "type" : 1,
   "status" : "A",
   "finished" : [ 18, 12 ],
   "badges" : [ "black", "blue" ]
}
```

_id 필드를 제하고는 필드를 빼고 더하는 것을 동시에 지정할수가 없습니다.

##Return Specific Fields in Embedded Documents¶
(.) dot notation을 사용해 전달되는 document의 내부document의 필드도 지정하는 것이 가능합니다.

다음은 name, status, favorites 의 food 필드를 추가하는 코드입니다.

```
db.users.find(
   { status: "A" },
   { name: 1, status: 1, "favorites.food": 1 }
)
```
결과는 다음과 같습니다.

```
{ "_id" : 2, "name" : "bob", "status" : "A", "favorites" : { "food" : "meringue" } }
{ "_id" : 3, "name" : "ahn", "status" : "A", "favorites" : { "food" : "cake" } }
{ "_id" : 6, "name" : "abc", "status" : "A", "favorites" : { "food" : "pizza" } }
```

##Suppress Specific Fields in Embedded Documents
(.) dot notation을 사용해 전달되는 document의 내부document의 필드를 제하는 것은 1대신 0을 지정하면 됩니다. 다음은 favorites 의 food 필드를 제거하는 코드입니다.

```
db.users.find(
   { status: "A" },
   { "favorites.food": 0 }
)
```
결과는 다음과 같습니다.

```
{
   "_id" : 2,
   "name" : "bob",
   "age" : 42,
   "type" : 1,
   "status" : "A",
   "favorites" : { "artist" : "Miro" },
   "finished" : [ 11, 25 ],
   "badges" : [ "green" ],
   "points" : [ { "points" : 85, "bonus" : 20 }, { "points" : 64, "bonus" : 12 } ]
}
{
   "_id" : 3,
   "name" : "ahn",
   "age" : 22,
   "type" : 2,
   "status" : "A",
   "favorites" : { "artist" : "Cassatt" },
   "finished" : [ 6 ],
   "badges" : [ "blue", "red" ],
   "points" : [ { "points" : 81, "bonus" : 8 }, { "points" : 55, "bonus" : 20 } ]
}
{
   "_id" : 6,
   "name" : "abc",
   "age" : 43,
   "type" : 1,
   "status" : "A",
   "favorites" : { "artist" : "Picasso" },
   "finished" : [ 18, 12 ],
   "badges" : [ "black", "blue" ],
   "points" : [ { "points" : 78, "bonus" : 8 }, { "points" : 57, "bonus" : 7 } ]
}
```

##Projection on Embedded Documents in an Array
array의 내부 document의 필드 또한 (.)dot notation을 사용해서 지정할수가 있습니다.

points array의 bonus 필드만 전달되도록 하는 코드입니다.

```
db.users.find( 
	{ status: "A" }, 
	{ name: 1, status: 1, "points.bonus": 1 } 
)
```
결과는 다음과 같습니다.

```
{ "_id" : 2, "name" : "bob", "status" : "A", "points" : [ { "bonus" : 20 }, { "bonus" : 12 } ] }
{ "_id" : 3, "name" : "ahn", "status" : "A", "points" : [ { "bonus" : 8 }, { "bonus" : 20 } ] }
{ "_id" : 6, "name" : "abc", "status" : "A", "points" : [ { "bonus" : 8 }, { "bonus" : 7 } ] }
```


##Project Specific Array Elements in the Returned Array

몽고디비는 array의 projection 연산자인  $elemMatch, $slice, $ 를 제공합니다.

다음의 예제는 $slice를 사용해서 points array의 마지막 엘리먼트를 제거한 결과를 리턴받는 코드입니다.

```
db.users.find( 
	{ status: "A" }, 
	{ name: 1, status: 1, points: { $slice: -1 } } 
)
```
결과는 다음과 같습니다.

```
{ "_id" : 2, "name" : "bob", "status" : "A", "points" : [ { "points" : 64, "bonus" : 12 } ] }
{ "_id" : 3, "name" : "ahn", "status" : "A", "points" : [ { "points" : 55, "bonus" : 20 } ] }
{ "_id" : 6, "name" : "abc", "status" : "A", "points" : [ { "points" : 57, "bonus" : 7 } ] }
```
$elemMatch, $slice, $  는 유일하게 전달되는 array의 엘리먼트를 특정지울수 있는 방법입니다. 예를들면 { "ratings.0": 1 } 같이 array의 인덱스 정보를 가지고 필드를 조절하는 것은 불가능합니다.






