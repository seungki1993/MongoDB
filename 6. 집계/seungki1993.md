### 집계



- 여러 도큐먼트의 데이터를 변환하고 결합하여 단일 도큐먼트에서 사용할 수 없는 새로운 정보를 생성
- elasticsearch에서도 집계를 지원



#### 집계파이프라인

- 파이프라인으로 구성

| 옵션     | 설명                                                         |
| -------- | ------------------------------------------------------------ |
| $project | 출력 도규먼트상에 배치할 필드를 지정한다.(즉 원하는 필드만 필터링) |
| $match   | 처리될 도큐먼트를 선택하는 것                                |
| $limit   | 다음 단계에 전달될 도큐먼트의 수를 제한한다.                 |
| $skip    | 지정된 수의 도큐먼트를 건너뛴다.                             |
| $unwind  | 배열을 확장하여 각 배열 항목에 대해 하나의 출력 도큐먼트를 생성한다.(배열 요소를 분해하여 배열 요소마다 하나의 다큐먼트를 만든다. 배열이 아닌 원소도 가능) |
| $group   | 지정된 키로 도큐먼트를 그룹화 한다.                          |
| $geoNear | 지리 공간위치 근처의 도큐먼트를 선택한다.                    |
| $sort    | 도큐먼트를 정렬한다.                                         |
| $out     | 파이프라인의 결과를 컬렉션에 쓴다.                           |
| $redact  | 특정 테이터에 대한 접근을 제어한다.                          |

- 이 외에도 다양한 옵션이 많다.
  - 해당 옵션에 또한 세부 옵션들이 존재한다.
- 옵션중에 여러번 사용할 수 있는 옵션이 있고 아닌 옵션이 존재한다.

```
db.mongodb.aggregate({option},{option},{option},...)
```

- 집계 파이프라인의 결과는 커서

#### $group

| 옵션      | 설명                                                     |
| --------- | -------------------------------------------------------- |
| $addToSet | 그룹에 고유한 값의 배열을 만든다(중복은 허용하지 않는다) |
| $first    | 그룹의 첫번째 값                                         |
| $last     | 그룹의 마지막값                                          |
| $max      | 최댓값                                                   |
| $min      | 최솟값                                                   |
| $avg      | 평균값                                                   |
| $push     | addToSet과 동일하나 중복허용                             |
| $sum      | 합계                                                     |

#### 도큐먼트 재구성

- RDB에 select 문에 as 라고 생각하면된다.
- $project를 통해 필터링된 필드의 key값을 변경할 수 있다.

```
db.est.aggregate({match:~~}{$project:{name:{first:$first_name,last:$last_name}}})

match문에서 걸러진 document의 first_name, last_name이 first, last에 매핑되어 결과를 반환 한다.
```

##### 문자열 함수

| 옵션        | 설명                                                         |
| ----------- | ------------------------------------------------------------ |
| $concat     | 문자열 합치기                                                |
| $strcasecmp | 대소문자를 구분하지 않는 문자열 비교를 하며 , 숫자를 반환한다. |
| $substr     | 부분 문자열(v3.4부터 deprecated)                             |
| $toLower    | 모두 소문자                                                  |
| $toUpper    | 모두 대문자로 변환한다.                                      |

##### 산술 함수

| 옵션       | 설명                                                         |
| ---------- | ------------------------------------------------------------ |
| $add       | db.sales.aggregate(    [      { $project: { item: 1, total: { $add: [ "$price", "$fee" ] } } }    ] )  price와 fee를 더한다 |
| $divide    | db.sales.aggregate(    [      { $project: { item: 1, total: { $add: [ "$price", "$fee" ] } } }    ] )  price와 fee를 나눈다 |
| $mod       | db.sales.aggregate(    [      { $project: { item: 1, total: { $add: [ "$price", "$fee" ] } } }    ] )  price와 fee를 나눈 나머지 |
| $mutiply   | db.sales.aggregate(    [      { $project: { item: 1, total: { $add: [ "$price", "$fee" ] } } }    ] )  price와 fee를 곱한다. |
| $substract | db.sales.aggregate(    [      { $project: { item: 1, total: { $add: [ "$price", "$fee" ] } } }    ] )  price와 fee를 뺀다. |

- 그이외에도 날짜 / 논리 함수 / 집합 함수에 대한 다양한 옵션들이 존재한다.
  - https://docs.mongodb.com/manual/reference/operator/aggregation/

#### 집계 파이프라인 옵션

- explain()  

  - 파이프라인 프로세스 세부 정보만 반환한다.

- allowDiskUse

  - 중간 결과를 위해 디스크를 사용한다.
  - 파이프라인은 기본적으로 RAM의 사용량을 제한하기 때문에 사용량을 넘어서게 되면 오류를 뱉어 낸다.
  - 즉 해당 옵션은 디스크에 저장하는 거고 당연하게 I/O 작업이 생기기에 성능저하가 생긴다.

- cursor

  - 파이프라인 결과는 cursor로 반환 된다.

  ```
  cursor.hasNext()
  cursor.next()
  cursor.toArray()
  cursor.forEach()
  cursor.map()
  cursor.itcount() 항목수를 반환한다.
  cursor.pretty()
  ```

  

#### limit

- 파이프 라인의 단계에서는 RAM사용량에 제한이 있다(100MB)
- 하지만 결과 커서를 반환 하거나 , 컬랙션에 저장시 16MB 제한이 있다.
- 대용량 처리에서는 allowDiskUse 옵션을 사용해 집계파이프라인 단계에서 데이터를 임시 파일에 작성(즉 성능저하가 일어난다.)
- $match , $sort에만 인덱스가 사용되고 나머지 파이프라인 연산자에는 인덱스를 타지 않는다.
- $match , $project연산자는 개별 샤드에서 실행되며 다른연산자는 프라이머리에서 실행된다.
  - $match로 시작하면 개별 샤드에서 실행된다.



#### 비교

| sql     | mongodb  |
| ------- | -------- |
| where   | $match   |
| groupby | $group   |
| Having  | $match   |
| select  | $project |
| orderby | $sort    |
| limit   | $limit   |
| sum()   | $sum     |



#

### MapReduce

- aggregation 이전에 집계기능을 제공하는 방법
- 4.2version이후로 deprecated됨
- 집계 프레임워크보다 느리다. 자바스크립트 언어를 사용한다.