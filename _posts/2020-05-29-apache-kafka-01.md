---

layout: post
title: "Apache Kafka 시작하기 #1"
tags: [Apache Kafka]
date: 2020-05-03
comments: true
show: false
---



<br>

# OverView



![apache kafka](https://kafka.apache.org/images/logo.png)

출처 : https://kafka.apache.org/

<br>

Apache Kafka는 2011년 링크드인에서 시작한 오픈소스 소프트웨어이다. 초기에 링크드인 웹 사이트에서 생성되는 로그를 처리하여 웹 사이트 활동을 추적하는 것을 목적으로 개발되었다. 

Apache Kafka는 **여러 대의 분산 서버에서 대량의 데이터를 처리하는 분산 메시징 시스템**이다. 메시지를 받고 받은 메시지를 다른 시스템이나 장치에 보내기 위한 목적으로 사용한다.

요즘에 정말 많이사용하는 Apache Kafka는 이제 더이상 옵션이 아니라 필수로 다가오는 것 같다 이 Apache Kafka에 대해서 자세하게 알아보자!

<br>

![apache kafka](https://kafka.apache.org/images/kafka_diagram.png)

출처 : https://kafka.apache.org/

# Kafka의 탄생목적

LinkedIn에서 Kafka를 개발하기 이전에도 비슷한 느낌의 Message Queue들이 오픈소스로 존재했는데 대표적인 오픈소스중에 하나로 **Rabbit MQ** 또는 **Active MQ** 등을 뽑을 수 있다.

실제로 Rabbit MQ의 release note( https://www.rabbitmq.com/versions.html)를 보면 2007년에 1.0.0 버전이 릴리즈된 것을 확인할 수 있다.

그렇다면 왜 LinkedIn은 기존에 있던 RabbitMQ를 사용하지 않고 Kafka를 개발했을까? 

<br>

이전 시스템들의 강력한 메시지 전달 보장, Scale Out, 메시지를 Disk에쓰는가? 아니면 Memory에 쓰는가? 등의 다양한 이유가 있지만 Kafka의 존재이유는 **매우 높은 처리량**에 있다.  전세계 사용자들의 로그데이터를 분산환경에서 적절하게 처리해야했기 때문에 이전시스템들보다 조금 더 좋은 성능의 아키텍쳐가 필요했던 것이다. 그렇다고 RabbitMQ나 ActiveMQ가 성능이 떨어지는 것은 절대 아니다.

<br>

![주석 2020-05-29 145728](https://user-images.githubusercontent.com/30790184/83240073-cdbb5880-a1d3-11ea-88e2-dcb37b22673e.png)

<br>

# LinkedIn의 Kafka 실현 목표

LinkedIn은 다음과 같이 Kafka의 실현 목적을 정의했다.

1. 높은 처리량으로 실시간 처리
   - 전 세계 사용자들의 방대한 액세스 로그를 감당하기위해 매우 높은 처리량을 필요로 함
2. 임의의 시간에 데이터를 읽기
   - 메시지를 단순히 실시간 처리 뿐만 아니라 임의의 시간에 읽어들일 수 있도록 메시지를 Memory에 저장하지 않고 Disk에 저장시키도록 함
3. 다양한 제품과 시스템에 쉽게 연동
   - 이용 목적에 따라 DB, 데이터웨어하우스 등의 보다 쉬운 API를 제공해서 손쉬운 연결을 하도록 함
4. 메시지 손실방지
   - Kafka 서버가 갑작스럽게 종료되더라도 Replication을 통해 메시지가 손실되는 것을 방지하도록 함

# Queueing Model VS Pub/Sub Model

카프카는 Queueing Model과 Pub/Sub Model 모델을 동시에 구현하기위해 새로운 개념인 Consumer Group과 Partition이라는 개념을 만들었다. Consumer Group과 Partition 설명 이전에 Queueing Model과 Pub/Sub Model 에 대해서 간단하게 알아보자

## Queueing Model

![주석 2020-05-29 150430](https://user-images.githubusercontent.com/30790184/83240063-ca27d180-a1d3-11ea-8683-54f8f92fa9df.png)

Queueing Model의 가장 핵심적인 부분은 **메시지의 소비를 하나의 컨슈머에게만 보장**한다는 점이다. 이 Queueing Model은 컨슈머의 개수대로 병렬처리를 할 수 있다는 특징이 있다.

## Pub/Sub Model

![주석 2020-05-29 150454](https://user-images.githubusercontent.com/30790184/83240068-cc8a2b80-a1d3-11ea-9a5b-f7e447420029.png)



Pub/Sub Model의 가장 핵심적인 부분은 **동일한 topic을 구독하는 Subscriber는 서로 동일한 메시지를 받는다.** 이다.



# Kafka의 Messaging Model

Kafka는 이 Queueing Model 과 Pub/Sub Model을 동시에 구현했다.

![Kafka_Architecture](https://user-images.githubusercontent.com/30790184/83240069-cd22c200-a1d3-11ea-8684-413d4192c156.png)

- Partition: Topic 하위의 개념으로 Topic 내부에 n개의 파티션 존재 가능함. Topic을 구성하는 Partition은 브로커 클러스터안에서 분산 배치되어 스케일 아웃이 가능함
- Consumer Group: Topic을 구독하는 Consumer를 Group화 시켜서 Consumer Group을 통해 여러 Partition에서 메시지를 취득할 수 있도록 함



하나의 파티션은 한개의 컨슈머에게 메시지를 전달할 수 있고 하나의 컨슈머는 1개 이상의 파티션에서 메시지를 소비할 수 있다.

![Kafka_Partition_Consumer](https://user-images.githubusercontent.com/30790184/83240072-cd22c200-a1d3-11ea-97d4-fe8b98fa667d.png)

만약 파티션이 3개가있고 컨슈머가 3개가있다면 처리량의 3배가 되는 구조가 된다.

# Kafka의 메시지 전달







포스팅은 여기까지 하겠습니다. 

퍼가실때는 출처를 반드시 남겨주세요!

<br>

**References**

-  실전 아파치 카프카 - 사사키 도루 등 6명 (한빛미디어)
-  https://dev.to/tbking/how-to-choose-between-kafka-and-rabbitmq-4jol
