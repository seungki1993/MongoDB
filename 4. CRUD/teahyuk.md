# CRUD

기본적인 CRUD 및 추가 기능들이다.

고려해야 할 것은 `many`메소드들은 atomic하지 않다 `단일 document atomic`

## Create

``` sh
db.inventory.insertOne({
  name:"tea",
  age:20,
  comment:"MaEumManEun20Sal ^^"
})
```

``` sh
db.inventory.insertMany([{
  name:"tea",
  age:20,
  comment:"MaEumManEun20Sal ^^"
},
{
  name:"syjung",
  age:80,
  comment:"SiruSiru"
},
{
  name:"seungki",
  age:24,
  comment:"NuLaGo BuReulGgae"
}])
```

## Read

### projection

``` sh
# (item필드와 status필드만 가져온다.)
db.inventory.find( { status: "A" }, { item: 1, status: 1 } )

# _id도 빼고 싶을 떄
db.inventory.find( { status: "A" }, { item: 1, status: 1, _id: 0 } )

# 특정필드 제외 기능 (status,와 instock필드 만 빼고 다가져온다)
db.inventory.find( { status: "A" }, { status: 0, instock: 0 } )
```

### where 또는 search

``` sh
# equals 사용법.
db.inventory.find( { age: 20 } )
```

``` sh
# operator사용법.
db.inventory.find( { age: { $in: [ 20, 24 ] } } )
```

``` sh
# or 연산
db.inventory.find( 
  { $or: [ 
    { name: "syjung" }, 
    { age: { $lt: [ 21 ] } } 
  ]})
```

``` sh
# and 연산
db.inventory.find( 
    { name: "syjung" }, 
    { age: { $lt: [ 21 ] }})
```

``` sh
# embadded 검색 subdocument 전체
db.inventory.find(  { size: { w: 21, h: 14, uom: "cm" } }  )

# embadded 검색 subdocument중 특별 필드만
db.inventory.find( { "size.uom": "cm" } )
```

### array 검색

``` sh
# 순서와 갯수가 정확히 일치해야 한다.
db.inventory.find( { tags: ["red", "blank"] } )

# 순서 no상관 element 포함된 모든 검색
db.inventory.find( { tags: { $all: ["red", "blank"] } } )

# 단일 엘리먼트 포함은 모든 검색이다.
db.inventory.find( { tags: "red" } )

# 조건식은 한개라도 통과하면 검색된다.
db.inventory.find( { dim_cm: { $gt: 25 } } )

# index를 특정하여 검색도 가능
db.inventory.find( { "dim_cm.1": { $gt: 25 } } ) #dim_cm의 1번인덱스가 25이상?

# size검색
db.inventory.find( { "tags": { $size: 3 } } )
```

``` sh
# null이랑 missing field는 둘다 null로 검색 된다.
db.inventory.find( { item: null } )

#결과
{ _id: 1, item: null }
{ _id: 2 }
```

### typeCheck

- [BsonType목록](https://docs.mongodb.com/manual/reference/bson-types/)

```sh
db.inventory.find( { item : { $type: 10 } } ) #BsonType 10 = nullType
```

## Update

대표 기능

- db.collection.updateOne(<filter>, <update>, <options>)
- db.collection.updateMany(<filter>, <update>, <options>)
- db.collection.replaceOne(<filter>, <update>, <options>)


``` sh
# filter에 맞는 첫 docu update
db.inventory.updateOne(
   { item: "paper" },
   {
     $set: { "size.uom": "cm", status: "P" },
     $currentDate: { lastModified: true }
   }
)

# filter에 있는 모든 docu update
db.inventory.updateMany(
   { item: "paper" },
   {
     $set: { "size.uom": "cm", status: "P" },
     $currentDate: { lastModified: true }
   }
)

# _id 필드를 제외한 나머지 모든 필드(즉 docu자체)를 변경 하고 싶을 떄
db.inventory.replaceOne(
   { item: "paper" },
   { item: "paper", instock: [ { warehouse: "A", qty: 60 }, { warehouse: "B", qty: 40 } ] }
)
```

## Delete

``` sh
# 전체 삭제
db.inventory.deleteMany({})
```

``` sh
# 여러개 삭제
db.inventory.deleteMany({ status : "A" })
```

``` sh
# 1개 삭제
db.inventory.deleteOne( { status: "D" } )
```

## 기타 추가 옵션들

### Upsert 옵션

Update나 Replace등등의 메소드들에 `{ upsert: true }` 옵션 추가시에는 Insert Or Update기능을 실행하게 된다.

### Bulk Write

FileI/O 는 줄일 수 없지만, Network 또는 api I/O는 줄일수 있는 방안으로 Bulk Write를 지원한다.

ex)

``` sh
db.characters.bulkWrite([
  { insertOne: { "document": { "_id": 4, "char": "Dithras", "class": "barbarian", "lvl": 4 } } },
  { insertOne: { "document": { "_id": 5, "char": "Taeln", "class": "fighter", "lvl": 3 } } },
  { updateOne : {
      "filter" : { "char" : "Eldon" },
      "update" : { $set : { "status" : "Critical Injury" } }
  } },
  { deleteOne : { "filter" : { "char" : "Brisbane"} } },
  { replaceOne : {
      "filter" : { "char" : "Meldane" },
      "replacement" : { "char" : "Tanys", "class" : "oracle", "lvl": 4 }
  } }
]);
```

#### Ordered vs Unordered Operations

Ordered는 순차쓰기를 지원하면서 실행 도중 실패시 나머지를 취소한다.
또한 순차실행으로 직렬 실행이다.

Unorderd는 병렬 실행이 가능하다, 하지만 Error에대한 보장을 따로 할 수가 없고
실패 하더라도 나머지는 전부 다 실행 된다.

default는 Ordered이다.

``` sh
# Unordered실행
db.characters.bulkWrite([
  <operations>...
  ], 
  { ordered : false }
);
```

### TextSearch

몽고DB가 이상하 짓을 합니다.

`$text`라는 명령어로 텍스트 서치를 지원합니다.
- **내부적으로 `Apache Lucene`을 사용합니다.**
- 텍스트 인덱스를 갖고 있어야 합니다.
- **유료**입니다.
- 성능이 썩 좋진 않다고합니다.
- 참조 : https://docs.mongodb.com/manual/text-search/


### Geospatial Query

지형에 대한 쿼리도 쉽게 지원함.

### Read Isolation

해당 기능을 통해 복제 세트 또는 복제 샤드에서도 일관성있는 데이터가 있는지 까지 확인하며 가져올 수 있다.

### Write Acknowledgment

해당 기능을 통해 복제 세트 또는 복제 샤드로의 쓰기에 대한 고려사항까지 생각하며 Write(Create or Update or Delete) 를 할 수 있다.

## 몽고디비의 CRUD 컨셉

### 읽기에대한 고립, 일관성, 최신 성

 - 몽고디비는 ES와 달리 실시간 성을 갖고 있다.

### 분산된 읽기 쿼리

 - 위에 설명한 Read Isolation, Write Concern등을 이용하여 레플리카셋을 통해 분산된 쿼리를 실행 할 수도 있다.

### Plan 지원

 - 쿼리 플랜을 지원하고 optimize할 수 있다.
