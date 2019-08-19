# 데이터모델

몽고DB에서의 데이터 모델링을, ElasticSearch 및 SQL과 비교하여 정리한다.

## Flexible Schema

Document 지향의 Schema-less DB 로써 SQL과는 결이 다르며 Elastic-Search와는 같은 결을 유지한다.

ElasticSearch와 다른 점은 같은 collection,index 내의 서로 다른 document가 같은 field 내에서 서로 다른 타입을 갖는 경우를 허용 한다는 것이다.

## Document Structure

몽고DB에서의 데이터 모델링의 중점은 RDBMS와 비슷하게 어떻게 데이터간의 관계가 이루어지는지에 대한 매핑하는지에 있다.

몽고DB는 RDBMS와 다르게 Document 안에 embadded Document를 넣는게 가능하다.

### Embadded Data

하나의 document안에 subDocument를 같이 저장하는 방식이다.

Reference방식 보다 읽기 및 업데이트가 훨씬 빠르며, 원자성이 보장되지만 document 자체 size 제한인 16MB에 걸릴 수 있고, 수정시 여러개의 document를 수정해야 할 수 있다.

### References

참조 방식이며, Embadded 방식보다 읽기 속도가 현저히 느리다.

Embadded 방식을 사용 했을 때에 데이터 중복으로 인한 손해가 많을 때에 `References방식`으로 바꾸거나 Many-to-Many 방식 또는 계층구조가 클 때 사용 하는게 좋다.

집계 함수 중 [$lookup](https://docs.mongodb.com/manual/reference/operator/aggregation/lookup/#pipe._S_lookup) 또는 [$graphLookup](https://docs.mongodb.com/manual/reference/operator/aggregation/graphLookup/#pipe._S_graphLookup) 를 사용하여 조회 가능하다.

## Atomicity of Write Operations (쓰기 원자성)

### Single Document Atomicity

몽고DB는 기본적으로 하나의 document에 대한 쓰기에 대해 원자성을 지원한다.

이로인해 얻을 수 있는 이점은 Embadded도 결국 하나의 document안에 들어가 있으므로 Embadded로 실행시 원자성에대한 이득을 볼 수 있다는 것이다.

한꺼번에 여러개의 update가 발생시에도 하나의 document에 대해서만 지원하므로 중간에 다른 작업이 interrupt걸 수 있다.

### Multi-Document Transections

필요한 상황에서는 `Multi-Document Transactions Api`를 이용하여 원자성을 만들 수 있다.

ES의 BulkIndex기능과는 반대로 해당 기능을 이용시에 몽고DB는 훨씬 더 비효율 적으로 작업하게 되므로, 해당 기능을 의존하여 사용 하는것 보단 설계를 잘 해서 Bulk와 상관 없이 작업 하는 것이 좋다.

## Data Use and Performance(Data 사용 성 및 성능)

몽고 DB에 대해 데이터 사용성에 대한 모델링을 먼저 하고서 저장하는것이 좋다. 보통 1차DB로 많이 사용 하며, 데이터 목적성을 가지고 최근 데이터만 필요 할 시에는 `CappedCollection`을 많이 사용하고, 읽기가 쓰기에 비해 많이 빈번하면 `index`를 추가 하는것이 좋다.

ES는 검색을 위한 시스템으로 역색인 구조의 full index 구조이지만, 몽고DB는 저장을 위한 용도로, index는 선택적이며 index가 많아질 수록 insert에 대한 부하가 커진다.

## Schema Validation

Schema-less 한 특성으로 인해 document가 망가지는것을 방지하기 위해서 `Schema Validation`에 대한 기능을 제공한다.

Collection단위로 설정이 가능하며, ES의 Mapping과 다른 부분은 ES는 효과적인 indexing 및 analyzing을 위해서 작업하는 부분이라면 (제약을 따로 걸지 않음) 몽고 DB는 제약을 위해서 추가하는 API이기에 해당 제약 사항에 위반하는 document insert 시 에러를 뱉어낸다. (에러메세지 까지 설정 할 수 있다.)

참조 https://docs.mongodb.com/manual/core/schema-validation/

## 모델링을 위한 몇가지 고려사항

### Atomicity

 - 위에 설명한 것 처럼 document단위에서의 원자성만 지원하므로 embadded vs reference방식을 고려해야한다.

### Sharding

 - 샤드키 설정 방식에 따라 성능에 영향이 미치는데 샤드키는 전체 document들 중 분포도가 고르게 나타나는것을 사용 하는게 좋다.

### 대량의 Collection

 - 특정 상황에 있어서 Collection을 쪼개는 것도 고려해야 한다.
 - 몽고디비는 Collection에 대한 메타데이터 저장소인 NameSpace파일에 대한 제한이 있으므로 이 NameSpace가 너무 커지는 방향이 되면 데이터를 특정 기준으로 나누는게 좋다.

### 대량의 작은 Document로 이루어진 Collection

 - 몽고디비는 join, 즉 여러 Collection 조회시 비용이 크므로 작은 Docu로 이루어진 Collection의 경우 연관된 다른 Collection으로의 편입도 고려 할 수 있다, 이를 `roleup`이라 한다.

### 작은 Document의 저장소 최적화

 - 기본적으로 Collection당 오버헤드 크기가 크기 때문에 Document크기 자체가 너무 작으면 예상한 Data보다 사용하는 양이 너무 커질 수 있다.
 - Field의 Key값도 모든 document가 갖고 있기 때문에, Key값의 길이도 줄이는게 좋다.

### Data Lifecycle 관리

 - 몽고DB는 데이터 Lifecycle 에 대한 유효한 기능을 지원한다
   - capped Collection
   - TTL Index
