---

layout: post
title: "Spring Cloud에서 Zuul로 Service Gateway 구현하기"
tags: [Spring Boot, Spring Cloud, Netflix Zuul, Service Gateway]
date: 2020-04-15
comments: true
grand_parent: Spring
parent: Cloud
---



<br>

# OverView

마이크로서비스에서 보안과 로깅, 사용자 추적 등의 기능이 필요한 시점이 오는데 개발자는 비지니스 로직에만 집중하기때문에 이런 보안, 로깅 등의 횡단관심사에 대한 기능을 적절하게 구현하기 어렵다. 설령 기능을 구현한다 하더라도 중복된 코드 또는 공용 라이브러리를 사용함으로써 서비스 간 복잡한 의존성을 만들 수 있다. 이런 문제들을 해결하려면 특정 서비스에서 이러한 횡단 관심사들을 추상화하고 독립적인 위치에서 어플리케이션의 모든 마이크로서비스 호출에 대한 필터와 라우터 역할을 해야한다. 이러한 횡단 관심사를 **서비스 게이트웨이**라고 한다. 

서비스 클라이언트는 서비스를 직접 호출하지 않고 서비스 게이트웨이로 모든 호출을 경유시켜 최종 목적지로 라우팅한다. 



# 서비스 게이트웨이란?

서비스 게이트웨이는 서비스 클라이언트와 호출될 서비스 사이에서 중개 역할을 한다. 서비스 게이트웨이가 구축되면 서비스 클라이언트는 개별 서비스의 URL을 직접 호출하지 않고 서비스 게이트웨이로 모든 호출을 보낸다.

서비스 게이트웨이에서 구현할 수 있는 횡단관심사는 다음과 같다.

- **정적 라우팅**: 서비스 게이트웨이는 단일 서비스와 URL과 API 경로로 모든 서비스를 호출하게 한다. 개발자는 모든 서비스에 대하 하나의 서비스 엔드포인트만 알면 되므로 개발이 간단해진다.
- **동적 라우팅**: 서비스 게이트웨이는 유입되는 서비스 요청을 조사하고 요청 데이터를 기반으로 서비스 호출자 대상에 따라 지능형 라우팅을 수행할 수 있다. 예를 들어 베타 프로그램에 참여하는 고객의 서비스 호출은 모두 다른 코드 버전이 수행되는 특정 서비스 클러스터로 라우팅 될 수 있다.
- **인증과 인가**: 모든 서비스 호출은 서비스 게이트웨이로 라우팅되므로 서비스 게이트웨이는 서비스 호출자가 자신을 인증하고 서비스를 호출할 권할 여부를 확인할 수 있는 최적의 장소이다.
- **측정 지표 수집과 로깅**: 서비스 게이트웨이를 사용하면 서비스 호출이 서비스 게이트웨이를 통과할 때 측정 지표와 로그 정보를 수집할 수 있다. 규격화된 로깅을 보장하기 위해 사용자 요청에서 주요 정보가 누락되지 않았는지 확인하는 데도 사용된다. 이는 각 서비스에서 측정 지표를 수집할 필요가 없다는 것이 아니라 서비스 게이트웨이를 사용하면 서비스가 호출된 횟수와 응답 시간처럼 많은 기본 측정 지표를 한곳에서 수집할 수 있다는 의미다.

# Spring Cloud와 Netflix Zuul

스프링 클라우드는 넷플릭스의 오픈 소스 프로젝트인 Zuul을 통합한다. Zuul은 다음의 서비스 게이트웨이 기능을 제공한다.



- **어플리케이션의 모든 서비스 경로를 단일 URL로 매핑**: Zuul의 매핑이 단일 URL로만 제한되는 것은 아니다 Zuul에서는 여러 경로 항목을 정의해 경로 매핑을 매우 세분화할 수 있다. 하지만 Zuul의 가장 일반적인 사용 사례는 모든 서비스 호출이 통과하는 단일 진입점을 구축하는 것이다.
- **게이트웨이로 유입되는 요청을 검사하고 대응할 수 있는 필터 작성**: 이러한 필터를 사용하면 코드에 정책시행지점을 주입해서 모든 서비스 호출에서 광범위한 작업을 일관된 방식으로 수행할 수 있다.



위 두가지 기능을 기반으로 간단하게 예제를 작성해 보자!



# Zuul을 사용해서 서비스 게이트웨이 구현하기

이 예제는 아래의 아키텍쳐를 기반으로 구성한다.

![주석 2020-04-15 105539](https://user-images.githubusercontent.com/30790184/79290703-cd0e8180-7f07-11ea-9179-703e88649245.png)

넷플릭스의 Zuul은 Spring Boot를 사용하면 아주아주 간단하게 구성 가능하다.

**pom.xml**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>

    <groupId>me.sup2is</groupId>
    <version>1.0-SNAPSHOT</version>
    <artifactId>zuul-server</artifactId>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.0.3.RELEASE</version>
    </parent>
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
    
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-zuul</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
    </dependencies>
</project>
```

이 예제는 유레카 서버 기반으로 동작하기때문에  **spring-cloud-starter-netflix-eureka-client**모듈을 넣어주고 Zuul 서버를 위한 **spring-cloud-starter-netflix-zuul**모듈도 넣어줬다. **spring-boot-starter-actuator**은 뒤에서 설명한다.

<br>

**ZuulServerApplication.java**

```java
package me.sup2is.zuulserver;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.zuul.EnableZuulProxy;

@SpringBootApplication
@EnableZuulProxy
public class ZuulServerApplication {

    public static void main(String[] args) {
        SpringApplication.run(ZuulServerApplication.class, args);
    }
}

```

부트스트랩 클래스에 **@EnableZuulProxy**를 추가해줘서 Zuul 서버에 대한 설정을 해준다.비슷한 느낌의   **@EnableZuulServer**가 있는데 이 어노테이션을 사용하면 자체 라우팅 서비스를 만들고 내장된 Zuul 기능도 사용하지 않을때 선택할 수 있다. 유레카가 아닌 서비스 디스커버리 엔진(ex: Consul)과 통합할 경우가 해당된다. 우리는 유레카와 통합할것이기 때문에  @EnableZuulProxy를 사용한다.

<br>

**application.yml**

```properties
eureka:
  instance:
    preferIpAddress: true
  client:
    registerWithEureka: true
    fetchRegistry: true
    serviceUrl:
      defaultZone: http://127.0.0.1:8761/eureka/

spring:
  application:
    name: zuulservice
  profiles:
    active:
      default

logging:
  level:
    com.netflix: WARN
    org.springframework.web: WARN
    com.thoughtmechanix: DEBUG
    
management:
  endpoints:
    web:
      exposure:
        include: routes

server:
  port: 5555
```

먼저 Zuul서버를 유레카 서버에 등록해야하므로 **eureka.client.serviceUrl.defaultZone**으로 유레카 서버를 바라보게 설정해주고 그 외 설정은 아래를 참고한다.

1. **eureka.client.registerWithEureka**: false로 지정했을때 자신을 유레카 서비스에 등록하지 않도록 설정
2. **eureka.client.fetchRegistry**: false로 설정시 유레카 서비스가 시작할때 레지스트리 정보를 로컬에 저장하지 않음. 스프링 부트 서비스로 된 유레카 클라이언트를 유레카에 등록할 경우 이 값을 바꿀 수 있음
3. **eureka.instance.preferIpAddress**: 서비스 호스트 이름이 아닌 IP 주소를 유레카에 등록하도록 지정
4. **management.endpoints.web.exposure.include**: actuator를 통해 Zuul이 매핑하는 서비스 목록을 확인할 수 있도록 'routes' 값을 지정

<br>

위와 같이 조금의 설정으로도 Zuul에서 제공하는 기능을 사용할 수 있다. 이렇게 설정하면 Zuul이 기존에 등록해놓은 **spring.application.name**을 기반으로 서비스 게이트웨이를 구성할 수 있다 간단하게 테스트해보자



**MemberService의 application.yml**

```properties
...
spring:
  application:
    name: memberservice
...
```



**OrderService의 application.yml**

```properties
...
spring:
  application:
    name: orderservice
...
```

<br>

아까 pom.xml과 application.yml에서 actuator 설정을 해줬는데 Zuul은 자신이 관리하는 서비스 목록을 http://127.0.0.1:5555/actuator/routes 경로로 다음과 같이 제공한다. 이런 기능은 서비스 엔드포인트의 실제 물리적 위치를 유레카와 소통하고 있어서 가능하다.  이런 구성은 단일 엔드포인트를 구성할 수 있다는 장점 뿐만 아니라 Zuul의 수정 없이 자유롭게 인스턴스를 추가 또는 제거할 수 있다.



![주석 2020-04-14 204107](https://user-images.githubusercontent.com/30790184/79236902-82a9e800-7ea8-11ea-81c9-2dae61a2a503.png)



현재 OrderService와 MemberService는 각각 /order 와 /member 라는 endpoint를 가지고 있다. 우리는 서비스 게이트웨이를 설정했기때문에 다음과 같은 endpoint를 얻어낼 수 있다.

- http://127.0.0.1:5555/orderservice/order
- http://127.0.0.1:5555/memberservice/member





## 몇가지 설정 더 해보기

다음과 같이 application.yml을 수정해보자

**application.yml**

```properties
...

zuul:
  ignored-services: memberservice, orderservice
  prefix: /api
  routes:
    memberservice: /my-member/**
    orderservice: /my-order/**

```

이런식으로 수정하면 위에서 설명한 endpoint를 Zuul은 다음과 같이 매핑시킨다.

- http://127.0.0.1:5555/api/my-order/order
- http://127.0.0.1:5555/api/my-member/member

<br>

- **zuul.prefix**는 접두사를 붙여서 우리가 일반적으로 api 서버를 구성할때와 동일하게 /api/**라는 endpoint를 얻어낼 수 있다.

- **zuul.routes.{applicationId}** 설정을 통해서 실제 매핑될 서비스 인스턴스의 매핑주소를 수동으로 직접 설정할 수 있다. 만약 Zuul은 앞에서 설명한 자동설정에서는 응답가능한 인스턴스가  없다면 /actuator/routes 를 호출해도 그 인스턴스 목록은 반환하지 않지만 이런식으로 수동으로 설정한다면 인스턴스 목록을 항상 반환한다. 
- **zuul.ignored-services **설정으로 기존에 사용하던 applicationId 로 매핑되는 주소를 제거시켜준다.

아래의 사진을 보면 이해가 조금 더 빠르게 될 것이다.

![주석 2020-04-14 204906](https://user-images.githubusercontent.com/30790184/79236903-83db1500-7ea8-11ea-9f50-7da35774f97f.png)

추가적으로 유레카에서 관리하는 서버가 아니여도 Zuul에서 직접 경로를 매핑해서 또다른 서비스의 endpoint로 라우팅 시킬 수 있다. 이 글에서는 다루지 않도록 하겠다.

<br>

Zuul은 히스트릭스와 리본 라이브러리를 사용해서 오래 수행되는 서비스 호출이 서비스 게이트웨이의 성능에 영향을 미치지 않도록 한다. 기본적으로 1초 이상이 걸리면 호출을 종료시키는데 주울로 실행중인 모든 서비스에 대해 히스트릭스 타임아웃을 설정하려면  hystrix.command.default.excution.isolation.thread.timeoutInMilliseconds 로 설정하면 된다. 특정 서비스에 대해 별도로 설정하고 싶다면 hystrix.command.**someservice**.excution.isolation.thread.timeoutInMilliseconds 로 설정하면 된다. 이 설정 말고 만약 5초 이상 걸리는 서비스가 있다면 리본의 타임아웃 설정도 해야한다. {service-name}.ribbon.ReadTimeout 프로퍼티를 사용해 리본의 타임아웃을 재정의할 수 있다. 



# Zuul의 강력한 Filter기능 사용하기

사실 진정한 Zuul의 기능은 이 Filter 기능이다. 모든 서비스 클라이언트들을 아우르는 서비스 게이트웨이 이기때문에 모든 서비스에 대한 로깅, 인증 및 권한 등의 처리를 한번에 묶어서 횡단으로 처리할 수 있다.

이런 Filter는 Spring 에서 일반적으로 사용하는 Filter랑 형태가 매우매우 비슷하기 때문에 익숙한 개발자들이라면 바로바로 쉽게 이해가 될 것이다.

<br>

## Pre-filter

Zuul에서 목표 대상에 대한 실제 요청이 발생하기 전에 호출된다. 일반적으로 사전 필터는 서비스의 일관된 메시지 형식을 확인하는 작업을 수행하거나 서비스를 이용하는 사용자가 인증 및 인가 되었는지 확인하는 게이트키퍼 역할을 한다.

**PreFilter.java**

```java
package me.sup2is.zuulserver;

import com.netflix.zuul.ZuulFilter;
import com.netflix.zuul.context.RequestContext;
import com.netflix.zuul.exception.ZuulException;
import org.springframework.stereotype.Component;

@Component
public class PreFilter extends ZuulFilter {

    private static final int FILTER_ORDER = 1;
    private static final boolean SHOULD_FILTER = true;
    private static final String PRE_FILTER_TYPE = "pre";

    @Override
    public String filterType() {
        return PRE_FILTER_TYPE;
    }

    @Override
    public int filterOrder() {
        return FILTER_ORDER;
    }

    @Override
    public boolean shouldFilter() {
        return SHOULD_FILTER;
    }

    @Override
    public Object run() throws ZuulException {
        RequestContext ctx = RequestContext.getCurrentContext();
        System.out.println("### PreFilter 동작!");
        System.out.println("### 요청한 URL : " + ctx.getRequest().getRequestURI());
        return null;
    }
}

```

ZuulFilter는 총 네가지 추상메서드를 제공하고 ZuulFilter를 알맞게 구현하면 용도에 맞는 filter를 직접 설정할 수 있다. 그리고 com.netflix.zuul.context.RequestContext 클래스를 통해서 현재 요청에 대한 Context를 얻어올 수 있다.

- **filterType()**: filter의 타입을 결정한다. 이후에 설명할 `post`, `route`가 있고 위와 같이 `pre`로 설정하면 사전필터로 등록된다.
- **filterOrder()**: 같은 타입의 filter 끼리의 동작 순서를 정한다. 
- **shouldFilter()**: 필터의 동작 여부를 정한다.
- **run()**: 실제 필터의 구현은 run() 내부에 작성한다.



이 글에서는 다루지 않지만 사전 필터는 상관관계 ID를 지정해서 연속적으로 이루어지는 마이크로 서비스 환경에서의 한 요청에 대한 트랜잭션을 관리할 수 있다. 이렇게 할 수 있는 이유는 모든 서비스는 Zuul이라는 서비스 게이트웨이를 지나쳐서 오기 때문이다.

<br>

## Post-filter

대상 서비스를 호출하고 응답을 클라이언트로 전송한 후 호출된다. 일반적으로 사후 필터는 대상 서비스의 응답을 로깅하거나 에러 처리, 민감한 정보에 대한 응답을 감시하는 목적으로 구현된다.

```java
package me.sup2is.zuulserver;

import com.netflix.zuul.ZuulFilter;
import com.netflix.zuul.exception.ZuulException;
import org.springframework.stereotype.Component;

@Component
public class PostFilter extends ZuulFilter {

    private static final int FILTER_ORDER = 1;
    private static final boolean SHOULD_FILTER = true;
    private static final String PRE_FILTER_TYPE = "post";

    @Override
    public String filterType() {
        return PRE_FILTER_TYPE;
    }

    @Override
    public int filterOrder() {
        return FILTER_ORDER;
    }

    @Override
    public boolean shouldFilter() {
        return SHOULD_FILTER;
    }

    @Override
    public Object run() throws ZuulException {
        System.out.println("### PostFilter 동작!");
        return null;
    }
}
```

PreFilter와 똑같이 ZuulFilter를 구현하도록 작성했고 filterType은 `post`로 작성했다.

상관관계 ID를 PreFilter에서 적용했다면 PostFilter에서는 클라이언트에게 다시 상관관계 ID를 반환하도록 프로그램을 작성하면 된다.

<br>

## Route-filter

대상 서비스가 호출되기 전에 호출을 가로채는데 사용된다. 일반적으로 라우팅 필터는 일정 수준의 동적 라우팅 필요 여부를 결정하는데 사용된다. 예를 들어 동일 서비스의 다른 두 버전을 라우팅할 수 있는 라우팅 단위 필터를 사용해 작은 호출 비율만 새 버전의 서비스로 라우팅 할 수 있다. 이렇게 하면 모든 사용자가 새로운 서비스를 이용하지 않고도 소수 사용자에게 새로운 기능을 노출할 수 있다. 라우팅 필터는 Zuul 외부의 서비스로도 동적 라우팅할 수 있다.

```java
package me.sup2is.zuulserver;

import com.netflix.zuul.ZuulFilter;
import com.netflix.zuul.exception.ZuulException;
import org.springframework.stereotype.Component;

@Component
public class RouteFilter extends ZuulFilter {

    private static final int FILTER_ORDER = 1;
    private static final boolean SHOULD_FILTER = true;
    private static final String PRE_FILTER_TYPE = "route";

    @Override
    public String filterType() {
        return PRE_FILTER_TYPE;
    }

    @Override
    public int filterOrder() {
        return FILTER_ORDER;
    }

    @Override
    public boolean shouldFilter() {
        return SHOULD_FILTER;
    }

    @Override
    public Object run() throws ZuulException {
        System.out.println("### RouteFilter 동작!");
        return null;
    }
}
```

마찬가지로 RouteFilter역시 ZuulFilter를 구현하는것으로 작성했다.





# 간단하게 테스트해보기

간단하게 지금까지 설정해놓은 Zuul 서버를 기반으로 http://127.0.0.1:5555/orderservice/order 으로 요청했을때 동작하는 필터의 순서와 정상 동작 여부를 확인해보자.



![주석 2020-04-14 233002](https://user-images.githubusercontent.com/30790184/79236905-8473ab80-7ea8-11ea-8519-ba1dfa4ea4d6.png)

직접 서비스 인스턴스로 요청하지 않고 Zuul을 통해 원하는 결과값을 얻어올 수 있다.

![주석 2020-04-14 232346](https://user-images.githubusercontent.com/30790184/79236904-83db1500-7ea8-11ea-84d6-0b0784cfe895.png)

예상했던 대로 pre-route-post 순서대로 동작하는걸 확인할 수 있다.



<br>

포스팅은 여기까지 하겠습니다.  모든예제는 제 github에서 확인하실 수 있습니다.

예제 : https://github.com/sup2is/spring-example/tree/master/spring-cloud-zuul



<br>

**References**

-  Spring 마이크로서비스 코딩 공작소 -존 카넬 (길벗출판사)
