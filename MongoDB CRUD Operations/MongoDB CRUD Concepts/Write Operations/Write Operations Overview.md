[목록](https://github.com/yuby/mongodb-ko)


#Write Operations Overview

쓰기 작업은 몽고디비 인스턴스에서 발생하는 생성하고 수정하는 작업을 말합니다. 몽고디비에서 쓰기 작업은 하나의 collection을 대상으로 이뤄집니다. 몽고디비에서의 모든 쓰기 작업은 하나의 document레벨에서 원자성을 가집니다.

몽고디비에서는 insert, update, remove 이 세가지 클래스를 제공합니다. Insert 는 collection에 새로운 데이터를 추가 할때 사용합니다. Update는 존재하는 데이터를 수정할때 사용합니다. Remove의 경우에는 collection으로 부터 데이터를 제거할 때 사용합니다. No insert, update, or remove can affect more than one document atomically.

update와 remove 작업에 대해서는 수정하거나 제거할 대상이 될 documents를 구분지어주는 특정 기준이나 조건을 지정할수 있습니다. 이 동작들의 경우 read 작업에 사용하는 문법과 동일합니다.

몽고디비는 어플리케이션이 쓰기 작업에 필요한 승인의 레벨을 정의할수 있게 해줍니다. 더 자세한 정보는 Write Concern 을 확인하세요.

##Insert

db.collection.insert() 메서드 콜렉션에 새로운 documents를 추가 할때 사용하는 매서드 입니다.

다음 다이어그램은 몽고디비가 insert 작업의 컴퍼넌트를 표시한 것입니다.

![The components of a MongoDB insert operations.](http://docs.mongodb.org/manual/_images/crud-annotated-mongodb-insert.png)

이 다이어그램을 동일한 SQL 쿼릴로 변환 시킨 것입니다.

![The components of a SQL INSERT statement.](http://docs.mongodb.org/manual/_images/crud-annotated-sql-insert.png)


>EXAMPLE
다음 insert 작업은 users  collection에 새로운 document를 생성하는 예제입니다. 새로운 document는 name, age, and status, 그리고 _id 필드를 가지고 있습니다. 몽고디비는 새롭게 생성되는 document에 _id 필드가 존재하지 않는 경우 _id 필드를  항상 추가를 합니다.
'''
db.users.insert(
   {
      name: "sue",
      age: 26,
      status: "A"
   }
)
'''
For more information and examples, see db.collection.insert().

###Insert Behavior

만약에 _id 필드가 없이 새로운 document를 추가한다면 클라이언트 라이브러리 또는 mongod 인스턴스가 _id필드를 추가하고 유니크한 값인 ObjectId 필드를 채우게 됩니다.
만약에 _id필드를 지정한다면 그 값은 반드시 collection내에서 유니크한 값이어야 합니다. write concern이 지정되어 동작이 되고 있는 경우 중복된 _id 값이 있는 경우 mongod는 중복된 키로 예외처리를 시킵니다.


###Other Methods to Add Documents

upsert 옵션을 통해 collection에 새로운 documents를 추가할수가 있습니다. 만약에 upsert 옵션이 true로 지정이 된경우라면 이 메서드는 존재하는 documents에 대해서는 수정을 하고 일치하는 documents가 존재하지 않는 경우에는 새로운 documents를 추가합니다. upsert에 대한 더많은 정보는 이곳에서 확인 하세요.


##Update

몽고디비에서 db.collection.update() 메서드는 collection에 존재하는 documents를 수정하는 역활을 합니다. db.collection.update() 메서드는 쿼리에 조건들을 통해서 옵션을 기반으로 하여 수정하는 작업을 합니다 . 예를 들면 mutil 옵션의 경우에는 다중 documents를 상대로 업데이트를 진행합니다.

몽고디비에서의 수정 작업은 모든 하나의 document를 상으로 원자성을 유지하여 실행이 됩니다. 예를들면 $inc나 $mul 의 경우에는 해당필드의 값을 동시에 그리고 매우 빈번하게 변경을 시킵니다.

아래의 다이어그램은 몽고디비의 update의 컴퍼넌트를 표시한 것입니다.

![The components of a MongoDB update operation.](http://docs.mongodb.org/manual/_images/crud-annotated-mongodb-update.png)

아래는 SQL 로 변경한 형태입니다.

![The components of a SQL UPDATE statement.](http://docs.mongodb.org/manual/_images/crud-annotated-sql-update.png)


>EXAMPLE
```
db.users.update(
   { age: { $gt: 18 } },
   { $set: { status: "A" } },
   { multi: true }
)
```
이 수정 작업은 users collection을 대상으로 age가 18이상인 경우에 status를 A 로 수정하는 동작입니다.
더 많은 정보는 db.collection.update() 과 update() Examples에서 확인 하세요.


###Default Update Behavior
기본적으로 db.collection.update() 메서드의 경우에는 하나의 document를 대상으로 수정작업을 합니다. 하지만 multi 옵션을 사용한다면 쿼리와 일치하는 collection의 모든 documents를 수정하는 작업이 가능합니다.
By default, the db.collection.update() method updates a single document. However, with the multi option, update() can update all documents in a collection that match a query.

db.collection.update()메서드는 이미 존재하는 document의 특정 필드 값을 수정하는 것이 가능하고 또는 해당 document를 대체해버리는것도 가능합니다. 더많은 정보는 db.collection.update()에서 확인 하세요.

document의 수정작업이 진행이 되면서 기존에 document가 가진 사이즈 보다 증가하는 경우가 발생하는 데 이경우에는 해당 document의 위치를 디스크상에서 새롭게 잡게 됩니다.

몽고디비는 다음의 경우를 제외하고는 document의 스기 작업에 필드의 순서를 유지합니다.

- _id필드는 항사 모든 필드의 첫번째에 위치합니다.
-  필드 이름을 재설정하는 경우에는 document의 필드의 순서를 재배열합니다.

2.6 버전이후 : 2.6버전 이전에는 필드의 순서를 유지하려는게 소극적이 었지만 2.6버전 이후부터는 document의 필드 순서를 유지하는 것이 활성화 되었습니다.
Changed in version 2.6: Starting in version 2.6, MongoDB actively attempts to preserve the field order in a document. Before version 2.6, MongoDB did not actively preserve the order of the fields in a document.

###Update Behavior with the upsert Option
만약에 update() 메서드가 upsert : true 를 가지고 있고 일치하는 document가 해당 collection 상에 존재하지 않는 다면 update 작업을 통해 새로운 document를 생성하게 됩니다.
If the update() method includes upsert: true and no documents match the query portion of the update operation, then the update operation creates a new document. If there are matching documents, then the update operation with the upsert: true modifies the matching document or documents.

upsert: true 설정을 통해서 어플리케이션은 수정을 위한 document 가 존재하지 않는 경우에는 생성작업이 이뤄져야 함을 인지하게 됩니다. update() 에서 upsert에 대한 더 많은 정보를 확인하세요.

2.6버전 이후 : Bulk() 매서드가 새롭게 추가 되어 한번의 호출로 여러번의 update(), upsert 포함 작업을 수행할수가 있습니다.

만약에 upsert 옵션을 사용해서 새로운 document를 생성한다면 중복된 동작을 막기 위해 유니크한 인덱스를 사용하는 것을 고려해야 합니다.


##Remove

db.collection.remove() 메서드는 collection으로 부터 documents 를 삭제할때 사용합니다.db.collection.remove() 메서드도 쿼리에 조건을 추가해 어떤 documents를 삭제 할것인지 지정할수가 있습니다.

다음 다이어그램은 몽고디비의 삭제 작업의 컴퍼넌트를 표시한것입니다.

![The components of a MongoDB remove operation.](http://docs.mongodb.org/manual/_images/crud-annotated-mongodb-remove.png)

이를 SQL로 변경한 모습입니다.


![The components of a SQL DELETE statement.](http://docs.mongodb.org/manual/_images/crud-annotated-sql-delete.png)

>EXAMPLE
```
db.users.remove(
   { status: "D" }
)
```
이 삭제 작업은 user collecion으로 부터 statusr가 D인 정보를 모두 삭제합니다. 더 많은 정보는 db.collection.remove() 과  Remove Documents에서 확인 하세요.

###Remove Behavior
db.collection.remove()메서드는 쿼리와 일치하는 모든 documents를 삭제합니다. 하지만 이 메서드는 플래그 값을 통해 하나의 documet 만을 삭제하도록 제한을 줄수도 있습니다.


##Isolation of Write Operations

하나의 document를 대상으로 하는 수정 작업은 항상 원자성을 가집니다. 심지어 document 내부의 다중 documets의 정보를 수정하는 작업에도 적용이 됩니다.
The modification of a single document is always atomic, even if the write operation modifies multiple embedded documents within that document. No other operations are atomic.

만약 여러 documents를 수정하는 작업을 한다면 각 작업에 대해서는 원자성이 보장되지 않습니다. 그래서 다른 작업이 현재작업을 방해할수도 있습니다. 하지만 우리는 isolation operator를 통해서 각각의 작업이 고립되게 하여 다른 작업으로 부터 현재작업이 영향을 받지 않게 할수 있습니다.
If a write operation modifies multiple documents, the operation as a whole is not atomic, and other operations may interleave. You can, however, attempt to isolate a write operation that affects multiple documents using the isolation operator.

더 많은 정보는  Atomicity 과 Transactions에서 확인 할수 있습니다.

##Additional Methods

db.collection.save()메서드도 수정과 일치하는 _id값을 찾지 못한다면 생성작업을 합니다. db.collection.save() 에대한 더많은 정보를 확인 하세요.

몽고디비는 또 덩어리씩 쓰기 작업을 진행하수 있는 Bulk()메서드를 제공하고 있습니다. Bulk()에 대한 정보를 확인하세요.


[목록](https://github.com/yuby/mongodb-ko)