---
layout: post
title: "Docker를 사용해서 Redis Replication 구성하기 Feat.Sentinel"
tags: [NoSQL, Redis, Redis Replication ,Redis Sentinel]
date: 2020-07-22
comments: true
---



<br>

# OverView

이번시간에는 Redis 에서 Replication을 구성하는 방법에 대해서 알아보고 자동 Failover를 위한  Redis Sentinel에 대해서 알아보는 시간을 갖도록 하겠다. Redis 서버는 전부 Docker Container로 구성한다.

# Redis Replication

Redis Replication은 Master & Slave로 구성되어있고 Slave는 Master의 복제본이 된다. Sentinel이나 Cluster를 구축하지 않는 이상. 고가용성은 구현하지않고 그냥 복제만 하는 구조다. 이 시스템은 세가지 매커니즘을 지원한다.

1. **마스터 & 슬레이브가 잘 연결된 상태라면 마스터에서 쓰기 작업, expire 만료 작업 등 마스터쪽에서 발생하는 데이터의 변경을 슬레이브에 스트림으로 전달하여 슬레이브를 복제본으로 유지시킨다.**
2. **네트워크 등의 문제로 마스터 & 슬레이브의 네트워크가 끊긴 뒤 재 연결되면 데이터의 일관성을 위해 부분 재 동기화를 시도한다.**
3. **부분 재 동기화가 불가능한 경우 슬레이브는 완전 재 동기화를 요청한다.**

다음은 몇가지 Redis Replication에 대한 정보이다.

1. **Redis의 마스터 슬레이브는 비동기로 데이터를 복제한다.**
2. **마스터는 여러개의 슬레이브를 가질 수 있다.**
3. **슬레이브는 또 다른 슬레이브와 연결될 수 있고 마스터 뿐만 아니라 cascading 같은 구조를 만들 수 있다.**
4. **마스터 측에서 복제작업은 non-blocking이기 때문에 하나 이상의 복제본이 동기화될때 마스터 측의 명령 처리 측면에서 영향을 받지 않는다.**
5. **마스터에서 전체 데이터를 Disk에 쓰는 비용을 줄이는 방법으로 슬레이브를 구성할 수 있다. 하지만 마스터가 재시작되면 데이터는 전부 비어있게 된다. 이때 슬레이브도 빈 데이터를 동기화시키는 것을 주의해야 한다.**

레디스에서 각 마스터는 랜덤 String 으로 이루어진 replication ID를 갖고 있고 offset정보를 갖고 있다. offset 정보는 슬레이브의 상태를 업데이트하기 위해 사용되는 정보다.

레디스 2.6 이상부터 슬레이브는 기본적으로 **Read-Only** 설정으로 되어있고 설정을 통해 바꿀 수 있다.

# Redis Master & Slave 구성하기

redis-server 같은 도구로 Replication을 구성할 수 있지만 번거롭기 때문에 docker 기반으로 구성하도록 하겠다. Redis 관련 이미지는 [https://hub.docker.com/r/bitnami/redis-sentinel](https://hub.docker.com/r/bitnami/redis-sentinel)에서 찾을 수 있다.

docker-compose를 사용하여 Replication을 구성해보도록 하자. 아직 sentinel에 대해서는 다루지 않기때문에 master & slave만 구성한다. 다음 명령어로 bridge network를 생성하자.

```
docker network create app-tier --driver bridge
```

이제 다음 docker-compose.yml로 마스터 1대와 슬레이브 2대를 구성하도록한다.

**docker-compose.yml**

```yaml
version: '2'

networks:
  app-tier:
    driver: bridge

services:
  redis:
    image: 'bitnami/redis:latest'
    environment:
      - REDIS_REPLICATION_MODE=master
      - ALLOW_EMPTY_PASSWORD=yes
    networks:
      - app-tier
    ports:
      - 6379:6379
  redis-slave-1:
    image: 'bitnami/redis:latest'
    environment:
      - REDIS_REPLICATION_MODE=slave
      - REDIS_MASTER_HOST=redis
      - ALLOW_EMPTY_PASSWORD=yes
    ports:
      - 6479:6379
    depends_on:
      - redis
    networks:
      - app-tier
  redis-slave-2:
    image: 'bitnami/redis:latest'
    environment:
      - REDIS_REPLICATION_MODE=slave
      - REDIS_MASTER_HOST=redis
      - ALLOW_EMPTY_PASSWORD=yes
    ports:
      - 6579:6379
    depends_on:
      - redis
    networks:
      - app-tier
```

다음 명령어로 컨테이너를 올린다.

```
docker-compose up -d
```

우리가 구성한 Redis Replication은 아래와 같다.

![20200721_095411](https://user-images.githubusercontent.com/30790184/88000336-4eddfe80-cb38-11ea-8dbe-35bb3f19f244.png)

이제 마스터노드에서 몇가지 명령어로 복제가 잘 되고 있는지 확인해보자. 쉘을 3개 띄워서 각각 redis 서버로 접속해서 테스트해보자.

<br>

> - redis shell에 접속하는 방법은 redis-cli 설치 (참고: [https://jojoldu.tistory.com/348](https://jojoldu.tistory.com/348))또는 다음 명령어를 통해 접속하는 방법이 있다. `docker run -it --network some-network --rm redis redis-cli -h some-redis` 
> - 이 글은 redis-cli 도구를 설치해서 사용한다.

<br>

마스터쪽 에 `set test-key test-value` 라는 명령어로 key-value 데이터를 지정하면 슬레이브쪽으로 데이터가 복제되는것을 확인할 수 있다.

![20200721_093518](https://user-images.githubusercontent.com/30790184/87999370-84cdb380-cb35-11ea-87b1-22d67db80b76.png)

![20200721_093507](https://user-images.githubusercontent.com/30790184/87999367-839c8680-cb35-11ea-9507-30940cf200ed.png)

![20200721_093513](https://user-images.githubusercontent.com/30790184/87999368-84351d00-cb35-11ea-8d58-0cb8f9459c0d.png)



> 이외에도 `info` 명령어를 통해 redis 관련 정보를 확인할 수 있다.

# Redis Sentinel

Redis Sentinel은 Redis Replication에서 만약 **마스터노드가 어떠한 원인에 의해 비정상적인 종료가되었을때 자동으로 슬레이브중 한 개를 채택해서 마스터로 승격시켜주는 역할(Failover)**을 한다. 가용성 측면에서 Redis Sentinel은 필수적인 요소이다.

Redis Sentinel은 다음 네가지 기능을 제공한다.

1. **Sentinel은 마스터 및 레플리카 인스턴스가 제대로 동작하는지 지속적으로 점검한다.**
2. **Sentinel API를 통해 관리자나 프로그램에 문제가 있음을 알리 수 있다.**
3. **마스터가 제대로 동작하지 않을 경우 복제본 중 한 개를 골라서 마스터로 승격시키고 승격된 마스터에 대한 새로운 엔드포인트를 제공한다.**
4. **Sentinel은 서비스 디스커버리를 지원한다.**

센티넬을 구성할때는 최소 3대 이상으로 구성하고 가능하면 홀수개로 구성해야 한다. 다음은 센티넬 구성시 필요한 설정이다.

```
sentinel monitor mymaster 127.0.0.1 6379 2
sentinel down-after-milliseconds mymaster 60000
sentinel failover-timeout mymaster 180000
```

첫번째줄에 `sentinel monitor mymaster 127.0.0.1 6379 2` 를 살펴보면 맨마지막에 입력된 `2` 라는 값은 quorum이라는 값인데 **마스터의 장애를 동의하는 센티널 개수**이다. 이후에 마스터의 장애를 판명지었다면 **센티넬 공정에따라 새로운 마스터의 승격 자격을 가진 센티널들을 투표하고 선택된 센티넬이 새로운 마스터를 승격**시킨다. 따라서 센티넬은 홀수개로 구성하는 것을 권장한다.

그외 자세한 설명은 생략한다. 이제 이전에 사용한 docker-compose.yml 파일에 다음 스크립트를 추가해서 센티넬도 구성하도록 해 보자.

## Redis Sentinel 설정하기

**docker-compose.yml**

```yaml
# docker-compose.yml 맨 하단부에 추가

  redis-sentinel:
    image: 'bitnami/redis-sentinel:latest'
    environment:
      - REDIS_SENTINEL_DOWN_AFTER_MILLISECONDS=3000
      - REDIS_MASTER_HOST=redis
      - REDIS_MASTER_PORT_NUMBER=6379
      - REDIS_MASTER_SET=mymaster
      - REDIS_SENTINEL_QUORUM=2
    depends_on:
      - redis
      - redis-slave-1
      - redis-slave-2
    ports:
      - '26379-26381:26379'
    networks:
      - app-tier
```

- **REDIS_SENTINEL_DOWN_AFTER_MILLISECONDS :** 센티넬이 마스터노드의 장애여부를 판단하는 시간이다. 테스트를위해 3초로 지정해놨다.
- **REDIS_MASTER_HOST :** master 노드의 호스트
- **REDIS_MASTER_PORT_NUMBER :** master 노드의 포트번호
- **REDIS_MASTER_SET :** 모니터링 할 Redis 인스턴스 set의 이름
- **REDIS_SENTINEL_QUORUM :** master의 장애를 동의하는 센티널의 개수

<br>

> 이외에도 추가적인 설정이 필요하다면 참고해도 좋다.
>
> - `REDIS_MASTER_HOST`: Host of the Redis master to monitor. Default: **redis**.
> - `REDIS_MASTER_PORT_NUMBER`: Port of the Redis master to monitor. Default: **6379**.
> - `REDIS_MASTER_SET`: Name of the set of Redis instances to monitor. Default: **mymaster**.
> - `REDIS_MASTER_PASSWORD`: Password to authenticate with the master. No defaults. As an alternative, you can mount a file with the password and set the `REDIS_MASTER_PASSWORD_FILE` variable.
> - `REDIS_MASTER_USER`: Username to authenticate with when ACL is enabled for the master. No defaults. This is available only for redis 6 or higher. If not specified, Redis Sentinel will try to authenticate with just the password (using `sentinel auth-pass <master-name> <password>`).
> - `REDIS_SENTINEL_PORT_NUMBER`: Redis Sentinel port. Default: **26379**.
> - `REDIS_SENTINEL_QUORUM`: Number of Sentinels that need to agree about the fact the master is not reachable. Default: **2**.
> - `REDIS_SENTINEL_PASSWORD`: Password to authenticate with this sentinel and to authenticate to other sentinels. No defaults. Needs to be identical on all sentinels. As an alternative, you can mount a file with the password and set the `REDIS_SENTINEL_PASSWORD_FILE` variable.
> - `REDIS_SENTINEL_DOWN_AFTER_MILLISECONDS`: Number of milliseconds before master is declared down. Default: **60000**.
> - `REDIS_SENTINEL_FAILOVER_TIMEOUT`: Specifies the failover timeout in milliseconds. Default: **180000**.
> - `REDIS_SENTINEL_TLS_ENABLED`: Whether to enable TLS for traffic or not. Default: **no**.
> - `REDIS_SENTINEL_TLS_PORT_NUMBER`: Port used for TLS secure traffic. Default: **26379**.
> - `REDIS_SENTINEL_TLS_CERT_FILE`: File containing the certificate file for the TSL traffic. No defaults.
> - `REDIS_SENTINEL_TLS_KEY_FILE`: File containing the key for certificate. No defaults.
> - `REDIS_SENTINEL_TLS_CA_FILE`: File containing the CA of the certificate. No defaults.
> - `REDIS_SENTINEL_TLS_DH_PARAMS_FILE`: File containing DH params (in order to support DH based ciphers). No defaults.
> - `REDIS_SENTINEL_TLS_AUTH_CLIENTS`: Whether to require clients to authenticate or not. Default: **yes**.
> - `REDIS_SENTINEL_ANNOUNCE_IP`: Use the specified IP address in the HELLO messages used to gossip its presence. Default: **auto-detected local address**.
> - `REDIS_SENTINEL_ANNOUNCE_PORT`: Use the specified port in the HELLO messages used to gossip its presence. Default: **port specified in `REDIS_SENTINEL_PORT_NUMBER`**.



<br>

컨테이너를 `docker-compose up --scale redis-sentinel=3 -d ` 명령어로 업데이트해서 센티넬 개수를 3개로 늘려주자. 센티넬 쉘에 접속한 뒤 `info` 명령어를 입력해보자.

센티넬 관련된 정보가 아래와 같이 나올것이다.

```
# Sentinel
sentinel_masters:1
sentinel_tilt:0
sentinel_running_scripts:0
sentinel_scripts_queue_length:0
sentinel_simulate_failure_flags:0
master0:name=mymaster,status=ok,address=192.168.0.2:6379,slaves=2,sentinels=3
```

추가로 `sentinel master mymaster` 를 입력하면 master에 대한 상세정보가 나온다.

```
 127.0.0.1:26379> sentinel master mymaster
 1) "name"
 2) "mymaster"
 3) "ip"
 4) "192.168.0.2"
 5) "port"
 6) "6379"
 7) "runid"
 8) "025f3747f2388af32ebbfdf23579607ee3802c20"
 9) "flags"
10) "master"
11) "link-pending-commands"
12) "0"
13) "link-refcount"
14) "1"
15) "last-ping-sent"
16) "0"
17) "last-ok-ping-reply"
18) "32"
19) "last-ping-reply"
20) "32"
21) "down-after-milliseconds"
22) "3000"
23) "info-refresh"
24) "3884"
25) "role-reported"
26) "master"
27) "role-reported-time"
28) "244829"
29) "config-epoch"
30) "0"
31) "num-slaves"
32) "2"
33) "num-other-sentinels"
34) "2"
35) "quorum"
36) "2"
37) "failover-timeout"
38) "180000"
39) "parallel-syncs"
40) "1"
```

이제 우리가 구성한 이 시스템은 아래와 같은 그림을 하게 된다.

![20200721_095419](https://user-images.githubusercontent.com/30790184/88000337-500f2b80-cb38-11ea-88df-d2996582fc29.png)



## Failover 테스트하기

Redis Sentinel이 문제없이 구성되었다면 마스터노드의 장애시 자동으로 Failover 되는지 한번 확인해보도록 하자. 현재 시스템에서는 **192.168.0.2** 서버가 마스터이다. 확인을 위해 센티넬 서버에서 다음 명령어를 입력해보자.

```
SENTINEL get-master-addr-by-name mymaster
```

![20200721_091750](https://user-images.githubusercontent.com/30790184/87999708-a8452e00-cb36-11ea-81a0-5c54dc2572fb.png)

이제 다음 명령어를 통해 약 30초간 마스터노드를 sleep 시켜보자.

```
redis-cli -p 6379 DEBUG sleep 30
```

`docker logs -f {sentinel id}` 명령어를 사용해서 센티넬 로그를 확인해보면 약 3초뒤에 다음과같은 로그가 찍히는 것을 확인할 수 있다.

![20200721_094929](https://user-images.githubusercontent.com/30790184/88000002-7e403b80-cb37-11ea-94be-0672019288df.png)

17ae6ef1 이라는 센티넬이 마스터를 승격시킬 권한을 얻고 마스터를 **192.168.0.2** 에서 **192.168.0.4** 서버로 승격시킨것을 확인할 수 있다.

이후에 센티넬서버에서 다시 아래의 명령어를 입력하면

```
SENTINEL get-master-addr-by-name mymaster
```

![20200721_091853](https://user-images.githubusercontent.com/30790184/87999850-0f62e280-cb37-11ea-94e2-d1e74b66775f.png)

마스터의 정보가 **192.168.0.4** 로 나오는 것을 확인할 수 있다. 위에서 지정한 30초가 지나면 **192.168.0.2** 서버는 다시 slave 서버로 추가된다. 마스터로 접속해서 확인해보자.

![20200721_092229](https://user-images.githubusercontent.com/30790184/88000070-b5aee800-cb37-11ea-8138-f5f6aa7c10d6.png)

failover가 적절하게 수행됐다면 아래와 같은 그림이 된다.

![20200721_095516](https://user-images.githubusercontent.com/30790184/88000338-500f2b80-cb38-11ea-8cdd-81f8abb2c146.png)

# 마무리

운영측면에서 고가용성은 필수적이다. 서버가 늘어나는 만큼 그만큼 관리포인트도 늘어나지만 이런식으로 장애에 대처할 수 있는 능력은 아주 매리트가 있는것 같다.



<hr>
포스팅은 여기까지 하겠습니다. 퍼가실때는 출처를 반드시 남겨주세요!


**References**

- [https://redis.io/topics/replication](https://redis.io/topics/replication)
- [https://redis.io/topics/sentinel](https://redis.io/topics/sentinel)