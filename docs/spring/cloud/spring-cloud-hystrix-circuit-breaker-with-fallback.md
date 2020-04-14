---

layout: post
title: "Spring Cloud에서 Hystrix를 사용한 Circuit Breaker 구현하기 Feat.Fallback"
tags: [Spring Boot, Spring Cloud, Netflix Hystrix, Fallback]
date: 2020-04-12
comments: true
grand_parent: Spring
parent: Cloud
---



<br>

# OverView

Spring Cloud에서는 서비스 하나가 완전히 종료되어 더이상의 통신이 불가능한 상태라면 쉽게 감지가 가능하다. 하지만 서비스가 느려졌을때 성능 저하를 감지하고 우회하는 방법은 매우 어려운데 다음 세가지 경우를 살펴보자.

1. **서비스 저하는 간헐적으로 발생하고 확산될 수 있다.** : 서비스 저하는 사소한 부분에서 갑자기 발생 가능하다. 순식간에 어플리케이션 컨테이너가 스레드 풀을 모두 소진해 완전히 무너지기 전까지 장애 징후는 일부 사용자가 문제점을 불평하는 정도이기 때문이다.
2. **원격 서비스 호출은 대개 동기식이며 오래 걸리는 호출을 중단하지 않는다.** : 서비스 호출자에게는 호출이 영구 수행되는 것을 방지하는 타임아웃 개념이 없다. 어플리케이션 개발자는 서비스를 호출해 작업을 수행하고 서비스가 응답할 때 까지 기다린다.
3. **어플리케이션은 대개 부분적인 저하가 아닌 원격 자원의 완전한 장애를 처리하도록 설계된다. **: 서비스가 완전히 붕괴되지 않는 이상 서비스를 계속 호출하고 빠른 실패 확인이 불가능하다. 호출하는 서비스는 제대로 동작하지 않는 어플리케이션을 호출해서 일부 호출은 비정상적인 종료가 될 수 있다.

마이크로서비스 환경에서는 서비스 단위로 어플리케이션이 세분화 되어 있기 때문에 이런 에러에 취약하다.

<br>

이번 시간에 알아볼 Netflix Hystrix는 위에서 소개한 에러들을 효율적으로 관리할 수 있도록 도와준다. Hystrix가 제공하는 **Circuit Breaker, FallBack, Bulkhead 패턴**을 간단한 예제와 함께 알아보는 시간을 가져보도록 하겠다. 이 글에서는 **Bulkhead 패턴**에 대해서는 다루지 않는다.

<br>

# Circuit Breaker

Circuit Breaker가 필요한 경우에 대해서 자세하게 한번 짚어보고 가자. 마이크로서비스에서 만약 한 부분의 서비스가 매우 느리게 응답할 경우 최초에 호출했던 서비스들의 스레드 풀이 계속해서 중첩된다. 결국에 호출된 모든 서비스들은 포화상태가 되어 전체 어플리케이션의 고장을 야기하는데 이 경우에 Circuit Breaker를 구성하면 느리게 답하는 한 부분의 특정 호출을 감지해서 스레드를 소진하지 않도록 빠르게 실패시키고 해당 원격자원을 사용하는 부분을 제거함으로써 그나마 나머지 엮이지 않는 부분까지는 제 기능을 하도록 구성할 수 있다.



# Fallback

만약 Circuit Breaker가 느리게 답하는 한 부분의 특정 호출을 차단시켰다면 예외가 발생할 것이다. 하지만 Fallback을 설정해 놓는다면 호출한 서비스에게 미리 작성해놓은 대체 경로를 실행해서 다른 방법으로 작업 수행이 가능하도록 한다. 예를들어 사용자에게 추천상품을 추천해주는 기능이 있는 웹서비스의 예를 들어보도록 하겠다. 마이크로 서비스 환경에서 이런 추천상품을 추천해주는 서비스가 만약 고장난다면 폴백 패턴을 통해 모든 사용자의 구매 정보를 기반으로 조금 더 일반화된 상품 목록을 조회하도록 구성할 수 있다.



# Hystrix

Circuit Breaker, Fallback 그리고 이 글에서 다루지 않는 Bulkhead 패턴을 직접 구현하는 것은 매우매우 어려운 일이다. 다행히도 Netflix는 실제 서비스를 운영하면서 검증된 Hystrix라는 모듈을 제공한다. 히스트릭스는 두가지 경우로 사용이 가능한데 **서비스와 서비스** 사이는 물론 **서비스와 데이터베이스** 사이에도 둘 수 있다.

이 Hystrix를 사용하여 Circuit Breaker, Fallback 를 구성해보기 전 간단한 시나리오에 대해서 알아보도록 하자



# 시작하기 전에

먼저 Spring Eureka Server를 기반으로 한 MemberService, OrderService 가 있고 OrderService가 MemberService의 사용자 정보를 호출하여 주문정보에 해당 사용자 정보를 기반으로 주문서를 작성한다. 그런데 MemberService에 원인 모를 서비스 성능 저하가 발생한다고 가정한다.

이런 성능 저하를 발견한 Circuit Breaker가 원격 호출 차단을 발동하고 MemberService는 Fallback 전략을 통해서 조금 더 일반화된 사용자 정보를 반환한다. 물론 실제 서비스환경에서 일반화된 사용자 정보는 없지만 예제를 이해하기엔 충분하다고 생각한다.




![주석 2020-04-02 144058](https://user-images.githubusercontent.com/30790184/78619343-12073800-78b8-11ea-98cf-1bb07c2dd359.png)



OrderService와 MemberService는 각각 주문과 회원에대한 기능을 제공하는데 Hello World 수준으로 매우매우 간단하게 구성했다. 클라이언트가 OrderService에 /order로 요청하면 OrderService는 Netflix Feign Client를 사용해서 Spring Eureka Server를 이용해 MemberService와 통신하여 회원 정보를 받아낸다. MemberService는 항상 id가 `1L`, name은 `sup2is` password는 `qwer!23`을 반환하지만 Circuit Breaker가 발동했을때는 id가 `null`, name은 `customer` password는 `undefined`를 반환한다.



이제 실제 예제를 구성하면서 Hystrix 설정에 대해 자세하게 알아보도록 하자!

만약 Spring Eureka Server 또는 Feign Client에 대한 내용이 궁금하다면 [이전글](https://sup2is.github.io/docs/spring/cloud/spring-cloud-eureka-with-netfix-feign-client-example.html)을 살펴보면 도움이 될 수 있다. 이 글에서는 Spring Eureka Server 와 Netflix Feign Client에 대한 내용은 자세하게 다루지 않는다.

<br>



# MemberService에 Hystrix 설정하기

호출당하는 MemberService에 Hystrix 설정을 해보도록 하자

먼저 pom.xml 부터 간단하게 살펴보자

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
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
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

**jaxb**관련된 모듈을 넣어주었는데 jaxb 모듈 같은 경우는 만약 java 11이하버전이라면 넣어주지 않아도 괜찮다. 주목할만한 라이브러리는 **spring-cloud-starter-netflix-eureka-client**, **spring-cloud-starter-netflix-hystrix**정도다.

<br>

Hystrix를 사용하기 위해서는 부스스트랩클래스에서 **@EnableCircuitBreaker**어노테이션을 다음과 같이 설정해주어야 한다.

**MemberServiceApplication.java**

```java
package me.sup2is.memberservice;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.circuitbreaker.EnableCircuitBreaker;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;

@SpringBootApplication
@EnableEurekaClient
@EnableCircuitBreaker
public class MemberServiceApplication {

    public static void main(String[] args) {
        SpringApplication.run(MemberServiceApplication.class, args);
    }
}
```

<br>

**MemberController.java**

```java
package me.sup2is.memberservice;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class MemberController {

    @Autowired
    private MemberService memberService;

    @GetMapping("/member")
    public Member getMember() {
        return memberService.getMemberById(1L);
    }

}
```

<br>

이제 실제 서비스로직에 Circuit Breaker를 설정해보자

**MemberService.java**

```java
package me.sup2is.memberservice;

import com.netflix.hystrix.contrib.javanica.annotation.HystrixCommand;
import com.netflix.hystrix.contrib.javanica.annotation.HystrixProperty;
import org.springframework.stereotype.Service;

@Service
public class MemberService {

    @HystrixCommand(
            fallbackMethod = "commonMemberInfo",
            commandProperties = {
                    @HystrixProperty(
                            name = "execution.isolation.thread.timeoutInMilliseconds", value = "2000"),
                    @HystrixProperty(
                            name="circuitBreaker.requestVolumeThreshold", value="10"),
                    @HystrixProperty(
                            name="circuitBreaker.errorThresholdPercentage", value="50"),
                    @HystrixProperty(
                            name="circuitBreaker.sleepWindowInMilliseconds", value="7000"),
                    @HystrixProperty(
                            name="metrics.rollingStats.timeInMilliseconds", value="15000")
            }
    )
    public Member getMemberById(long id) {
        randomlySleep();
        return new Member(id, "sup2is", "qwer!23");
    }

    private void randomlySleep() {
        int random = (int) (Math.random() * 10);
        System.out.println(random);
        if(random % 2 == 0) {
            try {
                Thread.sleep(3000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    public Member commonMemberInfo(long id) {
        return new Member(null, "customer" , "undefined");
    }

}
```

메서드 단위의 Hystrix설정은 **@HystrixCommand**를 사용해서 설정하면 된다. @HystrixCommand 내부에는 상세 설정을 위한 옵션이 몇가지 더 있는데 위에서 사용한 설정만 간단하게 알아보도록하자

- **fallbackMethod:** Circuit Breaker가 작동될때 동작하는 fallback 메서드 이름을 지정한다. 이 fallback 메서드는 @HystrixCommand를 사용한 클래스와 같이 있어야하고 파라미터 역시 동일해야 한다.
- **@HystrixProperty**: @HystrixCommand 내부 **commandProperties** 안에 상세설정 할 수 있도록 도와주는 어노테이션이다.
- **execution.isolation.thread.timeoutInMilliseconds:** 히스트릭스는 메서드 호출 이후의 시간을 모니터링해서 일정 시간이 지나면 호출을 강제종료시키는데 기본값은 1,000ms이다. 나같은 경우는 이 값을 2,000ms으로 지정해서 randomlySleep()을 통해 50프로 확률로 실패하도록 설정해 놓았다.
- **metrics.rollingStats.timeInMilliseconds:** 히스트릭스는 요청이 들어오는 시점부터 요청에 대한 오류감지 시간을 설정할 수 있다. 기본값은 10,000ms이다.
- **circuitBreaker.requestVolumeThreshold:** 히스트릭스는 오류감지 시간 설정해 놓은 시간인**metrics.rollingStats.timeInMilliseconds** 에서 설정해 놓은 값 동안 최소요청회수를 설정할 수 있다. 기본값은 20이다. 만약 설정해 놓은 시간이 15,000ms 인데 이 안에 요청이 19개 들어왔고 19개의 요청이 모두 실패하더라도 기본값 20을 넘기지 않았기 때문에 Circuit Breaker는 동작하지 않는다.
- **circuitBreaker.errorThresholdPercentage:** 오류감지시간, 최소요청회수를 모두 만족했을때 히스트릭스는 요청에 대한 통계를 내어 일정 확률 이상 실패했다면 Circuit breaker를 동작시킨다 기본값은 50% 이다.
- **circuitBreaker.sleepWindowInMilliseconds:** 히스트릭스가 서비스의 회복 상태를 확인할때까지 대기하는 시간이다 기본값은 5,000ms이다.

설정이 조금 복잡하긴한데 간단하게 풀어서 설명하면 요청이 들어오는순간 히스트릭스는 모니터링을 시작한다.(15,000ms) 설정한 시간 내에 최소요청회수(10)를 달성하면 요청에 따른 성공 또는 실패의 통계를 내어 실패율이 설정해놓은 값(50%)보다 높다면 히스트릭스는 Circuit breaker를 발동시켜서 이후 요청은 무조건 실패로 만들어 놓는다. 그리고 설정한 서비스 회복 시간(7,000ms) 이전까지는 무조건 실패시키다가 시간 이후 요청이 성공하면 Circuit Breaker는 종료된다 만약 서비스 회복 시간 이후에 첫 요청이 실패한다면 Circuit Breaker는 다시 동작한다.

더욱더 많은 정보나 자세한 Hystrix 설정은 [이곳](https://github.com/Netflix/Hystrix/wiki/Configuration#execution.isolation.strategy)을 참고하면 된다.

<br>

만약 클래스 단위로 Hystrix를 설정하고 싶다면 다음과같이 **@DefaultProperties**을 사용해서 설정할 수 있다.

```java
@DefaultProperties(
  commandProperties = {
    @HystrixProperty(
      name="execution.isolation.thread.timeoㅕtInMilliseconds", value="10000"
    )
  }
)
class MyService{...}
```

<br>

# Postman으로 테스트해보기

실제 OrderService는 MemberService의 /member 로 요청해서 name이 'sup2is' 인 회원 정보를 얻어올 것이다. 하지만 만약 MemberService의 randomlySleep()으로 sleep이 많이 발동된다면 히스트릭스는 실패로 간주하고 미리 준비해놓은 fallback 메서드를 통해 'customer' 를 반환한다. 이런 실패율이 지정해놓은 값보다 높게 설정되면 히스트릭스는 Circuit Breaker를 발동시켜서 항상 'customer'를 반환할 것이다. 물론 예제이기때문에 매우 간단하게 구성했다.

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

<br>

Postman에는 테스트를 위한 몇가지 도구를 제공해준다. 지정해놓은 횟수만큼 어떤 동작을 예상해서 테스트의 성공 여부를 결정할 수 있도록 도와준다 자세한 정보는 [이곳](https://learning.postman.com/docs/postman/scripts/test-scripts/)을 참고하도록 하자



OrderService는 MemberService와 통신해서 "sup2is님이 주문요청하셨습니다." 라는 문자열을 반환하도록 해놨으므로 test script는 아래와 같이 설정했다.

![주석 2020-04-07 095718](https://user-images.githubusercontent.com/30790184/79119168-ad257380-7dca-11ea-9cff-d8cf10201b82.png)



이제 실제 테스트를 진행해보자 

50회를 요청해봤다.

![녹화_2020_04_13_21_00_07_535](https://user-images.githubusercontent.com/30790184/79119334-14432800-7dcb-11ea-95dd-785555cc6b10.gif)

<br>

![주석 2020-04-13 204834](https://user-images.githubusercontent.com/30790184/79119170-ae56a080-7dca-11ea-9f85-21b0e35e9a87.png)



MemberService가 타임아웃으로 몇차례 종료되더니 어느순간 이후에는 Circuit Breaker가 발동되어 빠른 실패하는 것을 확인할 수 있다. 실패 위치를 확인해보면 최소 요청 횟수를 만족하고 실패율 역시 만족하는 것을 확인할 수 있다.



## 몇가지 알고가기

### 스레드 컨텍스트와 히스트릭스

@HystrixCommand가 실행될 때 개발자는 Thread 격리 방식, Semaphore 격리 방식을 선택할 수 있다. 기본적으로 히스트릭스는 Thread 격리 방식을 사용하는데 이 방식은 호출을 시도한 부모 스레드와 컨텍스트를 공유하지 않고 격리된 스레드 풀에서 수행된다. 이런 전략은 히스트릭스가 자기 통제 하에서 원래 호출을 시도한 부모 스레드와 연관된 어떤 활동도 방해햐지 않고 스레드를 중단할 수 있다는 것을 의미한다.

Semaphore 격리 방식은 히스트릭스는 새로운 스레드를 시작하지 않고 @HystrixCommand 어노테이션이 보호하는 분산 호출을 관리하며 타임아웃이 발생하면 부모 스레드를 중단시킨다.

```java
@HystrixCommand(
  commandProperties = {
    @HystrixProperty(
      name="execution.isolation.strategy", value="SEMAPHORE"
    )
  }
)
```

넷플릭스 역시 기본적으로 Thread 격리 방식을 추천하고 Thread 전략이 Semaphore 전략보다 격리 수준이 더 높다. Semaphore는 Netty같은 비동기 I/O 컨테이너를 적용할때 고려해야한다.

### ThreadLocal과 히스트릭스

기본적으로 히스트릭스는 부모 스레드의 컨텍스트를 히스트릭스 명령이 관리하는 스레드에 전파하지 않는다. 예를들면 Spring Securty에서 ThreadLocal 전략으로 UserContext 객체를 저장해서 하나의 컨텍스트에서 항상 UserContext 객체를 꺼내올 수 있지만 히스트릭스로 @HystrixCommand로 감싸진 부분은 전파되지 않는다.

다행히 히스트릭스와 스프링클라우드는 부모 스레드의 컨텍스트를 히스트릭스 스레드 풀이 관리하는 스레드에 전달하는 메커니즘을 제공한다. 이 메커니즘을 **HystrixConcurrencyStrategy**라고한다. HystrixConcurrencyStrategy에 대한 자세한 설명은 생략한다.



<br>

포스팅은 여기까지 하겠습니다.  모든예제는 제 github에서 확인하실 수 있습니다.

예제 : https://github.com/sup2is/spring-example/tree/master/spring-cloud-hystrix



<br>

**References**

-  Spring 마이크로서비스 코딩 공작소 -존 카넬 (길벗출판사)
