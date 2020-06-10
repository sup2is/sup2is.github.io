---

layout: post
title: "Kafka 조금 더 쉽게 이해하기"
tags: [Apache Kafka]
date: 2020-06-10
comments: true
---



<br>

# OverView

이번시간에는 내 멋대로 한번 카프카를 조금 더 쉽게 이해할 수 있도록 하기 위해서 docker-compose로 카프카 클러스터를 구성하고 replicas, partitions, consumer-group에 대해서 간단한 예제와 함께 설명하는 시간을 갖도록 하겠다.

# 시작하기전에

준비물이 몇가지 필요하다.

- docker
- docker-compose

예제는 docker 컨테이너로 카프카3대와 주키퍼 서버1대를 올릴 것이다. docker-compose를 사용한다.

kafka에서 제공하는 .sh 파일을 사용하기 위해서 kafka 관련 파일들을 다운받으려면 아래에서 다운 받을 수 있다

- apache kafka : [https://kafka.apache.org/](https://kafka.apache.org/)
- confluent kafka : [https://www.confluent.io/](https://www.confluent.io/)

confluent kafka가 조금더 많은 기능을 내장하고 있다. community 버전 이외에도 commercial버전도 제공하고 있다. 참고로 confluent는 linkedin에서 최초에 kafka를 만든 사람들이 따로 나와서 설립한 회사이다.

나는 이 예제에선 apache kafka를 사용한다.

# docker-compose로 클러스터 구성하기

docker-compose를 직접 작성하지 않아도 누군가 만들어 놓은 compose 파일들이 있다. 나는 [https://github.com/simplesteph/kafka-stack-docker-compose](https://github.com/simplesteph/kafka-stack-docker-compose)에 있는 **zk-single-kafka-multiple.yml** 을 사용한다.

카프카 서버 3대와 주키퍼 서버 1대의 구성으로 클러스터를 구성한다.

```
version: '2.1'

services:
  zoo1:
    image: zookeeper:3.4.9
    hostname: zoo1
    ports:
      - "2181:2181"
    environment:
        ZOO_MY_ID: 1
        ZOO_PORT: 2181
        ZOO_SERVERS: server.1=zoo1:2888:3888
    volumes:
      - ./zk-single-kafka-multiple/zoo1/data:/data
      - ./zk-single-kafka-multiple/zoo1/datalog:/datalog

  kafka1:
    image: confluentinc/cp-kafka:5.5.0
    hostname: kafka1
    ports:
      - "9092:9092"
    environment:
      KAFKA_ADVERTISED_LISTENERS: LISTENER_DOCKER_INTERNAL://kafka1:19092,LISTENER_DOCKER_EXTERNAL://${DOCKER_HOST_IP:-127.0.0.1}:9092
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: LISTENER_DOCKER_INTERNAL:PLAINTEXT,LISTENER_DOCKER_EXTERNAL:PLAINTEXT
      KAFKA_INTER_BROKER_LISTENER_NAME: LISTENER_DOCKER_INTERNAL
      KAFKA_ZOOKEEPER_CONNECT: "zoo1:2181"
      KAFKA_BROKER_ID: 1
      KAFKA_LOG4J_LOGGERS: "kafka.controller=INFO,kafka.producer.async.DefaultEventHandler=INFO,state.change.logger=INFO"
    volumes:
      - ./zk-single-kafka-multiple/kafka1/data:/var/lib/kafka/data
    depends_on:
      - zoo1

  kafka2:
    image: confluentinc/cp-kafka:5.5.0
    hostname: kafka2
    ports:
      - "9093:9093"
    environment:
      KAFKA_ADVERTISED_LISTENERS: LISTENER_DOCKER_INTERNAL://kafka2:19093,LISTENER_DOCKER_EXTERNAL://${DOCKER_HOST_IP:-127.0.0.1}:9093
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: LISTENER_DOCKER_INTERNAL:PLAINTEXT,LISTENER_DOCKER_EXTERNAL:PLAINTEXT
      KAFKA_INTER_BROKER_LISTENER_NAME: LISTENER_DOCKER_INTERNAL
      KAFKA_ZOOKEEPER_CONNECT: "zoo1:2181"
      KAFKA_BROKER_ID: 2
      KAFKA_LOG4J_LOGGERS: "kafka.controller=INFO,kafka.producer.async.DefaultEventHandler=INFO,state.change.logger=INFO"
    volumes:
      - ./zk-single-kafka-multiple/kafka2/data:/var/lib/kafka/data
    depends_on:
      - zoo1


  kafka3:
    image: confluentinc/cp-kafka:5.5.0
    hostname: kafka3
    ports:
      - "9094:9094"
    environment:
      KAFKA_ADVERTISED_LISTENERS: LISTENER_DOCKER_INTERNAL://kafka3:19094,LISTENER_DOCKER_EXTERNAL://${DOCKER_HOST_IP:-127.0.0.1}:9094
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: LISTENER_DOCKER_INTERNAL:PLAINTEXT,LISTENER_DOCKER_EXTERNAL:PLAINTEXT
      KAFKA_INTER_BROKER_LISTENER_NAME: LISTENER_DOCKER_INTERNAL
      KAFKA_ZOOKEEPER_CONNECT: "zoo1:2181"
      KAFKA_BROKER_ID: 3
      KAFKA_LOG4J_LOGGERS: "kafka.controller=INFO,kafka.producer.async.DefaultEventHandler=INFO,state.change.logger=INFO"
    volumes:
      - ./zk-single-kafka-multiple/kafka3/data:/var/lib/kafka/data
    depends_on:
      - zoo1
```

아래 명령어로 컨테이너를 구성해보자

```
export DOCKER_HOST_IP=127.0.0.1
docker-compose -f zk-single-kafka-multiple.yml up
```

docker-compose로 컨테이너들이 잘 구성되었다면 kafka home 디렉토리에서 다음 명령어를 입력해보자.

```
./bin/zookeeper-shell.sh localhost:2181 ls /brokers/ids

#결과
[1, 2, 3]
```

우리가 브로커 컨테이너를 생성할때 KAFKA_BROKER_ID를 각각 브로커 서버마다 1, 2, 3 으로 줬기 때문에 결과값도 [1, 2, 3] 으로 나온다.

현재까지의 구성을 그림으로 표현하자면 아래와 같다.



![주석 2020-06-10 082816](https://user-images.githubusercontent.com/30790184/84219980-baca5180-ab0c-11ea-9e99-c5816d825f4d.png)

# 카프카 토픽 사용하기

## 토픽 만들기

가장먼저 해볼일은 카프카에 토픽을 생성하는 것이다. 아래 명령으로 토픽을 생성해보자

```
./bin/kafka-topics.sh --create \
    --replication-factor 3 \
    --partitions 3 \
    --topic my-test-topic\
    --zookeeper  localhost:2181
```

--replication-factor와 --partitions는 아래에서 설명한다.

## 토픽 확인하기

```
./bin/zookeeper-shell.sh localhost:2181 ls /brokers/topics

#결과
[__confluent.support.metrics, __consumer_offsets, my-test-topic]
```

__이 붙은 토픽들은 카프카가 사용하는 토픽이라고 생각하면 된다. my-test-topic이 잘 생성된 것을 확인할 수 있다.

# Replication-factor  이해하기

카프카는 메시지를 중계함과 동시에 서버가 고장 났을때 수신한 메시지를 잃지 않기 위해서 **복제구조**를 갖추고 있다. 파티션은 단일 또는 여러개의 레플리카로 구성되고 여러 레플리카 중 최소 한개는 Leader, 나머지는 Follower가 된다. 

```
./bin/kafka-topics.sh --create \
    --replication-factor 3 \
    --partitions 3 \
    --topic my-test-topic\
    --zookeeper  localhost:2181
```

위에서 생성한 이 토픽은 아래의 그림처럼 생성이된다.



![주석 2020-06-10 104014](https://user-images.githubusercontent.com/30790184/84219984-bbfb7e80-ab0c-11ea-9316-e80229c9ffd5.png)

각각 브로커별로 Leader들이 분류되어있는 모습을 확인할 수 있다.

명령어로도 직접 확인할 수 있다.

```
./bin/kafka-topics.sh --zookeeper localhost:2181 --describe --topic my-test-topic
```



![주석 2020-06-10 104002](https://user-images.githubusercontent.com/30790184/84219982-bbfb7e80-ab0c-11ea-8cc5-06a6160ca465.png)



## ISR (In-Sync Replica)

사진 맨 끝에 보면 ISR이 있는데 ISR은 Leader 레플리카의 복제 상태를 유지하고 있는 레플리카로 Leader를 갖고 있는 서버가 고장났을때 후보가 될 수 있는 브로커를 나타낸다.

만약 도커 컨테이너에서 브로커 서버 2, 3을 내린다면 아래와 같은 그림이 된다.



![주석 2020-06-10 104103](https://user-images.githubusercontent.com/30790184/84219986-bc941500-ab0c-11ea-9cae-9fee1c2f849c.png)



명령어로도 직접 확인해 보자

```
./bin/kafka-topics.sh --zookeeper localhost:2181 --describe --topic my-test-topic
```



![20200610_112155](https://user-images.githubusercontent.com/30790184/84219989-bd2cab80-ab0c-11ea-94e9-58d7c317d2e0.png)



만약 다시 브로커 서버 2와 3을 올린다고 하더라도 Leader는 변경되지 않는다.

# 카프카의 kafka-console-consumer, kafka-console-producer 사용해보기

## kafka-console-producer 

카프카는 간편하게 테스트해볼 수 있게 producer와 consumer를 console 형태로 제공한다. 아래의 명령어로 producer를 실행시켜 보자.

```
./bin/kafka-console-producer.sh --broker-list localhost:9092,localhost:9093,localhost:9094 --topic my-test-topic
```

**--broker-list** 에 카프카 서버들의 주소를 입력하고 --topic 옵션으로 방금 생성한 토픽의 이름을 넣어준다.

쉘이 활성화되면 텍스트를 몇개 입력을 해보자. 아직 컨슈머가 없더라도 괜찮다.

카프카는 메시지를 디스크에 영속시키는 특징이 있다. 따라서 현재 컨슈머가 없더라도 offset 이라는 정보를 통해서 이 메시지를 소비했는지 아닌지를 판별한 뒤에 소비하지 않은 메시지라면 컨슈머를 통해서 메시지를 소비하게 한다. 

## kafka-console-consumer

이제 consumer console을 실행시켜보자

```
./bin/kafka-console-consumer.sh --bootstrap-server localhost:9092,localhost:9093,localhost:9094 --topic my-test-topic 
```

아까 producer에서 보냈던 메시지들이 입력된 것을 확인할 수 있다.

# 카프카의 consumer group과 partitions이해하기

카프카는 consumer group과 partitions라는 개념을 통해서 분산처리 모델을 구현했다. 간단하게 설명하면 **같은 consumer group을 가진 컨슈머 그룹이 같은 토픽을 구독하고있다면 메시지를 하나의 컨슈머에게만 소비**하게 한다. 먼저 컨슈머 그룹 없이 my-test-topic을 구독하고 있을때를 보자.


## Consumer Group이 없을때

producer쪽에서 메시지를 입력했을때 아래와 같이 구독하고 있는 모든 컨슈머에 메시지가 출력되는 것을 확인할 수 있다.



![2020-06-10-10-53-50](https://user-images.githubusercontent.com/30790184/84219991-be5dd880-ab0c-11ea-867c-4b17ac14ee86.gif)



## Consumer Group이 있을때

아래 명령어로 컨슈머 그룹을 my-group으로 할당시켜보자.

```
./bin/kafka-console-consumer.sh --bootstrap-server localhost:9092,localhost:9093,localhost:9094 --topic my-test-topic --from-beginning --consumer-property group.id=my-group
```

**--consumer-property group.id=my-group** 을 통해서 my-group이라는 컨슈머그룹을 할당했다. 이제 producer 쪽에서 메시지를 여러개 보내보면 메시지가 Consumer Group이 없을때와는 다르게 하나의 컨슈머에게만 메시지를 전달하는 것을 확인할 수 있다.



![2020-06-10-10-55-56](https://user-images.githubusercontent.com/30790184/84219990-bdc54200-ab0c-11ea-84ca-3be6d5badbb9.gif)



이 partition와 consumer의 관계는 다음과같다.

- **partition은 무조건 한개의 consumer를 갖는다.**
- **consumer는 0개 이상의 partition을 갖는다.**

아래 그림을 보면 조금 더 이해가 쉬울 것이다.

![그림1](https://user-images.githubusercontent.com/30790184/84220266-58be1c00-ab0d-11ea-93aa-14ca9b99bf0e.png)





## kafka-consumer-groups.sh 사용하기

마지막으로 kafka-consumer-groups.sh 을 사용해서 현재 파티션에 대한 상세정보들을 확인할 수 있다. 아래명령어를 입력해보자.

```
./bin/kafka-consumer-groups.sh --describe --group my-push-topic --bootstrap-server localhost:9092,localhost:9093,localhost:9094
```

현재는 모든 파티션에 컨슈머들이 연결되어있기때문에 아래와 같은 그림이 나온다.

![20200610_113852](https://user-images.githubusercontent.com/30790184/84221076-59f04880-ab0f-11ea-8702-784f74ad92c9.png)

이전에 실행했던 consumer 쉘을 두개 종료해서 한개만 실행시킨다면 아래와 같은 그림이 된다.

![20200610_113951](https://user-images.githubusercontent.com/30790184/84221080-5b217580-ab0f-11ea-98f2-a5bc0147f398.png)

이전과 비교했을때 **consumer-id**가 하나로 통일된 것을 확인할 수 있다.



마지막으로 모든 consumer 쉘을 모두 종료시킨 뒤 producer에 어느정도의 메시지를 보내고 한번 더 확인하면 아래와 같이 LAG 목록에 축적된 메시지의 개수가 나온다. CURRNT-OFFSET + LAG = LOG-END-OFFSET이 된다.

![20200610_114122](https://user-images.githubusercontent.com/30790184/84221202-a5a2f200-ab0f-11ea-89f2-c4232f0b3a72.png)

<hr>
포스팅은 여기까지 하겠습니다. 퍼가실때는 출처를 반드시 남겨주세요!


<br>