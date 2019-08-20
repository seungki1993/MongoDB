# CRUD

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

``` sh
db.inventory.find( { age: 20 } )
```

``` sh
db.inventory.find( { age: { $in: [ 20, 24 ] } } )
```

``` sh
db.inventory.find( 
  { $or: [ 
    { name: "syjung" }, 
    { age: { $lt: [ 21 ] } } 
  ]})
```

``` sh
db.inventory.find( 
  { $or: [ 
    { name: "syjung" }, 
    { age: { $lt: [ 21 ] } } 
  ]})
```

null이랑 missing field는 둘다 null로 검색 된다.

``` sh
db.inventory.find( { item: null } )
```

typeCheck도 가능하다.

- [BsonType목록](https://docs.mongodb.com/manual/reference/bson-types/)

```sh
db.inventory.find( { item : { $type: 10 } } ) #BsonType 10 = nullType
```

