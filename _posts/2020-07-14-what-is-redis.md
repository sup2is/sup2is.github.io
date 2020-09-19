---
layout: post
title: "Redis 시작하기"
tags: [NoSQL, Redis]
date: 2020-07-14
comments: true
---



<br>



# OverView

오늘은 NoSQL에서 Key-Value DB 역할을 하는 Redis에 대해서 알아보도록 하겠다.

# Redis란 ?

Redis(**RE**mote **DI**rectory **S**ystem)는 Key-Value Database로 분류되는 NoSQL 종류중 하나이다.  오픈소스이고 기업용 라이센스가 별도로 존재한다. Redis는 In-Memory 기반으로 동작하기 때문에 매우 빠른 Read/Write 성능을 자랑한다. 

Redis는 비정형 데이터를 다루며 그 종류에는 String, Set, Sorted Set, Hash, List, HyperLogLogs 등의 데이터를 저장할 수 있다. 필요한 환경에서 적절한 데이터 타입을 잘 선택하면 보다 손쉽게 개발이 가능하다. 예를 들어 Sorted Set 같은 경우는 랭킹 시스템에 아주 적합한 데이터 구조이다. 그리고 모든 데이터 타입의 CRUD는 Atomic하게 실행되고 트랜잭션처리를 지원한다.

Redis도 역시 Replication과 Clustering을 지원하기 때문에 장애에 보다 더 능숙하게 대처가 가능하다. Redis Replication과 Clustering의 자세한 내용은 다음시간에 알아보도록 하겠다.

Redis는 빠른 Read/Write 성능을 자랑하지만 Memory 기반이기 때문에 많은 양의 데이터를 지속적으로 관리하는 것은 한계가있다. 따라서 기존에 RDBMS를 대체하기 보다는 데이터수집, 캐싱용도로 사용하는 세컨더리 DB로 많이 사용한다. 물론 AOF, RDB 라는 개념으로 데이터를 디스크에 영속화시킬 수 있는 기능도 있다.

이번시간에는 Redis에 대해서 알아보고 자주 사용하는 데이터 타입에 대한 설명과 간단한 예제를 작성해보도록 하겠다!

# Redis 설치하기

Redis는 현재(20200713) 기준으로 stable 버전이 6.0.5버전이다. [링크](https://redis.io/download)에서 다운 받을 수 있다. 추가적으로 Docker 환경이 구축되어있다면 [https://hub.docker.com/\_/redis/](https://hub.docker.com\_/redis/) <- 여기에서 이미지에 대한 정보를 확인할 수 있다. 설치나 redis-cli에 대한 설명은 각각 링크에 잘 표기되어 있어서 Redis Server 실행 & Redis cli에 대한 자세한 설명은 생략한다.



# Redis의 Data Type

Redis에서 제공하는 데이터타입에 대해서 간략하게 알아보는 시간을 갖고 간단하게 api를 사용해보는 시간을 갖도록 하겠다.

## Redis Keys

일단 Redis는 Key-Value 기반이기때문에 모든 Value는 1개의 Key를 갖게 되어있다. key들은 object-type:id 같이 `:` 과 `-` 을 사용해서 조합할 수 있다.

## Redis Strings & Redis CRUD

Redis Strings는 Redis 데이터 타입중에 가장 간단한 타입이다. Redis Cli로 Redis Server에 접속한 뒤 다음 명령어를 입력해서 데이터를 저장해보자

```
my-redis:6379> set mykey somevalue
OK
my-redis:6379> get mykey
"somevalue"
```

여기에서 set 명령어는 데이터를 지정하는데 만약 기존에 "mykey" 라는 값이 있었다면 기존 값을 덮어 씌우기때문에 주의해야한다. 만약 `set mykey newval nx` 같은 형태로 `newval` 인자값을 주면 기존에 key가 있어야만 업데이트하도록 사용할 수 있다.

Redis에서는 숫자를 사용한 몇가지 명령어도 제공한다.

```
my-redis:6379> set count 100
OK
my-redis:6379> incr count
(integer) 101
my-redis:6379> incr count
(integer) 102
my-redis:6379> incrby count 100
(integer) 202
my-redis:6379> decrby count 50
(integer) 152
```

<br>

여러가지 키를 한번에 등록하고 한번에 꺼내올때는 `mset`과 `mget`을 사용하면 된다.

```
my-redis:6379> mset a 10 b 20 c 30
OK
my-redis:6379> mget a b c count
1) "10"
2) "20"
3) "30"
4) "152"
```

<br>

키를 삭제할때는 `del` 명령어를 사용해서 삭제하면 된다.

```
my-redis:6379> set mykey x
OK
my-redis:6379> type mykey
string
my-redis:6379> del mykey
(integer) 1
my-redis:6379> type mykey
none
```

<br>

`exist` 명령어를 사용해서 해당 키가 존재하는지에 대한 값을 1 또는 0 으로 받을 수 있다.

```
my-redis:6379> set mykey hello
OK
my-redis:6379> exists mykey
(integer) 1
my-redis:6379> del mykey
(integer) 1
my-redis:6379> exists mykey
(integer) 0
```

<br>

`expire` 명령어를 사용해서 Redis Key값을 일정시간 이후에 없애도록 지정할 수 있다. 또 `ttl` 명령어를 사용해서 해당 key의 유효시간을 확인할 수 있다.

```
my-redis:6379> set my-key some-value
OK
my-redis:6379> expire my-key 20
(integer) 1
my-redis:6379> ttl my-key
(integer) 13
my-redis:6379> 
my-redis:6379> ttl my-key
(integer) 11
my-redis:6379> 
my-redis:6379> ttl my-key
(integer) 10
my-redis:6379> 
my-redis:6379> ttl my-key
(integer) 8
my-redis:6379> get my-key
"some-value"
my-redis:6379> ttl my-key
(integer) -2
my-redis:6379> get my-key
(nil)
```

<br>

## Redis List

Redis에서 List는 LinkedList로 구현되어있다. 따라서 LinkedList의 특성을 그대로 갖고 있다. `lpush`와 `rpush`를 사용해서 List에 데이터를 저장해보자.

```
my-redis:6379> rpush mylist A
(integer) 1
my-redis:6379> rpush mylist B
(integer) 2
my-redis:6379> lpush mylist first
(integer) 3
my-redis:6379> lrange mylist 0 -1
1) "first"
2) "A"
3) "B"
```

`lpush`와 `rpush` 를 사용해서 list의 head와 tail에 접근해서 데이터를 추가할 수 있고 `lrange`를 통해서 list를 검색할 수 있다. `lrange`의 인자 두개는 to ~ from인데 -1로 할 경우 list의 length만큼의 데이터를 얻어올 수있고 -2 부터는 length - 1 형식으로 데이터를 가져올 수 있다.

<br>

`rpush` 와 `lpush`를 통해 데이터를 넣었다면 `rpop` , `lpop`으로 데이터를 꺼내올 수 있다.

```
my-redis:6379> rpush mylist a b c
(integer) 3
my-redis:6379> rpop mylist
"c"
my-redis:6379> rpop mylist
"b"
my-redis:6379> rpop mylist
"a"
my-redis:6379> rpop mylist
(nil)
```

<br>

`ltrim`을 사용해서 to ~ from 안에 범위에 있는 요소를 제외한 요소를 전부 삭제시킬 수 있다

```
my-redis:6379> rpush mylist 1 2 3 4 5
(integer) 5
my-redis:6379> ltrim mylist 0 2
OK
my-redis:6379> lrange mylist 0 -1
1) "1"
2) "2"
3) "3"
```

<br>

Redis의 List는 소셜 네트워크에서 가장 최신 피드의 목록을 나타내거나 producer/consumer모델을 구현하는 방식으로 사용할 수 있다. 만약 producer가 `lpush` 명령어로 list에 데이터를 넣고 consumer가 `rpop`으로 꺼내오는데 만약 `rpop`시점에 데이터가 없을때는 null 이 반환된다. Redis는 `brpop` 또는 `blpop`이라는 명령어를 지원해서 특정시간까지의 블로킹된 형태로 pop을 할 수 있도록 도와준다. shell을 두개 띄워서 테스트해보도록 하자.

**shell # 1**

```
my-redis:6379> brpop mylist 10

blocking ....

```

**shell # 2**

```
my-redis:6379> lpush mylist 3
(integer) 1
my-redis:6379> 
```

**shell # 1**

```
my-redis:6379> brpop mylist 10
1) "mylist"
2) "3"
(5.28s)
```

텍스트로 표현해서 크게 와닿을지는 모르겠지만 shell # 2 에서 `lpush` 하는 순간 바로 shell # 1에서 pop을 하는 모습을 볼 수 있다. 만약 지정한 시간 이내에도 데이터가 들어오지 않는다면 null을 반환한다.

<br>

Redis 의 List나 Sets, Sorted Set 등등은 위에서 확인한것처럼 key를 먼저 생성하지 않아도 `lpush` 같은 명령어를 통해 자동적으로 key를 생성해준다. 마찬가지로 더이상 요소에 아무런 데이터가 없을때는 key를 자동적으로 제거한다.

```
my-redis:6379> lpush mylist 10
(integer) 1
my-redis:6379> keys *
1) "mylist"
my-redis:6379> rpop mylist
"10"
my-redis:6379> keys *
(empty array)
```

## Redis Hash

Redis에서 Hash는 하나의 key 내부에 여러개의 key:value로 이루어진 데이터 타입이다.

```
my-redis:6379> hmset user:1000 username antirez birthyear 1977 verified 1
OK
my-redis:6379> hget user:1000 username
"antirez"
my-redis:6379> hget user:1000 birthyear
"1977"
my-redis:6379> hgetall user:1000
1) "username"
2) "antirez"
3) "birthyear"
4) "1977"
5) "verified"
6) "1"
```

여러개의 key를 가져올땐 `hmget` 을 사용하면 된다. 

```
my-redis:6379> hmget user:1000 username birthyear
1) "antirez"
2) "1977"
```

## Redis Set

Redis Set은 정렬되지 않은 String의 집합을 나타내는 데이터 타입이다. 요소의 존재 여부, 집합 등에 사용된다.

```
my-redis:6379> sadd myset 1 2 3
(integer) 3
my-redis:6379> smembers myset
1) "2"
2) "1"
3) "3"
my-redis:6379>  sismember myset 3
(integer) 1
my-redis:6379>  sismember myset 30
(integer) 0
```

myset에는 30에 값이 존재하지 않기 때문에 0을 반환한다.  set은 졍렬되지 않았기 때문에 데이터를 1, 2, 3 으로 넣었다고 하더라도`smember`  으로 꺼내올 때는 2, 1, 3이 반환될 수 있다.

<br>

카드게임을 생각했을때 deck은 다음과같은 요소들을 갖고 있어야 한다.

```
my-redis:6379> sadd deck C1 C2 C3 C4 C5 C6 C7 C8 C9 C10 CJ CQ CK
   D1 D2 D3 D4 D5 D6 D7 D8 D9 D10 DJ DQ DK H1 H2 H3
   H4 H5 H6 H7 H8 H9 H10 HJ HQ HK S1 S2 S3 S4 S5 S6
   S7 S8 S9 S10 SJ SQ SK
(integer) 52
my-redis:6379> smembers deck
 1) "DJ"
 2) "D10"
 3) "D7"
 4) "H3"
 5) "C4"
 6) "S1"
 7) "C1"
 8) "C8"
 9) "SK"
10) "D2"

...

51) "C9"
52) "D3"
```

이때 원본을 훼손시키지 않고 `sunionstore`으로 합집합연산을 사용해서 새로운 set에 저장시킬 수 있다.

```
my-redis:6379> sunionstore game:1:deck deck
(integer) 52
my-redis:6379> spop game:1:deck
"D6"
my-redis:6379> spop game:1:deck
"C10"
my-redis:6379> spop game:1:deck
"H4"
my-redis:6379> spop game:1:deck
"SJ"
my-redis:6379> scard game:1:deck
(integer) 48
my-redis:6379> scard deck
(integer) 52
```

## Redis Sorted Set

Redis에서 Sorted Set은 Hash + Set을 섞어놓은 데이터 타입이다. 

```
my-redis:6379> zadd hackers 1940 "Alan Kay"
(integer) 1
my-redis:6379> zadd hackers 1957 "Sophie Wilson"
(integer) 1
my-redis:6379> zadd hackers 1953 "Richard Stallman"
(integer) 1
my-redis:6379> zadd hackers 1949 "Anita Borg"
(integer) 1
my-redis:6379> zadd hackers 1965 "Yukihiro Matsumoto"
(integer) 1

my-redis:6379> zrange hackers 0 -1
1) "Alan Kay"
2) "Anita Borg"
3) "Richard Stallman"
4) "Sophie Wilson"
5) "Yukihiro Matsumoto"

```

`sadd` 와 비슷한 형태로 `zadd` 명령어를 사용해서 데이터를 저장하고 key:value 사이에 score값으로 졍렬한다. sadd에 데이터를 추가하는 시간복잡도는 O(log(N)) 으로 매우 빠른편이고 이미 데이터를 얻어오는 시점에는 정렬된 요소를 반환하기 때문에 용이하게 사용할 수 있다. 

Sorted Set은 score값을 비교해서 정렬하고 같은 score 값이 있으면 value의 String 값의 순서를 비교해서 정렬한다. Set이기때문에 같은 String 값을 가질 수 없어서 항상 정렬된 모습을 갖고 있는다.

```
my-redis:6379> zadd test 2000 "abcd"
(integer) 1
my-redis:6379> zadd test 2001 "abcde"
(integer) 1
my-redis:6379> zadd test 2000 "bcd"
(integer) 1
my-redis:6379> zadd test 2000 "abcd"  <- abcd는 이미 존재함
(integer) 0
my-redis:6379> zcard test
(integer) 3
my-redis:6379> zrange test 0 -1
1) "abcd"
2) "bcd"
3) "abcde"
```

`zrange`에  `withscores` 인수를 주어 score 값을 얻어올수도 있다.

```
my-redis:6379> zrange hackers 0 -1 withscores
 1) "Alan Kay"
 2) "1940"
 3) "Anita Borg"
 4) "1949"
 5) "Richard Stallman"
 6) "1953"
 7) "Sophie Wilson"
 8) "1957"
 9) "Yukihiro Matsumoto"
10) "1965"
```

`zrangebyscore `을 통해서 일정 score 이상/이하의 목록들을 가져올 수 있다. 

```


my-redis:6379> zrangebyscore hackers -inf 1950
1) "Alan Kay"
2) "Anita Borg"

```

`zrank` 명령어를 사용해서 해당 요소가 상위 몇번째인지 확인할 수 있고 `zrevrank` 를 통해서 하위 몇번째인지도 확인할 수 있다.

```
my-redis:6379> zrank hackers "Alan Kay"
(integer) 0
my-redis:6379> zrevrank hackers "Alan Kay"
(integer) 4
```



# 마무리

Redis의 기본 개념 + Redis에서 많이 사용하는 DataType에 대해서 알아봤다. 간단한 명령어로 다양한 데이터를 나타낼 수 있었다. 모든 명령어들의 목록은 [https://redis.io/commands](https://redis.io/commands) 에서 확인할 수 있기 때문에 필요할 때마다 확인하는 방식으로 사용하면 될 듯 하다. 



<hr>
포스팅은 여기까지 하겠습니다. 퍼가실때는 출처를 반드시 남겨주세요!


**References**

- 빅데이터 저장 및 분석을 위한 NoSQL & Redis - 주종면 
- [https://redis.io/](https://redis.io/)
