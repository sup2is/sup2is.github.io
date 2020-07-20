---
layout: post
title: "Redis와 Cache"
tags: [NoSQL, Redis]
date: 2020-07-20
comments: true
---

<br>

# OverView

이번시간에는 Redis의 Cache 전략과 Look-aside Cache 패턴에 대해서 알아보도록 하겠다. 그 전에 먼저 파레토 법칙에 대해서 알아보도록 하자.

# 파레토 법칙

> **파레토 법칙**( - 法則, [영어](https://ko.wikipedia.org/wiki/영어): Pareto principle, law of the vital few, principle of factor sparsity)[[1\]](https://ko.wikipedia.org/wiki/파레토_법칙#cite_note-1)[[2\]](https://ko.wikipedia.org/wiki/파레토_법칙#cite_note-2) 또는 **80 대 20 법칙**([영어](https://ko.wikipedia.org/wiki/영어): 80–20 rule)은 '전체 결과의 80%가 전체 원인의 20%에서 일어나는 현상'을 가리킨다.[[3\]](https://ko.wikipedia.org/wiki/파레토_법칙#cite_note-NYT-3) 예를 들어, 20%의 고객이 백화점 전체 매출의 80%에 해당하는 만큼 쇼핑하는 현상을 설명할 때 이 용어를 사용한다. **2 대 8 법칙**라고도 한다.

캐싱이라는 용어에서 빠지지 않는 파레토 법칙에 대한 설명이다. 간단하게 **전체 결과의 80%가 전체 원인의 20%에서 일어나는 현상**을 파레토 법칙이라고 한다.

만약 어떤 서비스를 구축할때도 마찬가지로 분석이 잘 되어있는 환경에서 전체 요청의 80프로에 해당하는 20프로의 데이터를 예측할 수 있다면 사용자에게 더 빠르고 좋은 서비스를 만들 수 있을 것이다.

# Look-aside Cache Pattern

Look-aside Cache 패턴이란 다음과같다.

1. **사용자의 요청이 서비스로 들어온다.**
2. **데이터베이스에 데이터를 요청하기 전에 캐싱된 데이터가 있는지 확인한다.**
3. **캐싱된 데이터가 있으면 6번으로 간다.**
4. **캐싱된 데이터가 없으면 데이터베이스에서 데이터를 가져온다.**
5. **데이터베이스에서 가져온 데이터를 캐싱한다.**
6. **데이터를 사용자에게 보낸다.**

만약 **캐싱된 데이터와 원본 데이터의 무결성을 보장할 수 있다면** 이 패턴을 활용해서 위에서 설명한 파레토 법칙을 웹서비스에 적용시킬 수 있다.

그런데 급격하게 변화는 비지니스 환경에서 전체 요청의 80프로에 해당하는 20프로의 데이터를 유지하는것은 쉽지 않을 것이다. 오래된 Key를 삭제하는 Cache전략, 또는 적은 참조 회수를 갖는 Key를 삭제하는 Cache 전략 등등 Cache evict 하는 전략을 알맞게 설정해야 한다.

# Redis의 Cache Evict 전략

Redis는 다양한 Cache Evict 전략을 제공한다. Redis에서는 총 6가지를 제공하는데 다음과 같다.

- **noeviction**: evict하지 않음. 메모리가 가득찰 경우 에러를 발생시키고 별도로 메모리를 증설하는 명령어를 사용해야 한다.
- **allkeys-lru**: 새로운 데이터를 위한 공간을 확보하기 위해 최근에 덜 사용된 (LRU)키를 먼저 제거한다.
- **volatile-lru**: 새로 추가된 데이터를 위한 공간을 확보하기 위해 덜 사용된 (LRU)키를 먼저 제거하지만 `set expire` 를 통해 만료시간이 지정된 키만 제거한다.
- **allkeys-random**: 모든 키가 정리 대상이 된다.
- **volatile-random**: 모든 키가 정리 대상이 되지만 `set expire` 를 통해 만료시간이 지정된 키만 제거한다.
- **volatile-ttl**: `set expire` 를 통해 만료시간이 지정된 키만 제거하고 그중에서 가장 짧은 TTL 을 가진 키를 제거한다.

주의해야할 점은 **volatile** 이 붙은 전략들은 `set expire`가 적용된 키가 아니라면 제거할 대상을 찾지 못한다. 이 경우 **noeviction**과 같이 동작한다.

적절한 전략을 사용하는것이 애플리케이션 성능에 매우 중요하고 애플리케이션 런타임 도중에도 전략을 바꿀 수 있다.

Redis 4.0부터는 LFU 교체알고리즘도 지원한다.

- **volatile-lfu**: 새로 추가된 데이터를 위한 공간을 확보하기 위해 덜 사용된 (LFU키)를 먼저 제거하지만 `set expire` 를 통해 만료시간이 지정된 키만 제거한다.
- **allkeys-lfu**: 새로운 데이터를 위한 공간을 확보하기 위해 최근에 덜 사용된 (LFU)키를 먼저 제거한다.

# Local Cache와 Global Cache

Redis가 만병통치약은 아니다. 대용량 트래픽 시점에서 봤을때 Redis 서버도 병목의 대상이 될 수 있다. 이때는 Local Cache를 사용해서 문제를 해결할 수 있지만 다중화된 WAS 환경에서는 각각 Local Cache의 동기화처리에 대한 비용이 발생한다.

Local Cache와 Global Cache의 장단점을 구분해서 현재 서비스에 맞는 적절한 모델을 선택하는것이 중요하다.

# 마무리

파레토법칙부터 Local Cache와 Global Cache까지 아주 간단하게 알아봤다. 대용량 트래픽 서비스라면 Cache를 반드시 도입해야하는데 어떤 모델을 선택하는지에 대한 고민을 조금 더 해봐야겠다.





<hr>
포스팅은 여기까지 하겠습니다. 퍼가실때는 출처를 반드시 남겨주세요!


**References**

- [https://redis.io/topics/lru-cache](https://redis.io/topics/lru-cache)
- [https://www.youtube.com/watch?v=mPB2CZiAkKM](https://www.youtube.com/watch?v=mPB2CZiAkKM)

