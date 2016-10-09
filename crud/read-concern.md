#Read Concern
몽고디비 3.2 버전부터 readConcern 이 replica set과 replica set의 shard의 옵션으로 설정이 가능합니다. 기본적으로 몽고디비는 read concern을 local로 설정되어 해당 데이터가 replica set의 majority로 지속되지 않아 롤백이 된다 하더라도 쿼리가 실행이 되는 순간 몽고디비의 가장 최신의 데이터를 리턴하게 됩니다.

##Storage Engine and Drivers Support
WiredTiger storage engine 은 readConcern옵선을 설정하는 것이 가능하도록 해서 읽기작업의 고립의 정도를 선택하게 합니다. read concern을 majority로 설정해 데이터를 읽게되면 replica set의 majority 에 생성된 데이터를 읽게되고 해당 데이터는 롤백이 되지 않습니다.

MMAPv1 storage engine 은 오직 local 로만 설정이 가능합니다.

>TIP:
>serverStatus 명령은  storageEngine.supportsCommittedReads 필드를 리턴해 엔진의 reac concern이  majority인지 아닌지를 확인하게 합니다.

| level	| Description|
|-----|-----|
|"local" | 기본적으로 쿼리는 인스턴스의 가장 최신에 복사된 데이터를 리턴합니다. 제공된 데이터는 replica set의 majority의 데이터임을 보장하지는 않습니다. |
|"majority" | majorityd에 저장된 가장 최신의 데이터를 리턴합니다. <br/> read concern의 레벨을 majority로 설정을 하면 반드시 WiredTiger storage engine을 사용해야 하고 mongod 인스턴스 시작시에 --enableMajorityReadConcern 를 커맨드라인 옵션으로 작성해야 합니다. <br/> 또는 설정파일에 replication.enableMajorityReadConcern 를 설정해줘야합니다. <br/> 오직 replica set은 protocol version 1이 "majority" 를 지원합니다. replica set이 version 0 으로 동작중일 경우 "majority" read concern을 지원하지 않습니다. <br/>  싱그쓰레드의 경우에는 "majority" read concern과 "majority" write concern을 사용해서 작성한 데이터를 그대로 불러오는 것이 가능합니다. |

read concern 레벨을 고려하지 않을경우 각 노드의 데이터들이 최신의 데이터를 반드시 가지고 있음을 보장하지는 않습니다.

##readConcern Option

reacConcern 옵션을 설정하는 방법은 다음과 같습니다.

```
readConcern: { level: <"majority"|"local"> }
```
readConcern는 다음의 메서드에 옵션값으로 설정될수 있습니다.

- find command
- aggregate command , db.collection.aggregate()
- distinct
- count
- parallelCollectionScan
- geoNear
- geoSearch

