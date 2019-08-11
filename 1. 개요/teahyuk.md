# 몽고DB 개요

몽고디비는 문서 지향적 NoSQL(NotOnly SQL) 데이터베이스 이다.

언어는 C++

## RDBMS 와 비교

|RDBMS|MongoDB|
|--|--|
|database|database|
|table|collection|
|row|document|
|column|field|
|join|linking or EmbeddedDocuments|
|pk|_id|

## 특징

- Schema-less
- pk는 자동 설정 (_id)
- join 은 가능하지만 비효율 적이다.
- CRUD 는 RDBMS에 비해 엄청 고속으로 동작한다.
- join 이 굉장히 느리고 CRUD가 빠르므로 RDBMS처럼 정규식을 통한 여러 테이블이 아닌 하나의 도메인이라고 느껴지면 최대한 크게 많이 저장한다.
- Scalability가 뛰어나다.
- 클러스터링이 가능하다.
