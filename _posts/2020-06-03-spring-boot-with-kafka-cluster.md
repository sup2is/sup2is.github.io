---

layout: post
title: "Kafka Cluster 구성하고 Spring Boot에서 Kafka 사용하기"
tags: [Spring Boot, Apache Kafka]
date: 2020-06-02
comments: true
---



<br>

# OverView

이전시간는 Kafka Cluster를 구성하고 거기에 Java 애플리케이션으로 프로듀서, 컨슈머를 구성하는 예제를 만들어봤었다. 하지만 순수 Java로만 짜여진 애플리케이션을 현업에서 사용하기는 쉽지 않은 선택이다. 이번 시간에는 Spring Boot와 Kafka Cluster를 구성해서 간단한 푸시 알람 아키텍쳐를 구성한다. 아키텍쳐는 다음과 같다

- Kafka Cluster (Kafka server 3대)
- Prouducer (Spring boot Rest Web Application)
- Consumer (Spring Boot GCM App Push Application)

<br>

간단하게 추가 설명하면

1. 카프카는 프로듀서와 컨슈머의 브로커 역할을 한다.
2. 프로듀서는 Rest API로 Client에게서 푸시알람 이벤트를 호출하는 endpoint를 제공한다.
3. 컨슈머는 카프카에게서 받은 데이터를 푸시서버에 전송하는 역할을 한다.

위에서 짜놓은 시나리오를 기반으로 간단하게 어플리케이션을 구성해보자!

# Kafka 3대로 Cluster 구성하기

나는 도커 컨테이너 기반으로 Kafka server를 3대 구성했다. 이전 글에서 컨테이너를 올리고 확인하는 부분까지 작성했으니 클러스터 구성이 필요한 분이면 [Kafka Cluster 구성하고 Java에서 Kafka 사용하기](http://127.0.0.1:4000/2020/06/02/java-with-kafka-cluster.html)를 참고하자

카프카에 토픽을 먼저 생성해도 되지만 이번에는 Spring Kafka 프로젝트에서 제공하는 **KafkaAdmin**를 사용해서 토픽을 생성해보도록 하겠다. 

# Spring Boot기반의 Producer 만들기

먼저 pom.xml에 필요한 모듈을 추가해주자.

**pom.xml**

```xml
<?xml version="1.0" encoding="UTF-8"?>

<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.3.0.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>producer</artifactId>

    <name>producer</name>

    <properties>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.kafka</groupId>
            <artifactId>spring-kafka</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
            <exclusions>
                <exclusion>
                    <groupId>org.junit.vintage</groupId>
                    <artifactId>junit-vintage-engine</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <dependency>
            <groupId>org.springframework.kafka</groupId>
            <artifactId>spring-kafka-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>

```

**spring-kafka** 모듈을 추가해주고 이 Producer는 Web Application으로 동작하기 때문에 **spring-boot-starter-web**역시 추가해준다.

<br>

추가적으로 토픽의 이름이나 Kafka 클러스터의 주소 등등을 하드코딩 하지 않도록 application.properties 파일을 생성해서 다음과 같이 설정해주자

**application.properties**

```properties
kafka.bootstrapAddress=192.168.56.107:9092,192.168.56.107:9093,192.168.56.107:9094
kafka.my.push.topic.name=app-push-topic
```

카프카 클러스터를 총 3대로 구성했기때문에 다음과 같이 콤마로 구분해서 넣어줬다.

<br>

설정이 모두 끝났다면 KafkaAdmin을 사용해서 Topic을 생성하기 위한 KafkaTopicConfig 클래스를 생성해보자

## Spring Kafka 로 Topic 생성하기

Spring for Apache Kafka는 KafkaAdmin을 사용해서 토픽을 생성할 수 있게 해준다. 사용법은 아래와 같다

**KafkaTopicConfig.java**

```java
package me.sup2is.producer;

import org.apache.kafka.clients.admin.AdminClientConfig;
import org.apache.kafka.clients.admin.NewTopic;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.kafka.core.KafkaAdmin;

import java.util.HashMap;
import java.util.Map;

@Configuration
public class KafkaTopicConfig {

    @Value("${kafka.bootstrapAddress}")
    private String bootstrapServers;

    @Value("${kafka.my.push.topic.name}")
    private String topicName;

    @Bean
    public KafkaAdmin kafkaAdmin() {
        Map<String, Object> configs = new HashMap<>();
        configs.put(AdminClientConfig.BOOTSTRAP_SERVERS_CONFIG, bootstrapServers);
        return new KafkaAdmin(configs);
    }

    @Bean
    public NewTopic newTopic() {
        return new NewTopic(topicName, 3, (short) 3);
    }
}

```

KafkaAdmin 객체를 적절하게 초기화시켜주고 NewTopic 객체를 빈으로 등록해서 애플리케이션 로딩 시점에 카프카에 토픽을 등록할 수 있다. 만약 같은 이름의 토픽이 등록되어있다면 아무런 동작도 하지 않는다. 위에서 사용한 NewTopic 객체의 생성자는 다음과 같다

> ```java
>     public NewTopic(String name, int numPartitions, short replicationFactor) {
>         this(name, Optional.of(numPartitions), Optional.of(replicationFactor));
>     }
> ```

나같은 경우는 app-push-topic이라는 이름으로 파티션 3개와 레플리케이션을 3개 구성한 토픽을 생성한 것이다 이것은 아래의 kafka shell을 이용한 명령어와 같은 역할을 한다.

```
./bin/kafka-topics.sh --create \
    --replication-factor 3 \
    --partitions 3 \
    --topic app-push-topic \
    --zookeeper  localhost:2181
```

이어서 Producer 관련 Config 객체를 생성해보자

## KafkaProducerConfig 작성하기

```java
package me.sup2is.producer;

import org.apache.kafka.clients.producer.ProducerConfig;
import org.apache.kafka.common.serialization.StringSerializer;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.kafka.core.DefaultKafkaProducerFactory;
import org.springframework.kafka.core.KafkaTemplate;
import org.springframework.kafka.core.ProducerFactory;
import org.springframework.kafka.support.serializer.JsonSerializer;

import java.util.HashMap;
import java.util.Map;

@Configuration
public class KafkaProducerConfig {

    @Value("${kafka.bootstrapAddress}")
    private String bootstrapServers;

    @Bean
    public ProducerFactory<String, GCMPushEntity> producerFactory() {
        Map<String, Object> configProps = producerFactoryConfig();
        return new DefaultKafkaProducerFactory<>(configProps);
    }

    private Map<String, Object> producerFactoryConfig() {
        Map<String, Object> configProps = new HashMap<>();
        configProps.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, bootstrapServers);
        configProps.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
        configProps.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, JsonSerializer.class);
        return configProps;
    }

    @Bean
    public KafkaTemplate<String, GCMPushEntity> kafkaTemplate() {
        return new KafkaTemplate<>(producerFactory());
    }

}

```

우리가 실제로 사용할 **KafkaTemplate** 객체를 적절한 설정과 함께 생성해준다.

여기에서 주의 깊게 볼 점은 **KEY_SERIALIZER_CLASS_CONFIG**과 **VALUE_SERIALIZER_CLASS_CONFIG**인데 자신이 메시지를 보내는 타입을 지정해줘야한다. 나같은경우는 **GCMPushEntity** 타입의 Entity 객체를 직접 생성해서 Consumer에게 보낼 것이므로 **VALUE_SERIALIZER_CLASS_CONFIG**은 **JsonSerializer.class**로 지정해줬다 만약 단순히 String 값만 보낼것이면 StringSerializer.class를 사용하면 된다.

## Kafka로 Message 보내기

실제 카프카로 메시지를 보내는 KafkaMessageSender 클래스를 작성해보자. 

**KafkaMessageSender.java**

```java
package me.sup2is.producer;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.kafka.core.KafkaTemplate;
import org.springframework.kafka.support.KafkaHeaders;
import org.springframework.kafka.support.SendResult;
import org.springframework.messaging.Message;
import org.springframework.messaging.support.MessageBuilder;
import org.springframework.stereotype.Component;
import org.springframework.util.concurrent.ListenableFuture;
import org.springframework.util.concurrent.ListenableFutureCallback;

@Component
public class KafkaMessageSender {

    @Autowired
    private KafkaTemplate<String, GCMPushEntity> kafkaTemplate;

    @Value("${kafka.my.push.topic.name}")
    private String topicName;

    public void send(GCMPushEntity gcmPushEntity) {

        Message<GCMPushEntity> message = MessageBuilder
                .withPayload(gcmPushEntity)
                .setHeader(KafkaHeaders.TOPIC, topicName)
                .build();

        ListenableFuture<SendResult<String, GCMPushEntity>> future =
                kafkaTemplate.send(message);

        future.addCallback(new ListenableFutureCallback<SendResult<String, GCMPushEntity>>() {

            @Override
            public void onSuccess(SendResult<String, GCMPushEntity> stringObjectSendResult) {
                System.out.println("Sent message=[" + stringObjectSendResult.getProducerRecord().value().toString() +
                        "] with offset=[" + stringObjectSendResult.getRecordMetadata().offset() + "]");
            }

            @Override
            public void onFailure(Throwable ex) {
                System.out.println("Unable to send message=[] due to : " + ex.getMessage());
            }
        });
    }

}

```

KafkaMessageSender클래스는 이전에 빈으로 등록해놓은 **KafkaTemplate** 을 주입받아서 사용한다. 주목할만한 부분은 **kafkaTemplate.send(message);** 로 메시지를 보내는부분인데 리턴타입으로 ListenableFuture 클래스를 받을 수 있다. 여기에서 Producer의 메시지 송신여부, 현재 보낸 파티션의 offset값을 확인하고싶다면 ListenableFuture의 addCallback 메서드에 new ListenableFutureCallback 을 구현하면 된다.

<br>

추가적으로 필요한 클래스들 역시 구성해준다.
**GCMPushRestController.java**

```java
package me.sup2is.producer;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class GCMPushRestController {

    @Autowired
    private KafkaMessageSender kafkaMessageSender;

    @PostMapping("/push")
    public String push(@RequestBody GCMPushEntity gcmPushEntity) {
        kafkaMessageSender.send(gcmPushEntity);
        return "success";
    }

}

```

그냥 단순히 "/push" 로 요청이 넘어왔을때 카프카에 넘어온 gcmPushEntity를 보내도록 구성했다.

<br>

**GCMPushEntity**

```java
package me.sup2is.consumer;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

@Data
@NoArgsConstructor
@AllArgsConstructor
public class GCMPushEntity {

    private String gcmToken;
    private String message;

}

```



# Spring Boot기반의 Consumer 만들기

Consumer 역시 필요한 의존성 모듈과 application.properties를 추가해준다.

**pom.xml**

```xml
<?xml version="1.0" encoding="UTF-8"?>

<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.3.0.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <modelVersion>4.0.0</modelVersion>
    <artifactId>consumer</artifactId>

    <name>consumer</name>

    <properties>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-autoconfigure</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.kafka</groupId>
            <artifactId>spring-kafka</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
            <exclusions>
                <exclusion>
                    <groupId>org.junit.vintage</groupId>
                    <artifactId>junit-vintage-engine</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <dependency>
            <groupId>org.springframework.kafka</groupId>
            <artifactId>spring-kafka-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-databind</artifactId>
        </dependency>
    </dependencies>
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>

```

<br>

**application.properties**

```properties
kafka.bootstrapAddress=192.168.56.107:9092,192.168.56.107:9093,192.168.56.107:9094
kafka.my.push.topic.name=app-push-topic
kafka.my.push.topic.group.name=app-push-group
```

## KafkaConsumerConfig 작성하기

KafkaConsumer 구성에 필요한 KafkaConsumerConfig 클래스를 작성해보자.

**KafkaConsumerConfig.java** 

```java
package me.sup2is.consumer;

import org.apache.kafka.clients.consumer.ConsumerConfig;
import org.apache.kafka.common.serialization.StringDeserializer;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.kafka.annotation.EnableKafka;
import org.springframework.kafka.config.ConcurrentKafkaListenerContainerFactory;
import org.springframework.kafka.core.ConsumerFactory;
import org.springframework.kafka.core.DefaultKafkaConsumerFactory;
import org.springframework.kafka.support.serializer.JsonDeserializer;

import java.util.HashMap;
import java.util.Map;

@EnableKafka
@Configuration
public class KafkaConsumerConfig {

    @Value("${kafka.bootstrapAddress}")
    private String bootstrapServers;

    @Value("${kafka.my.push.topic.group.name}")
    private String groupId;

    @Bean
    public ConsumerFactory<String, GCMPushEntity> pushEntityConsumerFactory() {
        JsonDeserializer<GCMPushEntity> deserializer = gcmPushEntityJsonDeserializer();
        return new DefaultKafkaConsumerFactory<>(
                consumerFactoryConfig(deserializer),
                new StringDeserializer(),
                deserializer);
    }

    private Map<String, Object> consumerFactoryConfig(JsonDeserializer<GCMPushEntity> deserializer) {
        Map<String, Object> props = new HashMap<>();
        props.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, bootstrapServers);
        props.put(ConsumerConfig.GROUP_ID_CONFIG, groupId);
        props.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        props.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, deserializer);
        return props;
    }

    private JsonDeserializer<GCMPushEntity> gcmPushEntityJsonDeserializer() {
        JsonDeserializer<GCMPushEntity> deserializer = new JsonDeserializer<>(GCMPushEntity.class);
        deserializer.setRemoveTypeHeaders(false);
        deserializer.addTrustedPackages("*");
        deserializer.setUseTypeMapperForKey(true);
        return deserializer;
    }

    @Bean
    public ConcurrentKafkaListenerContainerFactory<String, GCMPushEntity>
    pushEntityKafkaListenerContainerFactory() {
        ConcurrentKafkaListenerContainerFactory<String, GCMPushEntity> factory =
                new ConcurrentKafkaListenerContainerFactory<>();
        factory.setConsumerFactory(pushEntityConsumerFactory());
        return factory;
    }

}
```

주목할만한 부분은 클래스 상단에 **@EnableKafka** 를 붙인것과 consumerFactoryConfig() 메서드에서 Deserializer들을 Producer와 일치하도록 구성한 것이다. 근데 여기에서는 JsonDeserializer 객체를 직접 생성해서 등록했는데 그 이유는 Producer의 GCMPushEntity의 패키지명과 Consumer의 GCMPushEntity 패키지명이 달랐기 때문이다. 만약 package명이 다르다면 다른 객체로 판단하기때문에 **deserializer.addTrustedPackages("*");** 메서드로 해당 패키지를 신뢰할 수 있도록 추가해줘야한다. 이것때문에 고생좀 했다.

<br>

## @KafkaListener 사용하기

@KafkaListener 를 사용해서 Consumer 를 간단하게 구현할 수가 있다.

**KafkaMessageListener.java**

```java
package me.sup2is.consumer;

import org.springframework.kafka.annotation.KafkaListener;
import org.springframework.messaging.MessageHeaders;
import org.springframework.messaging.handler.annotation.Headers;
import org.springframework.messaging.handler.annotation.Payload;
import org.springframework.stereotype.Service;

@Service
public class KafkaMessageListener {

    @KafkaListener(topics = "${kafka.my.push.topic.name}"
            , groupId = "${kafka.my.push.topic.group.name}"
            , containerFactory = "pushEntityKafkaListenerContainerFactory")
    public void listenWithHeaders(@Payload GCMPushEntity gcmPushEntity,
                                  @Headers MessageHeaders messageHeaders) {

        // GCM으로 해당 데이터를 전달하는 로직 ....

        System.out.println(
                "Received Message: " + gcmPushEntity.toString() +
                        " headers: " + messageHeaders);
    }
}


```

**@KafkaListener** 어노테이션의 topics, gorupId 등으로 원하는 토픽과 consumer group까지 쉽게 구성할 수 있다. 여기에서 **@Headers MessageHeaders messageHeaders** 로 받아오는 부분으로 해당 메시지의 offset, partion정보등을 얻어올 수 있다.

# Example

이제 실제 작성한 프로그램을 프로듀서는 한개, 컨슈머 3개를 각각 따로 구동시켜서 어떻게 동작하는지 확인해보자.

만약 카프카 클러스터에 해당 컨슈머들이 적절하게 잘 분배되었는지 또는 각각 파티션의 offset 현황 등등을 kafka shell에서 직접 확인하고싶다면 아래 명령어로 확인 가능하다.

```
./bin/kafka-consumer-groups.sh --describe --group {group-id} --bootstrap-server localhost:9092,localhost:9093,localhost:9094
```

<br>

만약 적절하게 파티션 개수대로 컨슈머들이 잘 배치된것을 확인했다면 포스트맨의 Collection Runner 기능을 통해서 "/push" 에 적절한 json 데이터를 입력하고 100번정도의 request를 날리도록하겠다. "/push"는 프로듀서에서 작성했다.

![20200603_130347](https://user-images.githubusercontent.com/30790184/83595028-c6ad9500-a59b-11ea-9468-c4deaf4bd6ea.png)

![20200603_130316](https://user-images.githubusercontent.com/30790184/83595031-c7dec200-a59b-11ea-9922-80aa4c934483.png)

왼쪽은 프로듀서 이고 오른쪽 세개는 컨슈머를 각각 다른 쉘에 세개씩 띄워 놓은 결과다.

![2020-06-03-13-07-50](https://user-images.githubusercontent.com/30790184/83595167-2a37c280-a59c-11ea-8ab5-7fcee198bf0a.gif)

화면이 조금 작아서 메시지가 잘 안보이는데 아래와 같은 형태로 출력된다.

```
Received Message: GCMPushEntity(gcmToken=test1234, message=안녕하세요) headers: {kafka_offset=159, kafka_consumer=org.apache.kafka.clients.consumer.KafkaConsumer@c94a813, kafka_timestampType=CREATE_TIME, kafka_receivedPartitionId=2, kafka_receivedTopic=app-push-topic, kafka_receivedTimestamp=1591157398566, __TypeId__=[B@32a0aa24, kafka_groupId=app-push-group}
Received Message: GCMPushEntity(gcmToken=test1234, message=안녕하세요) headers: {kafka_offset=160, kafka_consumer=org.apache.kafka.clients.consumer.KafkaConsumer@c94a813, kafka_timestampType=CREATE_TIME, kafka_receivedPartitionId=2, kafka_receivedTopic=app-push-topic, kafka_receivedTimestamp=1591157398650, __TypeId__=[B@2fbaf3c8, kafka_groupId=app-push-group}
Received Message: GCMPushEntity(gcmToken=test1234, message=안녕하세요) headers: {kafka_offset=161, kafka_consumer=org.apache.kafka.clients.consumer.KafkaConsumer@c94a813, kafka_timestampType=CREATE_TIME, kafka_receivedPartitionId=2, kafka_receivedTopic=app-push-topic, kafka_receivedTimestamp=1591157398740, __TypeId__=[B@45b1911, kafka_groupId=app-push-group}
Received Message: GCMPushEntity(gcmToken=test1234, message=안녕하세요) headers: {kafka_offset=162, kafka_consumer=org.apache.kafka.clients.consumer.KafkaConsumer@c94a813, kafka_timestampType=CREATE_TIME, kafka_receivedPartitionId=2, kafka_receivedTopic=app-push-topic, kafka_receivedTimestamp=1591157398776, __TypeId__=[B@44f37836, kafka_groupId=app-push-group}
Received Message: GCMPushEntity(gcmToken=test1234, message=안녕하세요) headers: {kafka_offset=163, kafka_consumer=org.apache.kafka.clients.consumer.KafkaConsumer@c94a813, kafka_timestampType=CREATE_TIME, kafka_receivedPartitionId=2, kafka_receivedTopic=app-push-topic, kafka_receivedTimestamp=1591157398873, __TypeId__=[B@52913421, kafka_groupId=app-push-group}
Received Message: GCMPushEntity(gcmToken=test1234, message=안녕하세요) headers: {kafka_offset=164, kafka_consumer=org.apache.kafka.clients.consumer.KafkaConsumer@c94a813, kafka_timestampType=CREATE_TIME, kafka_receivedPartitionId=2, kafka_receivedTopic=app-push-topic, kafka_receivedTimestamp=1591157398911, __TypeId__=[B@2a4da908, kafka_groupId=app-push-group}

...
```



# 마무리

이번에 작성한 프로그램도 비록 간단한 예제지만 현업에서 어떻게 Spring과 Kafka를 통합해야할지에 대한 좋은 예제가 되었으면 좋겠다.







<hr>
포스팅은 여기까지 하겠습니다. 퍼가실때는 출처를 반드시 남겨주세요!

예제 : [https://github.com/sup2is/kafka-example/tree/master/spring-with-kafka-cluster](https://github.com/sup2is/kafka-example/tree/master/spring-with-kafka-cluster)

<br>

**References**

-  [https://www.baeldung.com/spring-kafka](https://www.baeldung.com/spring-kafka)
-  [https://stackoverflow.com/questions/51688924/spring-kafka-the-class-is-not-in-the-trusted-packages](https://stackoverflow.com/questions/51688924/spring-kafka-the-class-is-not-in-the-trusted-packages)
-  [https://spring.io/projects/spring-kafka](https://spring.io/projects/spring-kafka)
