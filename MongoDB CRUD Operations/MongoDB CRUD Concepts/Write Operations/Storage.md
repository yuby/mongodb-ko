[목록](https://github.com/yuby/mongodb-ko)


#Storage

몽고디비 3.0버전에서 부터 추가적인 stroage 엔진을 지원합니다. 몽고디비의 오리지날 storage 엔진인  mmapv1는 3.0버전의 기본 엔진입니다. 하지만 wiredTiger  엔진이 사용가능해 졌고 이를 통해 다양한 상황에 유연하고 향상된 처리량을 제공하고 있습니다.

##Data Model

몽고디비는 데이터를 풍부한 형태로의 키나 필드를 값과 매핑할수 있는  [BSON](http://docs.mongodb.org/manual/reference/glossary/#term-bson) documents 형태로 저장을 합니다. BSON은 풍부한  컬렉션의 형태를 지원하고 BSON의 필드는 배열이나 document의 형태의 값을 가질수도 있습니다. 모든 documents는  [BSON 의 document사이즈](http://docs.mongodb.org/manual/reference/limits/#BSON-Document-Size)인 16MB이하의 사이즈만을 가집니다.

모든 documents는 몽고디비 데이터베이스의  논리적인 documents의 그룹인 컬랙션의 일부입니다. [collecion](http://docs.mongodb.org/manual/reference/glossary/#term-collection)안의 documents는 설정된 index와 공통의 필드와 구조를 공유합니다.

몽고디비의 [데이터베이스](http://docs.mongodb.org/manual/reference/glossary/#term-database)의 구조는 연관된 collection의 그룹입니다. 각 데이터베이스는 구분되는 데이터 파일을 가지고 다수의 collecions을 가집니다. 하나의 몽고디비에는 다수의 데이터 베이스가 존재할수있습니다.

##WiredTiger Storage Engine

New in version 3.0.

WiredTiger는 선택적으로 64비트의 몽고디비 3.0에서 사용 가능한 엔진 입니다. 이는 복잡한 읽고 쓰는 작업에 탁월한 성능을 보여주고 있습니다.

##Document Level Locking
WiredTiger의 쓰기작업의 경우에는 document 컨택스트 내에서 일어나게 됩니다. 그 결과 다수의 사용자가 하나의 document를 동시에 수정하는 것이 가능합니다.  이는 동시 작업 컨트롤에 매우 훌륭하여 몽고디비로 더욱 효과적으로 읽고 쓰는 작업을 높은 처리량으로 처리할수 있습니다.

##Journal
WiredTiger 는 미리쓰기 트랜젝션 로그와 체트포인트를 조합하여 데이터의 지속성을 보장합니다. WiredTiger는 매 60초마다 체크포인트를 디스크에 저장을 합니다. 또는 매 2기가 의 데이터가 쓰일때마다 저장을 합니다. 각각의 체크포인트 간의 데이터 파일은 항상 유효합니다.

WiredTiger  journal은 각 체크포인트 간의 데이터 변화를 지속적으로 확인합니다. 만약에 체크포인트 사이에 존재한다면 journal을 통해 마지막 체크포인트로 부터 다시 모든 데이터의 수정을 시작합니다. 기본적으로 WiredTiger jounal은 [snappy](http://docs.mongodb.org/manual/reference/glossary/#term-snappy)압축라이브러리를 사용하여 압축되어있습니다.  [storage.wiredTiger.engineConfig.journalCompressor](http://docs.mongodb.org/manual/reference/configuration-options/#storage.wiredTiger.engineConfig.journalCompressor)를 사용하여 다른 압축알고리즘을 사용하거나 압축하지 않을 수도 있습니다.

journal을 과하게 유지하는 것을 줄이기 위해  [storage.journal.enabled](http://docs.mongodb.org/manual/reference/configuration-options/#storage.journal.enabled)를 false로 지정하면 jounaling을 하지 않게 할수도 있습니다.  [standalone](http://docs.mongodb.org/manual/reference/glossary/#term-standalone) 인스턴스는 journal를 사용하지 않습니다. 이는 만약에 몽고디비가 예상치 못하게 체크포인트 사이에 존재하게 된다면 약간의 데이터를 잃을 수도 있음을 의미합니다. [리플리카셋](http://docs.mongodb.org/manual/reference/glossary/#term-replica-set)의 멤버의  복제를 확실하게 보장합니다.

##Compression
몽고디비는 모든 인덱스와 collections의 압축을 WiredTiger를 사용통해서 지원합니다. 압축은 추가적인 CPU의 사용을 최소합 합니다.

기본적으로 WiredTiger는 [snappy](http://docs.mongodb.org/manual/reference/glossary/#term-snappy) compression library을 통해서 모든 collections와 [prefix compression](http://docs.mongodb.org/manual/reference/glossary/#term-prefix-compression)을 모든 인덱스를 블럭단위로 압축을 합니다.

모든 collections에 대해서 [zlib](http://docs.mongodb.org/manual/reference/glossary/#term-zlib)을 통한 블록단위 압축이 가능합니다. 다른 압축알고리즘의 설정과 압축을 사용하지 않는 방법은 [storage.wiredTiger.collectionConfig.blockCompressor](http://docs.mongodb.org/manual/reference/configuration-options/#storage.wiredTiger.collectionConfig.blockCompressor)을 통해서 설정가능합니다.

인덱스의 경우에는 [prefix compression](http://docs.mongodb.org/manual/reference/glossary/#term-prefix-compression)를 사용하지 않기 위해서는 [storage.wiredTiger.indexConfig.prefixCompression](http://docs.mongodb.org/manual/reference/configuration-options/#storage.wiredTiger.indexConfig.prefixCompression)를 통해 설정 가능합니다.

collection을 생성하거나 index를 생성할때 마다 압축을 설정할수 있습니다. 더 자세한 정보는 [Specify Storage Engine Options](http://docs.mongodb.org/manual/reference/method/db.createCollection/#create-collection-storage-engine-options) 과 [db.collection.createIndex() storageEngine option](http://docs.mongodb.org/manual/reference/method/db.collection.createIndex/#createindex-options) 에서 확인하시면 됩니다.

작업의 대부분의 경우에는 기본적인 압축의 세팅은 효과적인 저장공간과 요구사항의 처리의 균형을 맞춰서 설정되어있습니다.

WiredTiger journal 또한 기본적으로 압출이 됩니다. 더많은 정보는 [Journal](http://docs.mongodb.org/manual/core/storage/#storage-wiredtiger-journal)에서 확인하시면 됩니다.

SEE ALSO
[http://wiredtiger.com](http://www.wiredtiger.com/)



##MMAPv1 Storage Engine

MMAPv1은 파일과 메모리의 맵핑을 기반으로 하는 몽고디비의 오리지날 저장 엔진입니다. 이는 큰 볼륨의 생성과 읽기 , 수정작업에 최적화 되어있습니다. MMAPv1는 몽고디비 3.0버전과 그이전의 버전의 기본 엔진으로 설정되어있습니다.

###Journal
몽도디비가 디스크상에서 이뤄지는 모든 수정작업을 보장하기 위해서 모든 수정작업에 대한 정보를 journal에 쓰기 작업보다 더 빈번하게 기록 합니다. jounal은  몽고디비가 [mongod](http://docs.mongodb.org/manual/reference/program/mongod/#bin.mongod) 인스턴스가 모든 변경에 대해서 flushing 하지 않고 종료하는 경우에 모든 데이터를 성공적으로 복수할수 있게 해줍니다.

[Journaling Mechanics](http://docs.mongodb.org/manual/core/journaling/)에 대한 더 많은 정보를 확인 하세요.

##Record Storage Characteristics
모든 기록은 디스크상에서 이뤄집니다. 그리고 document의 크기가 기존에 위치한 곳의 크기 보다 더 커지는 경우에는 몽고디비는 반드시 새로운 곳으로 데이터를 이동시켜야 합니다. 이 경우에는 document의 이동과 모든 인덱스정보의 업데이트가 이뤄지게 됩니다. 이는  당연히 보통의 수정작업보다 더 많은 시간이 걸리게 되고 저장공간이 파편화가 되게 하는 원인이 됩니다.

3.0버전 이후.

기본적으로 몽고디비는 [Power of 2 Sized](http://docs.mongodb.org/manual/core/storage/#power-of-2-allocation)를 사용해서 추가적인 공간이나 [padding](http://docs.mongodb.org/manual/reference/glossary/#term-padding)과 함께 document를 저장합니다. padding은 수정을 통한 document의 사이즈가 커져서 새로운 위치로 이동되게 하는 일을 최소화 시킬수 있습니다.

##Record Allocation Strategies
MongoDB가 레코드를 생성 할 때 [mongod](http://docs.mongodb.org/manual/reference/program/mongod/#bin.mongod)가  document에 어떻게 padding을 추가할것인지에 대한 여러 방법들이 있습니다. 왜냐하면 몽고디비의 document는 생성이후에 크기가 커질수 있고 모든 저장은 디스크상에 연속적으로 위치하기 때문에 padding은 document의 수정에 따른 위치변경의 경우를 줄일수 있습니다. 위치의 변경은 보통의 수정보다 덜 효율적이고 저장공간의 파편화 시킵니다. 그 결과 padding을 추가하는 방법은 효율성을 증대 시키고 반대로 파편화를 감소시킵니다.

각기 다른 작업의 성격에 따라 각기다른 방법을 제공합니다.  [power of 2 allocations](http://docs.mongodb.org/manual/core/storage/#power-of-2-allocation)는 생성, 수정, 삭제작업에 더 효율적이지만 수정이나 삭제작업이 없이 처음사이즈 그대로 [위치(exact fit allocations)](http://docs.mongodb.org/manual/core/storage/#exact-fit-allocation)하고 있는것이 collection에게는 가장 이상적입니다.

###Power of 2 Sized Allocations
3.0.0버전 이후
Changed in version 3.0.0.

몽고디비는 power of 2 sizes 를  MMAPv1의 기본저장 방법으로 사용하고 있습니다. power of 2 sizes 는 저장공간이 2의 제곱근의 크기로 커집니다. (e.g. 32, 64, 128, 256, 512 ... 2MB). document의 크기가 2MB보다 클경우에 저장 공간은 2의 제곱근에 의 근사값을 가지게 됩니다.

power of 2 sizes를 이용한 방법에는 다음의 주요 속성을 가집니다.

- 효율적으로 새롭게 빈공간이 된 공간을 재사용하고 저장공간의 파편화를 감소시킵니다. 저장되는 공간을 정량화 하여 고정된 크기의 공간에 데이터가 위치하게 되면 생성하기 이전에 삭제되거나 재배치된 공간에 새롭게 생성될 document의 사이즈가 적절할 확률이 증가합니다.
- documents의 이동을 감소합니다. padding공간을 추가 함으로 움직이지 움직일 필요는 없을 정도 까지만 document 사이즈의 크기가 늘어날수 있습니다. 이런 documents의 이동에 드는 비용을 줄이면서 인덱스를 다시 업데이트 할필요가 없게 됩니다. 비록 power of 2 sizes 는 이동을 최소화 시킬수는 있지만 아예 없애지는 못합니다.

###No Padding Allocation Strategy
3.0.0버전 이후

collection의 document의 사이즈가 변화가 없는 경우, 예를 들면 생성만 계속 발생한다던지 업데이트가 발생은 하지만 사이즈의 변화가 일어나지 않는다던지(카운터 증가 같은 작업) 하는 경우에는 [power of 2 sizes](http://docs.mongodb.org/manual/core/storage/#power-of-2-allocation) 를 사용하지 않고 [collMod](http://docs.mongodb.org/manual/reference/command/collMod/#dbcmd.collMod)의 [noPadding](http://docs.mongodb.org/manual/reference/command/collMod/#noPadding) 플래그를 사용하거나  [db.createCollection()](http://docs.mongodb.org/manual/reference/method/db.createCollection/#db.createCollection)의 [noPadding](http://docs.mongodb.org/manual/reference/command/collStats/#collStats.paddingFactor) 옵션을 사용하여 padding을 주지않는 작업을 할수가 있습니다.

3.0.0 이전 버전의 몽고디비에서는 documernt크기에 따라 동적으로 [padding](http://docs.mongodb.org/manual/reference/command/collStats/#collStats.paddingFactor)을 계산해서 document의 위치를 잡았습니다.

##Capped Collections

[Capped collections](http://docs.mongodb.org/manual/reference/glossary/#term-capped-collection)는 사이즈가 고정된 collection으로 생성(insert) 명령에 높은 성능을 보여줍니다. Capped collections을 circular buffers처럼 동작을 합니다. 현재 가진 사이즈에 데이터가 모두 차게되면 collection상에서 가장 오래된 documents에 새로운 documents가 덧 쓰이게 됩니다.

[Capped Collections](http://docs.mongodb.org/manual/core/capped-collections/)에 대한 더많은 정보를 확인하세요,



[목록](https://github.com/yuby/mongodb-ko)