[목록](https://github.com/yuby/mongodb-ko)


#Distributed Write Operations

##Write Operations on Sharded Clusters

[sharded cluster](http://docs.mongodb.org/manual/reference/glossary/#term-sharded-cluster)환경 에서 sharded collections은 [mongos](http://docs.mongodb.org/manual/reference/program/mongos/#bin.mongos)의 쓰기 작업은 어플리케이션에서 부터 데이터 셋의 특정 부분을 대상으로 이뤄집니다. [mongos](http://docs.mongodb.org/manual/reference/program/mongos/#bin.mongos)는 cluster의 메타데이터를 [config 데이터베이스](http://docs.mongodb.org/manual/core/sharded-cluster-config-servers/#sharding-config-server)로 부터 가져와 적절한 shards에 쓰기 작업을 진행합니다.

![Diagram of a sharded cluster.](http://docs.mongodb.org/manual/_images/sharded-cluster.png)

sharded collection에 나누어지 데이터들은 기본적으로 [shard key](http://docs.mongodb.org/manual/reference/glossary/#term-shard-key) 를 기반]으로 나누어져 있습니다. 그리고 나서 몽고디비는 이 구분된 shards 덩어리를 배포하게 됩니다. shard key는 배포된 shards 덩어리를 구분짓는 역할을 합니다. 이는 클러스트에서 쓰기 동작의 성능에 영향을 줍니다.

![Diagram of the shard key value space segmented into smaller ranges or chunks.](http://docs.mongodb.org/manual/_images/sharding-range-based.png)

>IMPORTANT
하나의 document를 대상으로 수정작업을 하는 경우에는 반드시 [shard key](http://docs.mongodb.org/manual/reference/glossary/#term-shard-key) 나 _id 필드 값이 존재해야 합니다. [shard key](http://docs.mongodb.org/manual/reference/glossary/#term-shard-key)가 있는 경우에 다중 documents를 대상으로 수정작업을 하는 경우 훨씬 효과적으로 이뤄집니다. 하지만 모든 shards를 대상이 될수도 있습니다.

shard key가 매번 생성시마다 증가하거나 감소하는 경우 모든 생성작업의 경우에는 하나의 shard를 대상으로 이뤄집니다. 그 결과 해당 shard의 용량에 제한이 걸리게 됩니다.

더 많은 정보는 [Sharded Cluster Tutorials](http://docs.mongodb.org/manual/administration/sharded-clusters/) 과 [Bulk Write Operations](http://docs.mongodb.org/manual/core/bulk-write-operations/#write-operations-bulk-insert) 에서 확인 하시면 됩니다.

##Write Operations on Replica Sets

[리플리카 셋](http://docs.mongodb.org/manual/reference/glossary/#term-replica-set)에서 primary’s operation log 나 oplog를 기록 등 모든 쓰기 작업은 [primary](http://docs.mongodb.org/manual/reference/glossary/#term-primary)에서 진행이됩니다. 합니다. [oplog](http://docs.mongodb.org/manual/reference/glossary/#term-oplog)는 데이터 세트에 대한 복사가능한 순차적인 동작 정보입니다. [Secondary](http://docs.mongodb.org/manual/reference/glossary/#term-secondary) 멤버들은 이 oplog를 가지고 비동기적으로 primary에서 발생하는 모든 작업들을 지속적으로 자신에게 실행시키고 있습니다.

[Diagram of default routing of reads and writes to the primary.](http://docs.mongodb.org/manual/_images/replica-set-read-write-operations-primary.png)

bulk 작업 같은 사이즈 가 큰 쓰기 작업의 경우에는 Secondary 멤버가 지속적으로 replicating 작업을 충분하게 할 여력을 가지기 어렵습니다. 이는 Secondary 멤버가 Primary보다 뒤처질수 밖에 없는 원인이 됩니다. Primary에 비해 지나치게 뒤처진 Secondary는 리플리카 셋의 보통의 작업에서도 문제를 일으킵니다. 특히 데이터를 [일관성 있게 읽는것](http://docs.mongodb.org/manual/applications/replication/)은 물론이고 [롤백](http://docs.mongodb.org/manual/core/replica-set-rollbacks/#replica-set-rollback)에 대한 [failover](http://docs.mongodb.org/manual/core/replica-set-high-availability/#replica-set-failover-administration)등에 문제가 발생할합니다.

이러한 상황을 피하기 위해서 [write concern](http://docs.mongodb.org/manual/core/write-concern/#write-operations-write-concern)을 커스터 마이징 하여 리플리카 셋의 다른 맴버들의 100 번  또는 1000번의 작업마다  내용을 리턴하도록 만들수 있습니다. 이러한 기능은 Secondary가 Primary를 따라 잡을수 있는 기회를 제공합니다. Write concern은 전체적인  쓰기 작업을 늦출수 있지만  Secondary가 Primary의 현재상태를 유지할수 있도록 보장합니다.

[Write operation to a replica set with write concern level of ``w:2`` or write to the primary and at least one secondary.](http://docs.mongodb.org/manual/_images/crud-write-concern-w2.png)

리플리카 겟과 쓰기 작업에 대한 더 많은 정보는 [Replica Acknowledged](http://docs.mongodb.org/manual/core/write-concern/#replica-set-write-concern), [Oplog Size](http://docs.mongodb.org/manual/core/replica-set-oplog/#replica-set-oplog-sizing),그리고 [Change the Size of the Oplog](http://docs.mongodb.org/manual/tutorial/change-oplog-size/)에서 확인하세요.


[목록](https://github.com/yuby/mongodb-ko)