### MongoDB 스키마

- 스키마를 강제하지는 않지만 내부 기준 정도는 존재해야한다.

- URL에 document의 ID를 노출하고 싶지 않으면 slug를 사용해라
  - slug를 사용할 것이라면 고유한 인덱스를 생성해라

- MongoDB에서도 관계를 설정 할 수 있다.

  - RDBMS처럼 테이블을 나누고 JOIN을 하는 형식이 아니다.
  - document안에 RDBMS처럼 관계가 있는 document의 ID값을 명시한다.(Normalized Data Models, references )
    -   다대다
  - 혹은 중첩도큐 먼트를 사용해서 표현할 수 있다.(Embedded Data Models)
    -   일대일 , 일대다 관계

- 데이터 베이스를 생성하는 방법은 특별히 없으며 데이터 베이스내에 컬랙션에 쓰기 작업을 수행하면 데이터베이스가 생성이 된다.
  - 쓰기 작업이 수행 될때 데이터베이스가 디스크에 할당이 된다.

- mongodb 는 data path
  - centos 기준 /var/lib/mongo
  - 해당 폴더에 컬랙션 , 인덱스, 데이터베이스에 대한 메타정보가 들어 있다.
  - mongodb.lock
    -   서버 프로세스 ID를 저장
  - DataBaseName.ns 파일
    -   .ns 파일을 namespace
    -   컬랙션과 인덱스에 대한 메타 데이터로 hashTable로 구성
    -   26000개의 entry 저장가능(인덱스 + 컬렉션)
    -   default 16MB
  - DataBaseName-[0-9]*-~~
    -   데이터가 저장 되는 파일
    -  최대한 많은 데이터가 디스크에 연속적으로 저장되도록 미리 할당한다

- collection을 생성하는 방법
  - db.createCollection
  - collection은 이름은 128 이하
  - 컬랙션은 이름을 수정 할 수 도 있다.

- 캡드 컬랙션
  - 저번 설명처럼 일정한 사이즈를 가지고 있는 컬랙션
  - 컬랙션의 사이즈가 넘어가게 되면 오래된 도큐먼트를 덮어쓰는 방식
  - 단점은 개별 document를 삭제 할 수 없으며 document의 사이즈가 증가되는 작업은 할 수 가 없다.

- TTL 컬랙션
  - 사실 컬랙션이라는 개념은 아니고 index에 설정하는 값이다.
  - 설정된 시간보다 오래 된 document를 자동으로 삭제하는 기능
  - 제약사항
    - 인덱스가 이미 걸려 있는 field라면 ttl은 사용할 수 없다.

- Mongodb는 자체적으로 사용하는 컬랙션이 있다.
  - system.namespaces 
  - system.indexes

-   MongoDB 스키마 Validation
    -   validationLevel
        -   업그레이드 중에 기존 문서에 유효성 검사 규칙을 적용하는 정도
        -   moderate
            -   기존 데이터에 대해서 validation을 지키고 있는 document가 있으면 update 시에도 validation을 검사
            -   기존 데이터가 validation을 지키고 있지 않으면 update시 validation 검사를 하지 않는다.
        -   strict
            -   default
            -   모든 문서에 대한 삽입 , 업데이트에 대해 유효성 검사
    -   validationAction
        -   유효하지 않은 문서를 허용할지에 대한 여부
        -   error(default)
            -   유효성 검사를 위반하는 document는 허용하지 않는다.
        -   warn
            -   유효성 검사를 위반한 document도 삽입 , 업데이트 가능
     -  3.6 버전 부터 jsonSchema validator 지원
        ```
        // validation 지키지 않으면 오류 발생
        db.createCollection("students", {
           validator: {
              $jsonSchema: {
                 bsonType: "object",
                 required: [ "name", "year", "major", "address" ],
                 properties: {
                    name: {
                       bsonType: "string",
                       description: "must be a string and is required"
                    },
                    address: {
                      bsonType: "object",
                      required: [ "city" ],
                      properties: {
                      street: {
                        bsonType: "string",
                        description: "must be a string if the field exists"
                       },
                       city: {
                        bsonType: "string",
                        "description": "must be a string and is required"
                      }
                    }
                   }
                 }
              }
           }
        })
        ```
    -   $jsonSchema 이외에도 다양한 옵션이 존재
    -   기본적으로 mongodb에는 local, config, admin 데이터베이스가 존재하는데 해당 데이터베이스에는 validation 설정 불가
    -   마찬가지로 system이 들어가 있는 collection에도 불가
        -   system이 붙은 collection은 mongodb에 자체적인 컬랙션이다.
    -   bypassDocumentValidation 옵션으로 validation 생략 가능

    
  