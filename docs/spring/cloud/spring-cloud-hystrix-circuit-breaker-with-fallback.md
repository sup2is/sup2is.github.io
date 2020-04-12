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
3. **어플리케이션은 대개 부분적인 저하가 아닌 원격 자원의 완전한 장애를 처리하도록 설계된다. **: 서비스가 완전히 붕괴되지 않는 이상 서비스를 계쏙 호출하고 빠른 실패 확인이 불가능하다. 호출하는 서비스는 제대로 동작하지 않는 어플리케이션을 호출해서 일부 호출은 비정상적인 종료가 될 수 있다.

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

먼저 Spring Eureka Server를 기반으로 한 MemberService, OrderService 가 있고 OrderService가 MemberService의 사용자 정보를 호출하여 주문정보에 해당 사용자 정보






![주석 2020-04-02 144058](https://user-images.githubusercontent.com/30790184/78619343-12073800-78b8-11ea-98cf-1bb07c2dd359.png)

OrderService와 MemberService는 각각 주문과 회원에대한 기능을 제공하는데 Hello World 수준으로 매우매우 간단하게 구성했다. 클라이언트가 OrderService에 /order로 요청하면 OrderService는 Netflix Feign Client를 사용해서 Spring Eureka Server를 이용해 MemberService와 통신하여 회원 정보를 받아낸다.





<br>



















<br>

포스팅은 여기까지 하겠습니다.  모든예제는 제 github에서 확인하실 수 있습니다.

예제 : https://github.com/sup2is/spring-example/tree/master/spring-cloud-eureka-with-netfix-feign-client



<br>

**References**

-  Spring 마이크로서비스 코딩 공작소 -존 카넬 (길벗출판사)
