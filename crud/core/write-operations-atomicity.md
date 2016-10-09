#Atomicity and Transactions

몽고디비에서 쓰기작업은 하나의 document에 대해서는 원자성을 보장합니다. 심지어 document에 임베디드된 document의 수정인 경우에도 원자성을 보장합니다.

하나의 쓰기작업으로 다수의 document를 수정하는 경우 각각의 document는 원자성이 보장되지만, 전체 쓰기동작 자체에 대한 원자성이 보장되지는 않습니다. 쓰기작업으로 여러 document를 수정하는 과정에 다른 작업이 끼어들수도 있습니다. 하나의 쓰기작업에 대해서 원자성을 보장하기 위해서 $isolated 연산자를 사용하면 작업단위로 원자성이 보장됩니다.


##$isolated Operator

$isolated 연산자를 사용하면 쓰기작업의 "all-or-nothing"의 원자성을 제공하지 않습니다. 만약에 쓰기작업중에 에러가 발생을하면 쓰기작업 이전의 상태로 롤백을 하지 않게 됩니다.

>NOTE:
>$isolate 연산자는 쓰기작업이 collection에 lock을 걸어버립니다. 심지어 document단위의 lock을 거는 WiredTiger 엔진을 사용해도 마찬가지 입니다. $isolated 연산자가 WiredTiger를 작업중에는 싱글쓰레드로 만듭니다.

$isolated는 shared 클러스터 상에서는 동작하지 않습니다.


##Transaction-Like Semantics

하나의 document가 다수의 임베디드된 document를 가지는 경우 해당 document의 원자성을 보장하는 것이 여러 케이스를 대상으로 충분합니다. 예를들어 마치 하나의 트랜잭션처럼 단계적으로 쓰기작업이 이뤄저야하는 경우  two-phase commit 을 구현하여 이를 해결할수 있습니다.

하지만  two-phase commit 은 단지 트랜잭션 처럼 보이게 하는 역활을 할뿐입니다.  two-phase commit의 사용은 데이터의 일관성을 보장하지만 데이터가 two-phase commit를 진행하는 중이다 롤백을 진행하는 중간데이터를 반환하는 경우도 발생할 가능성이 있습니다.


##Concurrency Control

동시 실행제어(Concurrency Control)는 다수의 어플리케이션이 데이터의 충돌이나 일관관성을 해치지 않고 동작할수 있습니다.

첫번쨰 방법이 유니크한 index를 생성하고 유니크한 값을 가지게 하는 방법입니다. 이는 중복 생성이나 수정을 예방할수있습니다. 유니크한 인덱스를 다수의 필드에 부여하면 강제로 필드와 값을 유니크하게 만들수 있습니다.

다른 방법으로는 쓰기작업의 쿼리조건에 해당하는 필드의 값을 지정하는 것입니다. 
two-phase commit 은 쿼리가 예측하는 어플리케이션 인식자뿐만 아니라 쓰기작업에서의 예측되는 데이터 상태등의 변경을 규정합니다.(The two-phase commit pattern provides a variation where the query predicate includes the application identifier as well as the expected state of the data in the write operation.)
