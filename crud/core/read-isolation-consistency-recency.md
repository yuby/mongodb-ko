#Read Isolation, Consistency, and Recency

##Isolation Guarantees

###Read Uncommitted

몽고디비에서는 쓰기작업이 완료도기 이전에 데이터의 쓰기작업의 결과를 확인 할 수 있습니다.

- write concern에 상관없이 다른 클라이언트가 readConcern을 local 로 사용하는 경우 쓰기작업의 결과를 쓰기작업에 대한 결과를 클라이언트에 알려주기 이전에 알수가 있습니다.
- 클라이언트가 readConcern을 local로 작성하는 경우에 나중에 롤백이 될수도 있는 데이터도 읽을 수가 있습니다.

Read uncommitted 는 기본 isolation의 레벨이며 이는 mongod가 하나만 도는 경우는 물론이고 replica set이나 sharded되 cluster에서도 지원합니다.

###Read Uncommitted And Single Document Atomicity
읽기작업은 하나의 document를 대상으로는 원자성이 보장됩니다. 즉, document의 여러 필드를 수정하는 경우에, 해당 document를 읽는 사용자의 경우에는 각 필드들이 변경되고 있는 과정을 확인할수 는 없습니다.

하나의 mongod 인스턴스가 돌고 있는경우에 하나의 document를 대상으로 읽기와 쓰기는 순차적(serializable)입니다. 
replica set에서도 하나의 문서에 대해서 읽기와 쓰기 작업의 경우 롤백이 없는 경우에 한해서는 순차적(serializable)입니다.

비록 데이터를 읽는 사람이 데이터의 여러 필드의 변경에 대해서 각 필드가 업데이트 되는 순간을 볼수는 없지만  read uncommitted 이 의미하는 바는 실제 쓰기작업으로 데이터가 변경이 반영되기 이전에 변경된 데이터를 볼 수 있습니다.

###Read Uncommitted And Multiple Document Write
한번의 쓰기 작업으로 다수의 document를 수정하는 경우 각각의 수정에 대해서는 원자성이 보장이 됩니다. 하지만 전체 작업으로 봤을때는 원자성이 보장이 되지 않습니다. 다른 작업이 중간에 끼어들수가 있습니다. 하지만 하나의 작업에 대해서 원자성을 보장하기 위해서 $isolate 연산자를 사용하면 쓰기작업 단위로 원자성을 보장할수있습니다.

만악에 다수의 document를 대상으로 쓰기작업을 할때 별도로 isolate를 하지 않으면 다음의 행동을 보입니다.

1. 다수의 document의 읽기작업이 특정 시간 t1에 시작이되고, 그 중의 하나의 document를 대상으로 쓰기작업이 약간의 시간이 흐른다음 t2에 시작한다면, 읽기작업을 하는 사람은 수정된 document를 보게 됩니다. 그러므로 특정 시간대의 데이터의 스냅샷을 볼수는 없습니다.
2. document d1을 t1에 읽고 d1을 t3이 된시점에 수정을 한다 가정하겠습니다. 이는 읽기-쓰기 라는 순서로 동작의 의존성이 존재합니다. 예를들면 이 동작이 순차적으로 발생을 한다면 읽기작업이 쓰기작업에 우선되어 실행이 되야합니다. 하지만 위 작업의 중간에 d2에 대한 쓰기작업과 이어서 t4가 된 시점에 d2를 읽는 쓰기-읽기라는 순서의 의존성을 가진 동작이 발생하고 이또한 순차적인 스케줄에 따라 쓰기-읽기 순서에 따르게 됩니다. 이 각각의 의존된 동작의 경우에는 순차적으로 발생을 하지못합니다.
>역주:
>예를들어서 작업을 도표화 시켜보면
| Set1 | Set2 | Set1 | Set2 |
|---|---|---|---|
| t1 | t2 | t3 | t4 |
| D1 | D2 | D1 | D2 |
| Read | Update | Update | Read | 
> Se11의 작업 사이에 Set2가 중간에 끼게 됩니다. 즉, Set1이 순차적으로 작업을 완료할수가 없데 됨을 의미합니다.

3. 읽기작업의 도중에 수정이 이뤄진 document를 잘못 불러오는 경우가 발생할수 있습니다.

$isolated 연산자를 사용하면 다수의 document를 쓰기작업하는 경우 다른 프로세스가 중간에 끼어들게 하지 못하게 합니다. 이는 쓰기작업이 완전히 끝이 나기거나 에러가 발생하기 전까지는 클라이언트가 아무르너 변화를 볼수가 없습니다.

$isolated 연산자는 shared cluster상에서는 동작하지 않습니다.
$isolated 연산자는 "all-or-nothing"의 원자성을 보장하지 않습니다. 동작중에 에러가 발생을 하면 전체를 롤백을 하지않습니다.

>NOTE
>$isolated 연산자는 collection단위의 락을 걸게 됩니다. WiredTiger 엔진을 사용하면 document단위로 락을 걸지만 $isolated 연산자를 사용하면 collection단위의 락을 겁니다. WiredTiger의 동작을 single-thread 로 동작하게 합니다.
