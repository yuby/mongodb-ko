#Perform Two Phase Commits

##Synopsis

이번에는 다수의 document를 수정하는 작업이나 'multi-document transactons'을 처리하기 위해서 two-phase commit 패턴을 통해 쓰기작업을 진행합니다. 추가적으로 이러한 프로세스를 rollback-like 기능에도 적용할수있습니다.

##Background

하나의 document를 대상으로 동작하는 경우 항상 원자성이 보장됩니다. 하지만 다수의 document가 하나의 동작의 대상이 되는 경우 원자성이 보장되지 않습니다. document는 꽤나 복잡해질수 있고 다수의 nested document를 가질수가 있어서 하나의 document에 대해서는 원자성이 반드시 필요합니다.

이렇게 하나의 document의 원자성이 보장되기는 하지만 다수의 document의 경우에도 원자성이 필요로하는 경우가 있습니다. 만약에 순차적인 동작으로 구성된 transaction이 동작을 하면 다음의 이슈가 발생합니다.

- 원자성 : 만약에 작업중 하나가 실패하는 경우 이전의 작업은 반드시 롤백이 된어야합니다.
- 일관성 : transaction이 네트워크나 하드웨어의 원인으로 작업이 실패하는 경우 데이터 베이스는 상태를 복구해야합니다.

'multi document transaction'에서는 원자성과 일관성이 필요로 하고 이를 위해서 two-phase commit을 사용해서 에러상황에서 데이터의 일관성과 이전 동작에 대한 복구가 필요로 합니다. 하지만 이러한 동작중에 document는 pending 데이터나 상태로 나타날수 있습니다.

>NOTE
>왜냐하면 몽고디비는 오직 하나의 document를 대상으로만 원자성이 보장됩니다.
>two-pahse commit의 경우에는 트랜잭션 같은(transaction-like) 동작만 제공합니다. 그래서 작업도중에 데이터가 보이게 되는 현상이 발생할수가 있습니다.

##Patter

###Overview
시나리오를 가정해보겠습니다. A계좌에서 B계좌로 이체작업을 하려고합니다. 관계형 데이터 베이스 시스템 상에서는 A에서 금액을 빼고 B에 금액을 더하는 하나의 multi-statement 트랜잭션으로 동작을 합니다. 
몽고디비에서는 이러한 작업을 two-phase commit으로 같은 결과를 만들어 낼수있습니다.

두개의 collection을 구성합니다.

1. account collection을 구성해서 account의 정보를 저장합니다.
2. transactions collection을 구성해서 이체작업에 대한 정보를 저장합니다.

###Initialize Source and Destination Accounts
A와 B의 각각의 계좌정보를 생성합니다.

```
db.accounts.insert(
   [
     { _id: "A", balance: 1000, pendingTransactions: [] },
     { _id: "B", balance: 1000, pendingTransactions: [] }
   ]
)
```

작업의 결과로 BulkWriteResult() 객체가 리턴됩니다. 위작업이 성공을 했다면 BulkWriteResult() 객체는 nInserted가 2를 가집니다.

###Initialize Transfer Record
각각의 이체작업의 정보는 transactions collection에 document를 생성합니다. document는 다음의 필드정보를 가집니다.

- source, desctination 필드는 accounts필드의 A, B의 _id정보를 담습니다.
- value필드는 이체될 금액정보를 가지고 이는 source의 balance와 destination정보를 가집니다.
- state필드는 현재 이체 상태 정보를 가집니다. (initial, pending, applied, done,canceling, canceled)
- lastModified는 해당 document가 수정된 날짜정보를 가집니다.

A에서 100원을 B에 이채하는 작업을 시작합니다.

```
db.transactions.insert(
    { _id: 1, source: "A", destination: "B", value: 100, state: "initial", lastModified: new Date() }
)
```
작업의 결과로  WriteResult() 객체를 리턴하고 작업이 성공한 상태라면 nInserted가 1이됩니다.

###Trnasfer Funds Between Accounts Using Two-Phase Commit

1. Retrive the transaction to start

transaction의 필드에서 initial인 상태의 document의 정보를 가져옵니다. 현재의 transactions의 collection에는 오직 하나의 document가 존재하고 이를 'Initialize Transfer Record'단계라고 하겠습니다.
만약에 collection이 추가적인 docuemnt를 가지고 있다면 쿼리는 다른 조건을 쿼리에 추가해야합니다.

```
var t = db.transactions.findOne( { state: "initial" } )

```

mongo shell상에서  t를 입력하면 다음과 같은 결과를 리턴합니다.

```
{ "_id" : 1, "source" : "A", "destination" : "B", "value" : 100, "state" : "initial", "lastModified" : ISODate("2014-07-11T20:39:26.345Z") }
```

2. Update transaction state to pending

transaction의 state정보를 pending으로 변경을 하고 $currentDate 연산자를 통해서 lastModified필트의 정보를 수정합니다.

```
db.transactions.update(
    { _id: t._id, state: "initial" },
    {
      $set: { state: "pending" },
      $currentDate: { lastModified: true }
    }
)
```
작업의 결과로 WriteResult()객체를 리턴하고 성공을 했다면 nMatched와 nModified는 1을 가집니다.

3. Apply the transaction to both account

t정보를 사용해서 update() 작업을 진행합니다. pendingTransactions: {$ne: t._id} 를 포합해서 한번이상의 동작이 발생하지 않도록 합니다.

이체작업을 진행하기 위해서 balance필드와 pendingTransactions필드를 수정합니다.

source account의 balance로 부터 value 만큼의 금액을 빼고 이를 pendingTransactions의 배열에 _id정보를 추가합니다.

```
db.accounts.update(
   { _id: t.source, pendingTransactions: { $ne: t._id } },
   { $inc: { balance: -t.value }, $push: { pendingTransactions: t._id } }
)

```

작업이 성공했다면 WriteResult()의 nMatch, nModified는 1을 리턴합니다.

4. Update transaction state to applied

update()메서드를 사용해서 transaction의 state정보를 applied로 수정하고 lastModified필드의 정보도 수정합니다.

```
db.transactions.update(
   { _id: t._id, state: "pending" },
   {
     $set: { state: "applied" },
     $currentDate: { lastModified: true }
   }
)
```

5. Update both accounts' list of pending transactions

각각 accounts의 pendingTransactions배열로부터 _id필드 정보를 제거합니다.

```
db.accounts.update(
   { _id: t.source, pendingTransactions: t._id },
   { $pull: { pendingTransactions: t._id } }
)
```

```
db.accounts.update(
   { _id: t.destination, pendingTransactions: t._id },
   { $pull: { pendingTransactions: t._id } }
)
```

6. Update transaction state to done

모든작업이 완료가 되었다면 state의 정보를 done으로 수정하고  lastModified 필드의 정보도 수정합니다.


##Recovering from Failure Scenarios

트랜잭션에서의 가장 중요한 부분은 위의 예제의 이체 과정이 아닙니다. 중요한것은 다양한 에러상황에서 위의 트랜잭션과정이 완료가 되어버리면 안된다는 것입니다.

###Recovery Operations

two-phase commit패턴은 어플리케이션이 순차적으로 작업을 수행하고 일관된 상태를 유지하게 합니다. 복구작업의 경우에는 어플리케이션이 시작하는 순간부터 동작하고 특정시간, 미완료된 트랜잭션이 발생하는 순간 모두 동작합니다.

원래의 상태로 돌아오기 위한 시간은 얼마나 많은 작업들이 복구되어야하는지에 비례합니다. 만약 pending이나 applied가 30분동안 발생하지 않는다면 해당 트랜잭션은 복구가 필요한 상태로 인식합니다.

####Transactions in Pending State

"Update transaction state to pending" 단계에서 실패가 발생하면 "Update transaction state to applied" 이전에 해당 정보를 transactions collection으로 부터 불러와 복구작업을 해야합니다.

```
var dateThreshold = new Date();
dateThreshold.setMinutes(dateThreshold.getMinutes() - 30);

var t = db.transactions.findOne( { state: "pending", lastModified: { $lt: dateThreshold } } );

```

####Transaction in Applied State

"Update transaction state to applied" 단계에서 오류가 발생을 하면 "Update transaction state to done" 이전에 복가가 이뤄져야합니다.

```
var dateThreshold = new Date();
dateThreshold.setMinutes(dateThreshold.getMinutes() - 30);

var t = db.transactions.findOne( { state: "applied", lastModified: { $lt: dateThreshold } } );

``` 


###Rollback Operations

특정경우에 rollback 이나 transaction을 원래로 돌리는 작업이 필요합니다. 예를들면 상대 account의 정보가 존재하지 않는 경우 이전의 모든 작업을 취소해야합니다.

####Transactions in Applied State

"Update transaction state to applied" 이후에는 롤백이 이뤄질수 없습니다. 대신에 해당 트랜잭션을 완료시키고 새로운 트랜잭션을 생성해 A->B의 정보를 B->A로 바꿔동작시킵니다.

####Transactions in Pending State

"Update transaction state to pending" 이후 , "Update transaction state to applied" 이전에 롤백이 이뤄져야합니다.

1 . Update transaction state to canceling

```
db.transactions.update(
   { _id: t._id, state: "pending" },
   {
     $set: { state: "canceling" },
     $currentDate: { lastModified: true }
   }
)
```

2. Undo the transaction on both accounts

```
db.accounts.update(
   { _id: t.destination, pendingTransactions: t._id },
   {
     $inc: { balance: -t.value },
     $pull: { pendingTransactions: t._id }
   }
)
```

```
db.accounts.update(
   { _id: t.source, pendingTransactions: t._id },
   {
     $inc: { balance: t.value},
     $pull: { pendingTransactions: t._id }
   }
)
```

3. Update transaction state to canceled

```
db.transactions.update(
   { _id: t._id, state: "canceling" },
   {
     $set: { state: "cancelled" },
     $currentDate: { lastModified: true }
   }
)
```


##Multiple Applications

다수의 어플리케이션이 동시에 작업을 진행하는 경우에 데이터의 충돌이나 일관성이 깨지는 이리 발생하지 않고 동작해야합니다. 위의 예제에서 읽기나 쓰기작업의 경우에는  state필드를 사용해서 트랜잭션을 구분합니다.

예를들어 App1과 App2가 동일한 initial 상태라고 가정하겠습니다. App1은 App2이전에 모든 동작을 완료합니다. App2가 App1이 마친후에 "Update transaction state to pending"을 하려고 하는 경우에 transaction collection상에 initial상태의 정보가 존재하지 않습니다.

다수의 어플리케이션이 동작을 하는 경우 각 어플리케이션만의 트랜잭션이 지정되어야합니다. 

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

만약에 App1이 동작중에 실패를 한다면 복구작업이 진행되어야 하지만 복구작업이 반드시 실패한 어플리케이션의 복구작업임을 지정해야합니다. 

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

위의 예제는 매우 간단한 상황에 대한 예제입니다. 실제 이체 어플리케이션을 위해서는 current balance, pending credits, pending debits등의 추가적인 정보가 훨씬많이 필요합니다. 
모든 트랜잭션의 경우에 write concern을 사용해 동작을 더욱 확실하게 하게해야합니다.
