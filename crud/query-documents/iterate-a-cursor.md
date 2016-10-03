#Iterate a Cursor in the mongo Shell
db.collection.find() 메서드는 결과로 cursor를 리턴합니다. cursor의 document에 접근하기 위해서는 반복문을 통해서 접근이 가능합니다. 하지만 mongo shell에서는 cursor가 var키워드로 선언이 되지 않으면 결과의 처음 20개를 모두 화면에 보이게 합니다.

다음은 수동으로 cursor의 document에 접근해서 데이터를 확인하는 방법입니다.

##Manually Iterate the Cursor

var 키워드를 사용해서 find() 메서드를 통해서 리턴된 결과를 확인하면 자동으로 cursor를 반복해 내부의 document를 보여주지 않습니다.

다음은 일치된 결과의 20개를 화면에 보이게 하는 코드입니다.

```
var myCursor = db.users.find( { type: 2 } );

myCursor
```

next() 메서드를 사용해서 document에 접근하는 방법도 있습니다.

```
var myCursor = db.users.find( { type: 2 } );

while (myCursor.hasNext()) {
   print(tojson(myCursor.next()));
}
```
printjson() 메서드는 print(tojson()) 으로 바꿔사용할수 있습니다.

```
var myCursor = db.users.find( { type: 2 } );

while (myCursor.hasNext()) {
   printjson(myCursor.next());
}
```

cursor의 forEach() 메서드를 사용해서 cursor를 반복시켜 document에 접근할수 있습니다.

```
var myCursor =  db.users.find( { type: 2 } );

myCursor.forEach(printjson);
```

##Iterator Index
mongo shell에서 cursor에 toArray() 메서드를 사용해서 결과를 array화 시켜서 index기반으로 결과에 접근하는것도 가능합니다.

```
var myCursor = db.inventory.find( { type: 2 } );
var documentArray = myCursor.toArray();
var myDocument = documentArray[3];
```

toArray() 메서드는 리턴된 결과를 RAM에 저장 하기때문에 부담이 있습니다.

추가적으로 몇몇 드라이버의 경우에는 cursor자체에 index로 접근하도록 지원하는 경우도 있습니다.

```
var myCursor = db.users.find( { type: 2 } );
var myDocument = myCursor[1];
```

myCursor[1] 코드는 다음과 같습니다.

```
myCursor.toArray() [1];
```

##Cursor Behaviors

###Closure of Inactive Cursors

서버는 cursor가 리턴되고 10분동안 사용되지 않으면 자동으로 해당 cursor를 닫아버립니다. 또는 cursor가 부담이 되고 있는 경우라도 해당 cursor를 닫아버립니다. 
이러한 동작을 메서드를 통해서 제어할수 있습니다.
```
var myCursor = db.users.find().noCursorTimeout();
```
noCursorTimeout()   메서드를 사용하는 경우에는 반드시 수동으로  cursor를 닫아주는 cursor.close() 를 해줘야합니다.

###Cursor Isolation¶

리턴된 cursor에 다른 동작들이 쿼리에 존재할수 있습니다. MMAPv1 저장엔진의 경우에는 쓰기작업이 리턴된 document 에 한번 이상 발생할수가 있고 이는 document의 정보를 변경시킬수 있습니다. 이러한 상황을 조절하기 위해서 [snapshot mode](https://docs.mongodb.com/manual/core/read-isolation-consistency-recency/#faq-developers-isolate-cursors) 모드를 지원합니다.

###Cursor Batches
몽고디비 서버는 쿼리의 일괄으로 처리한 결과를 리턴합니다. 일괄처리되는 사이즈는 BSON 의 document가 지원하는 최대 크기를 넘지 않습니다. 대부분의 쿼리의 결과로는 101개의 document가 전달되거나 1mb 의 사이즈를 넘지 않습니다. 이후 계속되는 작업의 경우에는 하나의 작업당 4mb의 사이즈가 반영이 됩니다. 기본적으로 지정된 사이즈를 수정하기 위해서 [batchSize()](https://docs.mongodb.com/manual/reference/method/cursor.batchSize/#cursor.batchSize) 와 [limit()](https://docs.mongodb.com/manual/reference/method/cursor.limit/#cursor.limit) 에 대한 정보를 확인하시기 바랍니다.

반복을 통해 전달된 모든 결과의 데이터를 확인하려고 할 때 cursor.next() 메서드는 [getmore]() 동작을 수행하여 다음 결과들을 지속적으로 읽어옵니다. objsLeftInBatch()메서드를 사용해서 얼마나 많은 데이터들이 남아있는지를 확인 할 수있습니다.

```
var myCursor = db.inventory.find();

var myFirstDocument = myCursor.hasNext() ? myCursor.next() : null;

myCursor.objsLeftInBatch();
```

##Cursor Information

db.serverStatus() 메서드는 metrics 필드가 포함된 document를 리턴합니다. metrics 필드는 metrics.corsur 필드에 다음의 정보를 담고 있습니다.
- 마지막으로 서버가 재시작 된이후 시간이 지나버린 cursor의 갯수
- 현재 DBQuery.Option.noTimeout 옵션을 통해 현재도 열려있는 cursor의 갯수
- 고정된(pinned) 현재 열려있는 cursor
- 전체 열려있는 cursor

다음의 db.serverStatus() 메서드를 사용해 metrics.cursor 필드에 접근하는 코드입니다.

```
db.serverStatus().metrics.cursor
```
결과는 다음과 같습니다.

```
{
   "timedOut" : <number>
   "open" : {
      "noTimeout" : <number>,
      "pinned" : <number>,
      "total" : <number>
   }
}
```
















