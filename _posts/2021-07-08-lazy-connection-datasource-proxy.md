---
layout: post
title: "LazyConnectionDataSourceProxy 알아보기"
tags: [Spring Boot, LazyConnectionDataSourceProxy]
date: 2021-07-08
comments: true
---

<br>



# Overview

이번시간에는 `LazyConnectionDataSourceProxy`에 대해 알아보고 `LazyConnectionDataSourceProxy`을 사용하는 것과 사용하지 않는 것을 비교해보는 간단한 예제를 작성해보도록 하겠다.

# LazyConnectionDataSourceProxy란?

Spring은 트랜잭션에 진입하는 순간 Database Connection을 가져올까? 라는 질문부터 시작한다. 

결론부터 말하면 Spring에서는 트랜잭션에 진입하는 순간 설정된 Datasource의 커넥션을 가져온다. 이 결론으로 도달할 수 있는 단점은 아래와같다.

- Ehcache같은 Cache를 사용하는 경우 실제 Database에 접근하지 않지만 불필요한 커넥션을 점유
- Hibernate의 영속성 컨텍스트 1차캐시(실제 Database에 접근하지 않음) 에도 불필요한 커넥션을 점유
- 외부 서비스(http, etc ...)에 접근해서 작업을 수행한 이후에 그 결과값을 Database에 Read/Write하는 경우 외부 서비스에 의존하는 시간만큼 불필요한 커넥션 점유 
- Multi Datasource 환경에서 트랜잭션에 진입한 이후 Datasource를 결정해야할때 이미 트랜잭션 진입시점에 Datasource가 결정되므로 분기가 불가능

위 단점들을 해결할 수 있는 방법이 바로 `LazyConnectionDataSourceProxy`이다.

`LazyConnectionDataSourceProxy`을 사용하면 실제로 커넥션이 필요한경우가 아니라면 데이터베이스 풀에서 커넥션을 점유하지 않고 실제로 필요한 시점에만 커넥션을 점유하게 할 수 있다.



먼저 실제 Spring 트랜잭션에 진입한 뒤 아무런 작업을 하지않아도 데이터베이스 커넥션을 점유하는지 알아보자.

```java
package me.sup2is.lazyconnectiondatasourceproxy.domain;

import com.zaxxer.hikari.HikariDataSource;
import com.zaxxer.hikari.HikariPoolMXBean;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import javax.sql.DataSource;

@Service
@RequiredArgsConstructor
public class MemberService {

    private final DataSource dataSource;
    private final MemberRepository repository;

    @Transactional
    public void create(Member member) {
        printConnectionStatus();
    }

    private void printConnectionStatus() {
        final HikariPoolMXBean hikariPoolMXBean = ((HikariDataSource) dataSource).getHikariPoolMXBean();
        System.out.println("################################");
        System.out.println("현재 active인 connection의 수 : " + hikariPoolMXBean.getActiveConnections());
        System.out.println("현재 idle인 connection의 수 : " + hikariPoolMXBean.getIdleConnections());
        System.out.println("################################");
    }

}

```

아무런 작업도 하지 않는 MemberService의 `create()` 메서드이다. `HikariPoolMXBean`을 사용해서 현재 데이터베이스 커넥션 상황을 모니터링할 수 있게해뒀다. 실제 실행시켜보면 결과는 아래와 같다.

```
################################
현재 active인 connection의 수 : 1
현재 idle인 connection의 수 : 9
################################
```

위에서 언급한대로 repository계층의 아무런 작업을 하지 않아도 트랜잭션에 진입하는 순간 커넥션을 점유하는 것을 확인할 수 있다.



# LazyConnectionDataSourceProxy 적용하기

아래와 같이 `LazyConnectionDataSourceProxy`을 적용해보고 다시 아무런 작업도 하지 않는 MemberService의 `create()` 메서드를 실행해보도록 하겠다.

```java
package me.sup2is.lazyconnectiondatasourceproxy.config;

import com.zaxxer.hikari.HikariDataSource;
import org.springframework.boot.autoconfigure.jdbc.DataSourceProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.jdbc.datasource.LazyConnectionDataSourceProxy;

import javax.sql.DataSource;

@Configuration
public class DataSourceConfig{

    @Bean
    public DataSource lazyDataSource(DataSourceProperties properties) {
        HikariDataSource dataSource = new HikariDataSource();
        dataSource.setJdbcUrl(properties.getUrl());
        dataSource.setUsername(properties.getUsername());
        dataSource.setPassword(properties.getPassword());
        dataSource.setDriverClassName(properties.getDriverClassName());
        return new LazyConnectionDataSourceProxy(dataSource);
    }
}
```

결과는 아래와 같다.

```
################################
현재 active인 connection의 수 : 0
현재 idle인 connection의 수 : 10
################################
```



이부분을 어떤식으로 처리하는지 자세히 살펴보고 싶어서 소스름 조금 뒤적뒤적했는데 결론적으로 `JpaTransactionManager`의 `doBegin()` 메서드를 실행하는 부분에서 커넥션을 맺는 과정이 있는데 일반 `HikariDataSource` 인스턴스는 `doBegin()` 시점에 커넥션을 실제로 가져오지만 `LazyConnectionDataSourceProxy` 은 Proxy 객체로 한번 감싼 인스턴스를 `doBegin()` 시점에 리턴시켜서 실제 커넥션을 맺지는 않고 프록시 객체를 반환하도록 되어있다.

`JpaTransactionManager.doBegin()`

![스크린샷 2021-07-08 오전 10 33 54](https://user-images.githubusercontent.com/30790184/124850548-c754fe00-dfdb-11eb-855d-1c18db9b30af.png)

`HikariDataSource.getConnection()` 에서 실제로 커넥션을 가져오는 모습
![스크린샷 2021-07-08 오전 10 36 31](https://user-images.githubusercontent.com/30790184/124850543-c58b3a80-dfdb-11eb-9466-5697c4cbab8e.png)

`LazyConnectionDataSourceProxy.getConnection()` 에서 프록시를 리턴하는 모습

![스크린샷 2021-07-08 오전 10 53 17](https://user-images.githubusercontent.com/30790184/124850547-c6bc6780-dfdb-11eb-8a62-819856403c07.png)




# 마무리

사실 `LazyConnectionDataSourceProxy` 존재 유무자체도 몰랐는데 코드리뷰때 팀원이 코드리뷰에서 언급해줘서 이번에 처음 알았다. 보통은 Master/Slave 환경에서 트랜잭션 진입시점에 커넥션을 결정하지 않도록 하는 용도로 많이 사용하는것 같지만 개인적으로 다른부분에서도 얻어올 수 있는 강점이 굉장히 많아서 `LazyConnectionDataSourceProxy`의 사용을 적극 검토해보는것도 좋을 것 같다.



<br>

***

포스팅은 여기까지 하겠습니다. 퍼가실때는 출처를 반드시 남겨주세요!

전체 예제 : [https://github.com/sup2is/study/tree/master/spring/lazy-connection-data-source-proxy](https://github.com/sup2is/study/tree/master/spring/lazy-connection-data-source-proxy)

<br>

**References**

- [https://kwonnam.pe.kr/wiki/springframework/lazyconnectiondatasourceproxy](https://kwonnam.pe.kr/wiki/springframework/lazyconnectiondatasourceproxy)
- [https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/jdbc/datasource/LazyConnectionDataSourceProxy.html](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/jdbc/datasource/LazyConnectionDataSourceProxy.html)
