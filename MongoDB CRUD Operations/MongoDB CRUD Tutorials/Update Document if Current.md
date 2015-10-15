#Update Document if Current

##Overview
'Update if Current' 패턴은 여러 어플리케이션이 데이터에 [동시에 접근할때 이를 관리](http://docs.mongodb.org/manual/core/write-operations-atomicity/#concurrency-control)하는 방법입니다.

##Pattern

패턴은 document의 업데이트를 위한 쿼리입니다. 각 필드의 정보를 수정하기 위해서는 수정작업을 위한 쿼리의 조건에 의해 전달되는 document안에 있는 필드와 값이을 필드가 포함하고 있습니다. 이러한 방식으로 업데이트는 오직 쿼리 이후에도 변경되지 않은 document의 필드 정보만을 수정합니다.


##Example

다음 예제는 [몽고쉘](http://docs.mongodb.org/manual/reference/program/mongo/#bin.mongo)에서 quantity와 reordered 필드즤 정보를 업데이트 하고 일치하는 쿼리가 없을떄의 예제입니다.

2.6버전 이후
[db.collection.update()](http://docs.mongodb.org/manual/reference/method/db.collection.update/#db.collection.update) 메서드는 [WriteResult()](http://docs.mongodb.org/manual/reference/method/WriteResult/#WriteResult) 객체를 리턴하면서 작업의 상태를 담아 전달합니다. 이전 버전의 경우에는 [db.getLastErrorObj()](http://docs.mongodb.org/manual/reference/method/db.getLastErrorObj/#db.getLastErrorObj) 메서드를 호출해야 했습니다.

```
var myDocument = db.products.findOne( { sku: "abc123" } );

if ( myDocument ) {
   var oldQuantity = myDocument.quantity;
   var oldReordered = myDocument.reordered;

   var results = db.products.update(
      {
        _id: myDocument._id,
        quantity: oldQuantity,
        reordered: oldReordered
      },
      {
        $inc: { quantity: 50 },
        $set: { reordered: true }
      }
   )

   if ( results.hasWriteError() ) {
      print( "unexpected error updating document: " + tojson(results) );
   }
   else if ( results.nMatched === 0 ) {
      print( "No matching document for " +
             "{ _id: "+ myDocument._id.toString() +
             ", quantity: " + oldQuantity +
             ", reordered: " + oldReordered
             + " } "
      );
   }
}
```
> 추가적인 설명을 하면 검색의 조건을 oldQuantity, oldReordered처럼 변수화 시키면 현재 document가 가진 정보를 가지고 해당 document를 return받아 수정작업을 할수가 있습니다. 즉, 해당 document만을 업데이트 시키는데 특화시키는 개념으로 볼수가 있습니다.

##Modifications to the Pattern

다른 방법은 version 필드를 documents에 추가하는 것입니다. 어플리케이션은 각각의 업데이트 작업이 발생할때 마다 필드의 값을 증가 시킵니다. 데이터베이스에 연결하는 모든 클라이언트가 쿼리의  조건에 version 필드를 포함됨을 보장 할 수 있어야합니다. collection의 documents가 증가함에 따라 우리는  [Create an Auto-Incrementing Sequence Field](http://docs.mongodb.org/manual/tutorial/create-an-auto-incrementing-field/) 의 메서드중에 하나를 사용할수가 있습니다.

더 많은 정보는 [Concurrency Control](http://docs.mongodb.org/manual/core/write-operations-atomicity/#concurrency-control)에서 확인하세요.