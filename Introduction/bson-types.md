#Capped Collecions

BSON은 document를 저장하고 원격 프로 시저가 MongoDB를에 호출을하는 데 사용되는 binary serialization 포맷입니다.
BSON의 자세한 사항들은 [bsonspec.org](bsonspec.org)에서 확인 하시면 됩니다.

BSON은 다음의 데이터 형태의 값들을 지원합니다. 각각의 데이터 타입은 
각 데이터 유형은 BSON 타입별로 $형태의 연산자로 document를 조회(query)할 수 있도록 각 타입별 특정 문자나 숫자 형태의 별칭을 가지고 있습니다.



| Type	| Number | Alias	| Notes|
| ---- | ---- |---- |---- |
|Double|	1	|“double”||	 
|String|	2	|“string”||	 
|Object|	3	|“object”||
|Array|	4	|“array”||	 
|Binary data|	5|	“binData”||	 
|Undefined|	6|	“undefined”|	Deprecated.|
|ObjectId	|7	|“objectId”||	 
|Boolean|	8|	“bool”||	 
|Date|	9|	“date”||	 
|Null|	10|	“null”||	 
|Regular Expression|	11|	“regex”||	 
|DBPointer|	12|	“dbPointer”||	 
|JavaScript|	13|	“javascript”||	 
|Symbol|	14|	“symbol”||	 
|JavaScript (with scope)|	15|	“javascriptWithScope”|| 
|32-bit integer|	16|	“int” ||	 
|Timestamp|	17|	“timestamp”||	 
|64-bit integer|	18|	“long”	 ||
|Min key|	-1|	“minKey”||	 
|Max key|	127|	“maxKey”||

만약에 BSON을 JSON으로 변경하게 위해서는 [Extended JSON](https://docs.mongodb.com/manual/reference/mongodb-extended-json/) 문서를 참고하기 바랍니다.

###Comparison/Sort Order

서로다른 BSON타입의 값을 비교하는 경우 몽고디비는 다음의 우선순위를 가지게 됩니다. 낮음-> 높은 순으로 나타냈습니다.

1. MinKey (internal type)
2. Null
3. Numbers (ints, longs, doubles)
4. Symbol, String
5. Object
6. Array
7. BinData
8. ObjectId
9. Boolean
10. Date
11. Timestamp
12. Regular Expression
13. MaxKey (internal type)
	 
몽고디비에서 몇몇 타입의 경우에는 비교를 위한 목적으로 사용을 합니다. 예를들면 numeric 타입의 경우에는 비교 이전에 변환작업을 진행합니다.

> 3.0.0 버전부터는 Date 객체가 Timestamp 객체 이전에 정렬이 이뤄집니다. 이전버전에선 Date와 Timestamp 객체가 함께 정렬이 이뤄졌습니다.

값 비교시에 존재하지 않는 필드의 경우에는 empty BSON객체로 다룹니다. { } document와 { a: null } 객체는 정렬시에 동일한 레벨로 두고 정렬을 합니다.

배열에서는 '~보다작은' 비교 또는 오름차순 경우에는 배열의 가장 작은 요소를 비교하고, '~보다 큰' 비교나 내림차순의 경우에는 배열의 가장 큰 엘리먼트를 비교합니다.
하나의 엘리먼트를 가지는 배열(e.g. [1] )이 배열에 존재하지 않는 필드(e.g. 2)의 값과 비교하는 경우에는 1과 2사이의 값으로 비교를 합니다.
빈배열의 경우에는 null이나 필드의 값이 존재하지 않는 경우보다 더 낮은 레벨로 취급을 합니다.

몽고디지는 BinData의 정렬은 다음의 순서로 이뤄집니다.

First, the length or size of the data.

Then, by the BSON one-byte subtype.

Finally, by the data, performing a byte-by-byte comparison.

다음은 BSON 타입중에 특별히 고려해야하는 타입들에 대한 설명입니다.

###ObjectId

ObjectIds는 작고, 유일한 값이며, 빠르게 생성이 가능하고 순차적입니다. ObjectIds의 값은 12-bytes로 구성되어있고 처음 4바이트는 timestamp를 나타내어 ObjectIds의 생성시기를 나타냅니다. 특히

- 4바이트는 유닉스  기준의  초를 나타내고
- 3바이트는 머신의 구분자
- 2바이트는 프로세서 아이디
- 마지막 3바이트는 카운터로 랜덤한 값부터 시작합니다.

몽고디비에서 document는 유니크한 primary key로 사용 될 _id값을 필요로합니다. 만약에 _id필드가 지정되지 않는다면 몽고디비는 기본적으로 ObjectIds를 _id의 값으로 사용합니다. 예를들어 만약에 document의 최상단 레벨에 _id필드가 생성당시에 존재하지 않는다면, 몽고디비 드라이버는 _id필드의 값을 ObjectId값을 추가합ㅂ니다.

추가적으로, 만약에 mongod가 _id필드가 없는 document를 생성시에 전달받으면  자동으로 _id에 ObjectId값을 부여해서 추가시킵니다.
몽고디비 클라이언트는 반드시 _id를 유니크한 ObjectId를 가지게 합니다. ObjectId를 값으로 사용을 하면 다음과 같은 이점이 있습니다.

- 몽고 shell에서 , 우리는 ObjectId의 생성시의 시간을 접근할수가 있습니다. ObjectId.getTimestamp()
- _id필드로  저장을 하면 대략적으로 생성시간 순서로 정렬되어 저장이 됩니다.

>IMPORTANT
>ObjectId 값의 순서와 생성 시간의 관계는 반드시 둘관계가 일치한다고 보기 어렵습니다. 여러 시스템 또는 단일 시스템에서 여러 프로세스 또는 스레드가 값을 생성하는 경우에 ObjectId 값은 데이터 간에 생성된 순서를 완벽하게 대변하지 않습니다. 클라이언트 드라이버가 ObjectId가 값을 생성하기 때문에 클라이언트 간의 시간오차가 존재하기도 해서 정확한 생성의 순서를 표현한다고 보기도 어렵습니다.


###String

BSON의 문자열은 UTF-8형태입니다. 일반적으로 각 언어별 드라이버는 BSON으로 serializing 와 deserializing 을 하는 동안  언어의 문자열을 UTF-8형태로 맞춰 저장되는 데이터를 다양한 언어로 BSON의 문자열로 쉽게 저장될 수 있도록합니다. 추가적으로 몽고디비는 정규식 연산자인 $regex 사용할떄 UTF-8 형태를 지원합니다.

###Timestamps

BSON은 특별한 timestamp타입을 몽고디비 내부에서 사용하고 이는 일반적인 Date 타입과는 관련이 없습니다. timestamp값은 64bit의 값을 가집니다.
- 처음 32 비트는 유닉스 기준의 시간정보인 time_t 값을 가집니다.
- 다음 32 비트는 주어진 초 내에 작업을 위한 증가하는 순서이다.

하나의 mongod 인스턴스상에서는 timestamp가 항상 유니크한 값을 가집니다.

replication에서는  oplog가 ts필드를 가집니다. 이 값은 작업이 이뤄진 시간을 BSON의 timestamp값을 사용해서 나타냅니다.
