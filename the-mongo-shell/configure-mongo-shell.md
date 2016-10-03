#Configure the mongo Shell

##Customize the Prompt
mongo shell 에서 prompt 변수를 사용하면 prompt 의 내용을 수정할 수가 있습니다. prompt 변수의 경우에 javascript 코드를 가질수가 있습니다. prompt 가 javascript의 함수로 정의 되어있다면 mongo shell의 경우에는 함수에 해당하는 동적인 정보를 매번 보여줍니다.
추가적인 로직은 .mongorc.js 파을을 통해서 작성하면 mongo shell에 해당 환경으로 세팅이 됩니다.

###Customize Prompt to Display Number of Operations
예를들어 여러 작업을 현재의 세션에서 한번에 진행 하고 싶은 경우 다음과 같은 방법이 있습니다.

```
cmdCount = 1;
prompt = function() {
             return (cmdCount++) + "> ";
         }
```
결과는 다음과 같습니다.

```
1>
2>
3>
```

###Customize Prompt to Display Database and Hostname
```
host = db.serverStatus().host;

prompt = function() {
             return db+"@"+host+"$ ";
         }
```

결과는 다음과 같습니다.
```
test@myHost1$
```

###Customize Prompt to Display Up Time and Document Count
mongo shell에 시스템의 up time 과 document의 갯수를 표시하는 방법은 다음과 같습니다.

```
prompt = function() {
           return "Uptime:"+db.serverStatus().uptime+" Documents:"+db.stats().objects+" > ";
         }
```
결과는 다음과 같습니다.

```
Uptime:5897 Documents:6 >
```

###Use an External Editor in the mongo Shell
mongo shell의 경우 반드시 터미널 창에서 할 필요가 없습니다. 원하는 에디터를 연동하는 방법입니다. EDITOR 라는 환경변수에 값을 지정하고 mongo shell을 시작합니다.

```
export EDITOR=vim
mongo
```

이렇게 한번 지정을 하면 edit <variable> 이나 edit <function> 을 사용하면 설정된 에디터로 함수나 변수를 수정할 수 있습니다. 
1. myFunction 함수를 정의합니다.
```
function myFunction () { }
```

2. 에디터를 사용해서 해당 함수를 수정합니다.
```
edit myFunction
```

3. 수정을 하고 mongo shell 상에서 해당 함수를 입력하면 수정된 함수를 확인할 수 있습니다.

```
myFunction
```
```
function myFunction() {
    print("This was edited");
}
```

>NOTE
>mongo shell은 외부 에디터에서 수정된 코드를 javascipt 컴파일러에 의해서 해석 할 수 있습니다. mongo 는 1+1을 2로 변환하거나 주석을 제거하기도합니다.
> 그것은 자바 스크립트 컴파일러에 따라 기능 코드를 수정할 수 있습니다. 예를들어 mongo가 1 + 1의 코드를 2로 변환하거나 주석을 제거 할 수 있습니다. 이런 변화가 보기에는 코드가 변경이 되어있지만 실제 코드의 의미에는 아무런 영향이 없는 상태입니다.

##Change the mongo Shell Batch Size
db.collection.find() 메서드는  javascript 메서드로서 collection 상에서 document를 검색해서 리턴합니다. db.collection.find()메서드는 cursor를 리턴하지만 결과를 var 키워드를 통해 지정하지 않으면 20개의 결과만을 기본적으로 리턴합니다. **it** 를 입력할때마다 나머지 결과가 20개씩 리턴됩니다.

DBQuery.shellBatchSize 속성을 통해 기본으로 설정된 20개의 값을 바꿀수가 있습니다.

```
DBQuery.shellBatchSize = 10;
```


