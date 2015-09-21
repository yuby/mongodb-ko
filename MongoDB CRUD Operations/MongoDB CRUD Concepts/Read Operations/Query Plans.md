#Query Plans

몽고디비 쿼리 최적화기는 쿼리를 대상으로 진행이 되고 쿼리에 주어진 사용가능한 인덱스들을 기반으로 가장 효과적인 쿼리 계획을 선택합니다. 그리고 나서 쿼리 시스템은 쿼리가 동작할떄마다 이 시스템을 바탕으로 동작을 합니다.
쿼리 최적화기는 하나 이상의 실행가능한 계획을 쿼리 형태로 캐쉬합니다.
쿼리 최적화기는 가끔 collection의 내용이 달라 지는 경우 쿼리 계획을 재평가 하여 최적의 쿼리 플랜을 가지도록 합니다. 우리는 [인덱스 필터](http://docs.mongodb.org/manual/core/query-plans/#index-filters)를 통해서 특정 인덱스의 최적화 작압도 가능합니다.

 [db.collection.explain()](http://docs.mongodb.org/manual/reference/method/db.collection.explain/#db.collection.explain) 또는 [cursor.explain()](http://docs.mongodb.org/manual/reference/method/cursor.explain/#cursor.explain)메서드를 사용해서 주어진 쿼리의 쿼리계획의 통계를 볼수 있습니다. 더 많은 [인덱싱 정보](http://docs.mongodb.org/manual/applications/indexes/)에 대해서 확인 해보세요.

[db.collection.explain()](http://docs.mongodb.org/manual/reference/method/db.collection.explain/#db.collection.explain)은 [db.collection.update()](http://docs.mongodb.org/manual/reference/method/db.collection.update/#db.collection.update) 메서드 같은 다른 동작들에 대한 정보를 제공합니다. 더많은 정보는 [이곳](http://docs.mongodb.org/manual/reference/method/db.collection.explain/#db.collection.explain)에서 확인 하세요.

##Query Optimization

To create a new query plan, the query optimizer:
쿼리 최적화기를 통해 새로운 쿼리 계획을 세우는 방법입니다.

1. 쿼리가 서로다른 후보 인덱스가 병렬로 실행이 되는 경우.
2. 버퍼에 저장된 공통의 일치하는 결과를 기록하는 경우.
    - 만약에 후보 계획들이  [순차적인 쿼리](http://docs.mongodb.org/manual/reference/glossary/#term-ordered-query-plan) 계획만을 가지는  경우 오직 하나의 공통 결과 버퍼가 있습니다.
    - 만약에 후보 계획들이 [비 순차적인 쿼리](http://docs.mongodb.org/manual/reference/glossary/#term-unordered-query-plan) 계획만을 가지는 경우에도 오직 하나의 공통 결과 버퍼가 있습니다.
    - 만약에 후보 계획들이 [순차적인 쿼리](http://docs.mongodb.org/manual/reference/glossary/#term-ordered-query-plan), [비 순차적인 쿼리](http://docs.mongodb.org/manual/reference/glossary/#term-unordered-query-plan) 계획이 모두 있는 경우에는 두개의 공통 결과 버퍼들이 있습니다. 하나는 순차적인 결과 버퍼이고 다른 하나는 비 순차적인 결과 버퍼입니다.
    만약에 인덱스를 통해 리턴한 결과가 다른 인덱스에 의해 이미 리턴된 결과라면 최적화기는 중복에 대해서는 건너뛰게 됩니다.  위와 같이 두개의 버퍼가 있는 경우에는 두 버퍼에대한 중복 제거 작업(de-duped)이 이뤄집니다.
3. 후보계획들에 대한 테스트를 멈추고 다음에 에 설명할 이벤트가 발생하는 경우 인덱스를 선택합니다.
    - [비 순차적인 쿼리](http://docs.mongodb.org/manual/reference/glossary/#term-unordered-query-plan)  계획은 일치하는 모든 결과를 리턴합니다.
    - [순차적인 쿼리](http://docs.mongodb.org/manual/reference/glossary/#term-ordered-query-plan)  계획은 일치하는 모든 결과를 리턴합니다.
    - [순차적인 쿼리](http://docs.mongodb.org/manual/reference/glossary/#term-ordered-query-plan)  계획은 일치하는 결과의 한계 수치가 있습니다.
        - Version 2.0 : 한계 수치가 배치사이즈와 동일합니다. 기본적인 베치 사이즈는 101 입니다.
        - Version 2.2 : 한계 수치가 101입니다.


선택된 인덱스는 쿼리 계획의 인덱스로 지정이 되게 됩니다. 그리고 이후에 이 쿼리가 동작을 하거나 동일한 패턴의 쿼리가 동작을 할때면 이 인덱스를 사용하게 됩니다. 쿼리 패턴은 쿼리의 셀렉트의 조건의 값을 참조 하게 됩니다. 다음의 두 쿼리는 동일한 패턴의 쿼리입니다.
```
db.inventory.find( { type: 'food' } )
db.inventory.find( { type: 'utensil' } )
```

##Query Plan Revision

collections이 여러번 바뀌게 되면 쿼리 최적화기는 쿼리 계획을 모두 삭제합니다. 그리고 다음이벤트가 발생할시에 다시 최적화 작업을 진행합니다.
- 1000개의 쓰기 작업이 collection에 일어 날때.
- 인덱스의 [재설정](http://docs.mongodb.org/manual/reference/command/reIndex/#dbcmd.reIndex)작업이 이뤄질때.
- 인덱스를 추가 하거나 삭제 할때.
- [mongod](http://docs.mongodb.org/manual/reference/program/mongod/#bin.mongod) 프로세스가 다시 시작할때.

2.6버전 이후: explain() 동작은 더이상 쿼리 planner 캐쉬를 읽거나 쓰기가 지원되지 않습니다.

##Cached Query Plan Interface

New in version 2.6.

몽고디비는 [Query Plan Cache](http://docs.mongodb.org/manual/reference/method/js-plan-cache/)  메서드를 지원해서 캐쉬된 쿼리 플랜을 보고 수정할수 가 있습니다.

##Index Filters

New in version 2.6.

인덱스 필터는 어떤 인덱스가 최적의 [형태의 쿼리](http://docs.mongodb.org/manual/reference/glossary/#term-query-shape)를 가지게 하는지를 정의합니다. 쿼리의 형태는 쿼리와 정렬, 특정 필드에 대한 projection으로 구성이 되어있습니다. 만약에 인덱스 필터가 주어진 쿼리 형태에 존재하는 경우, 최적화기는 오직 필터에 지정된 인덱스만을 고려합니다.
인덱스필터가 쿼리 형태로 존재한다면 몽고디비는 [hint()](http://docs.mongodb.org/manual/reference/method/cursor.hint/#cursor.hint)메서드를 무시합니다. 몽고디비가 쿼리형태의 인덱스 필터를 적용하는지는 [indexFilterSet](http://docs.mongodb.org/manual/reference/explain-results/#explain.queryPlanner.indexFilterSet) 필드나  [db.collection.explain()](http://docs.mongodb.org/manual/reference/method/db.collection.explain/#db.collection.explain) 또는 [cursor.explain()](http://docs.mongodb.org/manual/reference/method/cursor.explain/#cursor.explain) 메서드를 확인하면 됩니다.
인덱스필터는 오직 최적화기의 평가를 받은 인덱스에 대해서만 영향을 줄것이고 최적화기는 여전히 collection을 선택하여 주어진 선택된 쿼리를 지속적으로 검사를 할것입니다.
인덱스 필터는 서버가 진행하는 동안에만 존재합니다. 그리고 종료가 된 이휴에는 지속되지 않습니다. 몽고디비는 수동으로 해당 필터를 제거하는 명령을 제공합니다.
인덱스 필터는 최적화기에 의해 예상된 동작뿐만아니라 [hint()](http://docs.mongodb.org/manual/reference/method/cursor.hint/#cursor.hint) 메서드를 오버라이드 하기 때문에 삼가면서 인덱스 필터를 사용합니다.
[planCacheListFilters](http://docs.mongodb.org/manual/reference/command/planCacheListFilters/#dbcmd.planCacheListFilters), [planCacheClearFilters](http://docs.mongodb.org/manual/reference/command/planCacheClearFilters/#dbcmd.planCacheClearFilters), 그리고 [planCacheSetFilter](http://docs.mongodb.org/manual/reference/command/planCacheSetFilter/#dbcmd.planCacheSetFilter) 에 대한 정보를 확인하세요.