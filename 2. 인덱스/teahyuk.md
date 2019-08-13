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
db.collection.createIndex( { <fieldName>: <sort(-1,1)>, <options> } )
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

### Text Indexes

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

필드명에 와일드카드를 주어 조건에 맞는 모든 필드를 텍스트 인덱스로 만들 수 있다.

``` sh
db.collection.createIndex( { a: 1, "$**": "text" } )
```

#### 제약 사항이 많다.

1. collection당 1개밖에 사용 못함
2. Multi-Key 인덱스, 2dsphere 인덱스(geological인덱스) 와 동시에 사용 못함.
3. 해당 인덱스로 소팅은 안됨.
4. 텍스트 필드값으 다 저장하기 때문에 용량도 배로 잡아먹고, 성능도 안좋아짐.
5. collection 성능이슈로 인해 인덱스 추가 못하게끔 옵션을 줄 수 있다.
