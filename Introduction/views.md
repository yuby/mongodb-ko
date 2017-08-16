#Views

>New in version 3.4

3.4버전 이후부터 몽고디비는 존재하는 collection 또는 다른 view를 기반으로 읽기전용의 view 기능을 제공합니다.

##Create View
view를 생성하고 정의하는 방법은 다음과 같습니다.

- 기존에 제공해주던 명령어를 기반으로 생성하고 정의하는 방법입니다.

```
db.runCommand( { create: <view>, viewOn: <source>, pipeline: <pipeline> } )
```
또는

```
db.runCommand( { create: <view>, viewOn: <source>, pipeline: <pipeline>, collation: <collation> } )
```

- 3.4버전부터 새롭게 제공하는 기능입니다.

```
db.createView(<view>, <source>, <pipeline>, <collation> )
```

##Behavior
Views는 다음과 같은 동작을 합니다.

###Read Only
Views는 오직 읽기 작업만 가능합니다. 쓰기작업을 시도하는 경우 에러를 발생시킵니다.

###Index Use and Sort Operations

