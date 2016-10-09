#Query for Null or Missing Fields
몽고디비의 연사자들은 저마다 방식으로 null 값을 다룹니다.
이에 대한 예제를 수행하기 위해 아래의 코드를 mongo shell 상에서 실행시키시기 바랍니다.

```
db.users.insert(
   [
      { "_id" : 900, "name" : null },
      { "_id" : 901 }
   ]
)
```

##Equality Filter
{ name : null } 쿼리를 작성을 하면 이경우에는 name 필드가 null 값을 가지는 경우와 name 필드가 존재하지 않는 경우 모두를 결과로 리턴합니다.

```
db.users.find( { name: null } )
```

결과는 다음과 같습니다.

```
{ "_id" : 900, "name" : null }
{ "_id" : 901 }
```

만약 쿼리가 sparse index를 사용한다면 쿼리은 오직 필드와 값이 존재하는 경우에 해당하는 document만을 리턴하게 됩니다.

```
{ "_id" : ObjectId("523b6e32fb408eea0eec2647"), "userid" : "newbie" }
{ "_id" : ObjectId("523b6e61fb408eea0eec2648"), "userid" : "abby", "score" : 82 }
{ "_id" : ObjectId("523b6e6ffb408eea0eec2649"), "userid" : "nina", "score" : 90 }
```
의 데이터에 대해서 score에 sparse index가 지정된경우 

```
db.scores.find( { score: { $lt: 90 } } )
```

위의 코드는 다음의 결과를 리턴합니다.

```
{ 
	"_id" : ObjectId("523b6e61fb408eea0eec2648"), 	"userid" : "abby", "score" : 82 }
```

##Type Check
{ name : { $type: 10 } } 쿼리는 name 필드의 값이 null document만을 리턴합니다.

```
db.users.find( { name : { $type: 10 } } )
```
결과는 다음과 같습니다.

```
{ "_id" : 900, "name" : null }
```

##Existence Check
{ name : { $exists: false } } 쿼리는 name 필드가 존재하지 않는 document에 대해서만 리턴을 합니다.

```
db.users.find( { name : { $exists: false } } )
```
결과는 다음과 같습니다.

```
{ "_id" : 901 }
```





