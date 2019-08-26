# Sharding

데이터 용량이 커지면 단일 서버로는 리소스에 대한 한계가 있을 수 있기 때문에 몽고DB는 Scale Out을 위해 샤딩을 지원한다.

## Sharded Cluster

샤드 클러스터의 구성은 다음과 같다

|엘라스틱 서치|몽고DB|
|---|---|
|노드|샤드|
|마스터노드,Ingest Node|mongos|
|마스터노드, 레플리카 샤드|Config Servers|
|샤드|청크|

### Shards

샤드는 클러스터를 구성하고있는 데이터들의 서브셋이다. 3.6 이후로는 가용성과 fail over를 위한 대체 레플리카 셋을 반드시 포함 하고 있어야 한다.

샤드에 직접 쿼리하면 샤드 내부의 subset만 반환한다, **mongos라는 프론트 라우터가 있어야 클러스터링이 완성된다.**

#### 기본 샤드

클러스터의 각 데이터베이스들은 각각 샤딩되지 않은 모든 collection을 갖고있는 기본 샤드를 갖고있다.

기본샤드는 복제셋의 `프라이머리`와는 연관성이 없다.

몽고스는 새로운 `database`를 생성 할 때마다 가용량이 많이 남은 샤드들을 확인하고 해당 샤드를 `기본 샤드`로 설정한다.

![기본샤드그림](https://docs.mongodb.com/manual/_images/sharded-cluster-primary-shard.bakedsvg.svg)

기본샤드를 변경하기위해서는 movePrimary 커맨드를 사용하면 된다. 

 - 해당 커맨드는 마이그레이션 동작을 하기 때문에 시간과 리소스를 잡아먹는다.

#### 샤드 상태

`sh.status()`를 사용하면 클러스터상태와 샤드내부의 청크 분포 상태 기본샤드 등등을 알 수 있다.

#### 샤드 클러스터 Security

각 샤드 실행 시 마다 (mongod) 실행 시 마다, 인증과정을 설정 해야 허가되지않은 다른 클러스터 구성요소가 해당 샤드로 불법 침입하는 것을 막을 수 있다.

- 보안을 위해서는 각 모든 샤드마다, 구성요소 마다 보안설정 해야 한다.

##### 샤드 로컬 유져

각 샤드들은 RBAC라는 롤기반의 접근제어를 제공한다. (mongod option `--auth`)

### Config Servers

설정 서버는 샤드클러스터의 메타 데이터를 저장하는데, 메타 데이터는 데이터 청크가 어떤 샤드에 위치하는지, 데이터의 상태가 어떤지, 청크 구조 정의 에 대한 정보들이 들어가 있다.

`mongos`는 해당 메타 데이터를 캐싱하고 메타 데이터를 가지고 각 샤드에 접근한다, 캐시된 메타데이터를 갱신하는 것은 청크의 분리나, 샤드가 추가되는 등의 변경이 일어날 때 `config server`를 통해 갱신한다.

`RBAC`라는 각 샤드들에 대한 접근제한 설정도 갖고있다.

주의 할 것은 각 클러스터마다 각각의 config server 를 사용 해야한다, 서로다른 클러스터가 같은 config server를 사용하면 안된다.

config server 를 통한 작업들은 샤드 클러스터 성능 및 가용성에 상당한 영향을 줄 수 있으므로 잘 알고 사용 해야 한다.

#### CSRS Config Server

몽고디비는 데이터 백업용으로 CSRS라는 레플리카 셋을 가지고 있을 수 있는데
Config Server 도 레플리카 셋으로 배포 될 수 있다.

CSRS를 사용하면 configServer를 일관성 있게 배포 할 수 있게 되고 configServer의 갯수를 여러개로 구성 해 놧을 시에도 배포가 가능해서 해당 기능을 사용하면 좋다.

버전은 3.2 이상이고 3.4에서는 이전 배포 방식이었던 SCCC는 deprecated됐다.

#### config server 의 Operation

system.*collectoins를 갖고있는 admin database는 configServer가 갖고있다.

config database는 청크 마이그레이션이나 청크 스플릿등등의 클러스터 메타 데이터를 갖고 있으며 해당 데이터베이스도 config server가 갖고있다.

따라서 인증을위한 admin database 또는 config server 사용 시에 mongos에서 config server를 조회하고 사용한다.

#### ConfigServer의 유효성

레플리카 셋을 통한 config server 배포 시에 configServer의 `Primary`정보를 잃게 되므로 다시 Primary선출 과정을 통한다.

Cluster된 서비스에서 config Server가 전부 죽으면 해당 클러스터는 동작하지 않는다.

### Router(Mongos)

클러스터에서 mongos는 외부 api 연동을 위한 프론트 라우터 app으로써 interface만 제공한다.

몽고스는 config server에서 클러스터 메타데이터를 캐싱해서 가져오므로서 어떤 샤드에 해당 청크가 있는지 확인하고 가져오게 된다.

mongos는 영속성 스테이트가 없다 

 - scale out이 간편하다.

몽고스가 클러스터에서 라우팅 하는 방법은 다음과 같다.

1. 쿼리에서 대상이 되는 샤드 결정
2. 커서에서 타겟팅되는 모든 샤드 결정


#### 몽고스에서의 집계

몽고스에서 샤드에서 가져온 모든 데이터들을 merge한다.

3.6 버전에서부터 mongos의 기능이 확장되면서 집계를 위해 기본 샤드가 켜져 있지 않아도 되게 되었다.

제약 사항이 있다.

1. 기본샤드를 필요로 하는 집계 파이프라인을 실행시 (ex: `$lookup`)
2. 기본샤드를 통해 저장하고 file I/O를 쓰게 되는 `allowDisUse:true`사용시

#### 몽고스의 핸들링

##### Sorting

각 샤드의 데이터셋이 sorting되어있지 않다면 mongos에서 내부적으로 cursor를 사용하여 roundrobin으로 전체 데이터 셋을 돌려 외부로 돌려준다.

##### Limits

`limit()`를 사용하게되면 몽고스는 해당 명령어를 샤드로 bypass시켜서 데이터를 가져온다.

##### Skips

`skip()`을 사용하게된다면 몽고스는 해당 샤드들 에서는 전부 데이터를 가져오고 merge이후에 몽고스에서 cursor를 skip시켜 보내게 된다.

##### 네트워크 쿼리 방식에 관하여

기본적으로 mongos는 샤드키와, configServer에서의 메타데이터를 기준으로 각 샤드로의 쿼리를 선택적으로 날리던지, 전체 샤드로 BroadCasting을 하던지 한다.

샤드키를 잘 사용하면 mongos에서 샤드들로의 쿼리를 진행 할 때 BroadCast방식이 아닌 Targetted 방식으로 내부 쿼리를 진행 할 확률이 높아져서 성능이 좋아진다.

## 샤드 키 

샤드 키는 하나의 컬렉션에 데이타 청크들을 어떻게 샤딩해서 나눌 것인가를 정하는 기준을 삼을 수 있는 index 의 일종으로 써, 서로 다른 샤드에서도 데이터 상태를 공유하여 unique를 유지 할 수도 있다.

> 보통 unique index는 해당 샤드 내에서 이므로 서로 다른 샤드 에서는 겹칠 수도 있다.

### Chunk

샤드키를 이용한 연속적인 데이터 범위로써 샤드 데이터의 부분 이라고 볼 수 있다.

기본크기는 64MB며 설정된 크기보다 넘어가면 청크는 분리된다.

> **중요!** 컬렉션이 한번 샤딩되면 샤드키는 불변이다. 하지만 4.2부터 _id필드를 샤드키로 사용하지 않으면 업데이트할 수 있다.</br> 샤드키의 변경은 데이터 마이그레이션, 청크 마이그레이션을 일으키므로 오래걸리고 민감하며 서비스 중단이 일어날 수 있으므로 조심히 써야한다.

### 샤드키 index

모든 샤딩된 컬렉션은 샤드키를 갖고 있어야 한다.

#### Unique index

`hashed index`는 `unique`속성을 가질 수 없다.

`_id`가 샤드키가 아닌 경우에는, `_id`키는 해당 샤드 내부 에서만 `unique`함을 갖는다.

### 고려사항

샤드키 선택시 고려 사항은 다음 과 같다.

1. [Cardinality 분포](https://docs.mongodb.com/manual/core/sharding-shard-key/#shard-key-cardinality) : 각 샤드키 마다 묶여진 데이터셋 크기가 차이가 많이나면 효과가 없다.
2. [같은 샤드키의 빈도](https://docs.mongodb.com/manual/core/sharding-shard-key/#shard-key-frequency) : 샤드키가 같은 document의 갯수가 많아지면 청크 분리도 힘들 수도 있으므로 같거나 비슷한 샤드키를 나타내는 빈도를 줄여야 한다.(병목이 걸릴 수 있다.)
3. [샤드키 변경의 용이성](https://docs.mongodb.com/manual/core/sharding-shard-key/#monotonically-changing-shard-keys) : 샤드키의 복잡성이 너무 낮아 샤드키의 변경이 쉬우면 안된다. 그렇게되면 샤드키의 단조로움으로 인해 청크가 무더기로 변경되는 경우가 생기기도 하고 쉽게쉽게 청크가 변경되기도 한다..

## 샤딩 방법들

### Hashed 샤딩

Hashed Index를 이용하는 Hashed샤딩 방법은, mongos가 각 샤드로의 쿼리를 BroadCasting방식으로 할지 TargetedCasting방식으로 할 지 결정하는 로직에 대한 비용을 줄여준다.

해시 인덱스의 ([float를 지원하지 않는](https://docs.mongodb.com/manual/core/index-hashed/#considerations))단점으로 인해 청크 분리에 대한 생각을 해야한다.

### Ranged 샤딩

Hashed 샤딩이나 zone을 만들지 않으면 기본적으로 Range-based 샤딩이다.

먼저 나온 고려사항과 같은 높은 카디널리티와, 적은 빈도수, 단조롭지 않은 샤드키 변경용이성 등을 고려해야 효과적이 샤딩이 가능하다.

### [zone](https://docs.mongodb.com/manual/core/zone-sharding/)

Range-Base샤딩의 경우, 청크 분배를 위한 zone을 만들 수 있다.

해당 zone을 만들 경우 청크 splite시에도 zone내부에서만 청크 분리 및 머지가 일어난다.

### chunk

몽고DB에서 한 컬렉션의 분산된 데이터 단위를 청크라 하며 이 청크를 분배함으로써 샤딩이 이뤄진다.

청크의 기본 최대 size는 64.2MB이고 설정을 변경 할 수 있다.

최대 사이즈가 넘어가면 Balancer라는 백그라운드 프로세스가 ChunkSplit을 수행하며, 특정 샤드에 청크가 너무 많이 몰리면 Chunck Migration이 일어난다. (청크이동)

#### 청크 마이그레이션 임계값

|청크 개수|임계값|
|---|---|
|20개미만|2|
|20~79|4|
|80개이상|8|

위의 표는 전체 청크 갯수에 따른 샤드별 청크갯수 차이에 대한 임계값이다. 예를들어 20개 미만의 전체 청크 갯수를 가진 컬렉션이 두개의 샤드에 갯수가 `[8,11]` 로 한쪽 샤드에 청크가 2개가 더 많다면 마이그레이션이 일어나서, 8개를 가진 쪽으로 청크가 이동하여 `[9,10]` 으로 변하게 된다.

### Balancer

청크를 관리하는 백그라운드 프로세스 이다.

## 비고

- 샤드 클러스터 어드민 설명 https://docs.mongodb.com/manual/administration/sharded-cluster-administration/

- 샤딩 레퍼런스 https://docs.mongodb.com/manual/reference/sharding/

