#Limit Number of Elements in an Array after an Update

New in version 2.4.

##Synopsis

어플리케이션에서 사용자는 점수를 입력할때 어플리케이션은 최고 점수 3개의 정보만을 필요로 하는 경우가 있습니다.

이러한 패턴에서는 [$push](http://docs.mongodb.org/manual/reference/operator/update/push/#up._S_push)를 [$each](http://docs.mongodb.org/manual/reference/operator/update/each/#up._S_each), [$sort](http://docs.mongodb.org/manual/reference/operator/update/sort/#up._S_sort), [$slice](http://docs.mongodb.org/manual/reference/operator/update/slice/#up._S_slice)와 함꼐 사용하면 됩니다.


##Pattern

collection에 저장된 document정보 입니다.
```
{
  _id: 1,
  scores: [
    { attempt: 1, score: 10 },
    { attempt: 2 , score:8 }
  ]
}
```
[$push](http://docs.mongodb.org/manual/reference/operator/update/push/#up._S_push)연산자를 사용해서 정보를 수정합니다.

- [$each](http://docs.mongodb.org/manual/reference/operator/update/each/#up._S_each) 연산자는 배열에 2개의 새로운 정보를 추가합니다.
- [$sort](http://docs.mongodb.org/manual/reference/operator/update/sort/#up._S_sort) 연산자를 사용해서 score 를 내림차순으로 정의 합니다.
- [$slice](http://docs.mongodb.org/manual/reference/operator/update/slice/#up._S_slice) 연산자를 사용해서 배열의 마지막 3 정보를 얻습니다.

```
db.students.update(
   { _id: 1 },
   {
     $push: {
        scores: {
           $each: [ { attempt: 3, score: 7 }, { attempt: 4, score: 4 } ],
           $sort: { score: 1 },
           $slice: -3
        }
     }
   }
)
```

>NOTE
[$sort](http://docs.mongodb.org/manual/reference/operator/update/sort/#up._S_sort)를 사용해서 배열의 엘리먼트에 접근할 때는[dot](http://docs.mongodb.org/manual/reference/glossary/#term-dot-notation) 을 사용하지 않고 바로 접근할수 있습니다.

모든 작업이 끝나고 나면 document는 오직 최대 값 3개를 배열에 담고 있습니다.
```
{
   "_id" : 1,
   "scores" : [
     { "attempt" : 3, "score" : 7 },
     { "attempt" : 2, "score" : 8 },
     { "attempt" : 1, "score" : 10 }
   ]
}
```

SEE ALSO
[$push](http://docs.mongodb.org/manual/reference/operator/update/push/#up._S_push) 
[$each](http://docs.mongodb.org/manual/reference/operator/update/each/#up._S_each) 
[$sort](http://docs.mongodb.org/manual/reference/operator/update/sort/#up._S_sort)
[$slice](http://docs.mongodb.org/manual/reference/operator/update/slice/#up._S_slice).