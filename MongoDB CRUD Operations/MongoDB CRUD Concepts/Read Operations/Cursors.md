[목록](https://github.com/yuby/mongodb-ko)

#Cursors

몽고shell 에서 읽기 작업에 가장 우선적으로 사용이 되는 메서드는 db.collection.find() 입니다. 이 메서드는 collection을 대상으로 쿼리를 실행하고 cursor를 리턴되는 documents에 전달합니다.

documents에 접근하려면 cursor를 반복시켜서 각각의 document에 접근할 수 있습니다. 하지만, 몽고shell에서는 만약에 리턴받은 cursor에 var 키워드에 값이 지정되지 않았다면 자동으로 20번 cursor를 반복시켜 총 20개의 documents[[1]](http://docs.mongodb.org/manual/core/cursors/#set-shell-batch-size)를 화면에 보여줍니다.

예를들어, 몽고shell에서 inventory collection을 대상으로 읽기 작업을 실행하면 type이 food인 documents를 20개 리턴해줍니다:
```
db.inventory.find( { type: 'food' } );
```
수동으로 cursor를 반복하여 documents에 접근할수 있습니다. [여기](http://docs.mongodb.org/manual/tutorial/iterate-a-cursor/)서 확인 하시면 됩니다.

> [1] DBQuery.shellBatchSize를 사용하여 기본으로 20번의 반복을 수정할 수 있습니다. 쿼리 실행에 대한 더많은 정보는  [이곳](http://docs.mongodb.org/manual/tutorial/getting-started-with-the-mongo-shell/#mongo-shell-executing-queries)에서 확인하시면 됩니다.


##Cursor Behaviors

###Closure of Inactive Cursors
기본적으로 서버에 10분동안 사용되지 않는 cursor나 클라이언트에서 더이상 사용하지 않는 cursor의 경우 자동으로 없애줍니다. 하지만 [cursor.addOption()](http://docs.mongodb.org/manual/reference/method/cursor.addOption/#cursor.addOption)의 noTimeout 플래그를 사용한다면 이 동작을 얼마든지 조절할 수 있습니다. 하지만 반드시 위의 두경우의 cursor에 대해서는 업애주는 작업이 이뤄저야 합니다. [몽고](http://docs.mongodb.org/manual/reference/program/mongo/#bin.mongo)shell에서 noTimeout을 설정하는 방법은 아래와 같습니다.
```
var myCursor = db.inventory.find().addOption(DBQuery.Option.noTimeout);
```
noTimeout에 대한 더 많은 정보는 [driver](http://docs.mongodb.org/manual/applications/drivers/) 문서를 확인하세요. [몽고](http://docs.mongodb.org/manual/reference/program/mongo/#bin.mongo)shell 에서 사용가능한 모든 cursor플래그 리스트를 [cursor.addOption()](http://docs.mongodb.org/manual/reference/method/cursor.addOption/#cursor.addOption) 을 통해 확인하세요.

###Cursor Isolation
cursor가 사용되는 동안은 cursor는 고립된 존재가 아닙니다. 중간에 쓰기 작업이 이미 한번 이상 리턴한 cursor의 document에 이뤄진다면 해당 cursor의 결과에 영향을 주게 됩니다. 이러한 문제를 다루기 위해서 [snapshot mode](http://docs.mongodb.org/manual/faq/developers/#faq-developers-isolate-cursors)를 제공하고 있습니다.

###Cursor Batches
몽고디비 서버는 배치를 통해 쿼리의 결과를 리턴합니다. 배치의 사이즈는 BSON document의 최대 사이즈를 초과하지 않습니다. 대부분의 쿼리의 경우에는 첫번째 배치의 결과로 101개의 documents를 리턴하거나 1메가바이트를 넘지 않는 한도 내에서 적당한 양을 리턴합니다. 그리고 그 이후의 배치에 대해서는 4메가바이트 씩을 전달합니다. batchSize() 와 limit()를 통해서 리턴하는 배치의 사이즈를 재설정할 수 있습니다.

쿼리에 정렬작업이 포함되어 있는데 인덱스가 존재하지 않을 경우에, 서버는 모든 documents를 메모리로 가져와 정렬을 시키고 리턴을 시킵니다.

커서의 반복작업을 통해서 리턴받은 배치의 데이터의 끝에 도달했을떄 더 이상의 결과가 존재하지 않을 경우 cursor.next()가 동작해서 다음 배치를 전달받기 위한 작업을 합니다. objsLeftInBatch()를 통해서 얼마나 많은 cursor가 배치에 남아있는지를 확인하는 방법입니다.
```
var myCursor = db.inventory.find();

var myFirstDocument = myCursor.hasNext() ? myCursor.next() : null;

myCursor.objsLeftInBatch();
```

##Cursor Information

db.serverStatus() 메서드는 metrics필드를 가진 document를 리턴합니다. metrics스 필드는 아래의 정보를 담고 있는 cursor필드 정보를 가지고 있습니다.

- 마지막 서버가 재시작 된 이후에 시간이 경과한 cursor의 갯수
- DBQuery.Option.noTimeout 으로 비활성화 되지 않은 실제 사용가능한 cursor의 갯수.
- 고정된(pinned) 사용가능한 cursor의 갯수
- 전체 사용가능한 cursors의 갯수

db.serverStatus() 메서드를 호출하여 결과로 부터 metrics 필드에 접근하여 cursor필드에 접근하면 해당 cursor의 정보를 얻을수있습니다.
```
db.serverStatus().metrics.cursor
```

결과는 아래와 같습니다.

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

[목록](https://github.com/yuby/mongodb-ko)