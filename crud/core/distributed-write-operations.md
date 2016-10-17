#Distributed  Write Operations

##Write Operations on Sharded Clusters

sharded된 클러스터에서 sharded된 collection의 경우 mongos는 어플리케이션에서 데이터의 특정 부분을 담당하는 shard를 대상으로 바로 이뤄집니다. 이를 위해 mongos는 config database의  route데이터를 기반으로 적절한 shard를 대상으로 동작합니다.

![sharded-cluster](https://docs.mongodb.com/manual/_images/sharded-cluster.png)

몽도디비는 sharded된 collection을 shard key를 기반으로 위치시킵니다. 그리고 몽고디비는 데이터를 key기반으로 덩어리째 분산시킵니다. 여기서 shard key의 경우에는 데이터 덩어리의 유니크한 키 역활을 합니다. 이러한 방법은 클러스터 상에서 쓰기작업의 성능에 영향을 미칩니다.

![sharding-range-based](https://docs.mongodb.com/manual/_images/sharding-range-based.png)


>IMPORTANT
>하나의 document를 대상으로 이뤄지는 수정작업의 경우에는 shard key나 _id필드가 반드시 포함되어 있어야합니다. 수정작업이 다수의 document를 대상으로 동작하는 경우가 더욱 효과적인 경우가 있는데, 예를들면 모든 shard를 대상으로 shard key가 존재하는 경우입니다.


만약에 하나의 shard를 대상으로 shard key의 값이 매번 쓰기작업때마다 증가하는 경우에는 허용된 키값의 범위를 초과하게 되어 해당 shard의 용량의 제한이 걸리게 됩니다.

##Write Operations on Replica Sets

replica set에서 모든 쓰기작업은 set의 primary를 대상으로 이뤄집니다. primary에서 쓰기작업이 이뤄지면 primary의 동작 log나 oplog에 정보가 저장됩니다. oplog의 경우에는 연속적인 작업을 진행할때 재사용가능합니다. secondary 멤버는 지속적으로 oplog를 복사하고 이를 기반으로 동기화작업을 진행합니다.

![replica-set-read-write-operations-primary](https://docs.mongodb.com/manual/_images/replica-set-read-write-operations-primary.png)