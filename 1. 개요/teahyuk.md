# 몽고DB 개요

몽고디비는 C++로 만든 문서 지향적 NoSQL(NotOnly SQL) 데이터베이스 오픈소스 프로젝트 이다.

## RDBMS, ES와 비교

|RDBMS|Elastic Search|MongoDB|
|--|--|--|
|database|cluster|database|
|table|index|collection|
|row|document|document|
|column|field|field|
|join|multiIndex Search|linking or EmbeddedDocuments|
|pk|_id(prefix)|_id(prefix)|

## 특징

- **Schema-less (Document 디비)**
- **CRUD 는 RDBMS에 비해 엄청 고속으로 동작한다.**
- **Rich한 CRUD 기능을 지원한다 (집계 포함)**
- **Scalability가 뛰어나다 (Auto Sharding 사용한 scale out).**
- **복제가 쉬움**
- **여러 Storage 엔진을 선택 가능하다**
- pk는 자동 설정 (_id)
- join 이 없고 CRUD가 빠르므로 RDBMS처럼 정규식을 통한 여러 테이블로 쪼개는 방식이 아닌 하나의 도메인이라고 느껴지면 최대한 하나의 collection에 크게 많이 저장한다.
- 클러스터링이 가능하다.
- 자유로운 indexing 
  - 그러나 양이 늘어날수록 느려짐...
- 내부적으로 BSON으로 저장한다.(Binary Json)

### documnet

- **Scheam-less!!**
- 위에 설명한 것 처럼 pk 는 "_id"라는 필드로 자동 설정 된다.
- size 제한이 있다, default설정은 16MB
- document Growth 가 비용이 많이든다.
  - join 방식 중에 **embedded 방식**이라고, document내부의 또다른 document를 넣는 방식으로 저장하는 방식이 있는데 만약 내부 document가 늘어난다면 늘어난 document를 update하려면 비용이 엄청 비쌀 것이다,,(차라리 지우고 새 docu를 넣는것이...)
  - 그래서 보통은 **linking방식**을 취한다
    - id들만 넣는 방식을 취하고, app-layer에서 손 조인....
- lock 방식이 read는 여러 lock으로 접근하지만, write는 collection당 한개의 lock만 사용한다. (concurrency 지원)
- 데이터 크기 자체는 RDBMS랑 비교했을때 2.5~3배정도 커진다. (I/O 부하가능성)

### 데이터 저장 방식

먼저 write는 memory에 하고 주기에 따라 다른 프로세스가 memory block을 주기적으로 disk write한다.

파일저장 방식

- NameSpace
  - 메타데이터 {collection이름}.ns
- data(Mapped file)
  - 실제 데이터 {collection이름}.0~
- journal
  - 백업 데이터 prealloc.0~
  - 0.1초마다 100MB를 백업하게 기본 설정되어있음 (조정가능)
  - MappedDate에 60초마다 동기화하는데 동기화완료되면 journal 파일 초기화.

메모리를 많이 잡아먹는다.
I/O도 많이 친다.

MappedFile크기가 2GB인 경우 요구되는 메모리는 10~12GB정도 이다.

#### Storage 엔진

1. WiredTiger Storage Engine
2. In-Memory Starage Engine (inmemory db로도 사용 가능하다는 뜻인듯.)
3. ~~MMAPv1 Storage Engine (현재 4.0 에서 Deprecated)~~ : WiredTiger Storage Engine으로 편입됨.

### ※ 주의 사항

- 클러스터드 인덱스가 없다. (_id를 통한 범위 검색시 Random IO발생->느림)
- 유니크인덱스는 