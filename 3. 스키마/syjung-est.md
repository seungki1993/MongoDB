## 스키마
 - 컴퓨터 과학에서 데이터베이스 스키마(database schema)는 데이터베이스에서 자료의 구조, 자료의 표현 방법, 자료 간의 관계를 형식 언어로 정의한 구조이다.
 - MongoDB는 스키마가 따로 정해져 있지 않지만 모든 애플리케이션은 데이터가 어떻게 저장되는지에 대한 기본적인 내부 기준 정도는 존재해야한다.

### 데이터베이스
 - 컬렉션과 인덱스의 물리적인 모음이며, 네임스페이스이다.
 - 데이터베이스내의 컬렉션에 쓰기를 하면 자동으로 생성된다.
 - db 접근
~~~
 use {database}
~~~
 - db 삭제 : 컬렉션을 지우는것은 단순히 그 안에있는 데이터를 지우는것. drop으로 따로 db를 지워줘야함.
~~~
  db.ropDatabase();
~~~
 - dbPath(지정되어있다면) | /data/db/ 에 물리적으로 저장되어있다.
~~~
 {database}.0
 {database}.1
 {database}.{N}
 {database}.ns
 mongod.lock
~~~
|항목|설명|
|--|--|
|mongod.lock|서버의 프로세스ID를 저장한다.|
|{database}.ns| - ns(=name space)<br> - 데이터베이스 내의 각각 컬렉션과 인덱스에 대한 메타데이터. 해시테이블로 구성되어있다.<br> - default 16MB<br> - 약 26,000개의 엔트리를 저장할 수 있다(=하나의데이터베이스에서 인덱스와 컬렉션의 갯수가 26,000개를 넘어설수 없다.)<br> - 26,000개 이상 넣고싶다면 size조절 가능 <br> - WiredTiger에선 사용하지않음. |
|{database}.{N}| - 실제 데이터가 저장되는부분. <br> - ns파일을 생성하고 컬렉션과 인덱스를 위한 공간을 미리 파일에 할당해놓음.<br> - 데이터를 업데이트하거나 질의할 때 연산이 디스크의 여기저기 흩어져 있는 데이터가 아니라 인접된 데이텅 대해 수행되도록 |
 - db.stats : db 상태를 보여줌
~~~
{
  "db": {database},
  ...
  "fileSize": {database}.{N} 파일들의 합,
  "dataSize": database에서 BSON객체의 실제 크기 ,
  "storageSiae": 컬렉션이 증가할 것을 대비한 여분의 공간+삭제되었지만 아직 할당되지 않은 공간,
  "indexSize": database 인덱스의 전체 크기
  ...
}
~~~

### 컬렉션
 - 구조적으로 혹은 개념적으로 비슷한 도큐먼트를 담고 있는 컨테이너.
#### 컬렉션 관리
 - 컬렉션 생성
 - size는 옵션, 알파벳이나 숫자로 시작해야함.
~~~
 db.createCollection("컬렉션명", {size: size})
~~~
 - 컬렉션 이름 변경
~~~
 db.{old_컬렉션명}.renameCollection("new_컬렉션명")
~~~
#### 캡드 컬렉션
 - 높은 성능의 로깅 기능을 위해 설계됨.
 - 고정된 크기를 갖고있음.
 - 캡드 컬렉션이 더이상의 공간이 없게 되면 도큐먼트를 삽입할 때 가장 오래된 도큐먼트를 덮어쓰게된다.
 - size뿐만아니라 max 매개변수로 캡드 컬렉션에 대해 도큐먼트 최대개수를 지정할 수 있다.
 - createCollection할때 caped:true|false 설정으로 만들수있음.
 - 개별도큐먼트 삭제, 업데이트 불가

#### TTL(Time-To-Live) 컬렉션
 - 특정 시간이 경과한 도큐먼트를 expire 시킬 수 있다. (expired보다는 delete)
~~~
 db.{컬렉션}.createIndex({기준필드}, {expireAfterSeconds: N초후 삭제})
~~~
 - id필드 기준으로 사용불가
 - 캡드 컬렉션 사용불가

#### 시스템 컬렉션
 - system.namespace, system.indexes 표준컬렉션, 디버깅할때 유용함

### 도큐먼트
 - 모든 도큐먼트는 MongoDB에 저장하기 전에 BSON으로 Serialize
 - 읽을땐 JSON으로 다시 DeSerialize된다.
 - max 16MB
 - 최대 100 depth

#### 문자열
 - UTF-8

#### 숫자
 - double, int, long

#### 날짜
 - datatime, 64비트 정수를 사용하여 유닉스 millisecond로 표현

#### 가상타입
 - 구지 타입에 구애받지 않고 알아서 Json객체를 만들어서 사용할 수 있다.
 - ex>
 ~~~
 {
   time_with_zone: {
     time: new Date(),
     zone: "EST"
   }
 }
 ~~~

### RDBMS, ES와 비교
|RDBMS|Elastic Search|MongoDB|
|--|--|--|
|database|cluster|database|
|table|index|collection|
|row|document|document|
|column|field|field|
|join|multiIndex Search|linking or EmbeddedDocuments|
|pk|_id(prefix)|_id(prefix)|
