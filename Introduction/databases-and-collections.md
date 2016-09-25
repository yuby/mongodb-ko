#Databases and Collections

몽고디비는 BSON documents를 저장합니다.그리고 이는 collections에 저장이되고 collections는 데이터베이스의 일부입니다.

![crud-annotated-collection](https://docs.mongodb.com/manual/_images/crud-annotated-collection.png)

##Databases
몽고디비는 documents로 구성된 collection의 모음을 데이터 베이스가 가지고 있습니다.

선택할 데이터 베이스를 사용하기 위해서는 mongo shell에서 **use <db> **를 사용합니다.

```
use myDB
```

###Create a Database
만약에 데이터베이스가 존재하지 않는 경우라면, 몽고디비의 데이터베이스를 우선 생성해야합니다. 다음은 myDB에서 존재하지 않는 데이터베이스(myNewDB)를 생성하고 동작시키는 방법을 mongo shell에서 사용하는 방법입니다

```
use myNewDB
db.myNewCollection1.insert( { x: 1 } )
```

insert() 메서드는 myNewDB 데이터베이스와 myNewCollection1 collection에 모두를 생성합니다.

##Collection
몽고디비는 documents를 collections에 저장을 하고, collection의 개념을 우리는 관계형 데이터베이스의 테이블과 같은 개념으로 생각하면 됩니다.

###Create a Collection
만약 collection이 존재하지 않는다면, 처음으로 데이터를 저장하기 위해서 정의하는 collection이 자동으로 생성이 됩니다.

```
db.myNewCollection2.insert( { x: 1 } )
db.myNewCollection3.createIndex( { y: 1 } )
```

insert()와 createIndex() 는 각각의 collection이 존재하지 않는다면 각각의 collection을 생성합니다.

###Explicit Creation
몽고디비는 db.createCollection() 메서드를 통해서 명시적으로  다양한 설정들과 함께 collection을 생성할수 있습니다.
예를 들면 최대 사이즈나, document의 규칙들을 정의 할수 있습니다. 만약 이러한 규칙들이 필요가 없다면 굳이 명시적으로 collection을 생성할 필요가 없습니다.
물론 한번 생성된 collection의 옵션은 collMod를 통해서 수정가능합니다.

###Document Validation
> new in version 3.2

기본적으로 collection은 동일한 형태의 스키마 형태를 가진 document가 필요로 하지 않습니다. 즉, 하나의 collection이라고 하더라도 다영한 형태의 document가 사용될수가 있습니다.
하지만 3.2버전부터는 수정이나 생성작업에 document의 규칙을 강제시킬수가 있습니다.

###Modifying Document Structure
collection내의 document의 구조를 변경하기 위해서, 예를들면 새로운 필드를 추가하거나 이미 존재하는 필드를 삭제하는 작업등 새로운 형태의 document를 사용하기 위해서는 그냥 변경된 형태의 구조로 해당 document를 업데이트 시키면됩니다.
