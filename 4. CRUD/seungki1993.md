## CRUD

- 몽고DB는  쿼리 결과를 Batch 단위로 가져온다
  - 배치 사이즈를 지정하는 방법은 limit를 사용하거나 batchSize를 조절
  - 해당 설정이 없다면 기본적으로 101개를 가지고 오며 1MB를 넘을 수 없다.
  - 설정이 있으면 설정 만큼 가져오나 4MB 이상은 한번에 가져 올 수 없다.
- 기본적으로 db.stocks.find({name:seungki,age:27}) 해당 쿼리는 키 - 값 쌍에 대하여 And연산이다
- 모든 텍스트 문자열은 대소문자를 구분하기 때문에 정규식을 사용하거나 Text 검색을 사용해야한다.
  - 대소문자를 구별하지 않는 옵션을 주거나 , 쿼리 진행시 인덱스를 타지 않는다.
- 정규식을 통해 검색도 가능하다
  - 프릭피스의 경우에는 인덱스를 타지만 다른 쿼리에는 타지 않는다.
- 찾아본결과 언어와 driver마다 지원하는 메소드가 다르고 심지어 몽고쉘에서만 지원하는 메소드도 존재!!! 
  - 사용할때 잘보고 사용해야한다.!

### find vs findOne

- findOne
  - document를 반환한다.
  - 하나의 도큐먼트를 얻고자 할때 사용
- find
  - 커서객체를 반환한다.
  - 커서
    - 쿼리에 대한 결과 집합에 대한 포인터
    - default 만료 시간 10분(타임아웃 설정 가능)
    - 타임아웃을 설정했다면 수동으로 cursor을 close 하거나 cursor를 다 소모해야한다.
    - find()만 사용시 default로 반환되는 개수는 20개
  - 여러 document가 필요할 때 사용
  - 어플리케이션에서 cursor객체를 반복해야한다.

### skip 

- cursor 객체에 메서드 체이닝으로 사용한다.

- db.est.find({age:27}).skip(1);

- skip메소드는 시작위치를 설정하는것

  ```
  일반 find()
  { "_id" : ObjectId("5d5b38a0f475ab33db4bc7da"), "name" : "sdf" }
  { "_id" : ObjectId("5d5b38adf475ab33db4bc7db"), "name" : "sdf", "age" : 11 }
  
  find().skip(1);
  { "_id" : ObjectId("5d5b38adf475ab33db4bc7db"), "name" : "sdf", "age" : 11 }
  ```

- default 파라미터는 0



### limit

- cursor 객체에 메서드 체이닝으로 사용한다.
- 반환할 개수를 설정한다.



### projection

- document에 모든 필드를 가져오는게 아니라 원하는 필드만 가지고 올수 있는 옵션
- db.stocks.findOne({name:"sdf"},**{_id:1}**);
- 1이면 포함 , 0이면 제외



### 범위

- $lt ,$lte,$gt,$gte
- 주의점
  - db.stocks.find({birth:{$gte:100},birth:{$lte:102}}) 는 마지막 lte만 작동한다
  - 올바른 표기 방식은  db.stocks.find({birth:{$gte:100,$lte:102}}) 
- 인덱스 탄다



### 집합연산자

- $in , $all , $nin
- $in
  - 하나라도 포함하고 있는 경우
- $all
  - 모두 일치하고 있는 경우
- $nin
  - 모두 일치하지 않는 경우ㅣ



### 부울 연산자

- $ne, $not, $or, $and, $nor, $exists
- $ne
  - 인수가 요소와 같지 않은 경우
  - 다른 연산자와 같이 사용하지 않으면 인덱스를 타지 않는다.
- $nor
  - 제공된 검색어 집합 중 어떤것도 true가 아닌 경우
- $exists
  - 요소가 도큐먼트안에 존재할 경우 일치
  - 즉 필드가 있냐 없냐



### 임베디드 검색

```
{
	id:1
	detail:{
		model_num : 1,
		age:27
	}
}
```

- db.stocks.find({detail.model_num:1})
- 만약 특정 필드가 아닌 객체에 전체에 질의를 하고 싶다면
  - db.stocks.find({detail:{model_num:1.age:27}})
  - 주의점
    - 해당 쿼리는 바이트 단위로 엄격하게 비교하기 때문에 키값의 순서가 다르다면 검색이 안된다.
    - 쉘은 키의 순서가 유지 되지만 다른 프로그래밍 언어에서는 순서가 유지가 안될 수 있기 때문에 자신이 사용하는 언어에서 오더드 딕셔너리데이터 구조를 지원하는지 확인해야한다. 지원안하면 항상 기 순서 유지



### 배열 검색

```
//ex 1
{
	tags:["solid","peida"]
}


//ex2
{
	address:[
		{
			name:home,
			state:ny
		},
		{
			name:work,
			state:ny
		}
	]
}
```

- db.stocks.find({tags:peida})
- 배열 검색도 엘라스틱서치의 array타입과 동일한 문제를 가지고 있다.
  - 다른 배열에 들어있음에도 쿼리를 날렸을때 해당이 되는문제 
  - 엘라스틱 서치에서는 이를 위해 nested를 붙인다.
  - mongodb는 $elemMatch사용
    - ex) db.stocks.find({address.name:home,address.state:NY}) 내가 원하는건 name이 home이고 state가 ny인것인데 해당 쿼리 실행하면 배열 두개 요소 다 나온다.
    - 올바른 결과를 위해서는
    - db.stocks.find({address:{$eleMatch:{'name':home,'state':'NY'}}})



### Javascript를 활용한 쿼리

- 몽고에서 지원하는 쿼리로 표현할 수 없다면 javascript문법으로 작성가능
- $where절에 사용
- 인덱스도 안타며 , 오버헤드가 발생하기 때문에 권장하지 않음 쓰고 싶다면 몽고에서 지원하는 쿼리로 최대한 document 수를 줄이고 사용
- 인잭션 공격에 취약



### Insert

- insertOne
  - document 저장 성공 시 _id 반환
- insertMany
  - dociment 저장 성공 시 _id 반환
- insert 당시 collection이 없다면 생성
- save 메소드도 가능
  - upsert 와 동일
  - 해당 아이디 값이 있으면 update하고 없으면 save해버린다.



### Update

- updateOne(filter,update,option)
- updateMany(filter,update,option)
- replaceOne(filter,update,option)
- upsert도 가능하다
- save와 다른점
  - update는 해당 필드에 대해서 업데이트를 하지만 save에 경우에는 document자체를 덮어쓰기를 한다
- mongoshell 에서는 db.stocks.update({},{},{multi:true}) 옵션을 줄 수 있는데 해당 옵션은 필터에 맞는 모든 document를 업데이트 한다 만약 true가 아니면 첫번째로 일치하는 도큐먼트만 영향을 준다
- 위치 연산자
  - 배열중 쿼리 셀럭터(filter)와 일치하는 배열 인덱스를 자신으로 대치해 업데이트 할 수 있도록한다.
  - $
  - ex) 배열  detail : [{name:1,value:2},{name:3,value:4},{name:2,value:3}]
  - detail.$.value : 2 이렇게 표시하면 value가 2인 인덱스가 $ 치환된다고 생각하면 쉽다.
- findAndModify
  - 기본적으로 모든 document에 대한 업데이트는 원자성을 보장한다.
  - 기본적은 update와 다른점은 변경후에 document를 반환한다(default는 변경되기 이전의 도큐먼트)
    - new : true를 하면 새롭게 변경된 다큐먼트를 반환한다.
- $set
  - rdb의 set과 동일
- $unset
  - document에서 해당 키를 삭제
- $rename
  - document에서 해당 키의 이름을 변경한다.
- 배열 update
  - $push ,$slice, $pop,$pull
  - $slice
    - ex) [91,92,93]
    - query $slice : -4 [94,95]
    - 배열은 [92,93,94,95]
    - $slice : 4 [94,95]
    - 배열은 [91,92,93,94]
  - pull , pop의 차이점은 인덱스로 삭제하냐 값으로 삭제하냐




### Delete

- deleteOne
- deleteMany



### Text 검색

- text 검색을 하기 위해서는 먼저 텍스트 전용 인덱스를 생성해야한다

```
MongoDB Atlas Full-Text Search Indexes leverage Apache Lucene to power rich text search with features like language analysis and scoring.


MongoDB Atlas 전체 텍스트 검색 색인은 언어 분석 및 평가와 같은 기능을 갖춘 고급 텍스트 검색에 Apache Lucene을 사용합니다.
```

- 루씬가져다가 text검색 만듬

- 컬랙션당 하나의 텍스트 인덱스를 가질 수 있다.

```
db.stocks.createIndex({name:'text'})
```

- 불용어는 무시 된다.
- 가중치도 줄 수 있다 default 1

```
db.stocks.createIndex({name:'text'}.{weights:{title:10}})
```



- 와일드 카드 형식으로 text 인덱스 생성가능

```
db.stocks.createIndex({$**:text})
```

- 기본적인 search 방법

```
db.stocks.find({$text:{$search:'actions'}})
```

- 기본적으로 부분일치도 검색 즉 actions로 검색해도 action도 포함이 된다.
- 역시 마찬가지로 search in action으로 검색하면 각각의 단어로 토큰화 되어서 검색한다
- 특정단어를 꼭 포함하는 document를 찾고 싶다면

```
db.stocks.find({$text:{$search:'"mongodb" in action verrrrrrrrrrry baaaaaaaad'}})
// "" 하면 된다.
```

- 특정 단어를 제외하고 싶다면

```
db.stocks.find({$text:{$search:'"mongodb" in action verrrrrrrrrrry  -baaaaaaaad'}})
// - 추가하면 된다.
```

- score을 표시하기 위해서는

```
db.stocks.find({$text:{$search:'mongodb'},{$meta:"textScore"}})
```

- score로 정렬도 가능

```
db.stocks.find({$text:{$search:'mongodb'},{$meta:"textScore"}}).sort({score:{$meta:"textScore"}})
```

- 다국어도 지원가능
  - 한국어는 없다.