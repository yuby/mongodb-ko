#Remove Documents

[db.collection.remove()](http://docs.mongodb.org/manual/reference/method/db.collection.remove/#db.collection.remove)메서드는 collection으로 부터 documents를 삭제합니다. 우리는 collection의 모든 documents를 제거할수 있고 조건에 일치하는 documents만을 골라 제거하는것이 가능합니다. 또는 document하나만 제거 하는 것도 가능합니다.

이 튜토리얼은 [db.collection.remove()](http://docs.mongodb.org/manual/reference/method/db.collection.remove/#db.collection.remove)을 사영해서 [몽고쉘](http://docs.mongodb.org/manual/reference/program/mongo/#bin.mongo)에서 삭제작업을 하는 예제를 제공합니다.

##Remove All Documents

collection으로 부터 모든 documents를 제거하기 위해 빈 쿼리 document {}를 [remove()](http://docs.mongodb.org/manual/reference/method/db.collection.remove/#db.collection.remove) 메서드의 인자로 전달합니다. [remove()](http://docs.mongodb.org/manual/reference/method/db.collection.remove/#db.collection.remove) 메서드는 indexes를 제거하지 않습니다.

다음의 예제는 inventory collection으로 부터 전체 documents를 제거합니다.
```
db.inventory.remove({})
```

모든 documents를 collection으로 부터 삭제하기 위해서는 [drop()](http://docs.mongodb.org/manual/reference/method/db.collection.drop/#db.collection.drop) 메서드가 더 효율적일수 있습니다. drop()메서드는 인덱스를 포합해 collection의 모든 내용을 제거하고 그리고 나서 collection을 다시 생성하고 index를 다시 빌드합니다.


##Remove Documents that Match a Condition

조건에 일치하는 documents를 제거하기 위해서 [remove()](http://docs.mongodb.org/manual/reference/method/db.collection.remove/#db.collection.remove) 메서드에 파라메터를 전달합니다.

다음 예제는 inventory collection으로 부터 type 필드의 값이 food인 모든 documents를 삭제합니다.

```
db.inventory.remove( { type : "food" } )
```

대량의 데이터를 삭제하는 경우에는 차라리 해당 collection으로 부터 필요한 documents를 새롭게 collection을 생성하여 옮겨두고 이전의 collection은 [drop()](http://docs.mongodb.org/manual/reference/method/db.collection.drop/#db.collection.drop)을 사용해서 collection 전체를 제거하는 것이 더 효율적입니다.

##Remove a Single Document that Matches a Condition

하나의 document를 삭제하기 위해서는 [remove()](http://docs.mongodb.org/manual/reference/method/db.collection.remove/#db.collection.remove) 메서드에 justOne 파라메터를 true나 1로 설정을 하면 됩니다.

다음 예제는 inventory collection으로 부터 type이 food 인 document를 하나만 삭제합니다.
```
db.inventory.remove( { type : "food" }, 1 )
```
[findAndModify()](http://docs.mongodb.org/manual/reference/method/db.collection.findAndModify/#findandmodify-wrapper-sorted-remove)메서드를 사용해서 documents에 순서를 적용해서 삭제하는 것도 가능합니다.
