## CRUD
### C(Create)
~~~
db.collection.insertOne({데이터})
~~~
~~~
db.collection.insertMany({데이터})
~~~

### R(Read)
 - 질의의 조건을 여러개 걸때 조건의 순서에 유의해야한다.
~~~
db.collection.find(
{query},
{projection}  
).{cursor modifier}
~~~
 - ex>
~~~
db.users.find(
    {age: {$gt: 18}},
    {name: 1, address: 1}
).limit(5)
~~~

#### query
 - 질의를 여러개까지 넣을수 있다.
 - 범위 연산자

|연산자|설명|
|---|---|
|$lt|~보다 작은|
|$gt|~보다 큰|
|$lte|~보다 작거나 같은|
|$gte|~보다 크거나 같은|

!!! 아래와같은 쿼리는 뒤의 lte 조건만 적용된다.
~~~
db.users.find( {birth_year:{$gt: 1990}, birty_year:{$lte: 2000}} )
~~~
 -> 이게 맞는 문법
~~~
db.users.find( {birth_year:{$gt: 1990, $lte: 2000}} )
~~~
 - 집합 연산자

|연산자|설명|
|---|---|
|$in|어떤 인수든 하나라도 참고 집합에 있는 경우|
|$all|모든 인수가 참고 집합에 있고 배열이 포함된 도큐먼트에서 사용되는 경우|
|$nin|그 어떤 인수도 참고 집합에 있지 않을 경우|
~~~
db.users.find( {color: {$in:["black","blue"]}} )
~~~
 - 논리 연산자

|연산자|설명|
|---|---|
|$ne|!=|
|$not|!|
|$or|or|
|$nor|not or(=검색결과중 그 어느것도 해당하지 않을때)|
|$and|and|
|$exists|요소가 도큐먼트 안에 존재할 경우|

 - 배열 연산자 : 배열의 순서로 접근할수있다.

|연산자|설명|
|---|---|
|$elemMatch|제공된 모든 조건이 동일한 하위 도큐먼트에 있는 경우|
|$size|배열 하위 도큐먼트의 크리가 제공된 리터럴 값과 같으면|

 - 기타 연산자

|연산자|설명|
|---|---|
|$mod[몫,결과]|몫으로 나눈 결과가 일치할 경우|
|$type|요소의 타입이 일치할 경우|
|$text|텍스트 인덱스로 인덱싱된 필드의 내용에 대해 텍스트 검색할수있다.|

#### projection
 - 결과값 도큐먼트에 대해 반환할 필드를 지정
 ~~~
 db.users.find( {color: {$in:["black","blue"]}} )
 ~~~

#### cursor modifier
 - limit
 - sort
      - 오름차순 : 1
      - 내림차순 : -1
 - skip

### U(Update)
~~~
db.collection.updateOne(
   <filter>,
   <update>,
   {
     upsert: <boolean>,
     writeConcern: <document>,
     collation: <document>,
     arrayFilters: [ <filterdocument1>, ... ]
   }
)
~~~
~~~
db.collection.updateMany(
   <filter>,
   <update>,
   {
     upsert: <boolean>,
     writeConcern: <document>,
     collation: <document>,
     arrayFilters: [ <filterdocument1>, ... ]
   }
)
~~~
~~~
db.collection.replaceOne(
   <filter>,
   <replacement>,
   {
     upsert: <boolean>,
     writeConcern: <document>,
     collation: <document>
   }
)
~~~

#### filter

#### replacement

#### option 



### D(Delete)
~~~
db.collection.deleteOne(
   <filter>,
   {
      writeConcern: <document>,
      collation: <document>
   }
)
~~~
~~~
db.collection.deleteMany(
   <filter>,
   {
      writeConcern: <document>,
      collation: <document>
   }
)
~~~
