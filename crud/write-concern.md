#Write Concern

Write Concerndms 몽고디비의 mongod 나 replica set, shard clusters의 쓰기작업에 대한 요청의 응답의 수준을 정의하는 옵션입니다. shard cluster의 mongos인스턴스는 shard에 해당 write concern의 값을 전달합니다.

3.2 버전부터는 replica set은 protocolVersion: 1 로 동작을 하고 journal이 활성화시키기 위해서는

- w: "majority" 와  j: true
- majority가 아닌 멤버들은 primary의 j 옵션이 값과 상관없이 각각의 디스크에 journal이 작성이되면 쓰기작업이 수행됩니다. 

2.6 버전부터는 쓰기작업에 write concern의 값을 함께 작성합니다. getLastError 메서드가 더이상 필요가 없습니다. 이전의 버전에서는 쓰기작업이 끝나는 즉시 getLastError 메서드를 실행해서 쓰기작업에 설정한 write concern의 설정값에 따른 응답을 확인했습니다.

##Write Concern Specification

Write Concern 은 다음의 필드를 가집니다.

```
{ w: <value>, j: <boolean>, wtimeout: <number> }
```

- w 옵션은 설정한 갯수만큼의 mongod 인스턴스나 특정 테그가 달린 mongod인스턴스의 쓰기작업의 확인을 설정합니다.
- j 옵션은 쓰기작업을 할때 jounal에 쓰기작업의 확인을 설정합니다.
- wtimeout 옵션은 쓰기작업으로 무제한으로 블록킹이 되는 형상을 방지하기 위해 시간제한을 설정합니다.

###w Option
w 옵션은 쓰기작업이 전달된 특성 수의 mongod 인스턴스나 특정 테그가 설정된 mongod인스턴스의 쓰기동작에 대한 확인을 요청합니다.

다음의 값들로 w: <value> 설정으로 write concern 설정이 가능해집니다.

>NOTE
>하나의 mongod 인스턴스와 replica set의 primary의 경우 j:true가 아니더라도 메모리상에 저장된 이후에 쓰기작업이 진행됩니다.
>3.2버전부터 replica set은 protocolVersion: 1로 사용되고 있습니다. 이는 primary가 아닌곳에서는 쓰기작업에 대해 해당 디스크에 journal이 저장이 이뤄진 다음에 데이터 저장이 이뤄집니다.

|Value	|Description|
|-----|-----|
|<number>|쓰기작업이 전달된 특성 수의 mongod 인스턴스나 특정 테그가 설정된 mongod인스턴스의 쓰기동작에 대한 확인을 요청합니다. 예를들면 <br/> **w:1** : 하나만 존재하는 mongod인스턴스나 replica set의 primary을 대상인경우 몽고디비는 기본적으로 w:1 을 설정합니다. <br/> **w:0** : 쓰기작업에 대한 응답을 요청하지 않습니다. 하지만 socket의 에러나 네트워킹상의 에러 정보를 리턴합니다. <br/> 만약w:0으로 설정했지만 j:true 인경우에는 j:true가 우선시되어 하나만 존재하는 mongod나 replica set의 primary를 대상으로 작업에 대한 확인을 요청합니다. <br/> 1이상의 숫자는 오직 replica set상에서  primary를 포함해 특정노드의 값으로 설정이 가능합니다.|
|"majority"| 쓰기 요청에 대한 확인이 majority로 선택된 노드나, 디스크상에 journal이 쓰여진 노드를 대상으로 쓰기작업에 대한 확인을 요청합니다. <br/> replica set에서는 protocolVersion: 1, w: "majority" 로 j: true를 의미합니다. 그래서 w: <number> 와 달리 w: "majority" 의 경우 primary도 디스크상에 journal쓰기를 하고 쓰기작업을 합니다. <br/> w: "majority"로 설정후에 쓰기작업을 한 결과에 대한 확인의 경우에는 readConcern이 "majority" 여야합니다.|
|<tag set> | 쓰기작업을 특정 테그가 정의된replca set 에 전달해 동작에 대한 확인을 요청합니다.|

###j Option

몽고디비로부터 journal에 쓰기작업이 쓰여 졌는지 확인하는 옵션입니다.

|||
|--|--|
|j| w: <value>로 지정된 곳에 디스크상에 journal이 쓰여졌는지에 대한 응답을 확인합니다. j:true가 이 자체로는 replica set 의 primary가 쓰기 작업에 실패하면 작업이 롤백되지 않음을 보장하지는 않습니다.<br/> 3.2 버전부터 j: true는 primary를 포함해 요청을 받은 멤버들이 journal에 쓰여졌을음 모두 확인한 결과를 리턴합니다. 이전 버전에서는 오직 primary의 결과만을 확인했습니다. <br/> replica set에서  protocolVersion: 1, w: "majority" 는 j:true를 의미하고 이는 journaling이 활성화 됨을 의미합니다. 기본적으로 journaling은 활성화 되어있습니다.|

2.6 버전 부터는 write concern에 j: true가 설정되어 mongod나 mongos가 --nojournal 옵션과 함께 실행이되면 에러를 발생합니다. 이전버전에서는 j:true를 무시했습니다.

###wtimeout
write concern 을 위한 밀리세컨드로 지정된 시간제한 옵션입니다. wtimeout 이 활성화 되기 위해서는 w의 값이 1보다 큰경우여야만 합니다.
만약 요구되는 write concern에 부합하더라도 wtimeout은 시간이 초과되는 쓰기작업에는 에러를 리턴합니다. 몽고디비는 wtimeout 의 시간을 초과하기 이전에 성공적으로 수행된 동작에 대해서는 에러가 발생해도 원래대로 데이터를 돌려놓지는 않습니다.

만약 wtimeout 의 값을 지정하지 않고 write concern의 레벨이 충족되지 않으면 무한정 블록이 발생합니다. wtimeout 를 0으로 설정하면 이는 wtimeout 을 설정하지 않은것과 동일한 결과입니다.







