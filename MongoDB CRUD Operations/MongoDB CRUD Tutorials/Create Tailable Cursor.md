#Create Tailable Cursor

##Overview

기본적으로 몽고디비는 생성된 cursor를 클라이언트가 cursor의 결과를 모두 사용하는 경우 자동으로 닫아버립니다. 하지만 capped collections에서는 Tailable Cursor 를 사용해서 클라이언트다 모두 사용해도 닫지않고 최초의 cursor를 열어둡니다.
Tailable Cursor 는 개념적으로는 Unix의 -f 명령어와 동일합니다. 클라이언트가 새로운 document를 capped collection에 생성을 하면 tailable cursor 는 지속적으로 documents를 검색을 합니다.
Tailable Cursor를 사용하는 capped collections에서는 대량의 쓰기 작업에 인덱스는 효과적이 못합니다. 예를들어 몽고디비의 리플리케이션에서는 primary의 oplog의 tail에 tailable cursors를 사용하고 있습니다.

>NOTE
만약에 쿼리가 엔덱스 필드를 사용하고 있다면 tailable cursors를 사용하지 마세요. 대신에 보통 cursor를 사용하세요. 지속적으로 쿼리를 통해 리턴되는 인덱싱이 매겨진 마지막  필드의 값을 기억합니다. 새롭게 추가된 documents를 검색하려면  인덱스가 매겨진 마지막 필드의 값을 찾아 이를 쿼리의 조건으로 추가를 합니다. 아래의 예를 보시겠습니다.
``` 
db.<collection>.find( { indexedField: { $gt: <lastvalue> } } )
```

다음은 tailable cursors의 특징입니다.

- Tailable cursors 는 인덱스를 사용하지 않고  기본 순서(natural order)에 따라 documents를 리넡합니다.
- Tailable cursors는 인덱스를 사용하지 않기 때문에 최초의 쿼리는 매우 큰 비용이듭니다. 하지만 최초의 cursor가 소모가 되고 나면 그다음에 추가된 document의 검색에 있어서는 적은 비용이 듭니다.
- Tailable cursors는 다음과 같은 경우에 죽거나 무효한 상태가 됩니다.
    - 쿼리로 일치하는 값을 리터하지 못했을때
    - collection의 마지막 document를 cursor로 리턴하고 이를 어플리케이션이 삭제를 하는 경우
죽은 cursor의 id는 0입니다.
driver documentation 에서 드라이버의 자세한 메서드를 확인하세요.

##C++ Example (내용 제외)