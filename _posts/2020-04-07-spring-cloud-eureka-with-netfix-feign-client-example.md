---

layout: post
title: "Spring Cloud에서 Eureka Server를 구성하고 Netflix Feign Client로 서비스 디스커버리 구현하기"
tags: [Spring Boot, Spring Cloud, Netflix Feign Client]
date: 2020-04-07
comments: true
grand_parent: Spring
parent: Cloud
---



<br>

# OverView

Spring Cloud에서는 여러개의 서비스 인스턴스의 물리적인 위치를 투명하게 구성하고 서비스 디스커버리를 사용해 실행하는 서비스 인스턴스 개수를 신속하게 수평 확장하거나 축소할 수 있다. 실제 서비스 인스턴스의 물리적 위치를 모르기 때문에 서비스 풀에서 새로운 서비스 인스턴스의 추가나 삭제가 자유롭다. 이런 서비스 디스커버리는 어플리케이션 회복성을 향상시키는데에도 도움이 된다. 마이크로 서비스 인스턴스가 비정상적이거나 가용중이 아니라면 서비스 디스커버리 엔진이 사용할 수 없는 서비스 인스턴스를 피해 라우팅하기때문에 다운된 서비스에 대한 피해를 최소화 시킬 수 있다.

이런 서비스 디스커버리를 사용하지 않는 곳이라면 DNS와 네트워크 로드 밸런서로 어플리케이션을 호출할 수 있는데이런 방법은 마이크로 서비스에 적합하지 못한다.

이번 시간에는 Spring Eureka Server를 구성하고 Netflix에서 제공하는 Feign Client를 이용해서 서비스 디스커버리를 직접 구현하는 시간을 가져보도록 하겠다.

이글의 예제는 아래처럼 구성한다.


![주석 2020-04-02 144058](https://user-images.githubusercontent.com/30790184/78619343-12073800-78b8-11ea-98cf-1bb07c2dd359.png)

OrderService와 MemberService는 각각 주문과 회원에대한 기능을 제공하는데 Hello World 수준으로 매우매우 간단하게 구성했다. 클라이언트가 OrderService에 /order로 요청하면 OrderService는 Netflix Feign Client를 사용해서 Spring Eureka Server를 이용해 MemberService와 통신하여 회원 정보를 받아낸다.





<br>

# Spring Eureka Server 구성하기

Spring Eureka Server 역시 Spring Boot를 사용한다면 굉장히 간단하게 Server를 구성할 수 있다.

**pom.xml**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>me.sup2is</groupId>
    <artifactId>eureka-server</artifactId>
    <version>1.0-SNAPSHOT</version>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.0.3.RELEASE</version>
    </parent>

    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
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
    
</project>
```

**spring-cloud-starter-netflix-eureka-server**모듈과 **jaxb**관련된 모듈을 넣어주었는데 jaxb 모듈 같은 경우는 만약 java 11이하버전이라면 넣어주지 않아도 괜찮다.

<br>

**EurekaServerApplication.java**

```java
package me.sup2is.eurekaserver;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;

@SpringBootApplication
@EnableEurekaServer
public class EurekaServerApplication {

    public static void main(String[] args) {
        SpringApplication.run(EurekaServerApplication.class,args);
    }

}
```

Spring Boot Application 작성을 위한 EurekaServerApplication 클래스를 생성하고 **@EnableEurekaServer**를 사용하여 이 어플리케이션이 Eureka Server의 기능을 하는것을 명시해둔다. 이 어노테이션으로 Eureka Server의 설정을 Spring Boot가 자동으로 설정해준다.

<br>

**application.yml**

```properties
server:
  port: 8761

eureka:
  client:
    registerWithEureka: false
    fetchRegistry: false
  server:
    waitTimeInMsWhenSyncEmpty: 5
  serverUrl:
    defaultZone: http://127.0.0.1:8671
```

마지막으로 application.yml을 설정해준다.

1. **eureka.client.registerWithEureka**: false로 지정했을때 자신을 유레카 서비스에 등록하지 않도록 설정
2. **eureka.client.fetchRegistry**: false로 설정시 유레카 서비스가 시작할때 레지스트리 정보를 로컬에 저장하지 않음. 스프링 부트 서비스로 된 유레카 클라이언트를 유레카에 등록할 경우 이 값을 바꿀 수 있음
3. **eureka.server.waitTimeInMsWhenSyncEmpty**: 유레카는 기본적으로 모든 서비스가 등록할 기회를 갖도록 5분을 기다린 후 등록된 서비스 정보를 공유함. 로컬에서 테스트할때 시간단축용으로 사용

Eureka는 등록된 서비스에서 10초 간격으로 연속 3회의 상태정보를 받아야하므로 등록된 개별 서비스를 보여주는데 30초가 걸린다.  테스트할때 이점을 항상 기억해두자.

<br>

위 설정만으로 간단하게 Eureka Server를 Spring Boot 기반으로 설정이 가능하다. 이어서 NetFlix의 Feign을 사용해서 서비스 디스커버리를 구현해보도록 하자.

<br>

# MemberService 구현하기

초반에 언급한대로 MemberService는 회원관련 서비스다. Spring Eureka Server에 등록하기 위해 몇가지 설정만 해주면 된다.

<br>

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
    <artifactId>member-service</artifactId>

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

별다른 특징은 따로 없고 **spring-cloud-starter-netflix-eureka-client**모듈을 추가해준다. 위에서 언급한대로  **jaxb**관련된 모듈을 넣어주었는데 jaxb 모듈 같은 경우는 만약 java 11이하버전이라면 넣어주지 않아도 괜찮다.

<br>

**MemberServiceApplication.java**

```java
package me.sup2is.memberservice;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;

@SpringBootApplication
@EnableEurekaClient
public class MemberServiceApplication {

    public static void main(String[] args) {
        SpringApplication.run(MemberServiceApplication.class, args);
    }
}
```

main 메서드가 있는 MemberServiceApplication 클래스에는 **@EnableEurekaClient**을 사용하여 이 서비스가 Eureka Client임을 명시해준다. **@EnableEurekaClient**는 자신을 유레카 서비스 디스커버리 에이전트에 등록하고 서비스 디스커버리를 사용해서 물리적인 위치의 투명성을 구현할 수 있다. 

<br>

**application.yml**

```properties
server:
  port: 8081

spring:
  application:
    name: memberservice

eureka:
  instance:
    preferIpAddress: true
  client:
    registerWithEureka: true
    fetchRegistry: true
    serviceUrl:
      defaultZone: http://127.0.0.1:8761/eureka/
```

**eureka.instance.preferIpAddress**: 서비스 호스트 이름이 아닌 IP 주소를 유레카에 등록하도록 지정한다.

기본적으로 유레카는 접속하는 서비스를 호스트이름 기반으로 등록한다. 이 설정은 DNS가 지원된 호스트 이름을 할당하는 서버 기반의 환경에서는 잘 동작하지만 컨테이너 기반에서는 잘 동작하지 않는다. 컨테이너는 DNS 엔트리가 없는 임의로 생성된 호스트 이름을 부여받아서 시작하기 때문이다.  **eureka.instance.preferIpAddress**을 true로 설정해주어 IP 주소를 전달받는것으로 설정해야 어플리케이션의 호스트를 구해올 수 있다.

spring.application.name을 통해 applicationId를 구성하는데 이 Id는 인스턴스 ID가 아니라 n개의 MemberService를 그룹화시켜주는 ID가 될 것이다.

<br>



**Member.java**

```java
package me.sup2is.memberservice;

import lombok.AllArgsConstructor;
import lombok.Data;

@Data
@AllArgsConstructor
public class Member {

    private Long id;
    private String name;
    private String password;


}
```

<br>

**MemberController.java**

```java
package me.sup2is.memberservice;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class MemberController {

    @GetMapping("/member")
    public Member getMember() {
        return new Member(1L, "sup2is", "qwer!23");
    }

}
```

최대한 간단하게 구성하기때문에 MemberService에 /member로 요청시에 하드코딩된 엔티티만 반환하도록 설정해놨다.

<br>

이 MemberService는 초기에 어플리케이션이 로딩되는순간 미리 준비해둔 Spring Eureka와 통신하여 자신을 서비스 디스커버리 에이전트에 등록할 것이다. 이어서 OrderService를 통해서 Feign Client를 구성해보자!

<br>

# OrderService 구현하기

OrderService는 주문관련 서비스이다. 주문할때 MemberService에서 회원정보를 가져온 뒤 주문관련 로직을 실행한다.



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

**spring-cloud-starter-netflix-eureka-client**와 동시에 **spring-cloud-starter-openfeign**를 추가해준다.

<br>

**OrderServiceApplication.java**

```java
package me.sup2is.memberservice;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.loadbalancer.LoadBalanced;
import org.springframework.cloud.openfeign.EnableFeignClients;
import org.springframework.context.annotation.Bean;
import org.springframework.web.client.RestTemplate;

@SpringBootApplication
@EnableFeignClients
public class OrderServiceApplication {

    public static void main(String[] args) {
        SpringApplication.run(OrderServiceApplication.class, args);
    }
}
```

main 메서드가 있는 OrderServiceApplication 클래스에는 **@EnableFeignClients**을 명시해준다. **@EnableFeignClients**는 Netflix Feign Client 기능을 사용하도록 설정해주는 어노테이션이다.

<br>

```properties
server:
  port: 8082

spring:
  application:
    name: orderservice

eureka:
  instance:
    preferIpAddress: true
  client:
    registerWithEureka: true
    fetchRegistry: true
    serviceUrl:
      defaultZone: http://127.0.0.1:8761/eureka/
```

MemberService와 마찬가지로 Eureka Server 관련한 설정을 해준다.

<br>

**MemberServiceFeignClient.java**

```java
package me.sup2is.memberservice;

import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.GetMapping;

@FeignClient("memberservice")
public interface MemberServiceFeignClient {

    @GetMapping(value = "/member",
            consumes = "application/json")
    Member getMember();

}
```

이부분이 실제 Feign Client를 사용하는 부분인데 이 글에서 자세하게 언급하지는 않지만 리본이나 서비스 디스커버리를 직접 구현해서 사용하는것보다 매우 추상적이고 개인적으로 Feign이 가장 깔끔하다고 생각한다. **@FeignClient("memberservice")**를 통해 이 전에 준비해놓은 memberservice를 바라보게 설정하고  @GetMapping을 통해서 MemberService에 질의할 URL을 정의할 수 있다. 여기에서는 @PutMapping 같은 REST 기반 메서드도 지원이 가능하고 @PathVariable 같은 파라미터라이징 기능도 사용이 가능하다. 만약 파라미터가 존재한다면 다음과 같은 형태로 사용 가능하다.

```java
@GetMapping(value = "/member/{memberId}",
        consumes = "application/json")
Member getMember(@PathVariable Long memberId);
```
<br>

**OrderController.java**

```java
package me.sup2is.memberservice;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class OrderController {

    @Autowired
    private MemberServiceFeignClient memberServiceFeignClient;

    @GetMapping("/order")
    public String order() {
        return memberServiceFeignClient.getMember().getName() + "님이 주문요청하셨습니다.";
    }

}
```

OrderController 클래스는 단순하게 MemberService에서 회원정보를 가져온 뒤 {name} 님이 주문요청하셨습니다. 라는 단순 String값을 반환하도록 구성했다. MemberService와 OrderService를 매우 간단하게 구성했기때문에 항상 같은 값을 반환할 것이다.

<br>

# Example

간단하게 테스트해보도록 하겠다.

1. Spring Eureka Server를 올린다. (port = 8761)
2. MemberService를 2개 올린다. (port = 8081, 8082)
3. OrderService를 1개 올린다 (port = 8080)



![20200407_095656](https://user-images.githubusercontent.com/30790184/78619736-21d34c00-78b9-11ea-84a9-d9bac5b4c030.jpg)

<br>



Spring Eureka Server는 **/eureka/apps**를 통해서 서비스 디스커버리에 등록된 서비스들을 조회하는 기능을 제공해준다. 일단 서비스를 서비스 디스커버리 엔진에 등록시킬 시간이 필요하므로 바로 조회했을때 나오지 않는다고 실망하지 않아도 된다. 조금 기다린 뒤에 **127.0.0.1:8761/eureka/apps**을 요청해보자.

![주석 2020-04-07 095718](https://user-images.githubusercontent.com/30790184/78619344-129fce80-78b8-11ea-9611-8f6d31baef65.png)

<br>

```json
{
    "applications": {
        "versions__delta": "1",
        "apps__hashcode": "UP_3_",
        "application": [
            {
                "name": "ORDERSERVICE",
                "instance": [
                    {
                        "instanceId": "host.docker.internal:orderservice:8080",
                        "hostName": "172.16.11.206",
                        "app": "ORDERSERVICE",
                        "ipAddr": "172.16.11.206",
                        "status": "UP",
                        "overriddenStatus": "UNKNOWN",
                        "port": {
                            "$": 8080,
                            "@enabled": "true"
                        },
                        "securePort": {
                            "$": 443,
                            "@enabled": "false"
                        },
                        "countryId": 1,
                        "dataCenterInfo": {
                            "@class": "com.netflix.appinfo.InstanceInfo$DefaultDataCenterInfo",
                            "name": "MyOwn"
                        },
                        "leaseInfo": {
                            "renewalIntervalInSecs": 30,
                            "durationInSecs": 90,
                            "registrationTimestamp": 1586220571961,
                            "lastRenewalTimestamp": 1586220992016,
                            "evictionTimestamp": 0,
                            "serviceUpTimestamp": 1586220571447
                        },
                        "metadata": {
                            "management.port": "8080"
                        },
                        "homePageUrl": "http://172.16.11.206:8080/",
                        "statusPageUrl": "http://172.16.11.206:8080/actuator/info",
                        "healthCheckUrl": "http://172.16.11.206:8080/actuator/health",
                        "vipAddress": "orderservice",
                        "secureVipAddress": "orderservice",
                        "isCoordinatingDiscoveryServer": "false",
                        "lastUpdatedTimestamp": "1586220571961",
                        "lastDirtyTimestamp": "1586220571400",
                        "actionType": "ADDED"
                    }
                ]
            },
            {
                "name": "MEMBERSERVICE",
                "instance": [
                    {
                        "instanceId": "host.docker.internal:memberservice:8082",
                        "hostName": "172.16.11.206",
                        "app": "MEMBERSERVICE",
                        "ipAddr": "172.16.11.206",
                        "status": "UP",
                        "overriddenStatus": "UNKNOWN",
                        "port": {
                            "$": 8082,
                            "@enabled": "true"
                        },
                        "securePort": {
                            "$": 443,
                            "@enabled": "false"
                        },
                        "countryId": 1,
                        "dataCenterInfo": {
                            "@class": "com.netflix.appinfo.InstanceInfo$DefaultDataCenterInfo",
                            "name": "MyOwn"
                        },
                        "leaseInfo": {
                            "renewalIntervalInSecs": 30,
                            "durationInSecs": 90,
                            "registrationTimestamp": 1586220915344,
                            "lastRenewalTimestamp": 1586220915344,
                            "evictionTimestamp": 0,
                            "serviceUpTimestamp": 1586220914832
                        },
                        "metadata": {
                            "management.port": "8082"
                        },
                        "homePageUrl": "http://172.16.11.206:8082/",
                        "statusPageUrl": "http://172.16.11.206:8082/actuator/info",
                        "healthCheckUrl": "http://172.16.11.206:8082/actuator/health",
                        "vipAddress": "memberservice",
                        "secureVipAddress": "memberservice",
                        "isCoordinatingDiscoveryServer": "false",
                        "lastUpdatedTimestamp": "1586220915344",
                        "lastDirtyTimestamp": "1586220914789",
                        "actionType": "ADDED"
                    },
                    {
                        "instanceId": "host.docker.internal:memberservice:8081",
                        "hostName": "172.16.11.206",
                        "app": "MEMBERSERVICE",
                        "ipAddr": "172.16.11.206",
                        "status": "UP",
                        "overriddenStatus": "UNKNOWN",
                        "port": {
                            "$": 8081,
                            "@enabled": "true"
                        },
                        "securePort": {
                            "$": 443,
                            "@enabled": "false"
                        },
                        "countryId": 1,
                        "dataCenterInfo": {
                            "@class": "com.netflix.appinfo.InstanceInfo$DefaultDataCenterInfo",
                            "name": "MyOwn"
                        },
                        "leaseInfo": {
                            "renewalIntervalInSecs": 30,
                            "durationInSecs": 90,
                            "registrationTimestamp": 1586220494224,
                            "lastRenewalTimestamp": 1586221003630,
                            "evictionTimestamp": 0,
                            "serviceUpTimestamp": 1586220493508
                        },
                        "metadata": {
                            "management.port": "8081"
                        },
                        "homePageUrl": "http://172.16.11.206:8081/",
                        "statusPageUrl": "http://172.16.11.206:8081/actuator/info",
                        "healthCheckUrl": "http://172.16.11.206:8081/actuator/health",
                        "vipAddress": "memberservice",
                        "secureVipAddress": "memberservice",
                        "isCoordinatingDiscoveryServer": "false",
                        "lastUpdatedTimestamp": "1586220494225",
                        "lastDirtyTimestamp": "1586220492999",
                        "actionType": "ADDED"
                    }
                ]
            }
        ]
    }
}
```

요청한 뒤에는 각각 등록된 서비스들의 정보나 상태값들을 반환시켜준다. 이렇게 조회했을때 나오는 서비스들이 서비스 디스커버리 엔진에 등록된 서비스들이고 만약 서비스들이 비정상적인 종료가 이루어졌을때는 일정 시간 뒤에 해당 서비스를 목록에서 제거시킨다.

<br>

Postman을 이용해서 127.0.0.1:8080/order로 요청하면 아래와 같은 응답 결과가 나온다.

![주석 2020-04-07 100835](https://user-images.githubusercontent.com/30790184/78619345-129fce80-78b8-11ea-94d5-272f3c6c1077.png)

위 결과에서 볼 수 있듯이 OrderService는 MemberService의 물리적 위치를 모르지만 Spring Eureka의 서비스 디스커버리를 통해서 서비스를 검색하고 질의할 수 있도록 구성했다.

<br>

포스팅은 여기까지 하겠습니다.  모든예제는 제 github에서 확인하실 수 있습니다.

예제 : https://github.com/sup2is/spring-example/tree/master/spring-cloud-eureka-with-netfix-feign-client



<br>

**References**

-  Spring 마이크로서비스 코딩 공작소 -존 카넬 (길벗출판사)
