---

layout: post
title: "Spring Cloud Sleuth와 Zipkin으로 MSA에서 분산추적하기"
tags: [Spring Boot, Distributed Tracing, Spring Cloud Sleuth, Zipkin]
date: 2020-05-03
comments: true
---



<br>

# OverView

MSA는 작은 단위의 컴포넌트로 이루어진 하나의 거대한 서비스다. 여러개의 작은 컴포넌트 중 어딘가 성능에 이상이 있거나 디버깅해야 할 경우가 생긴다면 단순히 모놀리틱환경에서 사용했던 IDE 브레이크 포인트는 거의 불가능 할 것이다. 이번시간에는 Spring Cloud Sleuth와 Zipkin을 사용해서 MSA 환경에서 각 서비스간의 트랜잭션 분산추적을 알아보도록 하자. 

<br>

# 시작전에

![주석 2020-04-22 215352](https://user-images.githubusercontent.com/30790184/79984204-cd001a00-84e3-11ea-9660-afe927f4a4f2.png)

이 예제는 다음과 같은 구조로 진행된다. OrderService는 주문관련 처리를 하고 MemberService는 회원정보에 관련된 처리를 한다.

<br>



# Spring Cloud Sleuth 적용하기

Spring Cloud Sleuth는 각 서비스간의 트랜잭션을 연결해주는 상관관계ID를 자동으로 주입해준다.  형태는 다음과 같다.

```
{application-id},{추적 id},{스팬 id},{집킨에 추적 데이터 전송 여부}
```

1. **어플리케이션 이름:** 로그를 출력하는 어플리케이션 이름이다. 기본적으로 스프링 클라우드 슬루스는 어플리케이션의 spring.application.name을 추적에서 기록할 이름으로 사용한다
2. **추적 id**: 추적 id는 상관관계 id와 동급 용어이며, 전체 트랜잭션에서 고유한 숫자다.
3. **스팬 id**: 스팬 id는 전체 트랜잭션의 일부를 나타내는 고유 id다 트랜잭션에 속한 각 서비스에는 고유한 스팬id가 있는데 특히 트랜잭션을 시각화하는데 유용하다.
4. **집킨에 추적 데이터 전송 여부**: 대용량 서비스에서 생성된 추적 데이터의 양은 엄청나고 모두 가치가 있는 것은 아니다. 스프링 클라우드 슬루스를 사용하면 트랜잭션을 집킨에 보낼 시점과 방법을 결정할 수 있다. 스프링 클라우드 슬루스의 추적 블록 끝부분에 있는 true/false를 표시해 추적 정보의 집킨 전송 여부를 결정한다.

<br>

실제 설정은 어떠한 추가 설정 없이 단순히 **spring-cloud-starter-sleuth** 모듈만 추가해준다면 자동적으로 상관관계 ID를 설정해준다.



**OrderService와 MemberService의 pom.xml**

```xml
...

<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-sleuth</artifactId>
</dependency>

...
```

<br>

사실 모듈만 추가해줌으로써 더이상 할 일이 없지만 추적관련한 로그 설정도 해주어야 실제로 추적이 어떻게 이루어지고 있나를 콘솔창에서 확인할 수 있다 따라서 다음 설정도 추가해준다.



**OrderService와 MemberService의 application.yml**

```properties
...

logging:
  level:
    org.springframework.web: DEBUG
    org.springframework.cloud.sleuth: DEBUG
    
...
```



<br>

# 간단하게 테스트해보기

postman을 통해서 /order로 요청했을때 OrderService는 MemberService에서 회원 정보를 가져오도록 간단하게 프로그램을 구성했다. 그럼 실제로 서비스들 간에 상관관계 ID가 어떤식으로 구성되는지 알아보도록 하자



요청과 동시에 OrderService의 콘솔에는 아래의 로그가 출력이 된다. 

```
2020-05-03 13:24:15.795 DEBUG [orderservice,bc7becdc872a7c88,bc7becdc872a7c88,false] 3720868 --- [nio-8082-exec-2] o.s.web.servlet.DispatcherServlet        : DispatcherServlet with name 'dispatcherServlet' processing GET request for [/order/1]
2020-05-03 13:24:15.795 DEBUG [orderservice,bc7becdc872a7c88,bc7becdc872a7c88,false] 3720868 --- [nio-8082-exec-2] s.w.s.m.m.a.RequestMappingHandlerMapping : Looking up handler method for path /order/1
2020-05-03 13:24:15.796 DEBUG [orderservice,bc7becdc872a7c88,bc7becdc872a7c88,false] 3720868 --- [nio-8082-exec-2] s.w.s.m.m.a.RequestMappingHandlerMapping : Returning handler method [public java.lang.String me.sup2is.memberservice.OrderController.order(java.lang.Long)]
2020-05-03 13:24:15.797 DEBUG [orderservice,bc7becdc872a7c88,bc7becdc872a7c88,false] 3720868 --- [nio-8082-exec-2] o.s.web.servlet.DispatcherServlet        : Last-Modified value for [/order/1] is: -1
2020-05-03 13:24:15.797 DEBUG [orderservice,bc7becdc872a7c88,bc7becdc872a7c88,false] 3720868 --- [nio-8082-exec-2] c.s.i.w.c.f.TraceLoadBalancerFeignClient : Before send
2020-05-03 13:24:15.798 DEBUG [orderservice,bc7becdc872a7c88,bc7becdc872a7c88,false] 3720868 --- [nio-8082-exec-2] o.s.c.s.i.w.c.f.LazyTracingFeignClient   : Sending a request via tracing feign client [org.springframework.cloud.sleuth.instrument.web.client.feign.TracingFeignClient@409526a3] and the delegate [feign.Client$Default@10d2a12a]
2020-05-03 13:24:15.798 DEBUG [orderservice,bc7becdc872a7c88,bc7becdc872a7c88,false] 3720868 --- [nio-8082-exec-2] o.s.c.s.i.w.c.feign.TracingFeignClient   : Handled send of NoopSpan{context=bc7becdc872a7c88/566cb51b1cb1811c}
2020-05-03 13:24:15.812 DEBUG [orderservice,bc7becdc872a7c88,bc7becdc872a7c88,false] 3720868 --- [nio-8082-exec-2] o.s.c.s.i.w.c.feign.TracingFeignClient   : Handled receive of NoopSpan{context=bc7becdc872a7c88/566cb51b1cb1811c}
2020-05-03 13:24:15.813 DEBUG [orderservice,bc7becdc872a7c88,bc7becdc872a7c88,false] 3720868 --- [nio-8082-exec-2] c.s.i.w.c.f.TraceLoadBalancerFeignClient : After receive
2020-05-03 13:24:15.813 DEBUG [orderservice,bc7becdc872a7c88,bc7becdc872a7c88,false] 3720868 --- [nio-8082-exec-2] o.s.w.c.HttpMessageConverterExtractor    : Reading [class me.sup2is.memberservice.Member] as "application/json;charset=UTF-8" using [org.springframework.http.converter.json.MappingJackson2HttpMessageConverter@75a4f014]
2020-05-03 13:24:15.815 DEBUG [orderservice,bc7becdc872a7c88,bc7becdc872a7c88,false] 3720868 --- [nio-8082-exec-2] m.m.a.RequestResponseBodyMethodProcessor : Written [sup2is님이 주문요청하셨습니다.] as "text/plain" using [org.springframework.http.converter.StringHttpMessageConverter@7cd01a27]
2020-05-03 13:24:15.815 DEBUG [orderservice,bc7becdc872a7c88,bc7becdc872a7c88,false] 3720868 --- [nio-8082-exec-2] o.s.web.servlet.DispatcherServlet        : Null ModelAndView returned to DispatcherServlet with name 'dispatcherServlet': assuming HandlerAdapter completed request handling
2020-05-03 13:24:15.816 DEBUG [orderservice,bc7becdc872a7c88,bc7becdc872a7c88,false] 3720868 --- [nio-8082-exec-2] o.s.web.servlet.DispatcherServlet        : Successfully completed request

```

로그에서 확인할 수 있듯이 **orderservice,bc7becdc872a7c88,bc7becdc872a7c88,false** 가 로그마다 출력되어 있는 것을 확인할 수 있다. 순서대로  {application-id},{추적 id},{스팬 id},{집킨에 추적 데이터 전송 여부} 이다.



MemberService도 확인해보자

```
2020-05-03 13:24:15.808 DEBUG [memberservice,bc7becdc872a7c88,566cb51b1cb1811c,false] 3719792 --- [io-8081-exec-10] o.s.web.servlet.DispatcherServlet        : DispatcherServlet with name 'dispatcherServlet' processing GET request for [/member/1]
2020-05-03 13:24:15.809 DEBUG [memberservice,bc7becdc872a7c88,566cb51b1cb1811c,false] 3719792 --- [io-8081-exec-10] s.w.s.m.m.a.RequestMappingHandlerMapping : Looking up handler method for path /member/1
2020-05-03 13:24:15.810 DEBUG [memberservice,bc7becdc872a7c88,566cb51b1cb1811c,false] 3719792 --- [io-8081-exec-10] s.w.s.m.m.a.RequestMappingHandlerMapping : Returning handler method [public me.sup2is.memberservice.Member me.sup2is.memberservice.MemberController.getMember(java.lang.Long)]
2020-05-03 13:24:15.810 DEBUG [memberservice,bc7becdc872a7c88,566cb51b1cb1811c,false] 3719792 --- [io-8081-exec-10] o.s.web.servlet.DispatcherServlet        : Last-Modified value for [/member/1] is: -1
2020-05-03 13:24:15.813 DEBUG [memberservice,bc7becdc872a7c88,566cb51b1cb1811c,false] 3719792 --- [io-8081-exec-10] m.m.a.RequestResponseBodyMethodProcessor : Written [Member(id=1, name=sup2is, password=qwer!23)] as "application/json" using [org.springframework.http.converter.json.MappingJackson2HttpMessageConverter@2e782682]
2020-05-03 13:24:15.813 DEBUG [memberservice,bc7becdc872a7c88,566cb51b1cb1811c,false] 3719792 --- [io-8081-exec-10] o.s.web.servlet.DispatcherServlet        : Null ModelAndView returned to DispatcherServlet with name 'dispatcherServlet': assuming HandlerAdapter completed request handling
2020-05-03 13:24:15.814 DEBUG [memberservice,bc7becdc872a7c88,566cb51b1cb1811c,false] 3719792 --- [io-8081-exec-10] o.s.web.servlet.DispatcherServlet        : Successfully completed request

```

마찬가지로 **memberservice,bc7becdc872a7c88,566cb51b1cb1811c,false** 의 형태로 출력이 된다.

<br>

```
| orderservice,bc7becdc872a7c88,bc7becdc872a7c88,false  |
| ----------------------------------------------------- |
| memberservice,bc7becdc872a7c88,566cb51b1cb1811c,false |
```



해당 상관관계 ID에서 맨 마지막 값인 false값은 이 트랜잭션을 zipkin에 전송하지 않는다는 뜻이다. 이 비율은 1/10 값이 기본 설정이다. 즉 10번요청시 1번의 비율로 zipkin에 해당 트랜잭션 정보를 전송한다는 뜻인데 다음 설정으로 비율을 조절 할 수 있다.

```properties
spring:
  sleuth:
    sampler:
      probability: 1.0
```

나는 테스트이기때문에 모든 요청을 zipkin에 전송하도록 1.0 값을 셋팅했다.

<br>

# Zipkin으로 분산 추적하기

위에서 설명한 상관관계 ID를 기반으로 트랜잭션의 흐름을 시각적으로 보여주는 대표적인 툴 중 하나인 zipkin을 소개한다. zipkin 모듈을 사용해서 직접 서버를 구성하는 방법도 있지만 executable jar 파일을 제공하기 때문에 예제에서는 이 jar파일을 사용하도록 하겠다. jar 파일은 [여기](https://search.maven.org/remote_content?g=io.zipkin&a=zipkin-server&v=LATEST&c=exec) 에서 다운받을 수 있다.



```
java -jar zipkin-server-2.21.1-exec.jar
```

이 명령어로 다운받은 jar파일을 실행시킨 뒤 127.0.0.1:9411로 접속하면 아래의 화면이 나올 것 이다.



![20200503_140155](https://user-images.githubusercontent.com/30790184/80899111-0ee05a00-8d47-11ea-86a0-564f644901f6.png)



2.21.x 버전으로 오면서 ui가 바뀐듯 싶다.. 구버전을 사용한다 하더라도 크게 다를 점은 없을 것 같다.

<br>

zipkin의 기본 port는 9411 포트이다. MemberService와 OrderService에는 알아서 9411 포트를 바라보게 되어 있지만 보안상의 이유 등으로 기본 포트나 주소를 변경하고 싶다면

```properties
spring:
  zipkin:
    base-url: http://localhost:9411
```

위의 설정으로 spring sleuth가 바라보는 zipkin의 엔드포인트를 설정할 수 있다.

<br>

# 간단하게 테스트해보기

그럼 postman으로 다시 /order 를 몇차례 요청해서 zipkin에 추적정보를 전송해보고 트랜잭션 정보를 확인해 보자. 

![20200503_140222](https://user-images.githubusercontent.com/30790184/80899112-10118700-8d47-11ea-85f2-09ce5086efd5.png)

아무것도 설정하지 않고 우측상단의 검색버튼을 누르면 service 이름에 관계 없이 들어온 모든 트랜잭션 정보를 확인할 수 있다.

<br>

![20200503_140231](https://user-images.githubusercontent.com/30790184/80899113-10aa1d80-8d47-11ea-8455-6796604b1595.png)

원하는 service만 검색해서 보고싶다면 selectbox를 통해서 검색하면 된다. application id 뿐만아니라 span id 등의 다양한 검색 조건이 가능하다.

<br>

![20200503_140253](https://user-images.githubusercontent.com/30790184/80899115-10aa1d80-8d47-11ea-8562-69c070c5afcd.png)

자세하게 보고싶은 트랜잭션 정보가 있다면 상세보기를 통해서 이 트랜잭션이 어디서부터 어디까지 이루어지고 있는지를 확인할 수 있다. 또 요청시간, 응답시간등을 체크해줘서 어떤 서비스에서 성능저하가 발생하는지 또한 체크가 가능하다.

<br>

포스팅은 여기까지 하겠습니다.  모든예제는 제 github에서 확인하실 수 있습니다.

예제 : https://github.com/sup2is/spring-example/tree/master/spring-cloud-stream-example



<br>

**References**

-  Spring 마이크로서비스 코딩 공작소 -존 카넬 (길벗출판사)
