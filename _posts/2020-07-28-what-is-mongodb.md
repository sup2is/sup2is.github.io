---
layout: post
title: "MongoDB 시작하기"
tags: [NoSQL, MongoDB]
date: 2020-07-28
comments: true
---



<br>

# OverView

이번 시간에는 NoSQL 점유율 1위인 Document 기반 NoSQL, MongoDB에 대해서 알아보도록 하겠다.

# MongoDB

MongoDB는 NoSQL 중 압도적인 1위를 자랑하는 **Document 타입 기반 NoSQL**이다. 2007년 10월 10gen(현재 MongoDB Inc)이라는 회사에서 처음 등장했고 2009년에 오픈소스로 전환하여 현재는 듀얼라이센스 정책으로 Community 버전과 Enterprise 버전을 제공하고 있다.

![20200722_094127](https://user-images.githubusercontent.com/30790184/88237937-5c6fc180-ccbb-11ea-8047-a10bb5787245.png)

> MongoDB는 2020년 7월 기준으로 데이터베이스 엔진 랭킹 5위에 있다. [](https://db-engines.com/en/ranking)

MongoDB는 JSON타입으로 데이터를 저장하는데 때문에 RDB에서는 사용하지 못했던 array,list 같은 형식의 데이터도 저장이 가능하다.

```
{
 "name": "notebook",
 "qty": 50,
 "rating": [ { "score": 8 }, { "score": 9 } ],
 "size": { "height": 11, "width": 8.5, "unit": "in" },
 "status": "A",
 "tags": [ "college-ruled", "perforated"]
}
```

몽고 DB는 Document 내부에 **컬렉션**이라는 개념에 데이터를 저장하는데 이 컬렉션은 RDB에서 테이블과 유사하다. 추가적으로 컬렉션을 조회하기위한 [Views](https://docs.mongodb.com/manual/core/views/)와 [On-demand materialized Views](https://docs.mongodb.com/manual/core/materialized-views/)를 제공한다.

몽고 DB는 임베디드형 데이터 모델을 제공하기 때문에 높은 성능을 갖고 있으며 더빠른 쿼리를 위해 인덱스를 지원한다.

기본적인 [CRUD api](https://docs.mongodb.com/manual/crud/)를 제공하는 것은 물론이고. [Data Aggregation](https://docs.mongodb.com/manual/core/aggregation-pipeline/)  같은 개념을 제공한다 Data Aggregation은 파이프라인의 개념을 토대로 데이터 집계에 사용하는데 Java에서 Stream을 생각하면 이해가 조금 더 쉬울 수 있다.

```
db.orders.aggregate([
   { $match: { status: "A" } },
   { $group: { _id: "$cust_id", total: { $sum: "$amount" } } }
])
```

Data Aggregation 이외에도 [Text Search](https://docs.mongodb.com/manual/text-search/) 같은 데이터 집계, 검색과 관련된 api를 제공한다.

몽고 DB는 [replica set](https://docs.mongodb.com/manual/replication/) 이라는 개념으로 replication을 구성한다. 이것을 통해 auto failover, 데이터 가용성을 지원한다. 물론 scale out 역시 가능하다. [Sharding](https://docs.mongodb.com/manual/sharding/#sharding-introduction)이라는 개념을 통해 데이터를 분산시킬 수 있다.

## 설치하기

몽고DB 설치는 [https://docs.mongodb.com/manual/installation/](https://docs.mongodb.com/manual/installation/)에서 찾을 수 있지만 나는 Docker 기반으로 설치할 예정이다. 이미지 관련된 정보는 [https://hub.docker.com/\_/mongo](https://hub.docker.com/_/mongo) 에서 찾을 수 있다.

다음 명령어를 통해 몽고DB 컨테이너를 올려보자.

```
docker run --name some-mongo -d mongo:tag
```

다음 명령어를 통해 몽고DB bash 쉘에 접속할 수 있다.

```
docker exec -it some-mongo bash
```

컨테이너 내부에서 `mongo` 라는 명령어를 치면 몽고DB를 사용할 수 있다.

## CRUD 연습하기

먼저 현재 사용하고있는 db를 확인하려면 `db` 라는 명령어로 확인할 수 있다.

```
> db
test
```

db를 변경하고싶다면 `use {dbname}` 을 사용해서 변경할 수 있다.

```
> use examples
switched to db examples
```



### Insert

이제 컬렉션이라는 곳에 데이터를 넣어볼 차례다. **inventory**라는 이름을 가진 컬렉션에 `insertMany()` 메서드를 사용해서 데이터를 넣는데 이때 inventory라는 컬렉션이 없다면 몽고DB가 알아서 컬렉션을 생성해준다.

```
> db.inventory.insertMany([
    { item: "journal", qty: 25, status: "A", size: { h: 14, w: 21, uom: "cm" }, tags: [ "blank", "red" ] },
    { item: "notebook", qty: 50, status: "A", size: { h: 8.5, w: 11, uom: "in" }, tags: [ "red", "blank" ] },
    { item: "paper", qty: 10, status: "D", size: { h: 8.5, w: 11, uom: "in" }, tags: [ "red", "blank", "plain" ] },
    { item: "planner", qty: 0, status: "D", size: { h: 22.85, w: 30, uom: "cm" }, tags: [ "blank", "red" ] },
    { item: "postcard", qty: 45, status: "A", size: { h: 10, w: 15.25, uom: "cm" }, tags: [ "blue" ] }
 ]);
{
        "acknowledged" : true,
        "insertedIds" : [
                ObjectId("5f17a3dd5bee84b4110e8521"),
                ObjectId("5f17a3dd5bee84b4110e8522"),
                ObjectId("5f17a3dd5bee84b4110e8523"),
                ObjectId("5f17a3dd5bee84b4110e8524"),
                ObjectId("5f17a3dd5bee84b4110e8525")
        ]
}

```

성공적으로 인서트되었다면 **ObjectId("5f179f1f5bee84b4110e84f4")**같은 _id를 반환한다. 이 _id는 몽고DB가 자동으로 부여하는 ID값이다.

### select

이제 컬렉션 내부에 데이터를 탐색해보도록 하겠다. 전체 데이터를 가져올때는 `db.inventory.find({}).pretty()` 명령어로 가져온다. `.pretty()` 옵션은 formating된 결과로 리턴하게 해준다.

```
> db.inventory.find({}).pretty()

...
{
        "_id" : ObjectId("5f17a3dd5bee84b4110e8525"),
        "item" : "postcard",
        "qty" : 45,
        "status" : "A",
        "size" : {
                "h" : 10,
                "w" : 15.25,
                "uom" : "cm"
        },
        "tags" : [
                "blue"
        ]
}
...
```

status와 같은 필드에 대한 질의를 수행하려면 `db.inventory.find( { status: "D" } );` 처럼 사용할 수 있다. 이렇게 사용하면 status가 D인 데이터만 가져온다.

```
> db.inventory.find( { status: "D" } ).pretty();
{
        "_id" : ObjectId("5f17a3dd5bee84b4110e8523"),
        "item" : "paper",
        "qty" : 10,
        "status" : "D",
        "size" : {
                "h" : 8.5,
                "w" : 11,
                "uom" : "in"
        },
        "tags" : [
                "red",
                "blank",
                "plain"
        ]
}
{
        "_id" : ObjectId("5f17a3dd5bee84b4110e8524"),
        "item" : "planner",
        "qty" : 0,
        "status" : "D",
        "size" : {
                "h" : 22.85,
                "w" : 30,
                "uom" : "cm"
        },
        "tags" : [
                "blank",
                "red"
        ]
}
```

`db.inventory.find( { qty: 0, status: "D" } );`와 같은식의 복합형태도 가능하고 size라는 임베디드 데이터 타입 내부에 uom 필드에 대한 질의를 하고싶다면 `db.inventory.find( { "size.uom": "in" } )` 와 같은 방식으로 검색이 가능하다.

array 타입이나 object 타입의 질의를 할때 주의해야 할 사항이 있는데 `db.inventory.find( { size: { h: 14, w: 21, uom: "cm" } } )` 와 같이 필드의 순서를 반드시 일치하게 해주어야한다. 이 명령어는 아무 행도 반환하지 않는다. 

```
db.inventory.find( { size: { h: 14, uom: "cm", w: 21 } } ) # 안됨!
db.inventory.find( { tags: [ "red", "blank" ] } ) # 역시 안됨!
```

특정 필드만 리턴받고싶다면 `db.inventory.find( { }, { item: 1, status: 1 } );` 명령어처럼 두번째 파라미터에 필드명 : 1(유) 또는 0(무) 의 값을 주어 특정 필드만 가져오게할 수 있다.

```
> db.inventory.find( { }, { item: 1, status: 1 } );
{ "_id" : ObjectId("5f18c7f87b0e45c9c6172062"), "item" : "journal", "status" : "F" }
{ "_id" : ObjectId("5f18c7f87b0e45c9c6172063"), "item" : "notebook", "status" : "A" }
{ "_id" : ObjectId("5f18c7f87b0e45c9c6172064"), "item" : "paper", "status" : "D" }
{ "_id" : ObjectId("5f18c7f87b0e45c9c6172065"), "item" : "planner", "status" : "D" }
{ "_id" : ObjectId("5f18c7f87b0e45c9c6172066"), "item" : "postcard", "status" : "A" }
> 
```

_id 값은 자동적으로 반환되며 만약 리턴받고싶지 않다면 _id : 0 으로 지정하면 된다.

### update

update는 다음과 같이 `db.inventory.updateOne({ status : "A" }, { $set: { status: "F"}})` 명령어 처럼 사용할 수 있다. 

```
> db.inventory.updateOne({ status : "A" }, { $set: { status: "F"}});
{ "acknowledged" : true, "matchedCount" : 1, "modifiedCount" : 1 }

> db.inventory.find({status: "F"})
{ "_id" : ObjectId("5f18c7f87b0e45c9c6172062"), "item" : "journal", "qty" : 25, "status" : "F", "size" : { "h" : 14, "w" : 21, "uom" : "cm" }, "tags" : [ "blank", "red" ] }
```

`updateOne()`의 첫번째 인자는 검색조건이고 두번째 인자는 `$set` 으로 변경 하고싶은 데이터를 나열할 수 있다. 이외에도 upsert 같은 기능을 제공한다. status 값이 A인 데이터를 F로 변경하고 조회하는 모습이다.



### delete

delete는 다음과 같이 `db.inventory.deleteOne({status : "F"})` 의 형태로 사용할 수 있다. 이전에 status를 F값으로 변경한 데이터를 삭제해보도록 하겠다.

```
> db.inventory.deleteOne({status : "F"})
{ "acknowledged" : true, "deletedCount" : 1 }
```



# 마무리

간단하게 MongoDB 에 대한 특성에 대해서 알아보고 기본적인 CRUD api 를 사용해서 MongoDB에서 어떻게 데이터를 사용하는지에 대해서 알아봤다. 이외에도 컬렉션 관련된 모든 메서드에 대한 정보는 https://docs.mongodb.com/manual/reference/method/js-collection/ 에서 찾을 수 있다.

<hr>
포스팅은 여기까지 하겠습니다. 퍼가실때는 출처를 반드시 남겨주세요!


**References**

- [https://docs.mongodb.com/](https://docs.mongodb.com/)