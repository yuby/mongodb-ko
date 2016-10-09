#MongoDB Extended JSON


JSON은 BSON이 지원하는 subset 타입으로 나타낼 수 있습니다. 타입 정보를 보존하기 위해 MongoDB는 JSON 형식으로 다음과 같은 확장 기능을 추가했습니다.

- Strict mode. BSON 타입의 Strict mode 표현은 [JSON RFC](http://www.json.org/) 를 따릅니다. 어떠한 JSON parser도 이 strict mode로 표현된 key/value 형태의 데이터를 파싱할수 있습니다. 몽고디비의 내부 JSON parser는 전달된 타입 정보로 타입정보를 인지합니다.
- mongo Shell mode.  몽고디비의 내부 JSON 파서와 mongo shell은 mongo Shell mode의 JSON을 파싱할 수 있습니다.

다양한 데이터 유형에 사용되는 표현들은 JSON 파싱되는 문맥에 의존합니다.

##Parsers and Supported format

###Input in Strict Mode
다음은 strict mode 에서의 타입정보와 표현식을 파싱할 수 있습니다.
- [REST Interfaces](https://docs.mongodb.com/ecosystem/tools/http-interfaces/)
- [mongoimport](https://docs.mongodb.com/manual/reference/program/mongoimport/#bin.mongoimport)
- 다양한 몽도디비 툴의 --query 옵션

mongo shell과 db.eval()을 포함한 다른 JSON parser는 stric mode에서는 key/value 형태의 표현을 파싱할수 있지만 타입을 인식하지는 못합니다.

###Input in mongo Shell Mode
다음은 mongo Shell Mode 에서의 타입정보와 표현식을 파싱할 수 있습니다.
- [REST Interfaces](https://docs.mongodb.com/ecosystem/tools/http-interfaces/)
- [mongoimport](https://docs.mongodb.com/manual/reference/program/mongoimport/#bin.mongoimport)
- 다양한 몽도디비 툴의 --query 옵션
- mongo shell

###Output in Strict mode
[mongoexport](https://docs.mongodb.com/manual/reference/program/mongoexport/#bin.mongoexport) 와 [REST , HTTP Interfaces](https://docs.mongodb.com/ecosystem/tools/http-interfaces/) 가 Strict mode에서의 아웃풋 데이터입니다.

###Output in mongo Shell mode
[bsondump](https://docs.mongodb.com/manual/reference/program/bsondump/#bin.bsondump) 이 mongo Shell mode의 아웃풋 데이터 입니다.

##BSON Data Types and Associated Representations
다음은 BSON의 데이터 타입과 Strict mode와 mongo Shell mode간의 표현입니다.

###Binary
####data_binary
|Strict Mode | mongo Shell Mode |
|---|---|
| { "$binary": "<bindata>", "$type": "<t>" } | BinData ( <t>, <bindata> ) |

- <bindata> 는 base64 기반의 binary 문자열 입니다.
- <t>는 데이터 종류를 나타내는 단일 바이트의 표현입니다. strict mode에서는 16진수의 문자열이고, Shell mode에서는 정수입니다. 확장된 BSON 문서를 확인해보시기 바랍니다. [http://bsonspec.org/spec.html](http://bsonspec.org/spec.html)

###Date
####data_date
|Strict Mode | mongo Shell Mode |
|---|---|
| { "$date": "<date>" } | new Date ( <date> ) |

strict 모드에서는 <date> 는 ISO-8601의 YYYY-MM-DDTHH:mmLss.mmm<+/-Offset> 형태의 필수 시간 대 필드입니다. 몽고디비 JSON 파서는 현재 로딩된 IOS-8601 로 표현된 이전 Unix epoch에 대해서는 지원하지 않습니다. pre-epoch 와 시스템의 time_t 타입의 과거 시간대를 포맷팅 할 때 다음과 같은 포맷을 사용합니다.
```
{ "$date" : { "$numberLong" : "<dateAsMilliseconds>" } }
```

Shell mode 에서는 <date> 는 64비트의 signed integer로 표현된 epoch UTC 이후부터의 milliseconds 값으로 나타냅니다.

###Timestamp
####data_timestamp
|Strict Mode | mongo Shell Mode |
|---|---|
| { "$timestamp": { "t": <t>, "i": <i> } } | Timestamp( <t>, <i> ) |

- <t> 는 32비트의 unsigned interger로 epoch 이후의 초를 나타냅니다.
- <i> 는 32비트의 unsigned의 증가된 값을 나타냅니다.

###Regular Expression
####data_regex
|Strict Mode | mongo Shell Mode |
|---|---|
| { "$regex": "<sRegex>", "$options": "<sOptions>" } | /<jRegex>/<jOptions> |

- <sRegex>는 검증된 JSON 문자열입니다.
- <jRegex>는 검증된 JSON 문자와 unescaped double quote (") 를 포함한 문자열입니다. 하지만 unescaped forward slash (/) 는 포함하지 않습니다.
- <sOptions> 는 알파벳으로 표현된 정규식 옵션 값을 나타냅니다.
- <jOptions> 는 오직  ‘g’, ‘i’, ‘m’ ,‘s’ 으로만 구성된 문자열입니다. 왜냐하면 자바스크립트와 mongo Shell의 옵션의 범위에 제한이 있고, 잘못된 옵션의 경우에는 표현식으로 변경하는 동안 무시하고 표현식을 구성합니다.

###OID
####data_oid
|Strict Mode | mongo Shell Mode |
|---|---|
| { "$oid": "<id>" } | ObjectId( "<id>" )|

<id>는 24-character 의 16진수 문자열입니다.

###DB Reference
####data_ref
|Strict Mode | mongo Shell Mode |
|---|---|
| { "$ref": "<name>", "$id": "<id>" } | DBRef("<name>", "<id>") |

<name> 는 검증된 JSON 문자열입니다.
<id> 는 어떠한 검증된 확장 JSON 타입입니다.

###Undefined Type
####data_undefined
|Strict Mode | mongo Shell Mode |
|---|---|
| { "$undefined": true } | undefined |

javascript/BSON 의 undefined 타입의 표현식입니다.

undefined 는 query document에서는 사용할 수가 없습니다. 
```
db.people.insert( { name : "Sally", age : undefined } )
```

다음의 쿼리는 에러를 리턴합니다.
```
db.people.find( { age : undefined } )
db.people.find( { age : { $gte : undefined } } )
```
하지만 사용을 원하는 경우 $type 을 사용해서 사용할 수가 있습니다.
```
db.people.find( { age : { $type : 6 } } )
```
이렇게 작성을하면 age가 undefined를 값으로 가진 모든 document를 리턴합니다.


###MinKey
####data_minkey
|Strict Mode | mongo Shell Mode |
|---|---|
| { "$minKey": 1 } | MinKey |
MinKey는 BSON의 모든 타입과 비교했을때 가장 낮은 우선순위를 가집니다.

###MaxKey
####data_maxkey
|Strict Mode | mongo Shell Mode |
|---|---|
| { "$maxKey": 1 } | MaxKey |
MaxKey는 BSON의 모든 타입과 비교했을때 가장 높은 우선순위를 가집니다.

###NumberLong
####data_numberlong
|Strict Mode | mongo Shell Mode |
|---|---|
| { "$numberLong": "<number>" } | NumberLong( "<number>" ) |

NumberLong 은 64비트의 signed integer입니다. 반드시 "" 쌍따옴표를 사용해서 나타내야하고 그렇지 않을 경우에는 floating point 숫자로 변경되어 정확도가 낮아집니다.

다음은 9223372036854775807 숫자를 삽입하는 쿼리입니다.

```
db.json.insert( { longQuoted : NumberLong("9223372036854775807") } )
db.json.insert( { longUnQuoted : NumberLong(9223372036854775807) } )

```

저장된 데이터를 불러오는 경우에 longUnquoted  값이 변경되고 longQuoted의 경우에는 정확도가 계속 유지가 됩니다.
```
db.json.find()
{ "_id" : ObjectId("54ee1f2d33335326d70987df"), "longQuoted" : NumberLong("9223372036854775807") }
{ "_id" : ObjectId("54ee1f7433335326d70987e0"), "longUnquoted" : NumberLong("-9223372036854775808") }
```


