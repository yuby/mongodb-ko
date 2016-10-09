#mongo Shell Quick Reference

##mongo Shell Command History
mongo shell에서 이전에 작업한 내용들을 확인 하기 위해서는 화살표 up/down을 사용해서 히스토리 확인이 가능합니다. 모든 히스토리는 ~/.dbshell 파일에 저장이됩니다.

##Command Line Options
mongo shell은 몇가지 옵션과 함께 시작할수 있습니다. [이곳](https://docs.mongodb.com/manual/reference/program/mongo/)에서 다양한 옵션을 확인할수 있습니다.
다음은 가장 보편적으로 사용하는 옵션들입니다.

|Option | Description |
|---------|-----------|
| --help|	커맨드라인 옵션을 보여줍니다. |
| --nodb |데이터 베이스 연결이 없이 mongo shell을 시작합니다.<br/> 추후에 DB에 새로운 연결을 확인할수 있습니다.|
|--shell | javaScript 파일(i.e. <file.js>)과 함께 mongo shell을 사용합니다. |

##Command Helpers
mongo shell은 다양한 help를 제공합니다. 다음은 가장 보편적으로 사용하는 help 메서드입니다.

|Help Methods and Commands | Description |
|---------|-----------|
|help | help를 보여줍니다.|
|db.help() | db의 help메서드를 보여줍니다.|
|db.<collection>.help() | collection 의 help 메서드를 보여줍니다.|
|show dbs | 서버의 모든 데이터베이스 정보를 보여줍니다.|
|use <db> | 현재의 사용중인 데이터 베이스는 <db> 가 됩니다. mongo shell의 db 변수가 현재의 데이터 베이스로 세팅이 됩니다.|
|show collections | 현재 데이터 베이스의 모든 collection 정보를 보여줍니다.|
|show users | 현재 데이터 베이스의 모든 사용자 정보를 보여줍니다.|
|show roles | 현재 데이터 베이스의 기본 시스템이 설정하고나 사용자가 정의한 role 정보를 보여줍니다.|
|show profile | 가장 최근에 작업 5개를 전달합니다.|
|show databases | 사용가능한 모든 데이터 베이스 정보를 전달합니다.|
|load() | javascript 파일을 실행합니다. |

##Basic Shell JavaScript Operations
mongo shell의 경우 javascript api를 데이터베이스 명령으로 제공합니다. mongo shell에서의 db 변수는 현재의 데이터 베이스 를 참조하고 있고 기본적으로는 test데이터를 참조하고 있습니다. 원하는 데이터 베이스는 user <db> 를 사용해서 설정할수 있습니다. 다음은 가장 보편적으로 사용하는 javascript명령입니다.

|JavaScript Database Operations	| Description|
|-|-|
|db.auth()| 보안모드에서 사용자 인증을 합니다. |
|coll = db.<collection>| coll 변수에 현재 db의 특정 collection을 지정합니다.|
|db.collection.find()	| collection에서 document를 찾아서 cursor에 담아 리턴합니다.|
|db.collection.insert()| 새로운 document 를 collection에 생성합니다.|	
|db.collection.update()| 존재하는 collection의  document를 수정합니다.|
|db.collection.save() |	 새로운 document를 생성하거나 존재한 document의 경우엔 수정합니다.|
|db.collection.remove() |	collection으로 부터 documents를 삭제합니다.|
|db.collection.drop()	| collection을 제거합니다.|
|db.collection.createIndex()|	새로운 index 정보를 생성합니다. 만약 index정보가 존재한다면 아무런 영향을 끼치지 않습니다.|
|db.getSiblingDB() | 동일한 접속상태에서 다른 데이터 베이스로의 전환을 할 수 있습니다.|


##Keyboard Shortcuts
다양한 키보드 단축키를 제공합니다.

|Keystroke|	Function|
|---------|-----------|
|Up-arrow|	previous-history|
|Down-arrow|	next-history|
|Home|	beginning-of-line|
|End|	end-of-line|
|Tab|	autocomplete|
|Left-arrow|	backward-character|
|Right-arrow|	forward-character|
|Ctrl-left-arrow|	backward-word|
|Ctrl-right-arrow|	forward-word|
|Meta-left-arrow|	backward-word|
|Meta-right-arrow|	forward-word|
|Ctrl-A	|beginning-of-line|
|Ctrl-B	|backward-char|
|Ctrl-C	|exit-shell|
|Ctrl-D	|delete-char (or exit shell)|
|Ctrl-E	|end-of-line|
|Ctrl-F	|forward-char|
|Ctrl-G	|abort|
|Ctrl-J	|accept-line|
|Ctrl-K	|kill-line|
|Ctrl-L	|clear-screen|
|Ctrl-M	|accept-line|
|Ctrl-N	|next-history|
|Ctrl-P	|previous-history|
|Ctrl-R	|reverse-search-history|
|Ctrl-S	|forward-search-history|
|Ctrl-T	|transpose-chars|
|Ctrl-U	|unix-line-discard|
|Ctrl-W	|unix-word-rubout|
|Ctrl-Y	|yank|
|Ctrl-Z	|Suspend (job control works in linux)|
|Ctrl-H |(i.e. Backspace)	backward-delete-char|
|Ctrl-I |(i.e. Tab)	complete|
|Meta-B	|backward-word|
|Meta-C	|capitalize-word|
|Meta-D	|kill-word|
|Meta-F	|forward-word|
|Meta-L	|downcase-word|
|Meta-U	|upcase-word|
|Meta-Y	|yank-pop|
|Meta-[Backspace]|	backward-kill-word|
|Meta-<	|beginning-of-history|
|Meta->	|end-of-history|

##Queries
mongo shell에서는 읽기 작업을 find() 와 findOne() 메서드를 사용합니다.
find() 메서드의 경우네느 cursor 객체에 docuemnt를 담아 리턴합니다. 기본적으로 20개의 document를 리턴합니다. "it"를 입력하면 다음 20개의 docuemnt를 확인할수 있습니다.

다음은 읽기 작업에 사용되는 명령어들입니다.

|Method|Description|
|---------|-----------|
|db.collection.find(<query>)| <query> 에 특정 조건을 담아 실행합니다. <query>가 정의 되지 안흔 경우에는 collection의 전체 document를 전달합니다.|
|db.collection.find(<query>, <projection>)| <query>으로 특정 조건의 데이터를 불러와 <projection> 정의한 필드의 데이터만 출력합니다.|
|db.collection.find().sort(<sort order>)|검색된 결과를 필드에 1(오름차순) , -1(내림차순)을 정의하면 해당 필드기준으로 정렬됩니다.|
|db.collection.find(<query>).sort(<sort order>)|<query>된 결과를 정렬합니다.|
|db.collection.find( ... ).limit( <n> )|n 개의 제한된 결과만 리턴합니다.|
|db.collection.find( ... ).skip( <n> )|n개의 결과를 건너뛰고 리턴합니다.|
|db.collection.count()|전체 collection에 존재하는 document의 갯수를 리턴합니다.|
|db.collection.find(<query>).count()|조건에 일치하는 결과 갯수를 리턴합니다.|
|db.collection.findOne(<query>)|조건에 일치하는 오직 하나의 document를 리턴합니다. 매치되는 결과가 없는 경우 nil 을 리턴합니다.|


더 많은 쿼리의 정보는 [이곳](https://docs.mongodb.com/manual/tutorial/query-documents/)에서 확인 하시면 됩니다.

##Error Checking Methods
이전에 Write concern의 경우에는 db.getLastError() 메서드들 통해서 확인이 가능했지만 지금은 쓰기작업의 메서드 상에 write concern을 함께 작성해서 실행할수가 잇습니다. 예를들어 현재는 WriteResult()메서드상에 작업의 결과 (작업상의 오류 나 write concern의 오류 정보) 를 담아 리턴합니다.

이전버전의 경우에는 db.getLastError() 와 db.getLastErrorObj() 메서드를 사용해서 해당 결과를 확인 했습니다.

##Administrative Command Helpers
다음은 데이터베이스 관리를 위한 메서드입니다.

|JavaScript Database Administration Methods|Description|
|---------|-----------|
|db.cloneDatabase(<host>)	| 현제의 데이터 베이스를 특정 host로 부터 복제합니다. 이경우 반드시 noauth mode 여야합니다.|
|db.copyDatabase(<from>, <to>, <host>)	| host상의 데이터 베이스 (<from> database) 부터 현재의 서버의 특정 데이터 베이스(<to> database) 로 데이터 베이스를 복제합니다. host 데이터 베이스의 경우 noauth mode입니다. |
|db.fromColl.renameCollection(<toColl>)	| 이전 collection의 이름을 다른 collection의 이름 toColl으로 변경합니다.|
|db.repairDatabase()|현재 데이터 베이스를 수리하고 압축하는 역활을 합니다. 이 동작의 경우 데이터베이스가 클 경우에는 매우 느립니다.|
|db.getCollectionNames()	| 현재 데이터베이스의 모든 collection의 정보를 리턴합니다.|
|db.dropDatabase()	| 현제 데이터베이스 정보를 삭제합니다. |

##Opening Additional Connections
mongo shell상에서 새로운 데이터 베이스 연결을 생성할수 있습니다.

|JavaScript Database Administration Methods|Description|
|---------|-----------|
|db = connect("<host><:port>/<dbname>") |새로운 데이터베이스 연결을 생성합니다.|
|conn = new Mongo()  db = conn.getDB("dbname") | new Mongo()를 사용해 새로운 연결을 생성합니다. getDB()메서드를 사용해 해당 데이터베이스의  연결 정보를 가져옵니다.|

##Miscellaneous
|Methods|Description|
|---------|-----------|
|Object.bsonsize(<document>)	| <document>의 BSON사이즈를 바이트 정보를 리턴합니다.|
