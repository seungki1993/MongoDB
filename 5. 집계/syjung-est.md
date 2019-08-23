## 집계
### 집계 파이프라인
|SQL|연산자|설명|
|---|---|---|
|SELECT|$project|=projection|
|WHERE, HAVING|$match|처리될 도큐먼트를 선택|
|LIMIT|$limit|다음단계에 전달될 도큐먼트의 수를 정한다.|
|-|$skip|지정된 수의 도큐먼트를 건너뛴다.|
|JOIN|$unwind|배열을 확장하여 각 배열 항목에 대해 하나의 출력 도큐먼트를 생성한다.|
|GROUP BY|$group|지정된 키로 도큐먼트를 그룹화한다. sum, min, avg 등 연산도 가능|
|ORDER BY|$sort|토큐먼트를 정렬한다.|
|-|$neoNear|지리 공간위치 근처의 도큐먼트를 선택한다.|
|-|$out|파이프라인의 결과를 컬렉션에 쓴다.|
|-|$redact|특정 데이터에 접은을 제어한다.|
 - ex>
~~~
db.collection.aggregate([{$match:...}, {$group:...}, {$sort:...}])
~~~

### 예시
~~~
db.reviews.aggregate([
  {$match: {product_id: product['_id']}},
  {$group: {
      _id: '$rating',
      count:{$sum:1}
    }}
])
~~~
=
~~~
SELECT RATING, COUNT(*) AS COUNT
FROM REVIEWS
WHERE PRODUCT_ID = 'ID'
GROUP BY RATING
~~~

#### $project
 - projection임
~~~
db.collection.aggregate([
  {$match: {user_name: 'kbanker', hashed_password: '~~~'}},
  {$project: {first_name:1, last_name:1}} -> 이름과 성을 반환하는 프로젝트 파이프라인 연산자  
])
~~~

#### $group
 - 집계함수를 이용하여 요약통계를 제공한다.

|연산자|설명|
|---|---|
|$addToSet|그룹에 고유한 값의 배열을 만든다. 중복값제거O|
|$first|그룹의 첫번째값. $sort를 선행해야 의미있음.|
|$last|그룹의 마지막값. $sort를 선행해야 의미있음.|
|$max|그룹의 필드 최대값|
|$min|그룹의 필드 최소값|
|$avg|필드 평군값|
|$push|그룹의 모든 값의 배열을 반환한다. 중복값제거X|
|$sum|그룹의 모든 값의 합계|

~~~
db.collection.aggregate([
{$match: {user_name: 'kbanker', hashed_password: '~~~'}},
{$group: {
    _id: {year: {$year: '$purchase_data'}, month: {$month: '$purchase_data'}},
    count: {$sum: 1},
    total: {$sum: '$sub_total'}
      }},
{$sort: {_id: -1}}      
])
~~~

#### $match, sort, skip, limit
~~~
db.collection.aggregate([
{$match: {product_id: product['_id']}},
{$skip: (page_number - 1)*12},
{$limit: 12},
])
~~~

#### $unwind
 - 배열의 모든 항목에 대해 하나의 출력 도큐먼트를 생성하여 배열을 확장함.
~~~
{ "_id" : 1, "item" : "ABC1", sizes: [ "S", "M", "L"] }
->
db.collection.aggregate( [ { $unwind : "$sizes" } ] )
->
{ "_id" : 1, "item" : "ABC1", "sizes" : "S" }
{ "_id" : 1, "item" : "ABC1", "sizes" : "M" }
{ "_id" : 1, "item" : "ABC1", "sizes" : "L" }
~~~

#### $out
 - 질의의 결과값을 컬렉션에 저장한다.
 - 파이프라인의 마지막 연산자여야함.
 - 이미 있는 컬렉션일 경우 덮어쓰게된다.
 - view 같이 사용
~~~
db.collection.aggregate( [
 { $group : { _id : "$author", books: { $push: "$title" } } },
 { $out : "authors" }
] )
~~~

### 함수
 - 문자열함수, 산술함수, 날짜/시간함수, 논리함수, 집합함수 등 다양하게 제공
 - https://docs.mongodb.com/manual/reference/operator/aggregation/

### 옵션
 - explain() : 파이프라인 프로세스 세부정보만 반환. 도큐먼트 몇개 스캔했고, 인덱스는 뭘 썼는지 등등.
 - allowDiskUse : 중간 결과를 위해 디스크를 사용한다.
 - cursor : 초기 배치 크기를 지정한다.

### 집계와 성능
 - 파이프라인에서 가능한 한 도큐먼트 수를 줄여나간다.
 - 인덱스는 $match, $sort 작업에서만 사용할 수 있다.
