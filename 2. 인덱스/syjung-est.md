## 인덱스
 - 잘만든 인덱스는 MongoDB는 하드웨어를 효율적으로 사용할 수 있고 쿼리를 빠르게 처리할 수 있지만, 잘못만들어놓으면 쿼리가 느려지고 하드웨어를 효율적으로 사용하지 못한다.

### 인덱스 개요
#### 단일키 인덱스
 - 인덱스 내의 각 엔트리는 인덱스 되는 도큐먼트 내의 한 값과 일치한다.
 - ex > id에 대해 default로 생성되는 인덱스, 빠른 검색을 위해 인덱스에 저장됨.

#### 복합키 인덱스
 - 하나 이상의 키를 사용하는 인덱스
 - 단일 인덱스만 사용하면 좋겠지만, 하나 이상의 속성으로 질의해야 할 경우를 위해 복합키 인덱스를 제공한다.

 |Key|디스크주소|
 |---|---|
 |Ace-8000|Ox12|
 |Acme-7699|OxFF|
 |Acme-7500|OxA1|

 |Key|디스크주소|
 |---|---|
 |8000-Ace|Ox12|
 |7999-Biz|OxF1|
 |7980-Dream|OxA2|
 - 복합키로 인덱스를 생성했을 때 어떤식으로 저장되어있는지 보여줌.
 - 복합키를 정의할때 키의 순서를 잘 정해야함. 중요하다. 순서대로 찾는다.

#### 인덱스 효율
 - 읽기 위주의 애플리케이션에서는 인덱스 비용은 인덱스로 인해 얻을 수 있는 효과로 상쇄된다. -> 당연한말..;
 - 인덱스에는 비용이 발생하니 조심해서 생성해야한다.
 - 새로운 데이터가 추가, 삭제, 재배치될 경우 인덱스에 대한 수정이 일어난다.
 - WiredTiger엔진에서는 모든 도큐먼트+컬렉션+인덱스(=페이지라고 부르는 4KB의 청크)는 운영체베에 의해 램으로 스왑됨. -> 해당 페이지에 대한 데이터가 요정될때마다 OS는 그 페이지가 램에 있는지 확인해야함. -> 없으면 -> pageFault 가 일어나고 메모리관리자는 해당 페이지를 디스크 -> 램으로 불러옴 -> 빈번하게 디스크 액세스가 일어난다면 -> thrashing현상으로, 성능 저하!
 - 즉, 인덱스가 너무 많으면 성능저하를 일으킬 수 있다. 꼭 필요한것만 생성하고 순서를 고려하여 잘 만들어야한다.

### MongoDB 인덱스
 - WiredTiger는 B-Tree와 LSM 둘다 지원함. 성능비교해놓은 사람이 있다. 책에는 B-Tree에 대한 설명밖에 없음.
 - 같이봅시다 : https://github.com/wiredtiger/wiredtiger/wiki/Btree-vs-LSM

#### 인덱스 생성
~~~
 db.{db명}.createIndex(옵션)
~~~
 - 순서대로 오름차순(1), 내림차순(-1) 로 정할수있음

#### unique
~~~
 db.{db명}.createIndex({필드명:1or-1}, {unique: true})
~~~
 - PK로, 같은 username값을 넣으려고 하면 duplicated error
 - 당연한 말이지만, 기존에 있던 데이터에 unique 걸순 있지만 에러날수있음.
 - dropDups로 중복된 항목들을 지울수 있지만 랜덤임

#### 희소인덱스
 - 인덱스의 키가 null이 아닌 값을 가지고 있는 document들만 존재한다.
 - 데이터 키가 대부분 null인 경우 유용하다.
~~~
 db.{db명}.createIndex({필드명:1or-1}, {unique:true, sparse: true})
~~~

#### 다중키 인덱스
~~~
 db.{db명}.createIndex({필드명:1or-1, "필드명2:1or-1" ... }, {unique: true})
~~~

#### 해시 인덱스
~~~
 db.{db명}.createIndex({필드명:'hashed'})
~~~
 - equals query는 정상작동, 범위 쿼리는 지원되지 않음.
 - 다중키 해시 인덱스는 허용되지 않는다.
 - 부동 소수점값은 해시가 되기 전에 정수로 변환 (ex> hashed 4.1 = hashed 4.2)
 - key와 Data가 균등하게 분배되지 않을 때 사용한다.

#### 지리공간적 인덱스
 - 각 document에 저장된 위도값과 경도값에 따라 document를 특정 위치에 가까이 배치한다.

### 인덱스 관리
 - ensureIndex() -> createIndex()로 바뀜
 - 인덱스 생성할때
    1. 인덱스할 값을 정렬한다.
    2. 정렬된 값들이 인덱스로 삽입된다.

#### 인덱스 검색
~~~
 db.system.indexes.find()
~~~

#### 인덱스 삭제
~~~
 db.{db명}.dropIndex("index명")
~~~

#### 인덱스 재구성
 - Changed in version 4.2, Deprecated됨 
~~~
 db.{db명}.reindex();
~~~

#### 오프라인 인덱스
 - 한 복제 노드를 오프라인 상태로 바꾸고 인덱스 구축 후 마스터 노드로부터 업데이트 받는다. 그 이후에 이 노드를 프라이머리 노드로 변경하고, 다른 세컨더리 노드들을 오프라인 상태로 바꾼 후에 인덱스 구축

### 기타 ~
 - 이미 존재하는 데이터의 양이 많은 상태에서 Index를 생성하려면 오래걸릴수있음.
 - 되도록 첨부터 구축하고 쓰세요
 - 인덱스 생성되는중엔 Client들이 데이터 읽기, 쓰기 불가능 -> background 인덱싱 가능
