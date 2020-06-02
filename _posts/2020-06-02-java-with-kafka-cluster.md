---

layout: post
title: "Kafka Cluster 구성하고 Java에서 Kafka 사용하기"
tags: [Java, Apache Kafka]
date: 2020-06-02
comments: true
---



<br>

# OverView

이전시간에 Apache Kafka의 기본 개념에 대해서 알아봤고 이번시간에는 카프카 서버 3대를 클러스터로 구성해서 프로듀서와 컨슈머를 사용하는 예제를 해보도록 구성해보도록 하겠다. 준비물은 다음과 같다.

- Kafka server 3대로 구성한 kafka cluster
- Java Producer Application
- Java Consumer Application

먼저 카프카 클러스터 구성을 해보자.

# Kafka 3대로 Cluster 구성하기

카프카 서버를 띄우는건 입맛에 맞춰서 진행하면 된다. 나같은 경우는 CentOS7에 도커 컨테이너로 카프카3대를 준비했다. 만약 카프카가 설치되지 않았다면 [https://kafka.apache.org/downloads](https://kafka.apache.org/downloads) 에서 다운로드 할 수 있다.

이제 kafka server를 준비해보자. 현재 도커 컴포즈 사용이 어려운 환경이라 단순히 도커 커맨드로 컨테이너를 준비했고 만약 도커 컴포즈를 사용하고 싶다면 [https://github.com/simplesteph/kafka-stack-docker-compose](https://github.com/simplesteph/kafka-stack-docker-compose) 에서 compose.yml 파일을 사용하면 될 듯 하다. 

먼저 zookeeper server를 준비한다

``` 
docker run -d -p 2181:2181 -e "ZOO_PORT=2181"  --name zoo1 --hostname zoo1 zookeeper:3.4.9
```

그리고 카프카 컨테이너를 세개띄운다 이미지는 confluentinc/cp-kafka:5.5.0 를 사용했다.

```
docker run -d \
-e "KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://192.168.56.107:9092" \
-e "KAFKA_LISTENERS=PLAINTEXT://0.0.0.0:9092" \
-e "KAFKA_ZOOKEEPER_CONNECT=zoo1:2181" \
-e "KAFKA_BROKER_ID=1" \
-e "KAFKA_LOG4J_LOGGERS=kafka.controller=INFO,kafka.producer.async.DefaultEventHandler=INFO,state.change.logger=INFO"  \
-p 9092:9092 --link zoo1:zoo1 --name kafka1 --hostname kafka1 confluentinc/cp-kafka:5.5.0

docker run -d \
-e "KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://192.168.56.107:9093" \
-e "KAFKA_LISTENERS=PLAINTEXT://0.0.0.0:9093" \
-e "KAFKA_ZOOKEEPER_CONNECT=zoo1:2181" \
-e "KAFKA_BROKER_ID=2" \
-e "KAFKA_LOG4J_LOGGERS=kafka.controller=INFO,kafka.producer.async.DefaultEventHandler=INFO,state.change.logger=INFO" -p 9093:9093 --link zoo1:zoo1 --link kafka1:kafka1 --name kafka2 --hostname kafka2 confluentinc/cp-kafka:5.5.0


docker run -d \
-e "KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://192.168.56.107:9094" \
-e "KAFKA_LISTENERS=PLAINTEXT://0.0.0.0:9094" \
-e "KAFKA_ZOOKEEPER_CONNECT=zoo1:2181" \
-e "KAFKA_BROKER_ID=3" \
-e "KAFKA_LOG4J_LOGGERS=kafka.controller=INFO,kafka.producer.async.DefaultEventHandler=INFO,state.change.logger=INFO"  -p 9094:9094 --link zoo1:zoo1 --link kafka2:kafka2 --link kafka1:kafka1 --name kafka3 --hostname kafka3 confluentinc/cp-kafka:5.5.0


```

차례대로 kafka1, kafka2, kafka3 서버를 띄웠고 환경설정을 간단하게 설명하면

- **KAFKA_ADVERTISED_LISTENERS :** 외부에서 Kafka 서버로 들어오는 IP를 지정할 수 있다. 이 값을 통해 Consumer와 Producer가 Kafka와 연결된다.
- **KAFKA_LISTENERS :** 내부에서 매핑될 IP를 지정할 수 있다.

KAFKA_ADVERTISED_LISTENERS 와 KAFKA_LISTENERS 에 대한 설명은 아래 링크에서 조금 더 자세한 설명을 확인할 수 있다. 

[https://medium.com/@iamsuraj/what-is-advertised-listeners-in-kafka-72e6fae7d68e](https://medium.com/@iamsuraj/what-is-advertised-listeners-in-kafka-72e6fae7d68e)

<br>

문제없이 컨테이너가 전부 띄워졌다면 docker ps -a을 했을때 아래와 같은 모습이면 된다. IP와 포트 같은 경우는 각자 컴퓨터에 맞는 환경으로 셋팅해야 한다.



![주석 2020-06-02 083309](https://user-images.githubusercontent.com/30790184/83471490-be802780-a4bf-11ea-90f1-fc373bfcaeb1.png)



카프카 클러스터가 적절하게 구성되었는지 확인해보려면 아래 명령어를 통해서 확인할 수 있다. 모든 명령어는 kafka home dir에서 실행한다. (.sh 이나 .bat이 있는 위치)

```
./bin/zookeeper-shell.sh localhost:2181 ls /brokers/ids
```

![주석 2020-06-02 085747](https://user-images.githubusercontent.com/30790184/83471491-bf18be00-a4bf-11ea-8f95-3e4ac714c3e6.png)

도커 컨테이너를 생성할때 KAFKA_BROKER_ID로 각각 1, 2, 3 으로 값을 줬기 때문에 다음과 같이 id가 출력된다.

이제 카프카 토픽을 생성해보자.

```
./kafka-topics.sh --create \
    --replication-factor 3 \
    --partitions 3 \
    --topic my-test-topic \
    --zookeeper  localhost:2181
```

- **replication-factor:** 레플리카를 구성하는 개수를 지정한다. 클러스터 개수보다 같거나 작아야한다.
- **partitions:** 해당 토픽의 병렬처리를 위해 파티션을 구성한다.

<br>

토픽이 생성되었다면 아래 명령어로 토픽에 대한 상세 정보를 확인할 수 있다.

```
./kafka-topics.sh --zookeeper localhost:2181 --describe --topic my-test-topic
```



![주석 2020-06-02 090457](https://user-images.githubusercontent.com/30790184/83471481-bc1dcd80-a4bf-11ea-923b-428026063b77.png)



레플리카가 잘 구성되어 있는 모습을 확인할 수 있다. 지금 생성한 토픽 이외에 다른 토픽의 목록을 확인하고 싶다면 아래 명령어로 확인할 수 있다.

```
./bin/zookeeper-shell.sh localhost:2181 ls /brokers/topics
```

<br>

카프카는 kafka-console-producer.sh 과 kafka-console-consumer.sh 을 통해 간단한 입출력을 테스트할 수 있는 도구를 제공한다.

```
./bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic my-test-topic --from-beginning --group my-test-topic

--------------------

./bin/kafka-console-producer.sh --broker-list localhost:9092 --topic my-test-topic

```

쉘을 두개 띄워서 프로듀서쪽에 문자를 입력하면 컨슈머쪽으로 데이터가 전달되는 것을 확인할 수 있다.





![2020-06-02-09-43-10](https://user-images.githubusercontent.com/30790184/83471486-bde79100-a4bf-11ea-8d58-7e829d765a0b.gif)



이렇게 카프카 서버 3대로 클러스터를 구성까지 완료했다면 Java로 프로듀서/컨슈머를 구성해보자!

# Java로 Producer 만들기 

의존성 도구로 maven을 사용했다.

**pom.xml**

```xml
...
        <dependency>
            <groupId>org.apache.kafka</groupId>
            <artifactId>kafka-clients</artifactId>
            <version>2.3.0</version>
        </dependency>
...


    <!--jar 파일로 추출시 dependencies 포함하도록 설정-->
    <build>
        <plugins>
            <plugin>
                <artifactId>maven-assembly-plugin</artifactId>
                <version>2.6</version>
                <configuration>
                    <descriptorRefs>
                        <descriptorRef>jar-with-dependencies</descriptorRef>
                    </descriptorRefs>
                </configuration>
                <executions>
                    <execution>
                        <id>make-assembly</id>
                        <phase>package</phase>
                        <goals>
                            <goal>single</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>


...
```

Kafka 관련 Class를 사용하기 위해 kafka-clients 의존성을 프로젝트에 적용시킨다.

<br>

**MyProducer.java**

```java
package me.sup2is.producer;

import org.apache.kafka.clients.producer.KafkaProducer;
import org.apache.kafka.clients.producer.Producer;
import org.apache.kafka.clients.producer.ProducerConfig;
import org.apache.kafka.clients.producer.ProducerRecord;
import org.apache.kafka.common.serialization.StringSerializer;

import java.util.Properties;
import java.util.Scanner;

public class MyProducer {

    public static void main(String[] args) {
        Properties properties = new Properties();
        properties.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG
                , "192.168.56.107:9092,192.168.56.107:9093,192.168.56.107:9094");
        properties.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
        properties.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());

        Producer<String, String> producer = new KafkaProducer<>(properties);

        while(true) {
            Scanner sc = new Scanner(System.in);
            System.out.print("Input > ");
            String message = sc.nextLine();

            ProducerRecord<String, String> record = new ProducerRecord<>("my-test-topic", message);
            try {
                producer.send(record, (metadata, exception) -> {
                    if (exception != null) {
                        // some exception
                        exception.printStackTrace();
                    }
                });

            } catch (Exception e) {
                // exception
            } finally {
                producer.flush();
            }

            if(message.equals("quit")) {
                producer.close();
                break;
            }
        }
    }
}

```

Properties 클래스를 사용해서 Kafka Producer 인스턴스를 초기화시킬 수 있다. 관련된 설정 key값은 ProducerConfig클래스에서 제공하니 필요한 값이 있다면 확인해서 사용하면 된다. 

- **ProducerConfig.BOOTSTRAP_SERVERS_CONFIG:** 이전에 설정했던 kafka 클러스터 ip들을 입력해준다.
- **ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG**: 카프카의 모든 메시지는 직렬화된 상태로 전송되는데 이때 사용할 시리얼라이저를 지정해준다. JSON, Apache Abro 등 적절한 상황에 맞춰서 사용하면 된다. 두 시리얼라이저가 같지않으면 메시지 직렬화 또는 역직렬화가 불가능하니 같은 시리얼라이저를 사용해야한다.

이 프로그램은 간단하게 "quit" 이라는 문자열이 들어오기 전까지 메시지를 계속 카프카 토픽에 넣는 프로그램이다. 매우 간단한 프로그램이니 자세한 설명은 생략한다.





# Java로 Consumer 만들기

**pom.xml**

```xml
...
        <dependency>
            <groupId>org.apache.kafka</groupId>
            <artifactId>kafka-clients</artifactId>
            <version>2.3.0</version>
        </dependency>
...

    <!--jar 파일로 추출시 dependencies 포함하도록 설정-->
    <build>
        <plugins>
            <plugin>
                <artifactId>maven-assembly-plugin</artifactId>
                <version>2.6</version>
                <configuration>
                    <descriptorRefs>
                        <descriptorRef>jar-with-dependencies</descriptorRef>
                    </descriptorRefs>
                </configuration>
                <executions>
                    <execution>
                        <id>make-assembly</id>
                        <phase>package</phase>
                        <goals>
                            <goal>single</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>

...
```

마찬가지로 Consumer 역시 Kafka 관련 Class를 사용하기 위해 kafka-clients 의존성을 프로젝트에 적용시킨다.

<br>

```java
package me.sup2is.consumer;

import org.apache.kafka.clients.consumer.*;
import org.apache.kafka.common.serialization.StringDeserializer;

import java.time.Duration;
import java.util.Collections;
import java.util.Properties;

public class MyConsumer {
    public static void main( String[] args ) {
        Properties properties = new Properties();
        properties.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG,
                "192.168.56.107:9092,192.168.56.107:9093,192.168.56.107:9094");
        properties.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
        properties.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
        properties.put(ConsumerConfig.GROUP_ID_CONFIG, "my-test-topic");

        Consumer<String, String> consumer = new KafkaConsumer<>(properties);
        consumer.subscribe(Collections.singletonList("my-test-topic"));

        String message = null;
        try {
            do {
                ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(100000));

                for (ConsumerRecord<String, String> record : records) {
                    message = record.value();
                    System.out.println(message);
                }
            } while (!message.equals("quit"));
        } catch(Exception e) {
            e.printStackTrace();
            // exception
        } finally {
            consumer.close();
        }

    }

}

```

이것 역시 크게 대단한점은 없고 Producer 애플리케이션과 매우매우 비슷한 느낌으로 구성했다. 만약 ConsumerRecords나 자세한 정보를 확인하고싶다면 api를 확인해보자.

- **ConsumerConfig.GROUP_ID_CONFIG :** 이 값을 통해 컨슈머 그룹을 지정해줬다. 카프카는 같은 토픽을 구독하는 컨슈머그룹 내의 컨슈머는 병렬 처리를 할 수 있도록 메시지를 공유하지 않는다.

# Example

지금까지 구성한 Kafka Cluster와 Java 애플리케이션을 한번 실행시켜보자

각각 프로듀서, 컨슈머의 jar파일을 추출하자

```
# 해당 프로젝트 pom.xml이 있는 dir
mvn package
```

프로듀서는 한개만 필요하므로 한개만 다음 명령어로 구동시킨다 패키지명과 클래스명은 각자 상황에 맞도록 설정하면 된다.

```
java -cp ./producer-1.0-SNAPSHOT-jar-with-dependencies.jar me.sup2is.producer.MyProducer
```

컨슈머는 총 세개를 준비할 것이다. 쉘을 3개 더 열어서 컨슈머를 아래 명령어로 구동시키자

```
java -cp ./consumer-1.0-SNAPSHOT-jar-with-dependencies.jar me.sup2is.consumer.MyConsumer
```





이렇게하면 아래와 같은 화면 구성이 될 것이다

왼쪽은 프로듀서를 구동시켰고 오른쪽 세개는 컨슈머를 구동시켰다



![20200602_104242](https://user-images.githubusercontent.com/30790184/83471488-be802780-a4bf-11ea-8a72-8f5b8ed9dd74.png)



한번 실행해보자



![2020-06-02-10-40-58](https://user-images.githubusercontent.com/30790184/83471484-bd4efa80-a4bf-11ea-972a-9b649f17c310.gif)

예상한대로 문자열을 입력했을때 메시지를 모두 공유하지 않고 컨슈머 그룹에 의해 분산된 처리를 하는 모습을 확인할 수 있다.



# 마무리

이런 메시징모델의 가장 큰 장점은 프로듀서와 컨슈머 사이에 의존성을 메시지큐가 담당하는 것이다. kafka는 아주 성능이 좋은 메시지 브로커이다. 만약 더 많은 컨슈머나 프로듀서가 필요하다면 단순히 프로듀서와 컨슈머의 개수만 늘려주면 된다. 다음시간에는 Spring에서 Kafka를 어떻게 사용하는지에 대해서 알아보도록 하겠다!



<hr>
포스팅은 여기까지 하겠습니다. 퍼가실때는 출처를 반드시 남겨주세요!

예제 : [https://github.com/sup2is/kafka-example/tree/master/kafka-with-java](https://github.com/sup2is/kafka-example/tree/master/kafka-with-java)

<br>

**References**

-  실전 아파치 카프카 - 사사키 도루 등 6명 (한빛미디어)
-  https://github.com/simplesteph/kafka-stack-docker-compose/blob/master/zk-single-kafka-multiple.yml
-  https://madplay.github.io/post/java-kafka-example
