#The mongo Shell

##Introduction
mongo shell 은 인터렉티브한 몽고디비 javascript 인터페이스 입니다. 우리는 mongo shell에서 query와 update는 물론 관리상의 필요한 작업들을 진행할 수 있습니다.
mongo shell은 몽고디비의 배포판의 구성요소입니다. 즉, 일단 몽고디비를 깔고 시작하기만 하면 mongo shell 이 몽고디비 인스턴스와 연동이 됩니다.

대부분의 몽고디비 매뉴얼의 예제의 경우 mongo shell을 사용하지만, 다양한 드라이버들 또한 비슷한 인터페이스를 제공하고 있습니다.

##Start the mongo shell

> IMPORTANT:
> mongo shell 시작 이전에 몽고디비가 동작하고 있는 중인지 확인 하세요.

mongo shell을 시작하고 몽고디비 인스턴스(localhost의 기본 포트)에 연결하는 방법입니다.

1. terminal을 을 열고 몽고디비가 설치된 폴더로 이동을 합니다.
```
cd <mongodb installation dir>
```
2. ./bin/mongo  을 입력하면 mongo가 실행됩니다.
```
./bin/mongo
```

만약에 PATH 에 몽고디비의 저장위치를 추가해 두었다면 간단히 mongo를 입력하기만 해도 동작을 시작합니다.

###Option
mongo를 실행할때 아무런 옵션사항을 입력하지 않은경우에는 localhost:27017 의 몽고디비 인스턴스와 연결을 합니다. 다른 host와 port에 연동하고 싶은 경우에는 [예제](https://docs.mongodb.com/manual/reference/program/mongo/#mongo-usage-examples) 와 [문서](https://docs.mongodb.com/manual/reference/program/mongo/)에서 자세한 내용을 확인 하시기 바랍니다.


###.mongorc.js File
mongo가 실행이 되기 시작하면 HOME 디렉터리에 .mongorc.js 라는 이름의 파일을 확인 하시기 바랍니다. 만약에 해당 파일이 존재한다면, mongo는 처음 프롬프트에 무언가를 내보이기 전에 해당 파일의 내용을 우선 해석을 합니다. 만약에 javascript 파일이나 표현식을 -eval 을 이용하거나 특정 .js 파일을 사용하는 경우 mongo는 javascript의 실행이 모두 끝이 나면 .mongorc.js 파일을 읽게 됩니다.
만약에 .mongorc.js 파일이 실행되지 않게 하기 위해서는 --norc 옵션을 사용하면 됩니다.

##Working whth the mongo Shell
우리가 사용하는 데이터베이스를 보이게 하기 하려면 db 를 입력하면 됩니다.

```
db
```

입력을 하면 test라는 기본 데이터 베이스를 리턴합니다. 만약 다른 데이터 베이스를 사용하고 싶다면 use <db> 를 사용합니다.

```
use <database>
```

상용가능한 데이터베이스의 리스트를 구하기 위해서는 **show dbs** 를 입렵합니다. db.getSiblingDB() 메서드는 현재 데이터 베이스 컨택스트와 연관된 다른 데이터 베이스로 접근하게 합니다.

현재 존재하지 않는 데이터베이스로 이동할수 도 있습니다. 만약 존재하지 않는 데이터베이스에 데이터를 저장하면 자동으로 해당 데이터베이스를 생성하고 collection을 만들어 줍니다. 

```
use myNewDatabase
db.myCollection.insert( { x: 1 } );
```
존재하지 않는 새로운 myNewDatabase 데이터베이스와  myCollection 콜렉션을 생성해서 데이터를 생성해줍니다. db.myCollection.insert() 는 shell 상에서 사용가능한 [메서드중](https://docs.mongodb.com/manual/reference/method/) 하나입니다.

- db는 현재의 데이터 베이스를 참조합니다.
- myCollection은 collection의 이름입니다.

만약 mongo shell이 collection이 이름을 못받아 들이는 경우, 예를들면 이름에 공백이 존재하거나, 하이픈, 또는 별표와 숫자가 있는 경우에는 조금 다른 방법으로 collection이름을 지정해야합니다.

```
db["3test"].find()
db.getCollection("3test").find()
```

###Format Printed Results
 db.collection.find() 메서드는 결과의 cursor를 리턴하는 메서드 입니다. 하지만 mongo shell에서 리턴하는 결과에 var 키워드를 사용해서 결과를 변수화 시키지 않는다면 cursor는 자동으로 20개의 document만을 창에 보이게 합니다. **it** 를 입력할떄 마다 추가적으로 20개씩을 추가로 보여줍니다.
리턴되는 결과값을 잘 정리된 형태로 확인하고 싶은 경우에는 .pretty() 를 추가하면 됩니다.

```
db.myCollection.find().pretty()
```

추가적으로 출력에 사용할수 있는 메서드들은 다음과 같습니다.

- print() 포맷팅이 되지 않은 형태로 결과를 보여줍니다.
- print(tojson(<obj>)) JSON 형태로 결과를 리턴합니다. printjson() 과 동일합니다.
- printjson() 은 JSON 형태로 결과를 리턴합니다. print(tojson(<obj>))와 동일한 결과입니다. 

###Multi-line Operations in the mongo Shell
만역에 mongo shell 상에서 (' **(** ') , (' **{** '), (' **[** ') 으로 괄호가 열리고 다음 라인 부터는 ("...") 로 시작해서 닫는 괄호가 오기전까지 계속 연속됨을 표시합니다.
```
> if ( x > 0 ) {
... count++;
... print (x);
... }
```
만약 중간에 ("...") 상태를 벗어나고 싶으면 두번 연속 빈칸, 즉 엔터 두번을 연속으로 입력하면 ("...") 에서 벗어나게 됩니다.

```
> if (x > 0
...
...
>
```

###Tab Completion and Other Keyboard Shortcuts
mongo shell은 키보드 단축키를 지원합니다. 
- up/down 키의 경우 커맨드라인의 히스토리를 검색합니다. 
- 입력중 tab 키는 자동 완성을 해주는 역활을 합니다.
```
db.myCollection.c<Tab>
```
더 많은 단축키 리스트는 [여기](https://docs.mongodb.com/manual/reference/program/mongo/#mongo-keyboard-shortcuts) 에서 확인 가능합니다.

###Exit the Shell
mongo shell을 끝내고 싶을때 quit(), ctrl+ c 를 입력하면 됩니다.








