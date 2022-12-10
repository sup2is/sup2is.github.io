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

우리는 Prometheus를 사용할 것이므로 Prometheus의존성과 spring-actuator 의존성을 추가만 해주면 Prometheus가 수집해야될 데이터에 대한 모든 설정을 자동으로 수행시켜준다. 

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

프로메테우스 의존성 + application.yaml이 준비되었으면 서버를 올려서 [http://localhost:8080/actuator/prometheus](http://localhost:8080/actuator/prometheus) 을 확인해보자. 아래와 같은 여러 메트릭정보들을 확인할 수 있다.

```
# HELP mongodb_driver_pool_size the current size of the connection pool, including idle and and in-use members
# TYPE mongodb_driver_pool_size gauge
mongodb_driver_pool_size{cluster_id="632907bc83262d21da4760dd",server_address="localhost:27017",} 0.0
# HELP system_load_average_1m The sum of the number of runnable entities queued to available processors and the number of runnable entities running on the available processors averaged over a period of time
# TYPE system_load_average_1m gauge
system_load_average_1m 2.3896484375
# HELP jvm_threads_live_threads The current number of live threads including both daemon and non-daemon threads
# TYPE jvm_threads_live_threads gauge
jvm_threads_live_threads 22.0
# HELP process_start_time_seconds Start time of the process since unix epoch.
# TYPE process_start_time_seconds gauge
process_start_time_seconds 1.663633304967E9
# HELP executor_completed_tasks_total The approximate total number of tasks that have completed execution
# TYPE executor_completed_tasks_total counter
executor_completed_tasks_total{name="applicationTaskExecutor",} 0.0
# HELP jvm_gc_memory_promoted_bytes_total Count of positive increases in the size of the old generation memory pool before GC to after GC
# TYPE jvm_gc_memory_promoted_bytes_total counter
jvm_gc_memory_promoted_bytes_total 8693136.0
# HELP logback_events_total Number of error level events that made it to the logs
# TYPE logback_events_total counter
logback_events_total{level="warn",} 0.0
logback_events_total{level="debug",} 0.0
logback_events_total{level="error",} 0.0
logback_events_total{level="trace",} 0.0
logback_events_total{level="info",} 7.0

...
```

이런 기본 설정은 `PrometheusMetricsExportAutoConfiguration` 에서 자동으로 설정값을 설정해준다. 

Intellij에서 `MetricsAutoConfiguration`으로 검색하면 수많은 클래스들을 볼 수 있는데 우리가 classpath에 특정 라이브러리만 import 하더라도 자동으로 metric 정보를 수집할 수 있게 도와준다.



사진

이제 metric 정보들은 준비가 되었으니 prometheus와 grafana를 올려서 테스트해보자.

```
// grafana

docker run -d -p 3000:3000 grafana/grafana-enterprise

// prometheus

docker run \
    -p 9090:9090 \
    -v /path/to/prometheus.yml:/etc/prometheus/prometheus.yml \
    prom/prometheus


// prometheus.yaml

global:
  scrape_interval: 10s # 10초 마다 Metric을 Pulling
  evaluation_interval: 10s
scrape_configs:
  - job_name: 'spring-boot-app'
    metrics_path: '/actuator/prometheus' # Application prometheus endpoint
    static_configs:
      - targets: ['host.docker.internal:8080'] # Application host:port

```



















# DefaultWebFluxTagsProvider를 사용해서 대시보드 구성하기













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
