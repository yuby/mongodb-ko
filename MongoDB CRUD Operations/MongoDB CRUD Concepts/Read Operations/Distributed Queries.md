[목록](https://github.com/yuby/mongodb-ko)


#Distributed Queries

##Read Operations to Sharded Clusters

[Sharded 클러스트](http://docs.mongodb.org/manual/reference/glossary/#term-sharded-cluster)는 어플리케이션에 거의 투명한 방법으로 [mongod](http://docs.mongodb.org/manual/reference/program/mongod/#bin.mongod)인스턴스의 클러스트 사이의 데이터 세트를 분할 할 수 있습니다. sharded클러스트에 대한 개략적인 내용은 [Sharding](http://docs.mongodb.org/manual/sharding/)섹션에서 확인 할 수 있습니다.

sharded클러스트에 대한 어플리케이션의 [mongos](http://docs.mongodb.org/manual/reference/program/mongos/#bin.mongos)인스턴스와 연관된 클러스트에 대한 다이어그램입니

![Diagram of a sharded cluster.](http://docs.mongodb.org/manual/_images/sharded-cluster.png)

sharded클러스트에 대한 읽기 작업에서는 특정 shard를 대상으로 작업을 하는 것이 가장 효과적입니다. sharded collections에 대한 쿼리는 반드시 collection의 [샤드키](http://docs.mongodb.org/manual/core/sharding-shard-key/#sharding-shard-key)를 포함하고 있어야 합니다. 쿼리가 샤드키를 가지고 있으면 [mongos](http://docs.mongodb.org/manual/reference/program/mongos/#bin.mongos)는 [config 데이터 베이스](http://docs.mongodb.org/manual/core/sharded-cluster-config-servers/#sharding-config-server)로 부터 클러스트에 대한 메타정보를 사용하여 특정 샤드에 대한 쿼리를 실행할수가 있습니다.

![Read operations to a sharded cluster. Query criteria includes the shard key. The query router ``mongos`` can target the query to the appropriate shard or shards.](http://docs.mongodb.org/manual/_images/sharded-cluster-targeted-query.png)

만약에 쿼리가 샤드키를 가지고 있지 않다면 [mongos](http://docs.mongodb.org/manual/reference/program/mongos/#bin.mongos)는 모든 cluster의 모든 shard를 대상으로 쿼리를 하게 됩니다. 이런 분산된 쿼리는 매우 비효율적입니다. 사이즈가 큰 클러스터의 경우에는 분산된 쿼리는 일상적으로 실행할 수 없는 작업입니다.

![Read operations to a sharded cluster. Query criteria does not include the shard key. The query router ``mongos`` must broadcast query to all shards for the collection.](http://docs.mongodb.org/manual/_images/sharded-cluster-scatter-gather-query.png)

sharded클러스트에서의 더 많은 읽기 작업에 대한 정보는 [Sharded Cluster Query Routing](http://docs.mongodb.org/manual/core/sharded-cluster-query-router/) 과 [Sharded key](http://docs.mongodb.org/manual/core/sharding-shard-key/#sharding-shard-key)에서 확인 하세요.


##Read Operations to Replica Sets

[리플리카 셋](http://docs.mongodb.org/manual/reference/glossary/#term-replica-set)은 [우선 읽기(read preferences)](http://docs.mongodb.org/manual/reference/glossary/#term-read-preference) 를 사용해서 어디에서 어떻게 리플리카 셋의 멤버에 데이터를 읽을지를 결정합니다. 기본적으로 몽고디비는 항상 리플리카 셋의 [primary](http://docs.mongodb.org/manual/reference/glossary/#term-primary)로 부터 데이터를 읽어 옵니다. 그리고 이런 우선권의 경우 [read preference mode](http://docs.mongodb.org/manual/reference/read-preference/#replica-set-read-preference-modes)를 변경하여 얼마든지 수정 가능합니다.

우리는 [read preference mode](http://docs.mongodb.org/manual/reference/read-preference/#replica-set-read-preference-modes)를 설정하여 primary가 아닌 [secondaries](http://docs.mongodb.org/manual/reference/glossary/#term-secondary)에서 각 연결마다 또는 각 동작마다 읽기 작업을 하도록 하는 것이 가능합니다.

- 다중 데이터 센터에서 대기시간을 줄인다.
- 높은 읽기 볼륨의 배포하여 읽기 동작의 처리량을 향상한다.
- 작업들의 백업역할을 한다.
- primary에서 [실패](http://docs.mongodb.org/manual/core/replica-set-high-availability/#replica-set-failover)가 발생하는 경우 읽기작업이 가능하도록 한다.

![Read operations to a replica set. Default read preference routes the read to the primary. Read preference of ``nearest`` routes the read to the nearest member.](http://docs.mongodb.org/manual/_images/replica-set-read-preference.png)

리플리카 셋의 Secondary 멤버에서의 읽기 작업은 primary의 현재 상태를 100% 보장하지는 못합니다. 그리고 secondary가 primary의 상태를 따라가는데는 약간의 시간이 필요합니다.

read preference에 대한 더 많은 정보는 [Read Preference](http://docs.mongodb.org/manual/core/read-preference/) 와 [Read Preference Modes](http://docs.mongodb.org/manual/reference/read-preference/#replica-set-read-preference-modes) 을 보시면 됩니다.


[목록](https://github.com/yuby/mongodb-ko)