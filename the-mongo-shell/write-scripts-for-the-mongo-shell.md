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
- 쓰기 작업을 할떄 mongo shell 은
