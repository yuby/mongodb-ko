#MongoDB CRUD Introduction

몽고디비는 데이터를 JSON같이 필드와 값으로 이뤄진 형태인 documents형식으로 저장을 합니다.  Documents는 키와 값으로 아뤄진 구조인데 이는 dictionaries, hashes, maps, and associative arrays 같은 우리에게 익숙한 구조입니다.
정식적으로는 몽고디비의 documents는 [BSON](http://docs.mongodb.org/manual/reference/glossary/#term-bson) documents입니다. BSON은 [JSON](http://docs.mongodb.org/manual/reference/glossary/#term-json)을 추가적인 정보와 같이 바이너리 형태로 표현하는 방식입니다. documents에서는 BSON이 허용하는 어떠한 데이터 타입도 값으러 저장이 될 수 있습니다. 다른 documents는 물론 배열, documents의 배열이 있습니다. 더 자세한 정보는 [이곳](http://docs.mongodb.org/manual/core/document/)에서 확인하세요.
![images](http://docs.mongodb.org/manual/_images/crud-annotated-document.png)

A MongoDB document.
MongoDB stores all documents in collections. A collection is a group of related documents that have a set of shared common indexes. Collections are analogous to a table in relational databases.

A collection of MongoDB documents.
Database Operations

Query
In MongoDB a query targets a specific collection of documents. Queries specify criteria, or conditions, that identify the documents that MongoDB returns to the clients. A query may include a projection that specifies the fields from the matching documents to return. You can optionally modify queries to impose limits, skips, and sort orders.

In the following diagram, the query process specifies a query criteria and a sort modifier:

The stages of a MongoDB query with a query criteria and a sort modifier.
See Read Operations Overview for more information.

Data Modification
Data modification refers to operations that create, update, or delete data. In MongoDB, these operations modify the data of a single collection. For the update and delete operations, you can specify the criteria to select the documents to update or remove.

In the following diagram, the insert operation adds a new document to the users collection.

The stages of a MongoDB insert operation.
See Write Operations Overview for more information.

Related Features

Indexes
To enhance the performance of common queries and updates, MongoDB has full support for secondary indexes. These indexes allow applications to store a view of a portion of the collection in an efficient data structure. Most indexes store an ordered representation of all values of a field or a group of fields. Indexes may also enforce uniqueness, store objects in a geospatial representation, and facilitate text search.

Replica Set Read Preference
For replica sets and sharded clusters with replica set components, applications specify read preferences. A read preference determines how the client directs read operations to the set.

Write Concern
Applications can also control the behavior of write operations using write concern. Particularly useful for deployments with replica sets, the write concern semantics allow clients to specify the assurance that MongoDB provides when reporting on the success of a write operation.

Aggregation
In addition to the basic queries, MongoDB provides several data aggregation features. For example, MongoDB can return counts of the number of documents that match a query, or return the number of distinct values for a field, or process a collection of documents using a versatile stage-based data processing pipeline or map-reduce operations.