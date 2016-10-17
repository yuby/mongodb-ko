#Distributed Queries

##Read Operations to Sharded Clusters

sharded 클러스터는 데이터를  mongod인스턴스중의 하나의 클러스트로 데이터를 분산시킵니다.

shared된 클러스트에서 어플리케이션은 monogs인스턴스의 하나의 cluster를 대상으로 동작을 합니다.

![sharded-cluster](https://docs.mongodb.com/manual/_images/sharded-cluster.png)

sharded된 클러스트 상에서의 읽기 작업을 하는 경우 가장 효율적으로 데이터를 불러오는 방법은 특정 shard로 바로접근하는 것입니다. 쿼리가 sharded된 colleciton을 대상으로 동작하는 경우 collection의 shard key를 가진 경우입니다. 쿼리가 shard key를 가지는 경우, mongos는 config database의 도움으로 대상이 되는 shard를 대상으로 바로 접근하도록 하여 클러스터의 meta데이터를 가져옵니다.

![sharded-cluster-targeted-query](https://docs.mongodb.com/manual/_images/sharded-cluster-targeted-query.png)

만약에 쿼리가 shard key를 가지지 못하는 경우에는 mongos가 클러스터의 모든 shard를 대상으로 쿼리를 동작시키빈다. 이렇게 쿼릴를 전체를 대상으로 동작을 하는 것은 매우 비효율적입니다. 클러스터의 크기가 큰 경우에는 전체를 대상으로 하는 쿼리의 경우 일반적인 쿼리작업을 하지 못할수도 있습니다.

![sharded-cluster-scatter-gather-query](https://docs.mongodb.com/manual/_images/sharded-cluster-scatter-gather-query.png) 


replica set의 primary가 아닌 다른 set을 대상으로 읽기작업을 하는 경우 현재 primary의 상태가 반영되지 않았을 경우도 있고 이는 원하지 않았던 결과가 리턴될수도 있습니다.

##Read Operations to Replica Sets

기본적으로 클라이언트는 replica set에서의 primary를 대상으로 읽기작업을 하지만, 클라이언트는 다른 set을 대상으로 읽기작업이 진행하도록 하는 것도 가능합니다.
예를들면 클라이언트는 읽기작업의 대상을 설정할때 두번째 우선순위의 set을 대상으로 동작을 하거나 또는 다음과 가장 가까운 set을 대상으로 동작합니다.


- multi-data-center 로의 배포의 대기시간이 가장적은 경우
- 읽기작업의 결과가 high read-volumes의 분산을 통해 향상된 경우
- 백업작업이 동작하는 경우
- 새로운 primary가 선택되기 이전에 읽기작업이 허용된 set의 경우


![replica-set-read-preference](https://docs.mongodb.com/manual/_images/replica-set-read-preference.png)


읽기작업이 secondary를 대상으로 이뤄진다면 최신상태의 데이터 상태를 반영하지 못할수도 있고 이는 원하지 않았던 결과를 리턴받게 될수있습니다.

그래서 기본적으로 읽기작업에 대한 설정을 연결때마다, 작업때마다를 기준으로 설정할수있습니다.