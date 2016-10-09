[목록](https://github.com/yuby/mongodb-ko)



#Iterate a Cursor in the mongo Shell

[db.collection.find()]() 메서드는 cursor를 리턴합니다. documents에 접근하기 위해서는 cursor의 반복을 통해서 가능합니다. 하지만 [몽고쉘]()에서는 만약에 cursor에 값이 정해 지지 않은 경우에는 자동으로 20번만 cursor를 반복해서 해당 결과를 보여줍니다. 즉 검색된 20개의 결과를 볼수가 있습니다. 물론 우리는 앞으로 설명하겠지만 인덱스를 사용해서 특정 결과에만 적근하거나 수동으로 cursor에 접근할수가 있습니다.

##Manually Iterate the Cursor

[몽고쉘]()에서 [find()]() 메서드에 우리가 리턴받을 cursor를 변수에 저정하수 있습니다. 이러면 자동으로 해당 결과를 20번 반복하는 작업을 하지 않습니다.

그리고 변수로 저장된 cursor를 불러도 쉘상에서 20번 반복할수 있습니다.

```
var myCursor = db.inventory.find( { type: 'food' } );

myCursor
```

[next()]() 메서드를 사용해서 documents에 접근할수 있습니다.

```
var myCursor = db.inventory.find( { type: 'food' } );

while (myCursor.hasNext()) {
   print(tojson(myCursor.next()));
}
```
출력을 위해서 위에서는 print(tojson()) 으로 표현을 했지만 다른 방법으로는 printjson() 메서드를 사용하는 방법도 있습니다.

```
var myCursor = db.inventory.find( { type: 'food' } );

while (myCursor.hasNext()) {
   printjson(myCursor.next());
}
```
[forEach()]() 메서드를 사용해서 반복작업을 할수도 있습니다.

```
var myCursor =  db.inventory.find( { type: 'food' } );

myCursor.forEach(printjson);
```

[JavaScript cursor methods]() 와 [driver]()문서를 확인해서 cursor메서드들을 더 확인해보세요.


##Iterator Index

[몽고쉘]()에서 [toArray()]()메서드를 사용해서 cursor에 저장된 documents들을 전달 받을수 있습니다.

```
var myCursor = db.inventory.find( { type: 'food' } );
var documentArray = myCursor.toArray();
var myDocument = documentArray[3];
```
[toArray()]()메서드는 RAM에서 작업을 합니다.

추가적으로 몇몇 [drivers]() 는 cursor에 index를 사용해서 접근할수 있도록 해줍니다. 이는 [toArray()]()메서드를 호출하고 인덱스를 통해 접근하는 방법의 shortcut입니다.

```
var myCursor = db.inventory.find( { type: 'food' } );
var myDocument = myCursor[3];
```

myCursor[3] 는 아래와 동일한 기능입니다.

```
myCursor.toArray() [3];
```



[목록](https://github.com/yuby/mongodb-ko)