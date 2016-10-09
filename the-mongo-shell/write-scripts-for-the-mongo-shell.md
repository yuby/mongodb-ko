#Write Scripts for the mongo Shell

mongo shell에 javascipt로 작성된 스크립트를 작성할수 있습니다. 깊이있는 내용은 [이곳](https://docs.mongodb.com/manual/core/server-side-javascript/#running-js-scripts-in-mongo-on-mongod-host) 에서 확인하시고 이번 튜토리얼에서는 javascript로 작성된 mongo shell이 몽고디비에 접속하는 코드를 작성해 보겠습니다.

##Opening New Connections
mongo shell 또는 javascript 파일을 통해서 데이터베이스의 연결을 인스턴스화 시킬수가 있습니다.

```
new Mongo()
new Mongo(<host>)
new Mongo(<host:port>)
```

다음은 localhost의 기본 포트에 새로운 연결 인스턴스를 생성하는 방법입니다. 그리고 글로벌 변수 db에 getDB() 메서드를 통해 myDatabase를 정의하는 방법입니다.

```
conn = new Mongo();
db = conn.getDB("myDatabase");
```

db.auth() 메서드를 사용하면 몽고디비 인스턴스의 접근 권한을 컨트롤 할 수가 있습니다.

추가적으로 connect() 메서드를 사용해서 몽고디비 인스턴스에 연결할수가 있습니다. 

```
db = connect("localhost:27020/myDatabase");
```

##Differences Between Interactive and Scripted mongo
mongo shell 상에서 스크립트를 작성할때 고려할 사항입니다.

- 전역 변수인 db 의 값은 getDB()또는 connnet() 메서드를 사용합니다. db라는 변수 말고 다른 변수로 데이터 베이스의 참조를 정의할수도 있습니다.
- 쓰기 작업을 할떄 mongo shell 은 기본적으로 { w : 1} 의 write concern 을 가집니다. write concern에 대한 자세한 내용은 추후에 다루도록 하겠습니다. 그리고 만약 대량의 데이터를 다루는 작업을 하는 경우에는 Bulk() 메서드를 사용합니다.
- javascript 파일 내에서는 어떠한 help 메서드를 사용할수가 없습니다. 왜냐하면 유요한 javascript 표현이 아닌기 때문입니다. 다음은 shell에서 사용하는 help와 javascript 에서의 help를 사용하는 방법입니다.

|Shell Helpers |	JavaScript Equivalents|
|---------|-----------|
| show dbs, show databases| db.adminCommand('listDatabases') |
| use <db> | db = db.getSiblingDB('<db>') |
| show collections | db.getCollectionNames() |
| show users | db.getUsers() |
| show roles | db.getRoles({showBuiltinRoles: true}) |
| show log <logname> | db.adminCommand({ 'getLog' : '<logname>' }) |
| show logs | db.adminCommand({ 'getLog' : '*' }) |
| it | cursor = db.collection.find() if ( cursor.hasNext() ){   cursor.next();}|

- 인터렉티브 모드에서는 mongo가 cursor의 내용을 포함해 명령의 결과를 출력합니다. javascript파일 내에서도 동일하게 print() 메서드나 JSON 형태로 리턴하는 printjson() 메서드를 사용할수 있습니다.
> EXAMPLE
```
cursor = db.collection.find();
while ( cursor.hasNext() ) {
   printjson( cursor.next() );
}
```

##Scripting
프롬프트에서 javascript 파일을 사용하는 것도 가능합니다.

###--eval option
--eval 옵션에 javascript 코드 일부를 함꼐 사용 할 수가 있습니다.

```
mongo test --eval "printjson(db.getCollectionNames())"
```

위의 코드는 db.getCollectionNames()의 결과를 mongo shell이 mongod나 mongos 인스턴스가 동작하는 localhost:27017에 연결해서 얻어옵니다.

###Execute a JavaScript file
mongo shell 상에서 특정 .js 파일을 지정해서 바로 javascript 파일을 실행할수가 있습니다.

```
mongo localhost:27017/test myjsfile.js
```
myjsfile.js파일을 localhost:27017의 test 데이터 베이스에 mongod를 통해 연결해서 스크립트를 실행합니다.

또다른 방법으로 앞에서 살펴보았던 new Mongo() 생성자를 .js 파일 내에서 생성해서 사용하는 것도 가능합니다.

또는 .js 파일을 load() 메서드를 사용해서 mongo shell 상에서 실행하는 것도 가능합니다.

```
load("myjstest.js")
```

load() 메서드는 상대경로와 직접경로 모두를 받아 들입니다. 만약 현재 mongo shell 이 동작하는 위치가 /data/db인경우에는 scripts 폴더를 만들어서 myjstest.js 파일을 위치시켜 동작할 수가 있습니다.

```
load("scripts/myjstest.js")
load("/data/db/scripts/myjstest.js")
```

> 원하는 스크립트가 현재 디렉토리 또는 지정된 경로에 있지 않으면, mongo는 파일에 액세스 할 수 없을 것이다.

