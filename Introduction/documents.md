#Documents
몽고디비는 BSON documents로 데이터를 저장합니다. BSON은 바이너리로 펴햔되는 JSON document입니다. josn의 비해 더 많은 데이터 타입을 저장 할 수있습니다.

![A MongoDB document.](http://docs.mongodb.org/manual/_images/crud-annotated-document.png)

##Document Structure
몽고디비 documents는 필드-값 의 쌍으로 데이터 구조를 가지고 있씁니다.

```
{
   field1: value1,
   field2: value2,
   field3: value3,
   ...
   fieldN: valueN
}

```

필드의 값의 경우에는 다른 document를 포함해서 BSON이 허용하는 모든 데이터 타입을 사용가능합니다.

```
var mydoc = {
               _id: ObjectId("5099803df3f4948bd2f98391"),
               name: { first: "Alan", last: "Turing" },
               birth: new Date('Jun 23, 1912'),
               death: new Date('Jun 07, 1954'),
               contribs: [ "Turing machine", "Turing test", "Turingery" ],
               views : NumberLong(1250000)
            }

```

###Field Names
필드 이름은 문자열입니다.
다음은 document의 필드이름 제한사항입니다.

- _id 필드이름은 primary key의 역활을 합니다. 값은 반드시 유니크한 값이고 collection상에서 반드시 유일해야합니다. 이는 변경이 불가능하고 배열이외의 모든 형태가 가능합니다.
- 필드이름의 경우 ($)를 시작 문자로 사용 할 수 없습니다.
- 필드이름의 경우 (.)을 시작 문자로 사용 할 수 없습니다.
- 필드이름의 경우 null 이 포함된 필드를 가질수가 없습니다.

BSON document의 경우에는 중복되는 이름의 필드를 가지는 것을 혀용하지만 몽도디비에서는 허용하지 않습니다. 만약 중복되는 필드를 가지게 허용하고 싶다면 driver documentation을 찾아서 필요한 driver를 추가해야합니다.

###Filed Value Limit
인덱싱이 적용된 collection의 경우에, indexed field의 경우에는 키길이의 제한이 있습니다.

##Dot Notation
몽고디비는  (.) 을 통해서 array의 엘리먼트를 접근하거나 document의 필드를 접근할수가 있습니다.

###Array
기본적으로 배열의 인덱스틑 0 부터 시작을 합니다.array의 이름이나 index의 포지션을 통해서 값을 접근 할 수 있습니다.

```
"<array>.<index>"
```
예를들면 다음과 같은 document가 있다고 가정하면

```
{
   ...
   contribs: [ "Turing machine", "Turing test", "Turingery" ],
   ...
}
```
세번째 엘리먼트를 접근하고 싶다면 "contribs.2" 를 통해 값에 접근하는 것이 가능합니다.

###Embeded Documents
document의 필드도 (.)을 통해서 접근이 가능합니다.

```
"<embedded document>.<field>"
```
예를들어 다음과 같은 형태의 documents가 있다고 가정해보갰습니다.

```
{
   ...
   name: { first: "Alan", last: "Turing" },
   contact: { phone: { type: "cell", number: "111-222-3333" } },
   ...
}
```
"name.last" 또는 "contact.phone.number"

##Document Limitations

###Document Size Limit

BSON의 최대 사이즈는  16 mb입니다. document의 최대 사이즈 제한은 하나의 document가 RAM의 용량을 넘어서거나 데이터 전송간에 대역폭을 넘어서는 일을 막기위해서 입니다. 만약 16mb이상의 데이터를 저장하고 싶다면 몽고디비는 GridFS API를 제공하고 있습니다.

###Document Field Order
몽고디비는 document의 필드의 순서를 보장하지만 다음의 경우에는 예외입니다.

- _id 필드의 경우에는 항상 모든 document의 처음에 위치합니다.
- 필드이름을 변경하는 경우 필드의 순서가 변경되기도 합니다.

>2.6 버전 이후 부터 필드의 순서를 보장하고 있습니다.

###The _id Field
몽고디비에서 document는 유니크한 _id값 primary key로서 사용하여 collection 에 저장이 됩니다.
몽고디비는 ObjectIds를 기본 _id 필드의 값으러 사용합니다. 만약에 데이터 입력하는 단계에서 _id 필드가 없다면 몽고디비가 자동으로 해당 document에 _id필드와 ObejctId를 값으로 부여하게 됩니다.

추가적으로, 만약 mongod가 _id 필드가 없는 document를 입력데이터로 받게 되어도 자동으로 ObjectID를 가진 _id필드를 추가하게 됩니다.

_id필드는 다음과 같은 동작과 제한 사항을 가집니다.

- 기본적으로 데이터 생성시에 유니크한 인덱스 값을 collection이 생성될때 생성합니다.
- _id 필드는항상 document의 첫번째 필드로 존해합니다. 만약 첫번째로 존재하지 않는 경우 해당 필드를 처음으로 자동으로 이동시킵니다.
- _id 필드는 array를 제외한  BSON이 제공하는 모든 데이터 형을 가질수 있습니다.

> WARNING
> replication을 위해서 BSON의 정규식 형태의 값은 _id값으로 저장하지 않습니다.

다음은 _id 값으로 사용가능한 값의 선택 사항입니다.

- ObjectId 를 사용합니다.
- 가능하다면 자연적 고유식별자를 사용하는 것이 저장공간을 절약하고 추가적인 인덱싱작업을 피할 수 있습니다.
- auto-increamenting숫자를 생성합니다.
- UUID를 어플리케이션 코드단에서 생성합니다. 효과적인 UUID값을 가지기 위해서는 BinData타입을 사용해서 저장하면 됩니다.
BinData 타입을 다음과 같은 형태로 저장을 한다면 더욱 인덱싱에 효과적입니다.
	- 바이너리 서브타입 값의 범위는 0-7 또는 128-135 입니다.
	- 바이트 array 길이의 범위는 0, 1, 2, 3, 4, 5, 6, 7, 8, 10, 12, 14, 16, 20, 24,32 일때 입니다.
- driver가 생성하는 UUID를 사용합니다. driver는 UUID 생성에 driver에 특화된 로직을 사용하기 때문에 다른 driver가 읽어들이지 못하는경우가 생실수 있습니다.

##Other Users of the Document Structure

몽고디비가 사용하는 document의 구조는 저장을 위한 형태 이외에도 query filters, update specifications documents, 그리고 index specification documents를 할때도 사용합니다. 

###Query Filter documents
Query filter는 읽기, 수정, 삭제시에 특정 조건을 나타낼때 사용합니다.
<field>:<value> 형태입니다.

```
{
  <field1>: <value1>,
  <field2>: { <operator>: <value> },
  ...
}
```
예를들면

```
 { 
 	status: { 
 		$in: [ "P", "D" ] 
	} 
 
 } 
```
같은 형태입니다.

###Update Specification Documents

update 메서드를 사용한 특정 document를 수정시에 특정 필드값을 지정하기 위해 사용하는 표현입니다.

```
{
  <operator1>: { <field1>: <value1>, ... },
  <operator2>: { <field2>: <value2>, ... },
  ...
}
```

예를들면
```
  {
     $set: { "favorites.food": "pie", type: 3 },
     $currentDate: { lastModified: true }
  }

```

###Index Specification Documents
document에 정의된 인덱스 필드와 타입

```
{ <field1>: <type1>, <field2>: <type2>, ...  }
```








