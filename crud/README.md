#MongoDB CRUD Operations
CRUD 는 생성(Create), 읽기(Read), 수정(Update), 삭제(Delete) 를 의미합니다.

##Create Operation
생성과 삽입은 새로운 document를 collection에 추가하는 작업입니다. 만약에 대상이 되는 collection이 존재하지 않는다면 collection을 생성하도 document를 추가하는 작업을 진행합니다.

몽고디비는 다음의 생성 메서드를 제공합니다.

- db.collection.insert()
- db.collection.insertOne() New in version 3.2
- db.collection.insertMany() New in version 3.2

몽고디비에서 삽입작업은 하나의 collection을 대상으로 이뤄집니다. 몽고디비에서의 모든 쓰기 작업은 하나의 document에 대해서는 원자성이 보장이 됩니다.
![crud-annotated-mongodb-insert](https://docs.mongodb.com/manual/_images/crud-annotated-mongodb-insert.png)

##Read Operations
collection상에서 document를 읽어 오는 작업을 하는 경우 예를 들어 다음과 같은 메서드를 사용할 수 있습니다.

- - db.collection.find()

물론 해당 메서드에 query criteria 나 projection 을 추가하거나 추가적인 메서드를 추가할수도 있습니다.

![crud-annotated-mongodb-find](https://docs.mongodb.com/manual/_images/crud-annotated-mongodb-find.png)

##Update Operations

현재에 collection 상에 존재하는 document를 수정하기 위해 다음의 메서드를 제공합니다.

- db.collection.update()
- db.collection.updateOne() New in version 3.2
- db.collection.updateMany() New in version 3.2
- db.collection.replaceOne() New in version 3.2

수정작업은 하나의 collection을 대상으로 이뤄집니다. 물론 읽기작업과 마찬가지로 수정의 조건을 추가하는 것도 가능합니다.

![crud-annotated-mongodb-update](https://docs.mongodb.com/manual/_images/crud-annotated-mongodb-update.png)

##Delete Operations

현재에 collection 상에 존재하는 document를 삭제하기 위해 다음의 메서드를 제공합니다.

- db.collection.remove()
- db.collection.deleteOne() New in version 3.2
- db.collection.deleteMany() New in version 3.2

삭제작업은 하나의 collection을 대상으로 이뤄지고 해당작업도 document 단위로 원자성이 보장이 됩니다. 삭제도 수정과 마찬가지로 삭제조건을 추가할수 있습니다.

![crud-annotated-mongodb-remove](https://docs.mongodb.com/manual/_images/crud-annotated-mongodb-remove.png)

##Bulk Write
몽고디비는 대량의 데이터를 쓰기위한 기능도 제공합니다. 자세한 내용은 [이곳](https://docs.mongodb.com/manual/core/bulk-write-operations/) 에서 확인 하시면 됩니다.
