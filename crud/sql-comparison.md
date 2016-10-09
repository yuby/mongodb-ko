#SQL to MongoDB Mapping Chart

다음의 차트들은 몽고디비에 대한 FAQ입니다.

##Terminology and Concepts

아래의 테이블은 SQL과 몽고디비 간의 표현을 대응시킨 표입니다.

|SQL Terms/Concepts	| MongoDB Terms/Concepts |
|----------------|----------------|
|database	|database |
|table |	collection|
|row |	document or BSON document| 
|column |	field|
|index |	index|
|table joins	|embedded documents and linking|
|primary key |primary key|
| 특정 컬럼의 또는 컬럼의 조합이 primary key |몽고디비에서는 자동으로 _id 필드가 primary key|
|aggregation (e.g. group by)	|aggregation pipeline. 자세한 내용은 [이곳](https://docs.mongodb.com/manual/reference/sql-aggregation-comparison/)에서 확인하시기 바랍니다.|


##Executables

다음은 데이터베이스를 실행어입니다.

||MongoDB |	MySQL |	Oracle | Informix |	DB2 |
|--------|--------|--------|--------|--------|--------|
|Database Server |	mongod	|mysqld|	oracle|	IDS|	DB2 Server|
|Database Client |	mongo|	mysql|	sqlplus|	DB-Access|	DB2 Client|

##Examples

다음은 SQL문과 이에 대응하는 몽도디비의 표현을 실제 코드를 기반으로 살펴보겠습니다.

- SQL에서의 테이블 이름은 users 입니다.
- 몽고디비의 collection이름은 users이고 다음의 document 형태를 가집니다.
```
{
  _id: ObjectId("509a8fb2f3f4948bd2f983a0"),
  user_id: "abc123",
  age: 55,
  status: 'A'
}
```

###Create and Alter

|SQL Schema Statements	|MongoDB Schema Statements|
|-----|-----|
|CREATE TABLE users (  <br/>id MEDIUMINT NOT NULL<br/>  AUTO_INCREMENT, <br/> user_id Varchar(30),<br/>age Number,<br/>status char(1), <br/> PRIMARY KEY (id) <br/>)| db.users.insert( { <br/>  user_id: "abc123", <br/>    age: 55, <br/>    status: "A" <br/> } ) |
| ALTER TABLE users <br/> ADD join_date DATETIME | db.users.update( <br/>    { }, <br/>    { $set: {join_date: new Date() } },<br/>     { multi: true } <br/>) |
|ALTER TABLE users <br/> DROP COLUMN join_date |db.users.update( <br/>    { }, <br/>    { $unset: {join_date: "" } }, <br/>    { multi: true }<br/>)|
|CREATE INDEX idx_user_id_asc <br/> ON users(user_id) | db.users.createIndex( { user_id: 1 } )|
|CREATE INDEX <br/>       idx_user_id_asc_age_desc <br/>ON users(user_id, age DESC)|db.users.createIndex( { user_id: 1, age: -1 } )|
|DROP TABLE users | db.users.drop()|


###Insert

다음은 SQL문에서 특정데이터를 기록하는 코드와 이에 대응하는 몽고디비 코드입니다.

|SQL INSERT Statements	| MongoDB insert() Statements |
|-----|-----|
|INSERT INTO users(user_id, age,status)<br/> VALUES ("bcd001",45,"A") |db.users.insert(   { user_id: "bcd001", age: 45, status: "A" })|


###Select
다음은 SQL문에서 특정 기록을 읽어오는 코드와 이에 대응하는 몽고디비 코드입니다.

>NOTE
> find() 메서드의 경우 항상 _id 필드를 따로 빼서 전달하는 옵션을 설정하지 않는 이상 항상 _id 필드를 포함해서 결과를 리턴합니다. 아래 SQL문에 id필드가 정의 되었는데 몽도디비 코드에서는 필드정의가 빠진 이유도 이때문입니다.

|SQL SELECT Statements	| MongoDB find() Statements |
|-----|-----|
|SELECT * FROM users | db.users.find()|
| SELECT id, user_id, status FROM users | db.users.find({ },{ user_id: 1, status: 1 })|
| SELECT user_id, status FROM users | db.users.find( { }, { user_id: 1, status: 1, _id: 0 }) |
| SELECT * FROM users WHERE status = "A" | db.users.find( { status: "A" } ) |
| SELECT user_id, status FROM users WHERE status = "A" | db.users.find( { status: "A" }, { user_id: 1, status: 1, _id: 0 })|
|SELECT * FROM users WHERE status != "A" | db.users.find( { status: { $ne: "A" } } )|
| SELECT * FROM users WHERE status = "A" AND age = 50 | db.users.find( { status: "A",age: 50 } ) |
| SELECT * FROM users WHERE status = "A" OR age = 50 | db.users.find( { $or: [ { status: "A" } ,{ age: 50 } ] }) |
| SELECT * FROM users WHERE age > 25| db.users.find( { age: { $gt: 25 } } )|
| SELECT * FROM users WHERE age < 25 | db.users.find( { age: { $lt: 25 } }) |
| SELECT * FROM users WHERE age > 25 AND   age <= 50 | db.users.find( { age: { $gt: 25, $lte: 50 } }) |
| SELECT * FROM users WHERE user_id like "%bc%" | db.users.find( { user_id: /bc/ } ) |
| SELECT * FROM users WHERE user_id like "bc%" |  db.users.find( { user_id: /^bc/ } ) |
| SELECT * FROM users WHERE status = "A" ORDER BY user_id ASC | db.users.find( { status: "A" } ).sort( { user_id: 1 } ) |
| SELECT * FROM users WHERE status = "A" ORDER BY user_id DESC | db.users.find( { status: "A" } ).sort( { user_id: -1 } ) |
| SELECT COUNT(*) FROM users | db.users.count() <br/>or<br/> db.users.find().count()|
| SELECT COUNT(user_id) FROM users | db.users.count( { user_id: { $exists: true } } ) <br/>or<br/> db.users.find( { user_id: { $exists: true } } ).count()|
| SELECT COUNT(*) FROM users WHERE age > 30 | db.users.count( { age: { $gt: 30 } } )<br/>or<br/>db.users.find( { age: { $gt: 30 } } ).count()| 
| SELECT DISTINCT(status) FROM users | db.users.distinct( "status" )|
| SELECT * FROM users LIMIT 1 | db.users.findOne()<br/>or<br/>db.users.find().limit(1)|
| SELECT * FROM users LIMIT 5 SKIP 10 | db.users.find().limit(5).skip(10) |
| EXPLAIN SELECT * FROM users WHERE status = "A"|db.users.find( { status: "A" } ).explain() |

###Update Records
다음은 데이블에 존재하는 데이터를 수정하는 SQL 코드와 이에 대응하는 몽고디비 코드입니다.

|SQL Update Statements	| MongoDB update() Statements|
|-----|-----|
| UPDATE users SET status = "C" WHERE age > 25 | db.users.update( { age: { $gt: 25 } }, { $set: { status: "C" } },{ multi: true }) |
| UPDATE users SET age = age + 3 WHERE status = "A" | db.users.update({ status: "A" } , { $inc: { age: 3 } }, { multi: true }) |

###Delete Records

다음은 데이블에 존재하는 데이터를 삭제하는 SQL 코드와 이에 대응하는 몽고디비 코드입니다.

| SQL Delete Statements	| MongoDB remove() Statements |
|-----|-----|
| DELETE FROM users WHERE status = "D" | db.users.remove( { status: "D" } )|
| DELETE FROM users | db.users.remove({}) |

