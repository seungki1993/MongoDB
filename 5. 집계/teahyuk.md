# Aggregation

몽고DB는 집계를 위해 3가지 기능을 제공한다.

- 파이프라인
- 맵 리듀스
- 단일 목적 집계 함수

## Pipeline

집계를 위한 파이프라인 프레임워크. 라고 소개함.

네티, RXjava와 비슷한 개념으로 생각하심 됨.

``` sh
db.orders.aggregate([
   { $match: { status: "A" } }, # "A"라는 status필드와 맞는 docu 를 전달
   { $group: { _id: "$cust_id", total: { $sum: "$amount" } } } # cust_id필드를 중심으로 그룹화 해서 amount필드의 합계를 구함
])
```

가장 많이 사용하는 집계방법 중 하나다.

pipeline최적화를 통해 효율성을 증대 시킬 수 있다. (다른 작업보다 최적화가 쉽다.?)

### 제약 사항

- 결과값 최대 16MB
- 각 stage별 최대 100MB만 쓸 수 있다.
  - `allowDiskUse`옵션으로 올릴 수 있다. 대신 파일 사용.
  - `$graphLookup`을 사용한 stage는 `allowDisUse`옵션도 필요없다.

### 샤드에서의 사용법

- 샤드키를 이용해 $match로 특정 샤드에서만 뽑아서 사용하게 하면 해당 샤드에서만 진행한다.
- 마지막 $out 스테이지랑 $lookup스테이지에서만 기본 샤드를 필요로 한다.

## map-reduce

전통적인 맵 리듀스를 사용한 내부적으로 자바스크립트 문법을 지원하는 집계기능을 제공한다.

``` sh
db.orders.mapReduce(
  function() { emit(this.cust_id, this.amount);},
  function(key.values) { return Array.sum(values)},
  {
    query: {status:"A"},
    out:"order_totals"
  }
)
```

## 단일 집계 함수

다음의 기능들을 지원한다

- estimatedDocumentCount()
- count()
- distinct()

``` sh
# cust_id 필드종류들 뽑기
db.orders.distinct("cust_id")
```

## ** ES와의 차이점 **

집계는 파이프라인을 사용하여 집계함수를 제공하며, 외부적으로 `샤드키`를 이용해 샤딩된 데이터들의 분포가 어느샤드에 있는지 알기 때문에 집계 실행시 `샤드` 중심의 최적화를 이뤄낼 수 있다.

ES는 그런거 없다.

파이프라인도 없다.