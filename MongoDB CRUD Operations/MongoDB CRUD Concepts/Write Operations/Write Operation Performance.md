[목록](https://github.com/yuby/mongodb-ko)


#Write Operation Performance

##Indexes

매번 생성, 수정, 삭제 작업이 끝이나면 몽고디비는 반드시 collection과 연관된 모든 인덱스를 수정합니다. 그러므로  쓰기 작업에 대한 약간의 추가적인  퍼포먼스 과 부하가 걸리게 됩니다.

일반적으로 인덱스가 제공하는 읽기작업에서의 이점은 생성작업에서는 페널티로 작용합니다. 하지만 쓰기작업의 성능을 최적화 하기 위해서는 새로운 인덱스를 생성하는데 주의 해야 하고 이미 존재하는 인덱스가 실제로 쿼리상에서 의미가 있는지를 확인해야 합니다.

쿼리와 인덱스의 정보는 [Query Optimization](http://docs.mongodb.org/manual/core/query-optimization/) 에서 확인 하시고 인덱스에 대한 더많은 정보는 [Indexes](http://docs.mongodb.org/manual/indexes/) 와 [Indexing Strategies](http://docs.mongodb.org/manual/applications/indexes/)에서 확인 하시면 됩니다.

##Document Growth and the MMAPv1 Storage Engine

수정작업으로 document에 새로운 필드가 추가 된다면 document의 사이즈가 증가 하게 됩니다.


MMAPv1저장엔진에서 만약에 수정 작업을 통해 현재 document의 [크기](http://docs.mongodb.org/manual/reference/glossary/#term-record-size)를 넘어 서게 된다면 몽고디비는 충분한 공간으로 디스크상에 새롭게 document의 위치를 잡게 됩니다. 위치를 다시 잡아야 하는 수정작업은 그렇지 않은 수정 작업에 비해서 더 많은 작업시간이 소요가 됩니다. 특히 collection에 인덱스가 포함된 경우 그렇습니다. 만약에 collection이 인덱스를 가지고 있다면 몽고디비는 반드시 전체 인덱스의 정보를 업데이트 시켜야 합니다. 그러므로 collection에 많은 인덱스가 잡혀있는 경우라면 쓰는 작업에 영향을 주게 됩니다.

3.0.0 버전 이후 :  기본적으로 몽고디비는 [2 Sized Allocations](http://docs.mongodb.org/manual/core/storage/#power-of-2-allocation)로 자동으로 [추가적인 공간](http://docs.mongodb.org/manual/core/storage/#record-allocation-strategies)을 가지고 있습니다. [2 Sized Allocations](http://docs.mongodb.org/manual/core/storage/#power-of-2-allocation)를 사용함으로써 몽고디비는 효과적으로 수정및 작제작업시 공간을 사용하게 되고 document의 위치를 다시 잡는 많은 경우를 줄여 줍니다.

비록 [2 Sized Allocations](http://docs.mongodb.org/manual/core/storage/#power-of-2-allocation) 가 재배열이 발생한는 경우를 줄여주지만 완전히 제거하지는 못합니다.

더많은 [Storage](http://docs.mongodb.org/manual/core/storage/)에 대한 정보를 확인 하세요.


##Storage Performance

###Hardware
storage 시스템의 성능은 몽고디비의 쓰기작업의 성능에 중요한 물리적인 한계를 만듭니다.  만은 유니크한 요소들이 storage상에서 관례를 가지고 쓰기작업이나 access patterns, disk caches, disk readahead , RAID configurations에 영향을 줍니다.

SSD의 경우에는 HDD보다 100배또는 그이상의 성능을 보여줍니다.


SEE
[Production Notes ](http://docs.mongodb.org/manual/administration/production-notes/)추가적인 하드웨어에 대한 권장사항이나 설정사항을 확인할수 있습니다.



###Journaling
몽고디비는 [쓰기작업](http://docs.mongodb.org/manual/core/write-operations/)을 보장하기 위해서 디스크상의 [journal](http://docs.mongodb.org/manual/reference/glossary/#term-journal)에 기록을 남기고 문제가 발생시에 이 정보를 제공합니다. 몽고디비의 데이터가 변경되기 이전에 몽고디비는 우선 journal의 정보를 변경하는 작업을 합니다.

journal이 동작에 대한 확실정을 보장하지만 반면에 journal은 쓰기작업에 추가적인 작업에 대한 부담을 안깁니다. 그래서 다음의 journal 과 성능상에 관계에 대해서 고려 할 필요가 있습니다.

- journal과 데이터가 동일한 디바이스 블럭에 위치한다면 데이터파일들과 journal 이  제한된 쓰기작업의 횟수를 가지고 싸우게 됩니다. journal을 다른 디바이스로 이동을 시켜서 쓰기작업에 성능을 향상시킬수가 있습니다.
- 어플리케이션이 [journaled](http://docs.mongodb.org/manual/core/write-concern/#write-concern-replica-journaled), [mongod](http://docs.mongodb.org/manual/reference/program/mongod/#bin.mongod) 를 포함해서 [write concern](http://docs.mongodb.org/manual/core/write-concern/)을 지정한다면 journal의 커밋의 시간을 감소시켜 전체적인 쓰기성능을 증가시킬수 있습니다.
- commitIntervalMs을 run-time 옵션으로 사용한다면 journal에 커밋되는 간격을 조절할수 있습니다. journal에 커밋의 간격을 줄인다면 쓰기의 횟수가 한정적인 몽고디비에서는 쓰기작업을 할 수 있는  횟수 가 증가하게 됩니다.   반대로 journal에 커밋되는 횟수가 증가하면 전체적인 쓰기작업의 횟수가 감소합니다. 하지만 journal에 커밋되는 횟수가 줄어든다는 말은 만약에 작업이 실패하는 경우 그 작업에 대한 기록을 하지 않을 횟수가 상대적으로 증가 하게 됨을 의미합니다.

추가적인 정보는 [Journaling Mechanics](http://docs.mongodb.org/manual/core/journaling/)에서 확인하시면 됩니다.



[목록](https://github.com/yuby/mongodb-ko)