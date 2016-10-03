
#Introduction to MongoDB

몽고디비는 오픈소스의 document 기반으로 뛰어난 성능과, 유용성, 자동 스케일링을 지원하는 데이터베이스 입니다.

##Document Database
몽고디비에서는 저장되는 데이터의 타입이 document 형태로 필드와 값이 쌍으로 존재하는 형태입니다.
document는 간단하게 JSON 객체와 유사한 형태입니다. 필드의 값으로 다른 document, array를 가질수가 있습니다.

![A MongoDB document.](http://docs.mongodb.org/manual/_images/crud-annotated-document.png)

document형태의 사용함으로서 발생하는 이점이 있습니다.

- document는 다양한 프로그래밍 언어가 기본적으로 제공하는 데이터 타입니다.
- Embedded된 document를 사용하면 값비싼 조인(join) 작업을 하지 않아도 됩니다.
- dynamic schema로 다양한 형태의 데이터 형태를 지원합니다.

##Key Features

###High Performance
몽고디비는 데이터를 높은 성능으로 저장합니다. 
- Embeded된 데이터 모델에 대한 지원으로 데이터베이스의 I/O 작업을 줄일수 있습니다.
- Indexes를 제공하여 높은 쿼리 성능을 제공하고 Embeded된 document나 array의 키에도 indexing이 가능합니다.

### Rich Query Language
몽고디비는 풍부한 쿼리를 지원합니다. CRUD는 물론이고 Data Aggregation, Text Search, Geospatial Queries를 제공합니다.

### High Availability
replica set이라고 불리우는 몽고디비의 replication은 
- 자동 복구
- 데이터 이중화
기능을 제공합니다.

replica set은 몽고디비 서버의 그룹으로 동일한 데이터 셋으로 유지가 되므로 데이터의 이중화와 데이터 가용성을 향상시켜 줍니다.

##Horizontal Scalability
몽고디비는 수평적인 확장을 핵심기능으로써 제공하고 있습니다.
- Shading을 통해서 클러스터 전체에 데이터를 분산시킬수있습니다.
- Tag를 통해 shading된 특정 데이터의 영역에 직접적으로 접근할수 있습니다.

##Support for Multiple Storage Engines
몽고디비는 여러개의 저장 엔진을 제공합니다.
- WiredTiger Storage Engine
- MMAPva Storage Engine

추가적으로 몽도디비는 다른 저장엔진 API를 사용할수 있는 환경을 제공하여 제공되는 엔진 이외에도 다른 저장 엔진을 사용 할 수 있게 합니다.



