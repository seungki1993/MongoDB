# 인덱스

인덱스는 RDBMS에서의 인덱스와 마찬가지로, 특별한 다른 파일에 다른 방식으로 해당 필드를 순차적으로 정렬하여 각 documentid를 저장해두는 색인이다.

인덱스를 사용하여 검색 시 범위 검색이나, 특별한 인덱스 필드 값에 대한 검색이 훨씬 빨라진다.

## Defatul Index (_id)

몽고DB는 `_id` 필드에 대한 유니크인덱스를 콜렉션 생성시에 자동으로 만든다. 이 인덱스를 drop할 수는 없고, 보통 auto-increment 형식으로 생성된다.

문제점은 해당 인덱스는 **하나의 샤드 내부에서만 유니크**하다는 것이다. 다르게 말하면 이 `_id` 필드를 가지고 샤드키(후에 배울 샤딩을 위한 기준이되는 키)를 만들면 안된다. 이유는 샤드내부에서만 유니크하기 때문에 서로 다른 샤드를 비교하면, `_id`는 중복이 일어나게 되고 샤딩의 기준으로써 동작을 안하기 때문에 문제가 나타난다.

## Index 생성

기본적인 인덱스 생성 방법이다.

쉘
```sh
db.collection.createIndex( { <fieldName>: <type>, <options> } )
db.collection.createIndex( { name: -1, unique : true } )
```

자바
```java
collection.createIndex(Indexes.descending("name"), IndexOptions.unique(true));
```

## Index 타입

### Single Field Index

단일 필드 인덱스 이다. Embedded Field에 대해서도 걸 수 있고 Embedded Docu에도 걸 수 있다.

``` sh
db.records.createIndex( { score: 1 } )
```

``` sh
db.records.createIndex( { "location.state": 1 } )
```

``` sh
db.records.createIndex( { location: 1 } )
db.records.find( { location: { city: "New York", state: "NY" } } )
```

### Multi Key Index

복수 키 인덱스 이다. 

주의할 점은 복수 키 인덱스 안에 **배열인 키 필드가 두개 이상 들어가면 안된다**는 것 이다.

배열인 키 인덱스 필드는 1개만 허용 되며, 인덱스 설정 뒤에는 인덱스 키들 중 2개이상이 배열인 docu를 넣으려하면 에러난다.

샤드키 및 hashed키 속성은 갖지 못한다.

``` sh
db.inventory.createIndex( { "stock.size": 1, "stock.quantity": 1 } )
```

### Text Index

텍스트 속성의 필드들을 위한 인덱스로서 다음과 같은 특징이 있다.

1. 대소문자 무시 ([비 라틴어 및 첨자등 비슷한 단어 포함](http://www.unicode.org/Public/8.0.0/ucd/CaseFolding.txt))
2. 성조 표시 무시 ([유니코드 DB목록에 있는 분음부호](http://www.unicode.org/Public/8.0.0/ucd/PropList.txt))
3. 자동 토큰화 ([유니코드의 텍스트 분류기호 모음](http://www.unicode.org/Public/8.0.0/ucd/PropList.txt))
4. 단어 별 토큰화된 뒤의 인덱싱 3번에서 나온 분류기호 기준.
5. 다중언어지원 (한국어는 없음)
6. sparse 인덱스 (null이거나 필드 없으면 인덱스에서 건너뜀)


또한 여러 파생 옵션들을 갖고 있다.

옵션 목록

- specify name
- weights
- wildcard

#### specify name

텍스트 인덱스는 옵션이 많아서 별칭을 줄 수 있다.

``` sh
db.collection.createIndex(
   {
     content: "text",
     "users.comments": "text",
     "users.profiles": "text"
   },
   {
     name: "MyTextIndex"
   }
)
```

#### weights

각 텍스트마다 가중치를 줄 수 있다.

``` sh
db.blog.createIndex(
   {
     content: "text",
     keywords: "text",
     about: "text"
   },
   {
     weights: {
       content: 10,
       keywords: 5
     },
     name: "TextIndex"
   }
 )
```

이렇게 하면 content에 맞는 문서를 높게 책정하여 찾게 된다.

#### wildcards

필드명에 와일드카드를 주어 조건에 맞는 모든 필드를 텍스트 인덱스로 만들 수 있다. 단 다른 인덱스와 통합은 안되고 텍스트인덱스 들만 모을 수 있다.

``` sh
db.collection.createIndex( { a: 1, "$**": "text" } )
```

#### 제약 사항이 많다.

1. collection당 1개밖에 사용 못함
2. Multi-Key 인덱스, 2dsphere 인덱스(geological인덱스) 와 동시에 사용 못함.
3. 해당 인덱스로 소팅은 안됨.
4. 텍스트 필드값으 다 저장하기 때문에 용량도 배로 잡아먹고, 성능도 안좋아짐.
5. collection 성능이슈로 인해 인덱스 추가 못하게끔 옵션을 줄 수 있다.

### Wildcard Index

위의 텍스트인덱스에서도 잠깐 설명햇지만 원래 몽고db는 인덱스 필드명에 와일드카드를 추가하여 해당되는 모든 필드를 섞은 CompoundIndex를 설정 할 수 있다.

4.2버전에 추가되었다. 단 지금은 일반 인덱스와, 텍스트 인덱스만 가능하다.

### 2dSphere Index

mongoDB에는 특별한 geoJson이라는 지도 좌표 또는 지역을 나타내는 타입을 가진 도큐먼트 필드를 설정 할 수 있는데 이러한 좌표들을 확장해서 특별한 지역 내부에 있는 좌표들을 검색 할 수 있는 2dsphere 인덱스를 지원한다.

2dSphere을 가진 document
``` sh
db.places.insert(
   {
      loc : { type: "Point", coordinates: [ -73.97, 40.77 ] },
      name: "Central Park",
      category : "Parks"
   }
)

db.places.insert(
   {
      loc : { type: "Point", coordinates: [ -73.88, 40.78 ] },
      name: "La Guardia Airport",
      category : "Airport"
   }
)
```

index설정 법

``` sh
db.collection.createIndex( { loc : "2dsphere" } )
```

index를 사용한 검색 방법

``` sh
db.<collection>.find( { <location field> :
                         { $geoIntersects :
                           { $geometry :
                             { type : "<GeoJSON object type>" ,
                               coordinates : [ <coordinates> ]
                      } } } } )

db.places.find( { loc :
                  { $geoWithin :
                    { $geometry :
                      { type : "Polygon" ,
                        coordinates : [ [
                                          [ 0 , 0 ] ,
                                          [ 3 , 6 ] ,
                                          [ 6 , 1 ] ,
                                          [ 0 , 0 ]
                                        ] ]
                } } } } )                     
```

### ~~2d Index~~

해당 인덱스는 위의 2dSphere Index의 레거시버전(2.2 이하) 좌표 docu를 위한 인덱스이다.

- [참고](https://docs.mongodb.com/manual/core/2d/)


### geoHaystack Index

지도좌표 document들 중 geoHaystack Index라는 특별한 인덱스는, 추가 필드들과 함께 아주작은 지역 좌표들을 위해 최적화된 인덱스들이다.

내부적으로 bucket이라는 영역을 만들어 인덱스를 활용한 검색시 성능을 최적화 시킨다.

추가 필드도 사용 할 수 있으며 bucket size는 도메인에 맞게 최대한 작게 줄이는게 적절한 방식 중 한가지이며 sparse는 false로 고정되어 무조건 null 또는 no key 를 제외시킨다.

생성

``` sh
db.coll.createIndex( { <location field> : "geoHaystack" ,
                       <additional field> : 1 } ,
                     { bucketSize : <bucket value> } )
```

``` sh
{ _id : 100, pos: { lng : 126.9, lat : 35.2 } , type : "restaurant"}
{ _id : 200, pos: { lng : 127.5, lat : 36.1 } , type : "restaurant"}
{ _id : 300, pos: { lng : 128.0, lat : 36.7 } , type : "national park"}

db.places.createIndex( { pos : "geoHaystack", type : 1 } ,
                       { bucketSize : 1 } )
```

사용 법

``` sh
db.runCommand( { geoSearch : "places" ,
                 search : { type: "restaurant" } ,
                 near : [-74, 40.74] ,
                 maxDistance : 10 } )
```

### hashed Index

해싱을 이용한 인덱스다.

샤딩을 위한 샤드키로 사용 될 수 있다.

``` sh
db.collection.createIndex( { _id: "hashed" } )
```

주의점은 **64비트 integer로 내림을 실행하니 해시결과 값이 float형이면 충돌이 날 수 있다는 것** 과 embedded document를 쓰면 충돌 날 수 있으니 조심해야 한다.

## 인덱스 속성

### TTL 인덱스

말 그대로 Time To Live 인덱스로써 해당 인덱스를 이용하면 collection에서 document를 삭제 하게끔 만들 수 있다.

보통 time 형태의 필드에 지정한 인덱스에 해당 속성을 추가 할 수 있다.

``` sh
db.eventlog.createIndex( { "lastModifiedDate": 1 }, { expireAfterSeconds: 3600 } )
```

### Unique Index

인덱스가 Unique한지 설정하는 속성이다. 충돌이 나면 insert가 안된다.

### Partial Index

특정 조건을 이용해 해당 조건을 만족하는 필드를 가진 document들만 인덱싱 하는 방법이다.

``` sh
db.restaurants.createIndex(
   { cuisine: 1, name: 1 },
   { partialFilterExpression: { rating: { $gt: 5 } } }
)
```

### CaseInsensitive Index

기본은 케이스 무시지만 해당 인덱스 속성을 통해 케이스를 무시하지 않게끔 사용 할 수 있다.

``` sh
db.fruit.createIndex( { type: 1},
                      { collation: { locale: 'en', strength: 2 } } )
```

strength 는 1,2만 사용 가능하다 **[참조](https://docs.mongodb.com/manual/reference/collation/#collation-document-fields)**

collation을 사용 할 때에는 default가 1 이다.

### Sparse Index

null값을 포함할지 말지에 대한 속성이다.

``` sh
db.addresses.createIndex( { "xmpp_id": 1 }, { sparse: true } )
```

## Index 적용

explain을 쓰면 쿼리 시에 인덱스를 거쳐갔는지 확인 가능하다.

#### 순서 적용 

compound index는 먼저 각 인덱스 필드의 순서가 중요한데 앞의 필드만 검색 시에는 인덱스가 적용이 되지만 뒤의 필드 사용시에 인덱스적용을 하기 위해서 앞단의 필드도 조건을 걸어서 검색해야 된다.

다음의 컴파운드 인덱스 실행 시

``` sh
db.collection.createIndex({ status: 1, ord_date: -1 })
```

적용되는 예

``` sh
db.collection.find( { status: { $in: ["A", "P" ] } } )
db.collection.find(
   {
     ord_date: { $gt: new Date("2014-02-01") },
     status: {$in:[ "P", "A" ] }
   }
)
```

적용 안되는 예

``` sh
db.collection.find( { ord_date: { $gt: new Date("2014-02-01") } } )
db.collection.find( { } ).sort( { ord_date: 1 } )
```

따라서 다른 필드 단독으로 인덱스적용하고 싶으면 각각 인덱스를 2번 생성하면 된다..

``` sh
db.collection.createIndex({ status: 1 })
db.collection.createIndex({ ord_date: -1 })
```

#### sort 와 find의 혼용

인덱스가 여러개 있을 때 한쪽 인덱스로 검색하고 다른쪽 인덱스로 sorting하는 등의 행위는 할 수 없다. (인덱스미적용됨.)

find와 sorting을 동시에 사용 할 때에는 반드시 양쪽에 다 걸치는 인덱스가 존재해야 한다.

다음의 collection에 여러개의 인덱스 적용 되어있을 때

``` sh
db.collection.createIndex({ qty: 1 })
db.collection.createIndex({ status: 1, ord_date: -1 })
db.collection.createIndex({ status: 1 })
db.collection.createIndex({ ord_date: 1 })
```

적용되는 예

``` sh
db.collection.find( { qty: { $gt: 10 } , status: "A" } ).sort( { ord_date: -1 } )
```

적용 안되는 예

``` sh
db.collection.find( { qty: { $gt: 10 } } ).sort( { status: 1 } )
```

## Index 측정 api

몽고디비는 인덱스에대한 많은 측정 api를 제공한다

1. $indextStats : 인덱스 사용량 통계
2. explain : 해당 쿼리가 인덱스를 이용하는지
3. hint : 쿼리에 인덱스 이용 할 수 있도록 힌트제공

## [인덱스 사용 전략](https://docs.mongodb.com/manual/applications/indexes/)

인덱스는 이럴때 사용 하면 좋다고 나와있음

1. 쿼리 support
2. 쿼리 sorting support
3. disk말고 메모리 사용을 위한 전략
4. 선택성을 보장하는 쿼리를 위한 전략