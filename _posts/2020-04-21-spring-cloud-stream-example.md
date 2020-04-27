---

layout: post
title: "Spring Cloud Stream을 사용해서 Event-Driven Architecture 연습해보기 feat.Kafka"
tags: [Spring Boot, Event-Driven Architecture, Spring Cloud Stream, Kafka, Message Queue]
date: 2020-04-21
comments: true
grand_parent: Spring
parent: Cloud
---



<br>

# OverView

이번시간에는 Spring Cloud Stream을 사용해서 각 서비스들간에 동기식 요청 방식을 MessageQueue 로 변경하 여 Event-Driven Architecture를 구성하는 연습을 해보도록 하겠다. EDA 에 대한 자세한 내용은 [여기](https://medium.com/dtevangelist/event-driven-microservice-%EB%9E%80-54b4eaf7cc4a)에 잘 정리되어 있다. 





# 시작전에







![주석 2020-04-22 215352](https://user-images.githubusercontent.com/30790184/79984204-cd001a00-84e3-11ea-9660-afe927f4a4f2.png)

이 예제는 다음과 같은 구조로 진행된다. OrderService는 주문관련 처리를 하고 MemberService는 회원정보에 관련된 처리를 한다.

<br>



![주석 2020-04-22 215053](https://user-images.githubusercontent.com/30790184/79984193-cb365680-84e3-11ea-9f29-da394c0de8b7.png)



우리는 실제 Redis를 이번 예제에서 구축하지 않지만 Redis에는 회원들의 캐시된 데이터가 있다고 가정한다.

클라이언트는 자신의 id값을 포함해서 OrderService에 주문을 요청한다. OrderService는 먼저 Redis에서 해당 회원의 정보를 얻어낸다. 만약 데이터가 존재하지 않을경우 실제 MemberService와 통신해서 회원정보를 얻어내 주문을 완성한다.

하지만 MemberService로 회원정보가 변경된다면 어떻게 해야할까?



# 동기식 요청 방식의 문제점

![주석 2020-04-22 215105](https://user-images.githubusercontent.com/30790184/79984200-cc678380-84e3-11ea-9c3d-aedbc6e75b4c.png)



MemberService에서 회원 정보가 변경된다면 MemberService는 OrderSerivce와 연결된 Redis에게 캐싱을 무효화처리하도록 요청해야한다. OrderService가 잘못된 회원 정보를 갖지 못하도록 말이다. 이런식의 동기식 요청 방식은 두 서비스간의 결합도를 높이게 되는 문제점이 있다. 예를들어 OrderService의 캐싱 무효화 엔드포인트가 변경된다면 MemberService역시 변경되어야 한다.



# Message 기반의 아키텍쳐

위와 같은 문제점을 해결하기 위해 Message 기반의 아키텍쳐를 다음과 같이 구상할 수 있다.

![주석 2020-04-22 215128](https://user-images.githubusercontent.com/30790184/79984203-cc678380-84e3-11ea-8a23-11bc976437d0.png)

MemberService에서 회원 정보가 변경된다면 MemberService는 다음과 같이 단순히 MessageQueue에 메시지를 발행하여 해당 회원정보가 변경되었음을 알린다. 해당 채널을 구독하고 있던 OrderService는 MessageQueue에 메시지를 받아서 해당 회원의 캐싱을 무효화시킨다.



이런식으로 MessageQueue를 도입하면 다음과 같은 이점 네가지가 제공된다.

1. **느슨한 결합**: 마이크로서비스 어플리케이션들은 수많은 작은 서비스로 분산 되어 상호작용하기때문에  서비스사이에 강한 의존성을 만든다. 이 의존성을 완전히 제거할 수는 없지만 서비스가 소유한 데이터를 직접 관리하는 엔드포인트만 노출함으로 의존성을 최소화 할 수 있다. 메시징을 도입하면 두 서비스가 서로 알지 못하므로 결합되지 않는다. MemberService는 단순히 발행만하고 OrderService는 단순히 수신만하기 때문이다.
2. **내구성**: 큐가 존재하기 때문에 서비스 소비자가 다운되어도 메시지 전달을 보장할 수 있으며 서비스간의 직접적인 통신이 없기 때문에 구독자가 가동중이 아니더라도 메시지를 계속 발행할 수 있다.
3. **확장성**: 메시지가 큐에 저장되므로 발신자는 메시지의 소비를 기다릴 필요가 없고 소비자 역시 많은 발행이 있을 경우 수평적으로 확장하여 성능을 향상시킬 수 있다. 
4. **유연성**: 발신자는 누가 메시지를 소비하는지 알 수 없다. 즉 원래 발신 서비스에 영향을 주지 않고 새로운 메시지 소비자를 쉽게 추가할 수 있다. 이것은 기존 서비스를 건드리지 않고 새로운 기능을 어플리케이션에 추가할 수 있는 매우 강력한 개념이다. 



# Spring Cloud Stream

스프링 클라우드를 사용하면 스프링 기반 마이크로서비스에 메시징을 쉽게 통합할 수 있다. 스프링 클라우드 스트림 프로젝트는 어플리케이션에 메시지 발행자와 소비자를 쉽게 구축할 수 있는 어노테이션 기반의 프레임워크이다. 스프링 클라우드 스트림은 메시징 플랫폼의 구현 세부 사항을 추상화해서 여러 메시지 플랫폼이 스프링 클라우드 스트림과 통합 될 수 있다. 



![spring cloud steam architecture](https://i0.wp.com/techrocking.com/wp-content/uploads/2019/12/spring-stream-architecture.png?resize=750%2C410&ssl=1)

출처: https://techrocking.com/event-driven-architecture-with-spring-stream-and-kafka/



스프링 클라우드 스트림의 아키텍쳐는 다음과 같다.

1. **소스**: 서비스가 메시지를 발행할 준비가 되면 소스를 사용해 메시지를 발행한다. 소스는 발행될 메시지를 표현하는 POJO를 전달받는 스프링의 어노테이션 인터페이스이다. 소스는 메시지를 받아 직렬화하고 메시지를 채널로 발행한다. 
2. **채널**: 채널은 메시지 생산자와 소비자가 메시지를 발행하거나 소비한 후 메시지를 보관할 큐를 추상화한 것이다. 채널 이름은 항상 대상 큐의 이름과 간련이 있지만 코드에서는 큐 이름을 직접 사용하지 않고 채널 이름을 사용한다. 따라서 채널이 읽거나 쓰는 큐를 전환하려면 어플리케이션 코드가 아닌 구성 정보를 변경한다.
3. **바인더**: 바인더는 스프링 클라우드 스트림 프레임워크의 일부인 스프링 코드로 특정 메시지 플랫폼과 통신한다. 스프링 클라우드 스트림의 바인더를 사용하면 메시지를 발행하고 소비하기 위해 플랫폼 마다 별도의 라이브러리와 api를 제공하지 않고도 메시징을 사용할 수 있다.
4. **싱크**: 스프링 클라우드 스트림에서 서비스는 싱크를 사용해 큐에서 메시지를 받는다. 싱크는 들어오는 메시지를 위해 채널을 수신 대기하고, 메시지를 다시 POJO로 직렬화한다. 이 과정에서 스프링 서비스의 비지니스 로직이 메시지를 처리할 수 있다.







# MemberService에 Producer 구현하기

이제 실제 MemberService에 publish 기능을 구현하도록 하겠다. MQ는 kafka를 사용해서 진행한다. 나도 이 글을 쓰는 시점에는 kafka에 대한 이해가 많이 부족하다. 잘 정리된 블로그가 있어서 필요하다면 [이곳](https://medium.com/@umanking/%EC%B9%B4%ED%94%84%EC%B9%B4%EC%97%90-%EB%8C%80%ED%95%B4%EC%84%9C-%EC%9D%B4%EC%95%BC%EA%B8%B0-%ED%95%98%EA%B8%B0%EC%A0%84%EC%97%90-%EB%A8%BC%EC%A0%80-data%EC%97%90-%EB%8C%80%ED%95%B4%EC%84%9C-%EC%9D%B4%EC%95%BC%EA%B8%B0%ED%95%B4%EB%B3%B4%EC%9E%90-d2e3ca2f3c2)을 참고해도 좋을듯 하다.



**pom.xml**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.0.3.RELEASE</version>
    </parent>

    <groupId>me.sup2is</groupId>
    <version>1.0-SNAPSHOT</version>
    <artifactId>order-service</artifactId>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-stream</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-stream-kafka</artifactId>
        </dependency>
        <dependency>
            <groupId>com.sun.xml.bind</groupId>
            <artifactId>jaxb-core</artifactId>
            <version>2.3.0.1</version>
        </dependency>
        <dependency>
            <groupId>javax.xml.bind</groupId>
            <artifactId>jaxb-api</artifactId>
            <version>2.3.1</version>
        </dependency>
        <dependency>
            <groupId>com.sun.xml.bind</groupId>
            <artifactId>jaxb-impl</artifactId>
            <version>2.3.1</version>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <scope>provided</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
        </dependency>
    </dependencies>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>Finchley.RELEASE</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

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

MemberService는 Eureka 서버에 등록되어야하기때문에 관련 모듈인 **spring-cloud-starter-netflix-eureka-client**을 추가하고 디스커버리 구현을 위해 **spring-cloud-starter-openfeign**을 추가해줬다 만약 Eureka 서버와 서비스 디스커버리 구현에 대해서 궁금하다면 [Spring Cloud에서 Eureka Server를 구성하고 Netflix Feign Client로 서비스 디스커버리 구현하기](https://sup2is.github.io/docs/spring/cloud/spring-cloud-eureka-with-netfix-feign-client-example.html)을 참고해도 좋을듯 하다.

실제 publish 기능 관련 모듈은 **spring-cloud-stream**과 **spring-cloud-starter-stream-kafka**이므로 참고 바란다.

<br>

**MemberServiceApplication.java**

```java
package me.sup2is.memberservice;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.circuitbreaker.EnableCircuitBreaker;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;
import org.springframework.cloud.stream.annotation.EnableBinding;
import org.springframework.cloud.stream.messaging.Source;

@SpringBootApplication
@EnableEurekaClient
@EnableBinding(Source.class)
public class MemberServiceApplication {

    public static void main(String[] args) {
        SpringApplication.run(MemberServiceApplication.class, args);
    }
}
```

이후에 부트스트래핑 클래스인 MemberServiceApplication 클래스에 **@EnableBinding(Source.class)**를 적용시켜준다. @EnableBinding(Source.class)은 스프링 클라우드 스트림에게 해당 서비스를 메시지 브로커에 바인딩하도록 설정하는 것이다. @EnableBinding(Source.class)를 사용하면 해당 서비스가 Source 클래스에 정의된 채널들을 이용해 메시지 브로커와 통신하게 된다. 스프링 클라우드 스트림은 메시지 브로커와 통신할 수 있는 기본 채널이 있다.

<br>

이 Source 인터페이스는 아래와 같이 output()이라는 메서드를 가진 인터페이스이다.

> ```java
> package org.springframework.cloud.stream.messaging;
> 
> import org.springframework.cloud.stream.annotation.Output;
> import org.springframework.messaging.MessageChannel;
> 
> /**
>  * Bindable interface with one output channel.
>  *
>  * @see org.springframework.cloud.stream.annotation.EnableBinding
>  * @author Dave Syer
>  * @author Marius Bogoevici
>  */
> public interface Source {
> 
>    String OUTPUT = "output";
> 
>    @Output(Source.OUTPUT)
>    MessageChannel output();
> 
> }
> ```

output() 메서드는 MessageChannel 클래스 타입을 반환하는데 이 클래스는 메시지 브로커에게 메시지를 보내는 방법을 정의한다. 사용방법은 아래와 같다. 기본적으로 채널이름은 `output` 이라는 채널을 사용한다.

<br>
**PublishMemberChange.java**

```java
package me.sup2is.memberservice;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.cloud.stream.messaging.Source;
import org.springframework.messaging.support.MessageBuilder;
import org.springframework.stereotype.Component;

@Component
public class PublishMemberChange {

    @Autowired
    private Source source;

    public void publishMemberChange(Long memberId) {
        MemberChangeModel model = new MemberChangeModel(memberId, "회원의 상태가 변경되었습니다");
        source.output().send(MessageBuilder.withPayload(model).build());
    }

}

```

PublishMemberChange 클래스에서 Source라는 인터페이스를 Spring에게 주입받고 실제 사용은 publishMemberChange라는 메서드에서 이루어진다. MessageBuilder.withPayload() 메서드를 사용해서 보내고싶은 객체를 보내면 직렬화된 메시지를 미리 설정해놓은 kafka 큐에 전달한다.

<br>

kafka, spring cloud stream과 관련된 실제 설정은 application.yml에 존재한다.

**pom.xml**

```properties
...

spring:
  application:
    name: memberservice
  cloud:
    stream:
      bindings:
        output:
          destination:  memberChangeTopic
          content-type: application/json
      kafka:
        binder:
          zkNodes: localhost
          brokers: localhost


...
```

- **spring.cloud.stream.bindings.{channel-name}.destination**: 생산할 kafka topic을 설정한다.

- **spring.cloud.stream.bindings.{channel-name}.content-type**: 발행하는 메시지의 content-type을 설정한다. json 이외에도 아래의 타입을 지원한다.

  > - **JSON** to/from **POJO**
  > - **JSON** to/from [org.springframework.tuple.Tuple](https://github.com/spring-projects/spring-tuple/blob/master/spring-tuple/src/main/java/org/springframework/tuple/Tuple.java)
  > - **Object** to/from **byte[]** : Either the raw bytes serialized for remote transport, bytes emitted by an application, or converted to bytes using Java serialization(requires the object to be Serializable)
  > - **String** to/from **byte[]**
  > - **Object** to **plain text** (invokes the object’s *toString()* method)

* **spring.cloud.stream.kafka.binder.zkNodes**: zookeeper 관련 설정을 한다 위와 같이 매핑해도 자동적으로 2181 포트와 매핑된다.
* **spring.cloud.stream.kafka.binder.brokers**: kafka 관련 설정을 한다 위와 같이 매핑해도 자동적으로 9092 포트와 매핑된다.



<br>

필요한 service 로직에 적절하게 사용한다 나같은경우는 회원정보가 변경될 때 메시지를 발행하도록 설정했다.

**MemberService.java**

```java
package me.sup2is.memberservice;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

@Service
public class MemberService {

    @Autowired
    private PublishMemberChange publishMemberChange;

    public Member getMember(long memberId) {
        return new Member(memberId, "sup2is", "qwer!23");
    }

    public void updateMember(long memberId) {
        // member update 로직 ...
        System.out.println("### MemberService에서 발행");
        System.out.println("MemberService에서 받은 memberId : " + memberId);
        publishMemberChange.publishMemberChange(memberId);
    }


}

```



위와 같이 updateMember() 메서드가 동작한다면 kafka queue에 다음과 같은 POJO 객체가 전달될 것이다.

```json
{"memberId":"1L" , "message":"회원의 상태가 변경되었습니다"}
```





# OrderService에 Consumer 구현하기

MemberService에서 kafka queue로 메시지를 발행하는 부분을 구현했으니 OrderService에서는 동일한 토픽을 구독해서 메시지를 소비하도록 구현해보자.

**pom.xml**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.0.3.RELEASE</version>
    </parent>

    <groupId>me.sup2is</groupId>
    <version>1.0-SNAPSHOT</version>
    <artifactId>order-service</artifactId>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-stream</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-stream-kafka</artifactId>
        </dependency>
        <dependency>
            <groupId>com.sun.xml.bind</groupId>
            <artifactId>jaxb-core</artifactId>
            <version>2.3.0.1</version>
        </dependency>
        <dependency>
            <groupId>javax.xml.bind</groupId>
            <artifactId>jaxb-api</artifactId>
            <version>2.3.1</version>
        </dependency>
        <dependency>
            <groupId>com.sun.xml.bind</groupId>
            <artifactId>jaxb-impl</artifactId>
            <version>2.3.1</version>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <scope>provided</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
        </dependency>
    </dependencies>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>Finchley.RELEASE</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

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

MemberService와 별다른 특이점 없이 spring-cloud-stream 모듈과 spring-cloud-starter-stream-kafka 모듈을 추가해 준다.

<br>

**OrderServiceApplication.java**

```java
package me.sup2is.memberservice;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.loadbalancer.LoadBalanced;
import org.springframework.cloud.openfeign.EnableFeignClients;
import org.springframework.cloud.stream.annotation.EnableBinding;
import org.springframework.cloud.stream.annotation.StreamListener;
import org.springframework.cloud.stream.messaging.Sink;
import org.springframework.context.annotation.Bean;
import org.springframework.web.client.RestTemplate;

@SpringBootApplication
@EnableFeignClients
@EnableBinding(Sink.class)
public class OrderServiceApplication {

    public static void main(String[] args) {
        SpringApplication.run(OrderServiceApplication.class, args);
    }

    @StreamListener(Sink.INPUT)
    public void memberChanged(MemberChangeModel model) {
        System.out.println("### OrderService에서 소비");
        System.out.println(model.getMessage() + "memberId : " + model.getId());
        System.out.println("memberId : " + model.getId() + "의 캐시정보를 삭제");
    }
}

```

OrderServiceApplication 클래스에는 **@EnableBinding(Sink.class)**를 사용해서 들어오는 메시지를 수신할 수 있게 Sink 인터페이스에 정의된 채널을 사용하도록 스프링에게 알려준다. Sink인터페이스는 input()이라는 단일메서드를 가진 인터페이스이다.

<br>

> ```java
> 
> package org.springframework.cloud.stream.messaging;
> 
> import org.springframework.cloud.stream.annotation.Input;
> import org.springframework.messaging.SubscribableChannel;
> 
> /**
>  * Bindable interface with one input channel.
>  *
>  * @see org.springframework.cloud.stream.annotation.EnableBinding
>  * @author Dave Syer
>  * @author Marius Bogoevici
>  */
> public interface Sink {
> 
> 	String INPUT = "input";
> 
> 	@Input(Sink.INPUT)
> 	SubscribableChannel input();
> 
> }
> 
> ```

기본적으로 채널이름은 `input` 이라는 채널을 사용한다.

또 OrderServiceApplication 클래스 하단부에 **@StreamListener(Sink.INPUT)**을 사용해서 kafka 메시지가 수신될 경우 memberChanged()를 사용해서 간단하게 로그를 찍는 부분도 있다.

마지막으로 application.yml에서 kafka와 spring cloud stream에 대한 설정을 한다.

<br>

**application.yml**

```properties
spring:
  application:
    name: orderservice
  cloud:
    stream:
      bindings:
        input:
          destination: memberChangeTopic
          content-type: application/json
          group: orderGroup
      kafka:
        binder:
          zkNodes: localhost
          brokers: localhost
```

- **spring.cloud.stream.bindings.{channel-name}.destination**: 생산할 kafka topic을 설정한다.
- **spring.cloud.stream.bindings.{channel-name}.content-type**: 발행하는 메시지의 content-type을 설정한다. json 이외에도 아래의 타입을 지원한다.

* **spring.cloud.stream.kafka.binder.zkNodes**: zookeeper 관련 설정을 한다 위와 같이 매핑해도 자동적으로 2181 포트와 매핑된다.
* **spring.cloud.stream.kafka.binder.brokers**: kafka 관련 설정을 한다 위와 같이 매핑해도 자동적으로 9092 포트와 매핑된다.



사실 MemberService 설정과 별다를게 없다. 추가된 부분은 **spring.cloud.stream.bindings.{channel-name}.group** 프로퍼티인데 마이크로서비스에서 가장 큰 장점은 수평적 확장이다. OrderService의 인스턴수 개수가 증가하여도 같은 group 값을 갖고 있다면 스프링 클라우드 스트림과 메시지 브로커는 해당 그룹에 속한 인스턴스에 메시지를 하나만 소비할 것을 보장해준다.





# 간단하게 테스트해보기

kafka 가 설치되지 않았다면 https://kafka.apache.org/downloads 에서 설치후 진행하면된다. 나같은경우는 윈도우이기 때문에 윈도우 기반으로 실행했다.

zookeeper server와 kafka server를 올려준다. 

```
kafka-topics.bat --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic memberChangeTopic
```

위 명령어를 통해 memberChangeTopic라는 토픽을 생성 후 예제를 진행했다.



이후외 MemberService와 OrderService를 콘솔로 띄우고 Postman을 통해서 회원정보 변경 endpoint인

PUT: http://127.0.0.1/member/{memberId} 로 요청해보았다.



![녹화_2020_04_22_21_49_45_377](https://user-images.githubusercontent.com/30790184/79985025-eeadd100-84e4-11ea-98fc-13e645d00e0f.gif)



왼쪽이 MemberService이고 오른쪽이 OrderService이다. 화면에서 보는것처럼 MemberService가 메시지를 발행하면 OrderService가 해당 메시지를 소비한다. 여기서 가장 주목할점은 MemberService는 OrderService의 존재를 모르고 OrderService 역시 마찬가지로 MemberService의 존재를 모른다.

이런식으로 EDA를 마이크로서비스에 적용한다면 서비스의 확장에 더욱 편리하고 결함에 더 잘 견디게 만들 수 있다.







<br>

포스팅은 여기까지 하겠습니다.  모든예제는 제 github에서 확인하실 수 있습니다.

예제 : https://github.com/sup2is/spring-example/tree/master/spring-cloud-stream-example



<br>

**References**

-  Spring 마이크로서비스 코딩 공작소 -존 카넬 (길벗출판사)
-  https://docs.spring.io/spring-cloud-stream/docs/Brooklyn.RELEASE/reference/html/contenttypemanagement.html
