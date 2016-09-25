#Capped Collecions

##Overview
capped collecion은 고정된 크기의 collecion사이즈를 통해서 데이터를 생성하는 명령에 대해서 생성과 불러오기에서 높은 처리율을 보입니다.

capped collection은 기본적인 동작은 circular buffer와 비슷하게 동작을 합니다.
일단 collecion이 제한된 크기로 공간을 차지하고 나면 , 새로운 데이터를 위해서 가장 오래된 데이터를 덮어쓰는 방식으로 데이터를 처리하고 있습니다.

##Behavior

###Insert Order
capped collection는 생성되는 순서를 확실하게 보장합니다. 그결과 쿼리는생성을 위한 별도의 index를 반환하는 작업이 필요가 업게되어 인덱싱작업에 대한 부담을 줄일수가 있습니다. 이게 바로 capped collecion이 쓰기에서 높은 성능을 유지하는 방법입니다.

###Automatic removal of Oldest Documents
새로운 document의 공간을 확보하기 위해서 capped collection은 자동으로 가장 오래된 document를 다른 별도의 스크립트상의 명명 없이 삭제를 합니다.

예를들어 oplog.rs collection은 replica set에서 작업들의 로그를 저장하는 역활을 합니다.
- 대용량 시스템에 의해서 생성된 로그정보를 저장할때 capped collection의 경우에는 별도의 index작업이 필요없으므로 바로 파일시스템에 해당 정보를 생성 할 수가 있습니다. 게다가, first-in-first-out 을 통해서 데이터의 저장이 보장됩니다.
- 작은 크기의 데이터의 경우 capped collection은 캐시를 하고 있습니다. 왜냐하면 캐시는 무거운 쓰기 작업보다는 읽기 작업이기 때문에, 우리는 확실하게 항상 작업이 이뤄지는 공간상에 남겨둘것인지(i.e. RAM) 또는 쓰기작업상에서 발생하는 부담을 감당할것인지를 정해야 합니다.

###_id Index
capped collection은_id 필드를 가지며 기본적으로 인덱싱이 이뤄진 상태입니다.

##Restrictions and Recommendations

###Updates
만약 capped collection상에서 일반적인 수정작업만 이뤄질 예정이라면 index를 생성해서 다음번 업데이트 작업에 있어서 전체 collection 을 스캔하는 일이 없도록합니다.

###Document Size
>Changed in version 3.2

업데이트 작업이나 document를 변경하는 작업으로 document의 크기가 변경이 되면 해당 작업은 실패하게 됩니다.

###Document Deletion
capped collectoin 상에서는 document의 삭제가 불가능 합니다. collection 상에서 document를 삭제하기 위해서는 drop() 메서드를 통해서  해당 collection을 삭제를 하고 다시 capped collection을 생성해야합니다.

###Shading
capped collectoin은 shading이 지원되지 않습니다.

###Query Efficiency
생성이 된 순서되로 가장 마지막에 저장된 엘리먼트를 읽어드리는것이 가장 효과적인 방법으로 collection을 사용합니다. 

###Aggregation $out
aggregation pipeline 연산자인 $out의 결과를 capped collection 상에 저장하는 것은 불가능합니다.


##Procedures

###Create a Capped Collection
capped collection의 경우에는 db.createCollection() 메서드를 통해서 명시적으로 생성이 이뤄져야 합니다. capped collection을 생성할때는 반드시 최대 사이즈가 정의 byte단위로 지정이 되어야하고 , 이 크기 대로 미리 공간을 차지하고 있게 됩니다.
여기에 지정하는 사이즈에는 외부의 오버해드를 위한 아주 약간의 공간을 같이 나타내게 됩니다.

```
db.createCollection('log', { capped: true, size: 100000 })
```

만약에 size 필드의 값이 4096 보다 작거나 같다면 collection은 4096 bytes를 무조건 가지게 됩니다. 한편 몽고디비가 제공하는 사이즈는 256의 배수로 존재합니다.

추가적으로, documents의 최대 갯수를 정하기 위해서는 max필드에 값을 지정하면 됩니다.

```
db.createCollection('log', { capped: true, size: 5242880, max: 5000 })
```

>Important
>size 필드의 경우 반드시 필요한 필드입니다. 만약에 document의 갯수를 지정하였다 하더라도 반드시 size는 지정이 되어야합니다.
>만약에 document의 최대 갯수 이전에 collection의 최대 사이즈를 넘어서는 경우에는 가장 오래된 document를 제거하는 작업이 이뤄저야하게 때문입니다.

###Query a Capped Collection
capped collection상에서 find() 메서드가 동작할때 별도의 순서를 정하고 저장이 이뤄지지 않은경우 저장된 순서대로 저장이 이뤄집니다.
입력된 반대 순서로 데이터를 불러들이기 위해서는 find() 와 sort()메서드를 함께 사용하면됩니다.

```
db.cappedCollection.find().sort( { $natural: -1 } )
```

###Check if a Collection is Capped
isCapped() 메서드를 사용한다면 해당 collection이 capped collection인지를 확인할수 있습니다.

```
db.collection.isCapped()
```

###Convert a Collection to Capped
non-capped collection을 capped collection으로 변경하는 것이 가능합니다.

```
db.runCommand({'convertToCapped': 'mycoll', size: 100000})
```
size필드의 값을 통해 해당 collection의 사이지를 지정 할 수 있습니다.

>Important
>위의 동작은 글로벌로 쓰기작업에 lock을 걸고 다른 작업들도 블락을 시키고 우선 동작을 완료시킵니다.

###Automatically Remove Data After a Specified Period of Time
몽고디비의 TTL 인덱스를 통해서 기간이 지난 데이터에 대해서 유연하게 대처할수 있습니다.
이 인덱스의 경우에는 기간이 지난 데이터를 일반 collection상에서 date-typed필드나 TTL의 인덱스 값을 기반으로 삭제하는 것이 가능합니다.

TTL collections의 경우에는 capped collection에서 지원하지 않습니다.

###Tailable Cursor
capped collection에서 tailable cursor를 사용 할 수가 있습니다. Unix의 **tail -f** 와 유사하게 tailable cursor를 통해서 capped collection상의 마지막을 가리키게 됩니다. 그래서 capped collection에 새로운 데이터가 생성이 되면 tailable cursor를 통해서 해당 document를 불러들일 수 있습니다.





