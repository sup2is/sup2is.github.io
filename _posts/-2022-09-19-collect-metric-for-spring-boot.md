---
layout: post
title: "Spring Boot에서 Metric데이터 수집하기 Feat. Prometheus, Grafana, Micrometer"
tags: [Spring Boot, Monitoring, Prometheus, Grafana, Micrometer]
date: 2022-09-19
comments: true
---

<br>



# Table Of Contents



# Overview

이번시간에는 Spring Boot에서 Prometheus, Grafana, Micrometer를 활용한 Custom Metric을 수집하는 방법에 대해서 알아본다. Prometheus와 Grafana에 대한 자세한 설명은 생략한다.



# Prometheus, Grafana, Micrometer로 기본 Metric 수집하기

Spring을 사용한다면 Micrometer를 사용해서 손쉽게 메트릭을 수집할 수 있다. 이 Micrometer는 "메트릭의 SLF4J" 자체로 설명되는 프레임워크다.

Micrometer는 아래와 같이 다양한 모니터링 시스템의 퍼사드 역할을 해주고 있다.

- [AppOptics](https://docs.spring.io/spring-boot/docs/current/reference/html/actuator.html#actuator.metrics.export.appoptics)
- [Atlas](https://docs.spring.io/spring-boot/docs/current/reference/html/actuator.html#actuator.metrics.export.atlas)
- [Datadog](https://docs.spring.io/spring-boot/docs/current/reference/html/actuator.html#actuator.metrics.export.datadog)
- [Dynatrace](https://docs.spring.io/spring-boot/docs/current/reference/html/actuator.html#actuator.metrics.export.dynatrace)
- [Elastic](https://docs.spring.io/spring-boot/docs/current/reference/html/actuator.html#actuator.metrics.export.elastic)
- [Ganglia](https://docs.spring.io/spring-boot/docs/current/reference/html/actuator.html#actuator.metrics.export.ganglia)
- [Graphite](https://docs.spring.io/spring-boot/docs/current/reference/html/actuator.html#actuator.metrics.export.graphite)
- [Humio](https://docs.spring.io/spring-boot/docs/current/reference/html/actuator.html#actuator.metrics.export.humio)
- [Influx](https://docs.spring.io/spring-boot/docs/current/reference/html/actuator.html#actuator.metrics.export.influx)
- [JMX](https://docs.spring.io/spring-boot/docs/current/reference/html/actuator.html#actuator.metrics.export.jmx)
- [KairosDB](https://docs.spring.io/spring-boot/docs/current/reference/html/actuator.html#actuator.metrics.export.kairos)
- [New Relic](https://docs.spring.io/spring-boot/docs/current/reference/html/actuator.html#actuator.metrics.export.newrelic)
- [Prometheus](https://docs.spring.io/spring-boot/docs/current/reference/html/actuator.html#actuator.metrics.export.prometheus)
- [SignalFx](https://docs.spring.io/spring-boot/docs/current/reference/html/actuator.html#actuator.metrics.export.signalfx)
- [Simple (in-memory)](https://docs.spring.io/spring-boot/docs/current/reference/html/actuator.html#actuator.metrics.export.simple)
- [Stackdriver](https://docs.spring.io/spring-boot/docs/current/reference/html/actuator.html#actuator.metrics.export.stackdriver)
- [StatsD](https://docs.spring.io/spring-boot/docs/current/reference/html/actuator.html#actuator.metrics.export.statsd)
- [Wavefront](https://docs.spring.io/spring-boot/docs/current/reference/html/actuator.html#actuator.metrics.export.wavefront)

우리는 Prometheus를 사용할 것이므로 Prometheus의존성을 추가만 해주면 Prometheus가 수집해야될 데이터에 대한 모든 설정을 자동으로 수행시켜준다. + actuator 의존성까지 추가해준다.

```
implementation("io.micrometer:micrometer-registry-prometheus")
implementation("org.springframework.boot:spring-boot-starter-actuator")
```

의존성을 설정했으면 application.yaml에 아래와 같이 actuator 설정을 해준다. 다른 actuator 설정을 하고싶다면 [https://docs.spring.io/spring-boot/docs/2.1.7.RELEASE/reference/html/production-ready-endpoints.html](https://docs.spring.io/spring-boot/docs/2.1.7.RELEASE/reference/html/production-ready-endpoints.html) 를 참고하자.

```yaml
management:
  endpoints:
    web:
      exposure:
        include: prometheus
```

의존성 + application.yaml이 준비되었으면 서버를 올려서 [http://localhost:8080/actuator/prometheus](http://localhost:8080/actuator/prometheus) 에 접속하면 여러 메트릭정보들을 확인할 수 있다.















# 정리

프로젝트를 진행하면서 공통으로 Response를 처리해야할 일이 있다면 `ResponseBodyResultHandler`를 적용해보자.



<br>

***

포스팅은 여기까지 하겠습니다. 감사합니다!

예제: [https://github.com/sup2is/kotlin-reactive-board/blob/main/api/src/main/kotlin/me/sup2is/kotlinreactiveboard/api/config/ResponseWrapper.kt](https://github.com/sup2is/kotlin-reactive-board/blob/main/api/src/main/kotlin/me/sup2is/kotlinreactiveboard/api/config/ResponseWrapper.kt)

<br>

**[References]**

- [https://stackoverflow.com/questions/71084011/spring-webflux-add-a-wrapping-class-before-serialization](https://stackoverflow.com/questions/71084011/spring-webflux-add-a-wrapping-class-before-serialization)
- [https://dreamchaser3.tistory.com/12](https://dreamchaser3.tistory.com/12)
