---
layout: post
title: "Spring Boot에서 Redis로 Session Clustering하기"
tags: [Spring Boot, NoSQL, Redis, Lettuce, Session Clustering]
date: 2020-07-15
comments: true
---



<br>

# OverView

오늘은 Redis를 활용해서 Session Clustering을 구성하는 방법에 대해서 알아보도록 하겠다.

# Session Clustering

Session Clustering은 다중화된 WAS 환경에서 Session을 통합해주는 기능이다. 만약 다음과 같은 환경이 있다고 생각해보자.

우리의 서비스가 많이 성장해서 더이상 한개의 WAS로 사용자의 요청을 받아들일 수 없다고 판단했다면 WAS를 스케일 아웃하는 방식을 사용할 수 있다. 아래와 같은 그림이다.

![37e2f93-web_sessions-small-copy](https://user-images.githubusercontent.com/30790184/87507328-31c4ae00-c6a8-11ea-936e-74cff22d6680.png)



위와 같은 환경에서 사용자의 로그인 요청을 APP SERVER 3 으로 했다고 가정하면 APP SERVER 3 서버에는 사용자의 로그인 정보를 갖는 세션갖게 된다.



![dae81d5-web_sessions_failed_instance-small](https://user-images.githubusercontent.com/30790184/87507323-30938100-c6a8-11ea-9c7a-8919a12d502a.png)



그런데 만약 사용자의 요청이 로드 밸런서에 의해 APP SERVER 1로 간다면 어떻게 될까? 권한이 없는 세션값으로 접근했기 때문에 사용자는 영문도 모른채 다시 로그인을 해야 할 것이다.

이럴때 사용하는게 바로 **Session Clustering**이다. Spring Boot는 Key Value 스토어인 Redis를 통해서 아주 간편하게 Session Clustering을 구현할 수 있게 해준다.

# Spring Boot + Redis로 Session Clustering 구성하기

## Redis 준비하기

먼저 Redis 서버가 필요하다. redis가 없다면 [https://redis.io/download](https://redis.io/download) 에서 다운받을 수 있다. 만약 Docker를 사용할 수 있는 환경이라면 [https://hub.docker.com/\_/redis/](https://hub.docker.com\_/redis/) 에서 이미지 관련된 정보를 얻을 수 있다. 개인적으로 Docker를 사용하는 방법을 추천한다.



## Spring Boot 서버 만들기

먼저 Redis를 사용하기 위해 필요한 라이브러리를 pom.xml에 세팅해준다.

**pom.xml**

```xml
...
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.session</groupId>
            <artifactId>spring-session-data-redis</artifactId>
        </dependency>
    </dependencies>
...

```

web application으로 사용할 것이기 때문에 **spring-boot-starter-web**을 추가해주고 redis session clustering 관련 라이브러리인 **spring-session-data-redis**을 추가해준다. redis client는 lettuce를 사용할것이고 lettuce모듈을 별도로 추가하지 않아도 **spring-boot-starter-data-redis**을 통해 Lettuce 관련 모듈을 사용할 수 있다.

<br>

이어서 Application 클래스를 작성해서 Redis와 Spring Boot를 연결해주는 작업을 해보도록 하자

**Application.java**

```java
package me.sup2is;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.session.data.redis.config.annotation.web.http.EnableRedisHttpSession;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.ResponseBody;

import javax.servlet.http.HttpSession;

@Controller
@SpringBootApplication
@EnableRedisHttpSession
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

    @GetMapping("/")
    @ResponseBody
    public String index(HttpSession session) {
        session.setAttribute("name", "sup2is");
        return session.getId() + "\nHello " + session.getAttribute("name");
    }
}

```

매우 간단하게 **@EnableRedisHttpSession**을 추가해주는 것 만으로도 Redis Session Cluster 환경을 구성할 수 있다.

사용자가 웹 페이지에 접속하면 Session에 'sup2is' 라는 문자열을 넣고 화면에는 Session ID를 뿌려주도록 구성했다.

실제 Redis Server 설정은 application.yml에서 이루어진다.

**application.yml**

```yml
spring:
  redis:
    host: 192.168.56.107
    port: 6379
```

이렇게 설정하면 Spring Boot가 알아서 Redis Connection을 얻어온다. 이제 실제로 테스트를 한번 해보자.

# 3개의 WAS, 1개의 Session

WAS는 총 3개를 준비할 예정이다. 각각 포트는 **8080, 8081, 8082**로 구성할 것이고 세션의 공유 여부만 확인할 것이므로 별도의 로드밸런서는 사용하지 않도록 하겠다. 다음 명령어로 3개의 쉘에 서버를 각각 띄워보자

```
mvn spring-boot:run -Dspring-boot.run.arguments=--server.port={port}
```

알맞게 서버들을 구성했다면 8080 포트로 접속해서 세션을 확인해보도록 하자

![20200715_140213](https://user-images.githubusercontent.com/30790184/87506867-510f0b80-c6a7-11ea-85c4-e7907fbf3df4.png)

현재 세션의 id는 60322로 시작한다. 이제 이어서 8081과 8082도 접속해서 session id를 확인해보도록 하자

![20200715_140245](https://user-images.githubusercontent.com/30790184/87506868-51a7a200-c6a7-11ea-8981-f7fe159603f8.png)

![20200715_140447](https://user-images.githubusercontent.com/30790184/87506869-51a7a200-c6a7-11ea-9185-a29c7e6fdbfc.png)

다른 WAS이지만 항상 같은 세션 ID를 반환하는 것을 확인할 수 있다.

## Redis 내부 확인하기

그렇다면 이 세션 ID는 어디에 어떻게 위치하는지 Redis Cli를 통해서 확인해보도록 하겠다.

Redis Cli를 통해서 `keys *` 명령어를 통해 내부의 Key값들을 확인해보면 위에서 확인했던 60322로 시작하는 세션 ID를 포함한 아래와 같은 key들이 존재할 것이다.

![20200715_140540](https://user-images.githubusercontent.com/30790184/87506873-52403880-c6a7-11ea-97ca-7c3a6a9d9789.png)

이중에서 **spring:session:sessions:{session ID}** 라는 key로 저장된 데이터의 타입은 hash라는 자료구조를 사용하는데 `hgetall` 명령어를 통해서 내부에 key-value를 확인할 수 있다.

![20200715_140633](https://user-images.githubusercontent.com/30790184/87506864-50767500-c6a7-11ea-83a2-aa05f9a63e35.png)

createTime, lastAccessedTime 그리고 이전에 저장한 name 이라는 속성까지 Redis 서버 내부에 저장되어있는 것을 확인할 수 있다.

# 마무리

Redis의 Session Clustering은 많이 사용되는 기능이므로 개념정도만 알고가도 이후에 적용하는건 크게 어려울 것 같지는 않다. 그리고 애너테이션 하나로 이런 환경을 구성할 수 있게 해주는 Spring Boot가 놀랍다....



<hr>
포스팅은 여기까지 하겠습니다. 퍼가실때는 출처를 반드시 남겨주세요!


예제 : [https://github.com/sup2is/study/tree/master/db/redis/session-clustering-with-redis](https://github.com/sup2is/study/tree/master/db/redis/session-clustering-with-redis)



**References**

- [https://apacheignite-mix.readme.io/docs/web-session-clustering](https://apacheignite-mix.readme.io/docs/web-session-clustering)
- [https://docs.spring.io/spring-session/docs/1.0.x/reference/html5/guides/rest.html](https://docs.spring.io/spring-session/docs/1.0.x/reference/html5/guides/rest.html)

