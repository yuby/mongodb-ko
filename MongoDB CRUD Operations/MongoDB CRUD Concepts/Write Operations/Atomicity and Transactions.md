[목록](https://github.com/yuby/mongodb-ko)


#Atomicity and Transactions

몽고디비는 하나의 document에 대해서는 원자성이 모장이 됩니다. 그리고 그 document에 포함된 여러 documents를 수정하더라도 마찬가지입니다.

하나의 수정 작업을 통해 다수의 documents를 수정하는 경우 수정에 대한 각각의 document는 원자성이 보장이 됩니다. 하지만 전체를 대상으로 동작을 하는 경우는 원자성이 보장되지 않아 중간에 다른 작업이 방해를 할수도 있습니다. 하지만 각 작업을 [$isolated](http://docs.mongodb.org/manual/reference/operator/update/isolated/#up._S_isolated) 를 통해 isolate 시키게 되면 다수의 documents에도 원자성이 보장이 됩니다.

##$isolated Operator

[$isolated](http://docs.mongodb.org/manual/reference/operator/update/isolated/#up._S_isolated) 사용하면 다수의 documents를 대상으로 하는 쓰기 작압에 먼저 수정이 된 document가 다른 프로세스로 부터 간섭 받는 일을 예방할 수 있습니다. 이는 클리아언트가 쓰기 작업이 모두 마치기거나 에러가 발생하기 전에는 변경에 대한 내용을 볼수가 없습니다.

Isolated 쓰기 작업의 경우에는 'all - or - nothing'이 제공되지 않습니다. 즉, 쓰기 작업중에 에러가 발생하면 이전에 변경된 내용이 모두 그 이전 상태로 돌아가지 않는다는 말입니다.

[$isolated](http://docs.mongodb.org/manual/reference/operator/update/isolated/#up._S_isolated) 는 sharded cluster 환경에서는 동작하지 않습니다.

[$isolated](http://docs.mongodb.org/manual/reference/operator/update/isolated/#up._S_isolated) 를 사용한 수정 작업의 예를 확인해보시기 바랍니다. 예제는 [$isolated](http://docs.mongodb.org/manual/reference/operator/update/isolated/#up._S_isolated) 사용해 제거하는 작업입니다. 더많은 정보를 [Isolate Remove Operations](http://docs.mongodb.org/manual/reference/method/db.collection.remove/#isolate-remove-operations)확인해 보세요.

##Transaction-Like Semantics

하나의 document가 내부에  여러개의 documents를 가질수 있기 때문에 하나의 document에 대한 원자성은 다양한 경우에 대해 매우 필수적인 요소입니다.  순차적인 쓰기 작업의 경우에는 동작이 하나의 트랜젝션으로 동작하듯이 해야 하기 대문에 어플리케이션은 [two-phase 커밋](http://docs.mongodb.org/manual/tutorial/perform-two-phase-commits/)의 방식으로 적용되어야 합니다.

However, two-phase commits can only offer transaction-like semantics.  two-phase커밋은 데이터의 일관성을 보장하지만 어플리케이션이 two-phase 커밋이나 롤백을 하는 도중에 데이터를 즉시 리턴하게 하는 것도 가능합니다.
[Perform Two Phase Commits](http://docs.mongodb.org/manual/tutorial/perform-two-phase-commits/)에 대한 더많은 정보를 확인 하세요.

##Concurrency Control

Concurrency control은 다수의 어플리케이션이 데이터의 충돌이나 불일치가 없이 동작하게 해줍니다.

필드에 [unique index](http://docs.mongodb.org/manual/core/index-unique/#index-type-unique)를 생성하여 필드의 값이 유일한 값을 가지게 해서 중복의 데이터 생성이데 데이터 수정을 예방 시켜줍니다. 더 많은 정보는   [update() 와 Unique Index](http://docs.mongodb.org/manual/reference/method/db.collection.update/#update-with-unique-indexes) , [findAndModify() Unique Index](http://docs.mongodb.org/manual/reference/method/db.collection.findAndModify/#upsert-and-unique-index) 에서 확인하시면 됩니다.

다른 접근 법은 쓰기 작업을 할때 쿼리가 예상하는 현재 필드가 가질만한  값을 설정하는 것입니다. 더 자세한 정보는  [Update if Current](http://docs.mongodb.org/manual/tutorial/update-if-current/) 에서 확인하시면 됩니다.

two-phase 패턴은 쓰기 작업에서 다양성을 제공하는데 쿼리의 조건이 [application identifuer](http://docs.mongodb.org/manual/tutorial/perform-two-phase-commits/#phase-commits-concurrency)를 포함하는지 뿐만 아니라  쓰기 작업에서 예상되는 데이터의 상태들을 확인할수 있습니다.



[목록](https://github.com/yuby/mongodb-ko)