---
layout: post
title: "Redis Cluster 시작하기"
tags: [NoSQL, Redis, Redis Cluster]
date: 2020-07-22
comments: true
---



<br>

# OverView

이번시간에는 Redis Cluster에 대해서 알아보고 Redis에서 기본으로 제공하는 스크립트를 통해서 Redis Cluster를 구축 및 테스트해보도록 하겠다.

# Redis Cluster

Redis Cluster는 여러 **Redis 서버에 데이터를 자동으로 sharding 해주는 기술**이다. 이전시간에 배운 Replication 구성을 한다면 Cluster 운영중에 노드중 일부가 장애를 겪고 있더라도 작업을 계속할 수 있게 해준다. (별도의 Replication 구성이 필요한 것은 아니다.)

모든 Redis Cluster의 노드들은 자기의 **기본 포트 (ex 6379) 와 기본포트 + 10000 (ex 16379) 같은 두개의 포트를 사용**한다. 여기에서 16379와 같은 포트는 레디스 클러스터 버스로 사용된다. 이 레디스 클러스터 버스는 **장애 감지, 구성 업데이트, failover 승인** 등에 사용된다. 따라서 반드시 두개의 포트는 통신을 보장해주어야한다.



## Redis Cluster Data Sharding

Redis Cluster는 **hash slot** 이라는 개념을 사용하는데 이 해시슬롯에는 **16384** 개의 해시슬롯이 있다. 데이터를 저장할때 주어진 키의 해시슬롯을 찾기 위해서 **HASH_SLOT = CRC16(key) mod 16384** 와 같은 알고리즘을 사용한다 CRC 16에 대한 자세한 정보는 [http://blog.daum.net/trts1004/12108957](http://blog.daum.net/trts1004/12108957)에서 확인할 수 있다.

예를들어 레디스 클러스터를 3개의 노드로 구성한다고하면 다음과 같이 해시슬롯이 부여된다.

- Node A contains hash slots from 0 to 5500.
- Node B contains hash slots from 5501 to 11000.
- Node C contains hash slots from 11001 to 16383.

레디스에서 데이터 샤딩하는 방법은 강대명님의 우아한 레디스 발표 (58분 58초)에서 영상을 통해 확인할 수 있다. [https://www.youtube.com/watch?v=mPB2CZiAkKM](https://www.youtube.com/watch?v=mPB2CZiAkKM)



## Redis Cluster master-slave

Redis Cluster는 별도의 가용성을 지원하지는 않고 이전시간에 배웠던 Redis Replication의 개념을 Cluster에도 사용할 수 있게 한다. 만약 클러스터 내부 노드 A, B, C로 이루어진 상태에서 각각 레플리카로 A1, B1, C1을 구성하면 마스터노드인 B에 장애가 나더라도 복제본인 B1을 통해서 Redis Cluster를 계속적으로 사용 가능한 상태로 만든다. 그러나 만약 B와 B1이 둘다 장애가 난다면 Redis Cluster는 사용이 불가능한 상태가 된다.



## Redis Cluster consistency guarantees

Redis Cluster는 강력한 일관성을 보장하지 않는데 다음과 같은 시나리오를 예를 들어보겠다.

1. **B는 B1, B2, B3라는 복제본을 갖고 있다.**
2. **사용자는 B에 데이터 aaa를 입력했다.**
3. **B와 B1, B2, B3 사이에 replication은 비동기로 이루어지기때문에 B1, B2, B3에 데이터 전파가 이루어지지 않더라도 사용자에게 'OK' 라는 메시지를 보낸다.** 
4. **비동기로 데이터를 복제하는 도중 aaa 데이터가 어떤 환경에 의해  B1, B2, B3에 들어가지 않았다.**
5. **이후에 B의 장애가 발생했을때 B1, B2, B3에는 aaa 라는 데이터가 없다.**

따라서 이런 문제에 대해서는 주의해야 한다.

# Redis Cluster 구성하기

먼저 레디스가 설치되지 않았다면 레디스를 설치한다. 다운로드: [https://redis.io/download](https://redis.io/download)

Redis Cluster를 구성하기 위해서는 최소 3대의 노드가 필요하고 각각 1개씩 레플리카를 구성하려면 총 6개의 노드가 필요하다. 직접 구성해보는것도 좋지만 **레디스에서 제공하는 create-cluster 스크립트를 활용해서 간단하게 구성할 수 있다.** 

<br>

> - 관련된 docker-compose는 여기에서 확인할 수 있다.[https://hub.docker.com/r/bitnami/redis-cluster](https://hub.docker.com/r/bitnami/redis-cluster)

<br>



레디스가 제공하는 create-cluster 스크립트의 위치는 다음과같다

```
{redis_home}/utils/create-cluster/create-cluster
```

이후에 `/create-cluster start` 명령어와 `./create-cluster create` 명령어를 통해서 cluster를 초기화시켜보자.

```
[root@test src]# ./create-cluster start
Starting 30001
Starting 30002
Starting 30003
Starting 30004
Starting 30005
Starting 30006
[root@test src]# ./create-cluster create
Master[0] -> Slots 0 - 5460
Master[1] -> Slots 5461 - 10922
Master[2] -> Slots 10923 - 16383
Adding replica 127.0.0.1:30005 to 127.0.0.1:30001
Adding replica 127.0.0.1:30006 to 127.0.0.1:30002
Adding replica 127.0.0.1:30004 to 127.0.0.1:30003
>>> Trying to optimize slaves allocation for anti-affinity
[WARNING] Some slaves are in the same host as their master
M: 741b7f1e34ed6186d2526133672be20002d65ccb 127.0.0.1:30001
   slots:[0-5460] (5461 slots) master
M: 6b688de2cc609c82721e90733cafabd2e7b1b78e 127.0.0.1:30002
   slots:[5461-10922] (5462 slots) master
M: 0ac65f89813907533d716ad6cd56551ae169d58d 127.0.0.1:30003


...

[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
```

각각 서버는 **30001~30005**로 할당되고 차례대로 마스터를 할당시키고 그뒤에 차례대로 레플리카를 할당시킨다. 맨 마지막에 **[OK] All 16384 slots covered.**같은 문구가 나온다면 정상적으로 생성된 것이다. 각각 마스터의 해시슬롯 할당은 다음과 같이 되었다.

```
Master[0] -> Slots 0 - 5460
Master[1] -> Slots 5461 - 10922
Master[2] -> Slots 10923 - 16383
```

# Redis Cluster 테스트하기

이제 몇가지 값을 넣어본 뒤 실제 해시슬롯 위치에 데이터가 샤딩되었는지 확인해보자.

```
[root@test src]# ./redis-cli -c -p 30001
127.0.0.1:30001> set foo bar
-> Redirected to slot [12182] located at 127.0.0.1:30003
OK
127.0.0.1:30003> get foo
"bar"
127.0.0.1:30003> set hello world
-> Redirected to slot [866] located at 127.0.0.1:30001
OK
127.0.0.1:30001> get hello
"world"
```

클러스터에 접속할때도 redis-cli 도구를 사용하지만 반드시 `-c` 옵션을 넣어주어야한다. 몇가지 값은 해싱알고리즘에 의해 적절한 곳으로 분산처리된다. 실제 **12182**번 슬롯을 담당하는 **30003** 서버에 접속해서 **foo**값을 확인해보자.

```
[root@test src]# ./redis-cli -c -p 30003
127.0.0.1:30003> get foo
"bar"
127.0.0.1:30003> get hello
-> Redirected to slot [866] located at 127.0.0.1:30001
"world"
```

30003 서버에는 **foo** 값은 존재하지만 **hello**라는 값은 866 슬롯을 담당하는 30001번에 있음을 알려주고 있다.

# Failover 테스트하기

앞에서 구성한 Redis Cluster는 총 3개의 마스터노드와 3개의 레플리카로 구성했다. `./redis-cli -p 30001 cluster nodes` 명령어를 통해서 확인할 수 있다.

```
[root@kafka-broker-base-1 src]# ./redis-cli -p 30001 cluster nodes
0ac65f89813907533d716ad6cd56551ae169d58d 127.0.0.1:30003@40003 master - 0 1595312805039 3 connected 10923-16383
8f7e69597479ac02276be531381854f292f88565 127.0.0.1:30005@40005 slave 0ac65f89813907533d716ad6cd56551ae169d58d 0 1595312805339 5 connected
06bad51df9b08df421666e8cca145146d5c6448e 127.0.0.1:30004@40004 slave 6b688de2cc609c82721e90733cafabd2e7b1b78e 0 1595312805239 4 connected
709f1e1beea7ce853f5efe17b2b4116e2805cf1e 127.0.0.1:30006@40006 slave 741b7f1e34ed6186d2526133672be20002d65ccb 0 1595312805039 6 connected
6b688de2cc609c82721e90733cafabd2e7b1b78e 127.0.0.1:30002@40002 master - 0 1595312805339 2 connected 5461-10922
741b7f1e34ed6186d2526133672be20002d65ccb 127.0.0.1:30001@40001 myself,master - 0 1595312805000 1 connected 0-5460
```

**0ac65f8981390** 시작하는 **30003** 서버의 slave가 **30005**번 서버임을 알 수 있다. 이제 **30003** 서버를 약 30초가량 중지시켜서 **30005**번 서버가 마스터로 승격되는지 한번 확인해보자.

```
[root@test src]# ./redis-cli -p 30003 DEBUG sleep 30
```

위 명령어로 30003번 서버를 30초간 sleep 시킨다.

```
[root@test src]# ./redis-cli -p 30001 cluster nodes
0ac65f89813907533d716ad6cd56551ae169d58d 127.0.0.1:30003@40003 master,fail - 1595313009109 1595313008108 3 connected
8f7e69597479ac02276be531381854f292f88565 127.0.0.1:30005@40005 master - 0 1595313038000 7 connected 10923-16383
06bad51df9b08df421666e8cca145146d5c6448e 127.0.0.1:30004@40004 slave 6b688de2cc609c82721e90733cafabd2e7b1b78e 0 1595313038089 4 connected
709f1e1beea7ce853f5efe17b2b4116e2805cf1e 127.0.0.1:30006@40006 slave 741b7f1e34ed6186d2526133672be20002d65ccb 0 1595313037988 6 connected
6b688de2cc609c82721e90733cafabd2e7b1b78e 127.0.0.1:30002@40002 master - 0 1595313038089 2 connected 5461-10922
741b7f1e34ed6186d2526133672be20002d65ccb 127.0.0.1:30001@40001 myself,master - 0 1595313038000 1 connected 0-5460
```

다시 `./redis-cli -p 30001 cluster nodes` 명령어로 확인하면 30003 서버는 **master,fail** 이라는 상태를 갖고 **30005** 서버가 마스터로 승격된것을 확인할 수 있다. 30초 정도가 지난 뒤에 다시 `./redis-cli -p 30001 cluster nodes`를 확인하면 

```
[root@test src]# ./redis-cli -p 30001 cluster nodes
0ac65f89813907533d716ad6cd56551ae169d58d 127.0.0.1:30003@40003 slave 8f7e69597479ac02276be531381854f292f88565 0 1595313040695 7 connected
8f7e69597479ac02276be531381854f292f88565 127.0.0.1:30005@40005 master - 0 1595313040095 7 connected 10923-16383
06bad51df9b08df421666e8cca145146d5c6448e 127.0.0.1:30004@40004 slave 6b688de2cc609c82721e90733cafabd2e7b1b78e 0 1595313040095 4 connected
709f1e1beea7ce853f5efe17b2b4116e2805cf1e 127.0.0.1:30006@40006 slave 741b7f1e34ed6186d2526133672be20002d65ccb 0 1595313040000 6 connected
6b688de2cc609c82721e90733cafabd2e7b1b78e 127.0.0.1:30002@40002 master - 0 1595313040095 2 connected 5461-10922
741b7f1e34ed6186d2526133672be20002d65ccb 127.0.0.1:30001@40001 myself,master - 0 1595313040000 1 connected 0-5460
```

다시 30003서버가 slave가 되어있는것을 확인할 수 있다.

create-cluster로 설정된 레디스 클러스터는 `stop` 명령어로 종료할 수 있고 만약 초기화가 필요하다면 `clean`으로 초기화시킨 뒤 다시 구동시킬 수 있다.

```
./create-cluster stop
./create-cluster clean
```



# Redis Cluster Hashtag

추가적으로 비슷한 키는 같은 슬롯에 저장시키고 싶은 경우가 있을 수 있다. 이때는 **hashtag**라는 개념을 사용해서 같은 슬롯에 저장됨을 보장시킬 수 있다. 우리가 저장할 데이터는 다음과같다.

- key : user = 1000, value = choi
- key : user = 2000, value = kim
- key : user = 3000, value = sup2is

일반적으로 데이터를 넣는다면 다음과같이 각각 다른 슬롯에 분배될 것이다.

```
127.0.0.1:30002> set user:1000 choi
-> Redirected to slot [1649] located at 127.0.0.1:30001
OK
127.0.0.1:30001> set user:2000 kim
-> Redirected to slot [7597] located at 127.0.0.1:30002
OK
127.0.0.1:30002> set user:3000 sup2is
-> Redirected to slot [11033] located at 127.0.0.1:30003
OK
```

하지만 hashtag를 사용하면 같은 슬롯에 저장시킬 수 있다.

```
127.0.0.1:30001> set {user}:1000 choi
-> Redirected to slot [5474] located at 127.0.0.1:30002
OK
127.0.0.1:30002> set {user}:2000 choi
OK
127.0.0.1:30002> 
127.0.0.1:30002> set {user}:3000 choi
OK
```

**5474**번을 담당하는 슬롯은 30002번 서버인데 30002 서버 내부에서 `keys *` 명령어를 사용하면 아래와 같이 key들이 묶여있는 것을 확인할 수 있다.

```
127.0.0.1:30002> keys *
1) "user:2000"
2) "{user}:2000"
3) "{user}:3000"
4) "{user}:1000"
```

hashtag에 자세한내용은 [https://redis.io/topics/cluster-spec](https://redis.io/topics/cluster-spec)에서 찾을 수 있다.

# 마무리

Redis Cluster는 최대 1000개의 노드로 구성이 가능하다. 실제 운영에 도입함에 있어서 많은 고민을 해야겠지만 좋은 성능을 기대할 수 있을 것 같다.

<hr>
포스팅은 여기까지 하겠습니다. 퍼가실때는 출처를 반드시 남겨주세요!


**References**

- [https://redis.io](https://redis.io)
- [http://redisgate.kr/redis/](http://redisgate.kr/redis/)