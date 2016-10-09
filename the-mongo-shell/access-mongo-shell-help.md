#Access the mongo Shell Help
몽고디비는 기본적으로 가이드 문서를 제공합니다. 하지만 시스템상에서 바로 필요한 도움을 언제든지 받을수 있습니다.


##Command Line Help
--help 옵션을 통해 사용가능한 도움목록들을 확인할 수 있습니다.

```
mongo --help
```

##Shell Help

mongo shell 상에서도 help 입력을 통해 도움 목록을 확인할 수 있습니다.
```
help
```

##Database Help

- 서버상에 존재하는 데이터베이스 목록을 확인 하기 위해서 show dbs 명령을 사용합니다.
```
show dbs
```

- db 객체에 대한 도움목록을 확인할 수 있습니다.
```
db.help()
```

- db의 메서드를 shell 상에서 실행 및 확인하기 위해서 db.<method name>를 사용합니다. db.updateUser() 를 확인 하기 위해서 () 를 제외하고 입력을 합니다.
```
db.updateUser
```

##Collection Help

- 현재 데이터베이스 상에 collection의 리스트를 리턴합니다.
```
show collections
```

- collection 객체에 대한 도움 목록을 확인합니다. db.<collection>.help()
```
db.collection.help()
```

- collection의 메서드를 shell 상에서 실행 및 확인하기 위해서 db.<collection>.<method>를 사용합니다. save()메서드를 확인하기 위해서 다음과 같이 입력합니다.
```
db.collection.save
```






