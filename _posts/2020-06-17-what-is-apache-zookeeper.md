---

layout: post
title: "Apache Zookeeper 시작하기"
tags: [Apache, Apache Zookeeper]
date: 2020-06-17
comments: true
---



<br>

# OverView

오늘은 Apache Zookeeper에 대해서 알아보도록 하겠다.



# Zookeeper란?

Zookeeper는 분산환경에서 사용하는 오픈소스 coordination 프로그램이다. coordination은 **조화 또는 합동**이라는 다음과 같은 뜻을 갖고 있는데 이뜻은 아래와 같다.


> **조화**(Harmony)는 두 개 이상의 여러가지 [디자인](https://ko.wikipedia.org/wiki/디자인) 요소들의 상호관계가 분리되지 않고 잘 아울려 나타나는 [미적](https://ko.wikipedia.org/wiki/미적분) [형식](https://ko.wikipedia.org/wiki/형식)이다. 균형감을 잃지 않는 상태에서의 변화와 통일을 포함한 전체적인 결합 상태이기도 한다.

주키퍼는 다음과 같은 몇가지 기능을 제공한다.

1. **분산환경에서 높은 수준의 동기화 기능**
2. **Configuration 유지보수 기능**
3. **Application의 그룹과 네이밍 기능**

주키퍼는 파일 시스템을 트리구조 형태로 사용하도록 설계되었다.

![](https://zookeeper.apache.org/doc/current/images/zknamespace.jpg)



주키퍼를 사용하면 분산 된 프로세스를 표준 파일시스템과 유사하게 구성된 공유 계층 namespace를 통해서 서로 조정할 수 있게 된다. 이 namespace는 주키퍼 내부에서 znodes라는 용어를 사용하고 데이터 레지스터로 구성되며 파일 및 디렉토리와 유사한 형태를 갖는다. 위 그림에서 znode들은 /app1, /app2 에 해당한다. 일반적인 파일 시스템과 달리 주키퍼는 메모리기반으로 동작하기때문에 높은 처리량과 낮은 대기시간을 갖고 있다.

![](https://zookeeper.apache.org/doc/current/images/zkservice.jpg)

또 주키퍼는 복제구성을 지원하는데 이것을 앙상블이라고 부르고 있다. 클라이언트는 단일 주키퍼 서버에 연결 되어 있고 TCP 연결을 유지하여 request, response, event 등의 정보를 받고 있는다. 만약 TCP 연결이 끊어진다면 다른 서버로 연결한다. 주키퍼에 조금 더 자세한 설명을 원한다면 [https://zookeeper.apache.org/doc/current/zookeeperOver.html](https://zookeeper.apache.org/doc/current/zookeeperOver.html) 을 확인해보자.

백문이불여일타! 직접 서버를 올려보고 주키퍼를 사용해보도록 하겠다!

# Zookeeper 시작하기

주키퍼를 사용하기 위해서 주키퍼를 다운받아보자. 이 글을 쓰는 시점에서 최신버전은 3.6.1이다.

[https://zookeeper.apache.org/releases.html](https://zookeeper.apache.org/releases.html)

centos7에서 진행할 예정이기 때문에 다음 명령어로 파일을 받고 압축을 풀어보자.

```
wget http://mirror.navercorp.com/apache/zookeeper/zookeeper-3.6.1/apache-zookeeper-3.6.1-bin.tar.gz
tar -zxvf apache-zookeeper-3.6.1-bin.tar.gz
```

다운로드가 끝나고 압축을 풀면 **apache-zookeeper-x.x.x-bin/conf** 디렉토리로 이동해서 **zoo.cfg** 파일을 만들고 구성정보를 입력해보자



```
vi {zk_home}/conf/zoo.cfg



----------------------------

tickTime=2000
dataDir=/var/lib/zookeeper
clientPort=2181


----------------------------
```

여기서 설정한 옵션 세가지는 다음과 같다

- **tickTime:** 주키퍼가 사용하는 기본적인 heartbeat 시간, 세션의 최소 timeout 시간은 tickTime의 두배가된다.
- **dataDir:** 메모리 데이터베이스 스냅샷, 데이터베이스 업데이트 트랜잭션 로그가 저장될 디렉토리 위치이다.
- **clientPort:** 클라이언에게 노출될 포트이다.

설정이 완료됐다면 **apache-zookeeper-x.x.x-bin/bin** 디렉토리로 이동해서 주키퍼 서버를 시작해보자

```
{zk_home}/bin/zkServer.sh start


...

ZooKeeper JMX enabled by default
Using config: /root/apache-zookeeper-3.6.1-bin/bin/../conf/zoo.cfg
Starting zookeeper ... STARTED
```

**Starting zookeeper ... STARTED**가 출력되었다면 주키퍼 서버가 적절하게 올라간거다.

주키퍼는 log4j를 사용하는데 추가 설정이 필요하다면 아까 zoo.cfg 파일을 만들었던 **apache-zookeeper-x.x.x-bin/conf** 디렉토리에서 log4j 관련 설정파일을 설정해주면 된다. 더욱 자세한 설명은 [https://zookeeper.apache.org/doc/current/zookeeperProgrammers.html#Logging](https://zookeeper.apache.org/doc/current/zookeeperProgrammers.html#Logging)을 확인해보자.

이번 예제는 간단하게 **standalone** 모드로 동작시키기때문에 replica 구성하지 않는다. 이말은 만약 주키퍼 서버가 어떠한 환경에 의해 종료되었을때 서비스가 완전히 종료된다는 것이다. 운영 서버에서는 반드시 앙상블을 구성해서 운영해야한다. 주키퍼 앙상블 구성은 [https://zookeeper.apache.org/doc/current/zookeeperStarted.html#sc_RunningReplicatedZooKeeper](https://zookeeper.apache.org/doc/current/zookeeperStarted.html#sc_RunningReplicatedZooKeeper) 을 확인해보자.



이제 주키퍼 쉘 도구로 주키퍼 서버에 직접액세스해보도록 하자

```
{zk_home}/bin/zkCli.sh -server localhost:2181
```

아무 문제없이 잘 접속했다면 **Welcome to ZooKeeper!** 의 출력과 함께 바뀐 커맨드라인 프롬프트를 확인하면 문제없이 잘 실행된 것이다.

```
Welcome to ZooKeeper!
JLine support is enabled

...


WATCHER::

WatchedEvent state:SyncConnected type:None path:null

[zk: localhost:2181(CONNECTED) 0] ...
```



주키퍼는 설정에 필요한 노드들을 생성할 수 있도록 간단한 api를 제공한다.

## Zookeeper Simple API

- *create* : 트리의 특정 위치에 노드를 생성한다.
- *delete* : 노드를 제거한다.
- *exists* : 특정 위치에 노드가 있는지 확인한다.
- *get data* : 노드의 데이터를 읽는다.
- *set data* : 노드에 데이터를 입력한다.
- *get children* : 노드의 자식목록을 검색한다.
- *sync* : 데이터를 전파시킨다.



기본적인 api 이외에도 클라이언트에서 실행할 수 있는 명령어를 확인하려면 커맨드라인에 **help**를 입력해서 확인하면 된다.

```
[zk: localhost:2181(CONNECTED) 0] help
ZooKeeper -server host:port -client-configuration properties-file cmd args
        addWatch [-m mode] path # optional mode is one of [PERSISTENT, PERSISTENT_RECURSIVE] - default is PERSISTENT_RECURSIVE
        addauth scheme auth
        close 
        config [-c] [-w] [-s]
        connect host:port
        create [-s] [-e] [-c] [-t ttl] path [data] [acl]
        delete [-v version] path
        deleteall path [-b batch size]
        delquota [-n|-b] path
        get [-s] [-w] path
        getAcl [-s] path
        getAllChildrenNumber path
        getEphemerals path
        history 
        listquota path
        ls [-s] [-w] [-R] path
        printwatches on|off
        
        ...
        
```



이제 아래 명령어를 사용해서 직접 노드를 생성하고 확인해보도록 하자

```
create /zk_test myData
ls /
```

`create` 옵션으로 `/zk_test` 라는 위치에 `myData` 라는 데이터를 입력한 뒤 `ls /` 을 통해서 생성한 노드를 확인할 수 있다.

```
[zk: localhost:2181(CONNECTED) 13] create /zk_test myData
Created /zk_test
[zk: localhost:2181(CONNECTED) 14] ls /
[zk_test, zookeeper]
[zk: localhost:2181(CONNECTED) 15] 
```

생성한 노드를 읽어들이려면 `get /zk_test` 를 사용해서 내부 데이터를 확인할 수 있다.

```
[zk: localhost:2181(CONNECTED) 15] get /zk_test
myData
```

`set /zk_test` 를 이용해서 데이터를 변경시킬수도 있다. 

```
[zk: localhost:2181(CONNECTED) 27] set /zk_test changed
[zk: localhost:2181(CONNECTED) 29] get /zk_test
changed
```

마지막으로 `delete /zk_test` 를 사용해서 생성한 노드를 지워보자.

```
[zk: localhost:2181(CONNECTED) 30] delete /zk_test
[zk: localhost:2181(CONNECTED) 31] ls /
[zookeeper]
```



# Zookeeper에서 Kafka 정보 확인하기

다들 아시다시피 kafka를 사용하기 위해서는 zookeeper 서버가 필요하다. 카프카 내부에 설정정보를 두지 않고 주키퍼를 통해서 설정정보나 클러스터 정보를 관리한다. 카프카에서 토픽을 생성할때 `--zookeeper` 옵션으로 주키퍼 서버를 가리킨적이 있을 것이다.

```
./bin/kafka-topics.sh --create \
    --replication-factor 3 \
    --partitions 3 \
    --topic my-test-topic\
    --zookeeper  localhost:2181
```

카프카의 모든 토픽정보는 주키퍼에 저장된다. 다음 명령어를 통해서 토픽의 목록을 확인해보자.

`ls /brokers/topics`

```
[zk: localhost:2181(CONNECTED) 10] ls /brokers/topics
[__confluent.support.metrics, __consumer_offsets, my-test-topic, my-test-topic-2]
```

테스트용으로 생성했던 토픽들의 목록이 잘 나오는 것을 확인할 수 있다.

토픽 내부에 데이터도 한번 확인해보자.

```
[zk: localhost:2181(CONNECTED) 16] get /brokers/topics/my-test-topic
{"version":2,"partitions":{"2":[2,3,1],"1":[1,2,3],"0":[3,1,2]},"adding_replicas":{},"removing_replicas":{}}
```

버전정보, 파티션정보 등등이 json 형식으로 입력되어 있는 것을 확인할 수 있다.

<br>

<br>

<hr>
포스팅은 여기까지 하겠습니다. 퍼가실때는 출처를 반드시 남겨주세요!


<br>

**References**

- [https://zookeeper.apache.org/](https://zookeeper.apache.org/)
- [https://zookeeper.apache.org/doc/r3.6.1/index.html](https://zookeeper.apache.org/doc/r3.6.1/index.html)