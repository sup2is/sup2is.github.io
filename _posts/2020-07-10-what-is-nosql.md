---
layout: post
title: "NoSQL 개념 알아보기"
tags: [DataBase, NoSQL]
date: 2020-07-10
comments: true
---



<br>



# OverView

요즘 엔터프라이즈급 회사에서의 최대 관심사는 어떻게 사용자들에게 조금 더 빠른 환경을 제공하냐가 관건이다. 그러나 기존에 사용하는 데이터베이스 솔루션인 RDBMS는 대용량 트래픽, 빅데이터 관점에서 봤을때 더이상 적합하지 않다.

오늘은 대용량 트래픽, 빅데이터에 사용하기 적합한 데이터베이스 솔루션인 NoSQL에 대해서 알아보도록 하겠다

# NoSQL이란 ?

NoSQL이라는 개념은 2000년대 초반에 등장하기 시작했다. 수많은 데이터들이 기하급수적으로 늘어남에 따라 기존 RDB의 저장 및 관리 기술만으로 감당하기에는 불가능했기 때문이다. 먼저 NoSQL과 기존 RDBMS를 비교했을때 어떠한 장단점이 있는지 확인해보도록 하겠다.



## NoSQL vs RDBMS

**1.NoSQL은 클라우드 환경에 적합하다.**

Oracle 같은 RDBMS는 라이센스 정책을 서버의 개수만큼 측정하기 때문에 스케일 아웃보다는 스케일 업에 적합하다. 하지만 스케일 업은 확장에 한계가 있고 비용이 높은 편이다. 그렇다고 스케일 아웃을 하기엔 라이센스 비용이 부담될 수 있다.

그러나 NoSQL은 보통 오픈소스로 되어 있고 Community 또는 Commercial을 동시에 지원하는 듀얼 라이센스 정책을 사용한다. 때문에 오픈소스를 사용한다고하면 비용부담이 적은 스케일 아웃 방식을 사용할 수 있기 때문에 클라우드 환경에서 적합하다.

**2.NoSQL도 충분한 관계 데이터 모델을 표현할 수 있다.**

NoSQL은 비정형 데이터를 저장하기 때문에 기존 RDBMS에서 사용하는 관계 데이터 모델에 대한 제한이 있을것이라는 오해가 있을 수 있는데 NoSQL도 다양한 데이터모델을 사용해서 RDBMS에서 사용하는 관계형 데이터모델 표현이 가능하다.

**3.NoSQL은 빠르다**

기존 RDBMS에 비해 NoSQL은 빠르게 동작한다. 예를 들어 NoSQL의 여러 솔루션중 하나인 Redis 같은 경우는 In-Memory 기반 동작 방식을 사용한다. 때문에 Disk를 사용하는 방식보다 훨씬 빠르게 동작할 수 있다.



<br>

# NoSQLDatabase Type

RDBMS에서 오라클, MySQL 등등 모두 같은 관계형 데이터 저장 구조를 사용하는 반면에 NoSQL에서 MongoDB, Redis, Cassandra등등 의 솔루션들은 반드시 같은 데이터 저장 구조를 사용하지 않는다.

데이터 저장방식은 총 4가지로 Key-Value 방식, Document DB 방식, Wide-Column (Column Family) 방식, Graph DB 방식이 있다. 간단하게 알아보도록 하자.



## Key-Value 방식

Key-Value 방식은 각 항목이 키와 값을 포함하는 단순한 유형의 데이터베이스이다. 보통의 경우 1개의 키와 N개의 값으로 표현 가능하고 일반적으로 사용자의 랭킹 표시, 캐싱을 구현하기 위해 사용한다. 가장 대중적으로 쓰는 데이터베이스 솔루션은 Redis, DynonDB 등이 있다.

## Document DB 방식

Document DB 방식은 JSON 객체와 유사한 문서에 데이터를 저장한다. 각 데이터에는 필드 및 값 쌍이 포함되어있고 값은 다양한 데이터로 표현 가능하다. Document DB 방식의 장점은 일반적인 애플리케이션에서 사용하는 JSON 타입과 동일하기 때문에 쉬운 접근이 가능하다. 가장 대중적으로 쓰는 MongoDB는 Docmuent DB 타입 중 하나이다.

## Wide-Column (Column Family) 방식

Key-Value, Document DB 방식은 행(row)에 초점을 둔 반면 Wide-Column 방식은 열(Column)에 초점을 둔다. RDB에서 모든 행은 같은 열을 가져야하는 반면 Wide-Column에서는 모든 행이 같은 열을 가질 필요가 없기 때문에 기존 RDBMS 에서 데이터를 표현하는 방식보다 유연하다. 가장 대중적으로 쓰는 데이터베이스 솔루션은 Cassandra와 HBase가 있다.

## Graph DB 방식

Graph DB는 말그대로 Graph 이론에 토대를 뒀다. 데이터를 노드에 저장하고 이 노드들은 그래프처럼 연결될 수 있다. 가장 대중적으로 쓰는 데이터베이스 솔루션은 Neo4j가 있다.



# 마무리

어떤 NoSQL 데이터베이스 솔루션 사용할지 결정하기 위해서는 현재 회사에서 사용하는 데이터의 특성이나 사용처에 대한 충분한 검토가 필요하다. 예를 들어 현재 데이터들의 무결성 제약조건이 필요한지? 아니면 트랜잭션 제어가 필요한지? 등등에 대한 충분한 분석, 검토가 필요하다.

당분간은 .. NoSQL쪽을 많이 공부할 것 같다... 



<hr>
포스팅은 여기까지 하겠습니다. 퍼가실때는 출처를 반드시 남겨주세요!


**References**

- 빅데이터 저장 및 분석을 위한 NoSQL & Redis - 주종면 
- [https://www.mongodb.com/nosql-explained](https://www.mongodb.com/nosql-explained)
- [https://m.blog.naver.com/PostView.nhn?blogId=gkenq&logNo=10184332901&proxyReferer=https:%2F%2Fwww.google.com%2F](https://m.blog.naver.com/PostView.nhn?blogId=gkenq&logNo=10184332901&proxyReferer=https:%2F%2Fwww.google.com%2F)
