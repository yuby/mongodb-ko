#Data Types in the mongo Shell
몽고디비  BSON은 JSON이 제공하는 이외에 추가적인 데이터 타입을 지원합니다. 이는 드라이버가 지원하기도 하도 mongo shell 또한 지원하고 있습니다.

##Types

###Date
mongo shell은 문자열이나 날짜객체같은 형태의 날짜정보를 리턴하는 메서드를 제공합니다.
- Date()  메서드는 현재의 날짜 정보를 문자열로 전달합니다.
- new Date() 생성자는 Date 객체를 ISODate() 객체로 쌓여 전달이 됩니다.
- ISODate() 생성자는 Date 객체를 ISODate() 객체로 쌓여 전달이 됩니다.

내부적으로 Date 객체는 64비트 Unix epoch(Jan 1, 1970) 이후로의 시간을 밀리초로 표현해서 과거부터 현재 까지 290,000,000  년의 범위를 표현할수가 있습니다.


###Return Date as a String
Date() 메서드를 사용해서 날짜 정보를 확인하는 방법입니다.
```
var myDateString = Date();
```
날짜 정보를 확인하고 싶으면 변수이름을 shell에 작성하면됩니다.
```
myDateString
```

결과값은 다음과 같습니다.
```
Wed Dec 19 2012 01:03:25 GMT-0500 (EST)
```
typeof 메서드를 사용하면 타입정보를 확인합니다.
```
typeof myDateString
```

string 이라는 결과를 리턴합니다.

###return Date
mongo shell에서의 Date객체는 ISODate 에 쌓여있지만 여전히 Date타입입니다.
다음의 예제는 new Date() 생성자와 ISODate() 생성자를 사용해 Date 객체를 전달하는 예제입니다.
```
var myDate = new Date();
var myDateInitUsingISODateWrapper = ISODate();
```
ISODate()의 경우에는 new 를 사용해도 되고 그냥 쓰지 않아도 됩니다. myDate 변수를 shell에 입력을하면 ISODate() 로 감싸여 다음과 같은 결과를 리턴합니다.
```
ISODate("2012-12-19T06:01:17.171Z")
```
instanceof 연산자를 사용해 타입확인이 가능합니다.
```
myDate instanceof Date
myDateInitUsingISODateWrapper instanceof Date
```
모두 true의 결과를 리턴합니다.

###ObjectId
mongo shell은 ObjectId() wrapper class를 사용한 ObjectId 데이타 타입을 제공합니다. 새로운 ObjectId를 생성하는 방법은 다음과 같습니다.

```
new ObjectId
```

###NumberLong
기본적으로 mongo shell은 모든 숫자를 floating-point(부동소수) 로 처리를 합니다. mongo shell은 64비트 정수를 다루기 위해 NumberLong() 메서드를 제공합니다.

NumberLong()메서드는 문자열을 전달 받습니다.

```
NumberLong("2090845886852")
```
다음은 NumberLong()을 사용해서 데이터를 저장하는 예제입니다.

```
db.collection.insert( { _id: 10, calc: NumberLong("2090845886852") } )
db.collection.update( { _id: 10 },
                      { $set:  { calc: NumberLong("2555555000000") } } )
db.collection.update( { _id: 10 },
                      { $inc: { calc: NumberLong(5) } } )
```
해당 정보를 가져옵니다.
```
db.collection.findOne( { _id: 10 } )
```
결과는 calc 필드에 NumberLong 형태로 값이 들어있습니다.
```
{ "_id" : 10, "calc" : NumberLong("2555555000005") }
```

만약에 $inc 메서드로 float 형태의 NumberLong객체의 숫자가 부동소수점(floating point value) 형태로 형이 변환이 됩니다. 다음의 예제를 확인 하시기 바랍니다.

1. $inc 를 사용해서 mongo shell이 float으로 다루는 calc 필드의 값을 5증가 시킵니다.
```
db.collection.update( { _id: 10 },
                      { $inc: { calc: 5 } } )
```
2. 수정된 document를 확인 합니다.
```
db.collection.findOne( { _id: 10 } )
```
결과로 calc의 값이 부동수소점으로 변경된 상태로 전달이 됩니다.
```
{ "_id" : 10, "calc" : 2555555000010 
```

###NumberInt
기본적으로 mongo shell의 경우에는 모든 숫자를 부동 소수점을 처리를 합니다. mongo shell은 NumberInt()의 경우에는 32비트 정수형으로 제공 합니다.

##Check Types in the mongo Shell
mongo shell에서 instanceof 와 typeof 를 사용해서 필드의 타입을 확인할 수 있습니다.

###instanceof
instanceof는 boolean 으로 타입의 비교 결과를 리턴합니다. 예를들어 _id 필드가 ObjectId 타입인지를 확인하려면 다음과 같습니다.

```
mydoc._id instanceof ObjectId
```
결과로 true를 전달합니다.

###typeof
typeof는 해당 필드의 타입정보를 리턴합니다. 예를들어 _id 필드의 타입을 확인 할때는 다음과 같습니다.
```
typeof mydoc._id
```
이경우에는 ObjectId 타입보다는 포괄적인 객체 타입을 전달합니다.








