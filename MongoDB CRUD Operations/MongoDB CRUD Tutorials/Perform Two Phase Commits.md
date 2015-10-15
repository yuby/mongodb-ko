[목록](https://github.com/yuby/mongodb-ko)



#Perform Two Phase Commits


##Synopsis

이번에는 여러 documents를 업데이트 하거나 여러 documents의 트랜잭션에서 two-phase 커밋을 통해 다수의 documents를 쓰는 방법에 대한 설명입니다. 추가적으로 [롤백같은 (rollback-like)](http://docs.mongodb.org/manual/tutorial/perform-two-phase-commits/#phase-commits-rollback) 기능을 재공하는 것으로 확장하여 생각할수도 잇습니다.


##Background

몽도디비에서는 하나의 [document](http://docs.mongodb.org/manual/reference/glossary/#term-document)를 대상으로 하는 동작은 항상 원자성이 보장이 됩니다. 하지만 다수의 documents를 대상으로 하는 작업이라면(multi-document transactions) 원자성이 보장이 되지 않습니다.
document는 매우 복잡하고 여러 "중첩(nested)" documents를 포함 할 수 있기 때문에, 단일 document의 원자성은 반드시 다양한 실질적인 경우들을 지원해야합니다.

이처럼 단일 document에 대한 원자성을 보장함에도 불구하고 여러 documents 트랜잭션이 필요한 경우가 있습니다. 순차적인 작업에 트랜젝션이 실행이 되면 몇가지 이슈가 발생합니다.

- Atomicity :  만약에 하나의 작업이 실패를 한다면 이전 작업업으로 상태로 반드시 롤백되어야 합니다. all or nothing 에서 nothing 상태가 예입니다.
- Consistency : 네트워크나 하드웨어때문에 트랜잭션이 실패를 한다면 데이터 베이스는 일관성을 반드시 유지시켜 줘야합니다.

여러 document에 대한 트랜젝션이 필요로 하게 되는 상황에서 우리는 two-phase commit을 적용해서 다수의 document를 수정하는 작업을 지원 할수 있습니다. two-phase commit은 데이터가 일관되고 에러가 발생하는 경우 트랜잭션의 이전 상태로 [복구할수 있어야 합니다(recoverable)](http://docs.mongodb.org/manual/tutorial/perform-two-phase-commits/#phase-commits-rollback). 하지만 진행되는 동안에 document는 데이터와 상태값의 지연이 있을수 있습니다.

>NOTE
오직 단일 document의 작업만이 원자성을 가지기 때문에 two-phase commit은 트랜잭션과 같은 의미를 제공 할 수 있습니다. 애플리케이션이 two-phase commit 또는 롤백을 하는 중간에  데이터를 반환하는 것이 가능합니다.


##Pattern

###Overview
하나의 시나리오를 가정해 보겠습니다. 여러분이 돈을 A계좌에서 B계좌로 이체를 하려합니다. 관계형 데이터베이스 시스템에서는 A계좌로 부터 차감을 하고 그만큼을 B계좌로 더해주는 작업이 하나의 다중 상태 트랜잭션을 통해 구현이 됩니다. 몽고디비에서는 이와 같은 작업을 two-phase commit을 통해서 동일하게 작업을 진행할수있습니다.

다음의 예제는 두개의 collection을 가지고 구현이 됩니다.
- accounts라는 이름의 계좌정보를 담고 있는 collection입니다.
-transactions 은 계좌의 이채 트랜젝션의 정보를 저장하고 있습니다.

###Initialize Source and Destination Accounts
accounts collection 상에 A계좌 정보와 B계좌 정보를 저장합니다.
```
db.accounts.insert(
   [
     { _id: "A", balance: 1000, pendingTransactions: [] },
     { _id: "B", balance: 1000, pendingTransactions: [] }
   ]
)
```
작업의 결과로 [BulkWriteResult()](http://docs.mongodb.org/manual/reference/method/BulkWriteResult/#BulkWriteResult)  객체를 리턴할것입니다. 위 작업이 성공적이라면 [BulkWriteResult()](http://docs.mongodb.org/manual/reference/method/BulkWriteResult/#BulkWriteResult) 의 [nInserted](http://docs.mongodb.org/manual/reference/method/BulkWriteResult/#BulkWriteResult.nInserted)가 2가 됩니다.

###Initialize Transfer Record
각각의 이체 작업은 transactions collection에 이체정보를 저장합니다. document는 다음과 같은 필드의 정보가 필요합니다.
- source 와 destination 필드는 각가의 계좌정보를 담고있는 documents의 _id정보를 각각 참조하고 있습니다.
- value 필드는 source와 destination계좌에 영향을 미칠 결국 이체를 하게 될 금액정보를 담습니다.
- state 필드는 현재 이체 상태에 대한 정보입니다. state 필드는  initial, pending, applied, done, canceling, canceled 를 값으로 가지고 있습니다.
- lastModified 필드는 가장 최근에 수정된 시간의 정보를 가지고 있습니다.

우선 100을 A에서 B계좌로 전송해보겠습니다. transactions collection에 이체 정보를 저장하고 상태는 initial, lastModified는 현재 시간으로 정합니다.
```
db.transactions.insert(
    { _id: 1, source: "A", destination: "B", value: 100, state: "initial", lastModified: new Date() }
)
```
작업의 결과로 [WriteResult()](http://docs.mongodb.org/manual/reference/method/WriteResult/#WriteResult) 객체에 작업의 상태를 담아 함께 전달합니다. 생성작업이 성공적이라면 [WriteResult()](http://docs.mongodb.org/manual/reference/method/WriteResult/#WriteResult) 객체의 [nInserted](http://docs.mongodb.org/manual/reference/method/WriteResult/#WriteResult.nInserted)가 1이 됩니다.

###Transfer Funds Between Accounts Using Two-Phase Commit

####1 Retrieve the transaction to start.
transactions collection으로 부터 transaction의 최초 상태 값을 찾습니다. 현재 transactions collection은 오직 하나의 document를 가지고 있습니다. 최초 계좌 전송단계에서 설정한 initial 의 값을 가집니다. 만약에 collection이 추가적인 documents를 가지고 있다면 쿼리는 initial 상태의 다른 정보도 전달하게 됩니다. 이경우에는 추가적인 조건을 더해줘야 합니다.
```
var t = db.transactions.findOne( { state: "initial" } )
```
t 변수를 [몽고쉘](http://docs.mongodb.org/manual/reference/program/mongo/#bin.mongo)에 입력을 하면 해당변수가 담고 있는 내용을 프린트 합니다. 프린트된 내용은 우리가 입력했던 내용과 동일하고 다만 lastModified가 실제 날짜값으로 들어가있음을 확인할수 있습니다.
```
{ "_id" : 1, "source" : "A", "destination" : "B", "value" : 100, "state" : "initial", "lastModified" : ISODate("2014-07-11T20:39:26.345Z") }
```

#### 2 Update transaction state to pending.
두번째 단계에서는 initial 단계의 상태를 pending 단계로 수정해줘야 합니다. [$currentDate](http://docs.mongodb.org/manual/reference/operator/update/currentDate/#up._S_currentDate) 연산자를 통해서 lastModified필드의 값을 현재 날짜로 설정해 줍니다.

```
db.transactions.update(
    { _id: t._id, state: "initial" },
    {
      $set: { state: "pending" },
      $currentDate: { lastModified: true }
    }
)
```
작업의 결과로 [WriteResult()](http://docs.mongodb.org/manual/reference/method/WriteResult/#WriteResult) 객체에 작업의 상태를 담아 함께 전달합니다. 생성작업이 성공적이라면 [WriteResult()](http://docs.mongodb.org/manual/reference/method/WriteResult/#WriteResult) 객체의 [nMatched](http://docs.mongodb.org/manual/reference/method/WriteResult/#WriteResult.nMatched)와 [nModified](http://docs.mongodb.org/manual/reference/method/WriteResult/#WriteResult.nModified)가  1이 됩니다.

이 단계에서 상태값이 initial이 아니라면 다른 작업을 진행하면서 해당 상태 정보를 수정했다는 말입니다. 즉 [nMatched](http://docs.mongodb.org/manual/reference/method/WriteResult/#WriteResult.nMatched) 와 [nModified](http://docs.mongodb.org/manual/reference/method/WriteResult/#WriteResult.nModified)가 0인 경우에는 처음 단계로 돌아가 다시 단계를 진행해야 합니다.

#### 3 Apply the transaction to both accounts.

이제 두 계좌의 정보를 [update()](http://docs.mongodb.org/manual/reference/method/db.collection.update/#db.collection.update) 메서드를 통해서 업데이트를 해줘야합니다.  업데이트를 위한 조건에  pendingTransactions: { $ne: t._id }를 추가해서 업데이트가 중복으로 발생하는 일을 예방합니다.

balance필드의 정보와 pendingTransactions 필드의 정보를 모두 업데이트 합니다.

보내는 사람의 계좌에서는 이체하는 금액만큼 잔고에서 빼주고 pendingTransactions 배열에 진행하고있는 거래의 (transaction)의 _id값을 추가합니다.
```
db.accounts.update(
   { _id: t.source, pendingTransactions: { $ne: t._id } },
   { $inc: { balance: -t.value }, $push: { pendingTransactions: t._id } }
)
```
성공적으로 업데이트가 이뤄졌다면 [WriteResult()]() 객체의 [nMatched]() 와 [nModified]() 가 각각 1이 됩니다.

받는 사람의 계좌의 경우에는 잔고에 전달받은 돈만큼 추가 되고 역시 pendingTransactions에 현재 진행되는  거래의 (transaction)의 _id값을 추가합니다.

```
db.accounts.update(
   { _id: t.destination, pendingTransactions: { $ne: t._id } },
   { $inc: { balance: t.value }, $push: { pendingTransactions: t._id } }
)
```
성공적으로 업데이트가 이뤄졌다면 [WriteResult()](http://docs.mongodb.org/manual/reference/method/WriteResult/#WriteResult) 객체의 [nMatched](http://docs.mongodb.org/manual/reference/method/WriteResult/#WriteResult.nMatched) 와 [nModified](http://docs.mongodb.org/manual/reference/method/WriteResult/#WriteResult.nModified) 가 각각 1이 됩니다.


#### 4 Update transaction state to applied.

[update()](http://docs.mongodb.org/manual/reference/method/db.collection.update/#db.collection.update) 메서드를 사용해서 작업의 상태를 바꿔주고 lastModified 필드의 정보를 수정해줘야합니다.
```
db.transactions.update(
   { _id: t._id, state: "pending" },
   {
     $set: { state: "applied" },
     $currentDate: { lastModified: true }
   }
)
```
성공적으로 업데이트가 이뤄졌다면 [WriteResult()](http://docs.mongodb.org/manual/reference/method/WriteResult/#WriteResult) 객체의 [nMatched](http://docs.mongodb.org/manual/reference/method/WriteResult/#WriteResult.nMatched) 와 [nModified](http://docs.mongodb.org/manual/reference/method/WriteResult/#WriteResult.nModified) 가 각각 1이 됩니다.

#### 5 Update both accounts’ list of pending transactions.
작업이 완료가 되었다면 진행했던 작업의 정보를 두 계좌의 pendingTransactions 으로 부터 제거를 해야합니다.

보내는 사람의 정보를 수정하고
```
db.accounts.update(
   { _id: t.source, pendingTransactions: t._id },
   { $pull: { pendingTransactions: t._id } }
)
```
반는 사람의 정보를 수정합니다.
```
db.accounts.update(
   { _id: t.destination, pendingTransactions: t._id },
   { $pull: { pendingTransactions: t._id } }
)
```
각 정보가 성공적으로 업데이트가 이뤄졌다면 [WriteResult()](http://docs.mongodb.org/manual/reference/method/WriteResult/#WriteResult) 객체의 [nMatched](http://docs.mongodb.org/manual/reference/method/WriteResult/#WriteResult.nMatched) 와 [nModified](http://docs.mongodb.org/manual/reference/method/WriteResult/#WriteResult.nModified) 가 각각 1이 됩니다.

#### 6 Update transaction state to done.
이제 모든 작업이 끝이 났습니다. 이제는 transaction collection의 정보에 모든 작업이 완료되었음으로 수정해주고 lastModified 정보도 수정해야 합니다.

```
db.transactions.update(
   { _id: t._id, state: "applied" },
   {
     $set: { state: "done" },
     $currentDate: { lastModified: true }
   }
)
```
성공적으로 업데이트가 이뤄졌다면 [WriteResult()](http://docs.mongodb.org/manual/reference/method/WriteResult/#WriteResult) 객체의 [nMatched](http://docs.mongodb.org/manual/reference/method/WriteResult/#WriteResult.nMatched) 와 [nModified](http://docs.mongodb.org/manual/reference/method/WriteResult/#WriteResult.nModified) 가 각각 1이 됩니다.


##Recovering from Failure Scenarios

대부분의 이체과정이 위처럼 이상적으로 동작하지 않습니다. 그래서 작업이 성공적으로 완료되지 못하는 다양한 실패의 경우를 대비해서 복구가 가능하도록 해야합니다. 이번에는 작업도중 실패가 발생하는 경우를 확인해보고 각 이벤트마다 복구하는 시나리오를 전체적으로 확인해 보겠습니다.

###Recovery Operations
two-phase commit 패턴은 어플리케이션이 순차적으로 이체작업이 재개되어 일관된 상태가 유지되도록 해줍니다. 복구작업이 어플리케이션이 시작하면서 동작을 하면 완료되지 못한 이체작업들을 모두 잡아 냅니다.

어플리케이션은 복구중인 각각의 이체작업이 일관된 상태가 되기 위한 시간이 필요합니다.

다음의 복구 과정은 lastModified 의 날짜 정보를 가지고 복구가 필요한 이체작업인지 아닌지를 판단합니다. 특히 이체작업의 상태가 pending나 applied로 최근 30분간 업데이트가 이뤄지지 않는다면 프로시져는 그 이체작업은 복구될 필요가 있는 작업이라고 인지를 압니다. 물론 lastModified의 날짜 정보가 아닌 다른 정보를 가지고도 충분히 이런 작업이 가능합니다.

####Transactions in Pending State
이체작업의 상태를 pending으로 업데이트하는 단계 이후에 그리고 applied 이전에 작업의 실패가 발생을 하면 transactions collection으로 부터 pending 중인 이체작업의 정보를 찾아 복구를 해줘야 합니다.

```
var dateThreshold = new Date();
dateThreshold.setMinutes(dateThreshold.getMinutes() - 30);

var t = db.transactions.findOne( { state: "pending", lastModified: { $lt: dateThreshold } } );
```
그리고는 각 계좌로부터 이체작업을 요청하는 단계부터 다시 시작하면 됩니다.

####Transactions in Applied State
이체작업의 상태를 applied로 업데이트하는 단계 이후에 그리고 done 이전에 작업의 실패가 발생을 하면 transactions collection으로 부터 applied 중인 이체작업의 정보를 찾아 복구를 해줘야 합니다.

```
var dateThreshold = new Date();
dateThreshold.setMinutes(dateThreshold.getMinutes() - 30);

var t = db.transactions.findOne( { state: "applied", lastModified: { $lt: dateThreshold } } );
```
그리고는 두 계좌에 pending 작업의 리스트를 업데이트 하는 곳 부터 다시 시작하면 됩니다.

###Rollback Operations
가끔은 작업을 roll back 하거나 되돌리기가 가능하게 해줘야합니다. 만약에 이체작업의 중단이 필요하거나 이체를 하려는 계좌의 정보가 존재하지 않는 경우등 이체작업이 중단되어야 하는 경우가 있습니다.

####Transactions in Applied State
이체작업의 상태가 applied 로 업데이트가 된이후에는 roll back 작업이 이뤄지면 안됩니다. 대신에 새로운 이체작업을 생성하여 이체가 이뤄졌던 단대로 이체작업을 진행해줘여합니다. (A->B 를 B-> A로)

###Transactions in Pending State
이체작업이 pending이고 applied가 되기 이전단계에서는 rollback이 다음과 같은 방식으로 이뤄집니다.

#### 1 Update transaction state to canceling.
이체정보를 우선 pending에서 calceling으로 바꿉니다.
```
db.transactions.update(
   { _id: t._id, state: "pending" },
   {
     $set: { state: "canceling" },
     $currentDate: { lastModified: true }
   }
)
```
성공적으로 업데이트가 이뤄졌다면 [WriteResult()](http://docs.mongodb.org/manual/reference/method/WriteResult/#WriteResult) 객체의 [nMatched](http://docs.mongodb.org/manual/reference/method/WriteResult/#WriteResult.nMatched) 와 [nModified](http://docs.mongodb.org/manual/reference/method/WriteResult/#WriteResult.nModified) 가 각각 1이 됩니다.

#### 2 Undo the transaction on both accounts.
두 계좌의 정보를 되돌려야 합니다. 만약에 이미 이체가 적용된 상태라면 다시 반대로 이체작업을 지원해줘야 합니다. 수정을 위한 조건은 pendingTransactions: t._id 를 통해서 오직해당 계좌의 정보를 업데이트 합니다.

받는 사람의 계좌 정보에서 받은 금액만큼을 잔고에서 빼주는 작업이 필요합니다. 그리고 pendingTransactions에서 진행중인 작업의 정보를 제거합니다.
```
db.accounts.update(
   { _id: t.destination, pendingTransactions: t._id },
   {
     $inc: { balance: -t.value },
     $pull: { pendingTransactions: t._id }
   }
)
```


그리고 반대로  보낸 사람의 계좌 정보에서 보낸 금액만큼을 잔고에 더해주는 작업이 필요합니다. 그리고 pendingTransactions에서 진행중인 작업의 정보를 제거합니다.
```
db.accounts.update(
   { _id: t.source, pendingTransactions: t._id },
   {
     $inc: { balance: t.value},
     $pull: { pendingTransactions: t._id }
   }
)
```

각 작업이 성공적으로 수정이 이뤄졌다면 [WriteResult()](http://docs.mongodb.org/manual/reference/method/WriteResult/#WriteResult)  객체의  [nMatched](http://docs.mongodb.org/manual/reference/method/WriteResult/#WriteResult.nMatched) 와 [nModified](http://docs.mongodb.org/manual/reference/method/WriteResult/#WriteResult.nModified)가 1이 됩니다. 만약  보류중인 이체작업이 이전에 이 계좌에 적용되지 않았다면 업데이트를 하기 위한 조건에 일치하는 결과가 없습니다. 이경우 [nMatched](http://docs.mongodb.org/manual/reference/method/WriteResult/#WriteResult.nMatched) 와 [nModified](http://docs.mongodb.org/manual/reference/method/WriteResult/#WriteResult.nModified) 는 0이 됩니다.

#### 3 Update transaction state to canceled.
rollback이 끝이 나면 이체작업의 상태는 canceling 에서 cancelled로 수정이 됩니다.
```
db.transactions.update(
   { _id: t._id, state: "canceling" },
   {
     $set: { state: "cancelled" },
     $currentDate: { lastModified: true }
   }
)
```
작업이 성공적으로 수정이 이뤄졌다면 [WriteResult()](http://docs.mongodb.org/manual/reference/method/WriteResult/#WriteResult)  객체의  [nMatched](http://docs.mongodb.org/manual/reference/method/WriteResult/#WriteResult.nMatched) 와 [nModified](http://docs.mongodb.org/manual/reference/method/WriteResult/#WriteResult.nModified)가 1이 됩니다.

##Multiple Applications

다수의 어플리케이이션이 생성과 실행작업을 동시에 진행헤도 데이터의 일관성이 유지 되고 충돌이 일어나지 않도록 하는 부분이 있습니다. 위의 예제에서는 이체정보를 수정하거나 검색할때 수정을 위한 조건에 state 필드를 이용해서 다수의 어플리케이션에 의해 작업이 중복으로 적용 되는 일을 방지 합니다.

예를들여 App1과 App2가 동일한 이체 정보를 가지고 작업을 시작하여 한다고 가정해 보겠습니다. App1 모든 이체작업을 App2가 시작하기 전에 모두 적용했습니다. App2 가 pending으로 업데이트 하기 위해서 이체정보가 initial인 이체정보를 transaction collection에서 검색을 하지만 일치하는 정보가 없습니다. 즉 [nMatched](http://docs.mongodb.org/manual/reference/method/WriteResult/#WriteResult.nMatched) 와 [nModified](http://docs.mongodb.org/manual/reference/method/WriteResult/#WriteResult.nModified)가 0이라는 말입니다. 이는 App2가 처음으로 돌아가 다른 이체정보를 가지고 다시 작업을 시작해야 함을 의미합니다.

여러 어플리케이션이 동작을 하는 경우, 하나의 어플리케이션이  특정 시점에 특정 트랜잭션을 처리 할 수​​있는 것은 매우 중요합니다. 그래서 트랜잭션의 상태를 알수 있는 정보를 포함하는 것 이외에 우리는 추가적으로 현재 작업중인  어플리케이션을 식별할수 있는 정보를 추가 해두는 것이 좋습니다. [findAndModify()](http://docs.mongodb.org/manual/reference/method/db.collection.findAndModify/#db.collection.findAndModify) 메서드를 사요해서 이체작업에 대한 정보를 수정하여 이를 가지고 작업을 진행하면 됩니다.
```
t = db.transactions.findAndModify(
       {
         query: { state: "initial", application: { $exists: false } },
         update:
           {
             $set: { state: "pending", application: "App1" },
             $currentDate: { lastModified: true }
           },
         new: true
       }
    )
```
이젝정보를 수정하여 현재 작업중인 이체가 있음을 확인해서 오직 일치하는 이체정보만을 가지고 작업을 진행해 나갈수가 있습니다.

만약에 App1가 작업중에 실패를 한다면 우리는 복구과정을 거쳐야합니다. 하지만 어플리케이션은 자신이 작업을 해오던 이체정보를 가지고 작업을 진행해야 합니다. 예를들어 pending중인 작업을 검색하기 위해서는 다음과 같이 검색을 하면 됩니다.

```
var dateThreshold = new Date();
dateThreshold.setMinutes(dateThreshold.getMinutes() - 30);

db.transactions.find(
   {
     application: "App1",
     state: "pending",
     lastModified: { $lt: dateThreshold }
   }
)
```

##Using Two-Phase Commits in Production Applications

위의 예제의 경우에는 매우 간략하게 과정을 설명했습니다. 예를들면 모든 계좌에 대한 rollback이 가능하다는 가정과 잔고가 마이너스 값을 가질수 있게 처리가 되어있습니다.

하지만 실제 제품에는 조금 더 복잡할 것입니다. 보통 계좌의 경우에는 현재 잔고(balance, pending credits, and pending debits) 등 더 많은 정보가 필요합니다.

모든 이체에  적절한 레벨의 [write concern](http://docs.mongodb.org/manual/core/write-concern/)을 함께 사용한다면 훨씬 안정작인 작업의 결과를 보장할수 있습니다.




[목록](https://github.com/yuby/mongodb-ko)