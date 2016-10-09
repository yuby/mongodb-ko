#Update Documents

##Update
다음은 몽고디비가 제공하는 document를 수정하는 메서드입니다.

|Method| Description|
| ------------- |-------------|
|db.collection.updateOne() | 가장 일치하는 하나의 document를 수정합니다. |
|db.collection.updateMany() | 모든 일치하는 document를 수정합니다. |
|db.collection.replaceOne() | 가장 일치하는 하나의 documet를 새로운 document로 대체합니다. |
|db.collection.update() | 수정하거나 대체하는 적업을 합니다. 기본적으로 하나의 document를 대상으로 동작을 하지만 multi 옵션 값을 사용한다면 다수의 document를 수정할수 있습니다. |

이 메서드들은 파라메터를 받아들입니다.

- 수정 대상의 조건을 지정합니다. 보통 읽기작업에 사용하는 표현과 동일합니다.
	- 필터쿼리는 <field>:<value>  형태로 특정 field의 value와 일치하는 대상을 찾습니다. { <field1>: <value1>, ... }
- 쿼리 연산자를 사용해 특정 조건을 부여할 수 있습니다. { <field1>: { <operator1>: <value1> }, ... }
- 특정 document를 수정하거나 해당 document 전체를 대체하는 경우에는 _id 필드와 option 정보를 제하고 동작합니다.


##Behavior

###Atomicity
몽고디비에서의 모든쓰기 작업의 경우에는 하나의 document단위로 원자성을 보장합니다.

###_id Field
한번 document가 저장이 되면 _id 필드의 정보는 수정할수 없습니다.

###Document Size
수정동작을 하는 경우 기존의 document의 사이즈를 넘어 서는 경우가 있습니다. 이경우에는 디스크 상에 주어진 자리를 벗어다 다른곳으로 이동해 해당 document를 저장합니다.

###Field Order
몽고디비는 document 필드의 순서를 다음의 경우를 제하고는 보장합니다.
- _id 필드는 항상 document의 최상단에 위치합니다.
- 필드이름을 재정의 하는 경우 변경된 이름으로 재배열 됩니다.

###Upsert Option
db.collection.update(), db.collection.updateOne(), db.collection.updateMany(),  db.collection.replaceOne() 메서드는 upsert : true 옵션을 포함하고 있어서 일치하는 수정 대상이 존재하지 않는경우 해당 document 정보를 생성합니다. 만약 일치하는 데이터가 존재한다면 해당 데이터 수정을 수행합니다.


##Example Collection
예제 수행을 위해 아래의 코드를 shell 상에서 실행하시고 users collection이 존재하는 경우에 해당 collection을 삭제(db.users.drop())하고 진행하시기 바랍니다.

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
       badges: [ "blue", "Picasso" ],
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
       badges: [ "Picasso", "black" ],
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

##Update Specific Fields in a Document
document의 특정 필드정보를 변경하기 위해서 $set 연산자 같은 것을 사용할수 있습니다. 수정연산자를 사용하는 document의 형태입니다.

```
{
   <update operator>: { <field1>: <value1>, ... },
   <update operator>: { <field2>: <value2>, ... },
   ...
}
```
$set 같은 몇몇 연산자들이 경우 해당 필드가 존재하지 않는경우에는 해당 필드를 생성합니다.


###db.collection.updateOne
db.collection.updateOne() 메서드를 사용해서 users collection의 favorites.artist가 Picasso인 document의 첫번째 document를 수정합니다.

- $set 연산자를 사용해서 favorites.food 필드의 값을 pie로 수정을 하고 type 필드의 값을 3으로 수정합니다.
- $currentDate 연산자를 사용해서 lastModified 필드의 값을 현재 날짜로 수정을 합니다. 필드가 존재하지 않는 경우 lastModified  필드를 추가합니다.

```
db.users.updateOne(
   { "favorites.artist": "Picasso" },
   {
     $set: { "favorites.food": "pie", type: 3 },
     $currentDate: { lastModified: true }
   }
)
```

###db.collection.updateMany
 db.collection.updateMany() 메서드를 사용해서 users collection의 favorites.artist가 Picasso인 document의 모든 document를 수정합니다.
- $set 연산자를 사용해서 favorites.artist  필드의 값을 Pisanello로 수정을 하고 type 필드의 값을 3으로 수정합니다.
- $currentDate 연산자를 사용해서 lastModified 필드의 값을 현재 날짜로 수정을 합니다. 필드가 존재하지 않는 경우 lastModified  필드를 추가합니다.

```
db.users.updateMany(
   { "favorites.artist": "Picasso" },
   {
     $set: { "favorites.artist": "Pisanello", type: 3 },
     $currentDate: { lastModified: true }
   }
)
```


###db.collection.update

db.collection.update() 메서드를 사용해서 users collection의 favorites.artist가 Picasso인 하나의 document를 수정합니다.
- $set 연산자를 사용해서 favorites.food   필드의 값을 pizza로 수정을 하고 type 필드의 값을 0으로 수정합니다.
- $currentDate 연산자를 사용해서 lastModified 필드의 값을 현재 날짜로 수정을 합니다. 필드가 존재하지 않는 경우 lastModified  필드를 추가합니다.

```
db.users.update(
   { "favorites.artist": "Pisanello" },
   {
     $set: { "favorites.food": "pizza", type: 0,  },
     $currentDate: { lastModified: true }
   }
)
```

db.collection.update() 메서드에 multi: true 옵션을 사용해서 다수의 document를 수정할 수 있습니다.

```
db.users.update(
   { "favorites.artist": "Pisanello" },
   {
     $set: { "favorites.food": "pizza", type: 0,  },
     $currentDate: { lastModified: true }
   },
   { multi: true }
)
```

##Replace the Document
document 전체를 대체하기 위해서는 _id 필드를 제하고 전체 document 정보를 db.collection.replaceOne() 나 db.collection.update() 메서드의 두번째 파라메터로 전달합니다. document를 대체하기 위해서 반드시 전달되는 document는 <field> : <value> 형태로 구성되야합니다.

대채되는 document의 경우에 기존의 document와 다른 필드를 가질수가 있습니다. 대체되는 document의 경우에는 _id 필드를 제하고 전달이되어야하고 만약에  _id 필드가 전달이 된다면 기존의 document와 동일한 값을 가지고 있어야합니다.


###db.collection.replaceOne
db.collection.replaceOne() 메서드를 사용해 name 이 "abc"인 document의 정보를 대채합니다.

```
db.users.replaceOne(
   { name: "abc" },
   { name: "amy", age: 34, type: 2, status: "P", favorites: { "artist": "Dali", food: "donuts" } }
)
```

###db.collection.update

db.collection.update() 메서드를 사용해 name 이 "xyz"인 document의 정보를 대채합니다.

```
db.users.update(
   { name: "xyz" },
   { name: "mee", age: 25, type: 1, status: "A", favorites: { "artist": "Matisse", food: "mango" } }
)
```


##Additional Methods
다음 메서드도 수정작업이 가능한 메서드들입니다.

- db.collection.findOneAndReplace().
- db.collection.findOneAndUpdate().
- db.collection.findAndModify().
- db.collection.save().
- db.collection.bulkWrite().

##Write Acknowledgement
write concern 으로 쓰기요청에 대한 응답(acknowledgement )의 수준을 지정할수 있습니다.











