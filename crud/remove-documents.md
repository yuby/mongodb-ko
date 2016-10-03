#Delete Documents

##Delete Methods

몽고디비는 collection의 document를 제거하기 위해서 다음의 메서드들을 제공합니다.

| Me        | Are           |
| ------------- |-------------|
|db.collection.remove()	| 조건에 일치하는 하나의 document나 전체 documents 를 삭제 합니다. |
|db.collection.deleteOne()	| 조건에 가장 일치하는 하나의 document를 삭제합니다.|
|db.collection.deleteMany() |	조건에 일치하는 모든 document를 삭제합니다.|

메서드에 특정 조건을 명시해서 해당 document를 삭제하는 것이 가능합니다. 방식은 읽기작업에 사용하는 조건 document작성법과 동일합니다.

- <field>:<value> 형태로 document를 작성하면 해당 필드의 값에 해당하는 document를 가져옵니다.
```
{ <field1>: <value1>, ... }
```
- 쿼리연산자를 사용해 조건을 작성할수 있습니다.
```
{ <field1>: { <operator1>: <value1> }, ... }
```

##Delete Behavior

###Indexes
삭제작업을 통해서는 인덱스 정보를 삭제하지 않습니다. 심지어 collection의 모든 정보가 삭제되도 삭제되지가 않습니다.

###Atomicity
몽고디비에서의 모든쓰기 작업의 경우에는 하나의 document단위로 원자성을 보장합니다.

###Example Collection
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

##Delete All Documents
collection의 모든 document를 조건 document를 db.collection.deleteMany() 나 db.collection.remove() 메서드에 명시해서 삭제할수 있습니다.

###db.collection.deleteMany()
다음의 코드는 db.collection.deleteMany() 메서드를 사용해 users collection을 삭제합니다.

```
db.users.deleteMany({})
```
해당 메서드의 결과로 다음을 리턴합니다.

```
{ "acknowledged" : true, "deletedCount" : 7 }
```

###db.collection.remove()
db.collection.remove()메서드를 사용해 users collection을 모두 삭제합니다.

```
db.users.remove({})
```

collection상의 모든 정보를 삭제하는 것은 db.collection.drop() 메서드를 사용하는것이 성능상에 이점이 있습니다. 이경우 index정보도 모두 삭제합니다.


##Delete All Documents that Match a Condition
db.collection.deleteMany() 나 db.collection.remove() 메서드를 사용해 특정 조건의 데이터를 삭제합니다.

###db.collection.deleteMany()
db.collection.deleteMany()메서드에 조건을 추가해 일치하는 document전체를 삭제합니다.

```
db.users.deleteMany({ status : "A" })
```

작업의 결과로 다음의 결과를 리턴합니다.

```
{ "acknowledged" : true, "deletedCount" : 3 }
```

###db.collection.remove()
다른 방법으로 db.collection.remove() 메서드를 사용합니다.

```
db.users.remove( { status : "P" } )
```

collction상의 삭제가 대량으로 이뤄지는 경우에는 필요한 document를 따로 저장을 해두고 collection을 전체 삭제한 이후에 따로 저장한 document를 가져오는 것이 성능상 이점이 있습니다.


##Remove Only One Document that Matches a Condition
db.collection.deleteOne() 나 db.collection.remove() 메서드를 사용해 하나의 document를 삭제합니다.

###db.collection.deleteOne()
db.collection.deleteOne() 메서드는 조건에 일치하는 document의 첫번째 document만 삭제합니다.

```
db.users.deleteOne( { status: "D" } )
```

###db.collection.remove()

db.collection.remove() 메서드의 <justOne> 파라메터를 전달하면 조건에 일치하는 document중 하나만을 삭제합니다.

```
db.users.remove( { status: "D" }, 1)
```

##Additional Methods

- db.collection.findOneAndDelete()
- db.collection.findOneAndModify()
- db.collection.findOneAndModify()
- db.collection.bulkWrite()

##Write Acknowledgement
write concern 으로 쓰기요청에 대한 응답(acknowledgement )의 수준을 지정할수 있습니다.

