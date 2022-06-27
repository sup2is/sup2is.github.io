---
layout: post
title: "Mono와 Flux랑 친해지기 #2 with Kotlin"
tags: [Reactive, Mono, Flux, Kotlin]
date: 2022-06-27
comments: true
---

<br>

# Overview

[이전글](https://sup2is.github.io/2022/06/26/reactive-with-kotlin-1.html)에 이어서 2편이다.



# 시퀀스 picking하기

시퀀스를 picking하는 메서드는 매우매우 많다. 이름이 굉장히 직관적이기 때문에 메서드명만보고 실제 기능이 어떻게 동작하는지 어느정도 유추해볼 수 있기 때문에 이런게 있구나~ 하고 넘어가고 실제로 필요할때 찾아서 사용하면 될 것 같다. 

아래 예제에서 `$it`로 받을 수 있는 함수들은 인자의 타입이 `Consumer`이고 그냥 print만 하는 메서드들은 전부 `Runnable` 타입의 인자를 받는다.

만약 내부에서 어떤일이 일어나는지 확인하고싶다면 `log()` 메서드를 사용하면 된다.

```kotlin
package me.sup2is.reactiveexam.ch03

import me.sup2is.reactiveexam.Person
import reactor.core.publisher.Flux

fun main(args: Array<String>) {

    Flux.just(
        Person(name = "choi", age = 29),
        Person(name = "woo", age = 31)
    ).doOnNext { println("doOnNext: $it") }
        .doOnComplete { println("doOnComplete!") }
        .doOnError {
            println("doOnError: $it")
        }
        .doOnCancel { println("doOnCancel!") }
        .doFirst { println("doFirst!") }
        .doOnSubscribe { println("doOnSubscribe: $it") }
        .doOnRequest { println("doOnRequest: $it") }
        .doOnTerminate() { println("doOnTerminate!") }
        .doAfterTerminate { println("doAfterTerminate!") }
        .doOnEach { println("doOnEach: $it") }
        .doFinally { println("doFinally: $it") }
        .subscribe { println("subscribe: $it") }
}

// console

doFirst!
doOnSubscribe: reactor.core.publisher.FluxPeekFuseable$PeekFuseableSubscriber@371a67ec
doOnRequest: 9223372036854775807
doOnNext: Person(name=choi, age=29)
doOnEach: doOnEach_onNext(Person(name=choi, age=29))
subscribe: Person(name=choi, age=29)
doOnNext: Person(name=woo, age=31)
doOnEach: doOnEach_onNext(Person(name=woo, age=31))
subscribe: Person(name=woo, age=31)
doOnComplete!
doOnTerminate!
doOnEach: onComplete()
doFinally: onComplete
doAfterTerminate!
```



# Sequence 필터링하기

## filter(), filterWhen(), ofType(), distinct(), distinctUntilChanged()

`filter()`는 자바의 스트림이랑 매우 비슷하다. 주어진 `Predicate`에 의해 필터링된 요소들을 반환한다. `filterWhen()` 은 `Publisher`로 감싸서 필터링한다.

`ofType()` 은 메서드명 그대로 타입별로 필터링한다. `distinct()` 역시 중복을 제거한다.

`distinctUntilChanged()`도 중복을 제거하긴 하지만 연속된 중복 요소만 제거한다.

```kotlin
package me.sup2is.reactiveexam.ch04

import me.sup2is.reactiveexam.Person
import reactor.core.publisher.Flux
import reactor.core.publisher.Mono

fun main(args: Array<String>) {

    val group1 = Flux.just(
        Person(name = "choi", age = 29),
        Person(name = "woo", age = 31),
        Person(name = "yoon", age = 30),
        Person(name = "hwang", age = 30)
    )

    println("\n### Flux.filter()\n")

    group1.filter {
        it.age >= 30
    }.log()
        .subscribe { println("subscribe: $it") }

    println("\n### Flux.filterWhen()\n")

    group1.filterWhen {
        Mono.just(it.age >= 30)
    }.log()
        .subscribe { println("subscribe: $it") }

    println("\n### Flux.ofType()\n")

    group1.ofType(
        Person::class.java
    )
        .log()
        .subscribe { println("subscribe: $it") }

    println("\n### Flux.distinct()\n")

    Flux.just(
        1, 1, 2, 3, 4, 5, 6, 6, 6, 7, 8, 9, 9
    )
        .log()
        .distinct()
        .collectList()
        .subscribe { println("subscribe: $it") }

    println("\n### Flux.distinctUntilChanged()\n")

    Flux.just(
        1, 2, 1, 2, 2, 2, 3, 1, 2
    )
        .log()
        .distinctUntilChanged()
        .collectList()
        .subscribe { println("subscribe: $it") }
}

// console

### Flux.filter()

[ INFO] (main) | onSubscribe([Fuseable] FluxFilterFuseable.FilterFuseableSubscriber)
[ INFO] (main) | request(unbounded)
[ INFO] (main) | onNext(Person(name=woo, age=31))
subscribe: Person(name=woo, age=31)
[ INFO] (main) | onNext(Person(name=yoon, age=30))
subscribe: Person(name=yoon, age=30)
[ INFO] (main) | onNext(Person(name=hwang, age=30))
subscribe: Person(name=hwang, age=30)
[ INFO] (main) | onComplete()

### Flux.filterWhen()

[ INFO] (main) onSubscribe(FluxFilterWhen.FluxFilterWhenSubscriber)
[ INFO] (main) request(unbounded)
[ INFO] (main) onNext(Person(name=woo, age=31))
subscribe: Person(name=woo, age=31)
[ INFO] (main) onNext(Person(name=yoon, age=30))
subscribe: Person(name=yoon, age=30)
[ INFO] (main) onNext(Person(name=hwang, age=30))
subscribe: Person(name=hwang, age=30)
[ INFO] (main) onComplete()

### Flux.ofType()

[ INFO] (main) | onSubscribe([Fuseable] FluxMapFuseable.MapFuseableSubscriber)
[ INFO] (main) | request(unbounded)
[ INFO] (main) | onNext(Person(name=choi, age=29))
subscribe: Person(name=choi, age=29)
[ INFO] (main) | onNext(Person(name=woo, age=31))
subscribe: Person(name=woo, age=31)
[ INFO] (main) | onNext(Person(name=yoon, age=30))
subscribe: Person(name=yoon, age=30)
[ INFO] (main) | onNext(Person(name=hwang, age=30))
subscribe: Person(name=hwang, age=30)
[ INFO] (main) | onComplete()

### Flux.distinct()

[ INFO] (main) | onSubscribe([Synchronous Fuseable] FluxArray.ArrayConditionalSubscription)
[ INFO] (main) | request(unbounded)
[ INFO] (main) | onNext(1)
[ INFO] (main) | onNext(1)
[ INFO] (main) | request(1)
[ INFO] (main) | onNext(2)
[ INFO] (main) | onNext(3)
[ INFO] (main) | onNext(4)
[ INFO] (main) | onNext(5)
[ INFO] (main) | onNext(6)
[ INFO] (main) | onNext(6)
[ INFO] (main) | request(1)
[ INFO] (main) | onNext(6)
[ INFO] (main) | request(1)
[ INFO] (main) | onNext(7)
[ INFO] (main) | onNext(8)
[ INFO] (main) | onNext(9)
[ INFO] (main) | onNext(9)
[ INFO] (main) | request(1)
[ INFO] (main) | onComplete()
subscribe: [1, 2, 3, 4, 5, 6, 7, 8, 9]

### Flux.distinctUntilChanged()

[ INFO] (main) | onSubscribe([Synchronous Fuseable] FluxArray.ArrayConditionalSubscription)
[ INFO] (main) | request(unbounded)
[ INFO] (main) | onNext(1)
[ INFO] (main) | onNext(2)
[ INFO] (main) | onNext(1)
[ INFO] (main) | onNext(2)
[ INFO] (main) | onNext(2)
[ INFO] (main) | request(1)
[ INFO] (main) | onNext(2)
[ INFO] (main) | request(1)
[ INFO] (main) | onNext(3)
[ INFO] (main) | onNext(1)
[ INFO] (main) | onNext(2)
[ INFO] (main) | onComplete()
subscribe: [1, 2, 1, 2, 3, 1, 2]
```



## take(), next(), takeLast(), takeUntil(), takeWhile()

`take()` 는 시퀀스에서 n개의 요소만 가져온다. 인자로 `Duration` 을 넘겨주면 텀을둬서 가져올 수 있다.

`next()`는 `Flux`에서 방출된 첫 번째 항목만 `Mono` 로 반환해준다. 비어있는 `Flux`라면 empty `Mono`를 리턴한다.

`takeUntil()` 은 주어진 `Predicate` 가 true가 될 때 까지 요소를 릴레이한다. 최초에 true가 된 요소까지만 반환한다. `takeWhile()` 은 반대로 주어진 `Predicate`가 true일 때만 요소를 릴레이한다.

```kotlin
package me.sup2is.reactiveexam.ch04

import me.sup2is.reactiveexam.Person
import reactor.core.publisher.Flux
import java.time.Duration

fun main(args: Array<String>) {

    val group1 = Flux.just(
        Person(name = "choi", age = 29),
        Person(name = "woo", age = 31),
        Person(name = "yoon", age = 30),
        Person(name = "hwang", age = 30)
    )

    println("\n### Flux.take()\n")

    group1.take(2)
        .log()
        .subscribe { println("subscribe: $it") }

    println("\n### Flux.take()\n")

    group1.take(Duration.ofSeconds(1))
        .log()
        .subscribe { println("subscribe: $it") }

    println("\n### Flux.next()\n")

    group1.next()
        .log()
        .subscribe { println("subscribe: $it") }

    println("\n### Flux.takeLast()\n")

    group1.takeLast(2)
        .log()
        .subscribe { println("subscribe: $it") }

    println("\n### Flux.takeUntil()\n")

    group1.takeUntil {
        it.age == 30
    }
        .log()
        .collectList()
        .subscribe { println("subscribe: $it") }

    println("\n### Flux.takeWhile()\n")

    group1.takeWhile {
        it.age <= 30
    }
        .log()
        .collectList()
        .subscribe { println("subscribe: $it") }
}

// console

### Flux.take()

[ INFO] (main) | onSubscribe([Fuseable] FluxTake.TakeFuseableSubscriber)
[ INFO] (main) | request(unbounded)
[ INFO] (main) | onNext(Person(name=choi, age=29))
subscribe: Person(name=choi, age=29)
[ INFO] (main) | onNext(Person(name=woo, age=31))
subscribe: Person(name=woo, age=31)
[ INFO] (main) | onComplete()

### Flux.take()

[ INFO] (main) onSubscribe(SerializedSubscriber)
[ INFO] (main) request(unbounded)
[ INFO] (main) onNext(Person(name=choi, age=29))
subscribe: Person(name=choi, age=29)
[ INFO] (main) onNext(Person(name=woo, age=31))
subscribe: Person(name=woo, age=31)
[ INFO] (main) onNext(Person(name=yoon, age=30))
subscribe: Person(name=yoon, age=30)
[ INFO] (main) onNext(Person(name=hwang, age=30))
subscribe: Person(name=hwang, age=30)
[ INFO] (main) onComplete()

### Flux.next()

[ INFO] (main) onSubscribe(MonoNext.NextSubscriber)
[ INFO] (main) request(unbounded)
[ INFO] (main) onNext(Person(name=choi, age=29))
subscribe: Person(name=choi, age=29)
[ INFO] (main) onComplete()

### Flux.takeLast()

[ INFO] (main) onSubscribe(FluxTakeLast.TakeLastManySubscriber)
[ INFO] (main) request(unbounded)
[ INFO] (main) onNext(Person(name=yoon, age=30))
subscribe: Person(name=yoon, age=30)
[ INFO] (main) onNext(Person(name=hwang, age=30))
subscribe: Person(name=hwang, age=30)
[ INFO] (main) onComplete()

### Flux.takeUntil()

[ INFO] (main) onSubscribe(FluxTakeUntil.TakeUntilPredicateSubscriber)
[ INFO] (main) request(unbounded)
[ INFO] (main) onNext(Person(name=choi, age=29))
[ INFO] (main) onNext(Person(name=woo, age=31))
[ INFO] (main) onNext(Person(name=yoon, age=30))
[ INFO] (main) onComplete()
subscribe: [Person(name=choi, age=29), Person(name=woo, age=31), Person(name=yoon, age=30)]

### Flux.takeWhile()

[ INFO] (main) onSubscribe(FluxTakeWhile.TakeWhileSubscriber)
[ INFO] (main) request(unbounded)
[ INFO] (main) onNext(Person(name=choi, age=29))
[ INFO] (main) onComplete()
subscribe: [Person(name=choi, age=29)]
```

## elementAt(), last(), skip(), skipLast(), skipUntil(), skipWhile()

`elementAt()` 은 예상대로 특정 위치에 있는 요소를 내보내고 만약 길이가 짧다면 `IndexOutOfBoundException`을 내보낸다.

`last()` 도 예상대로 마지막에 있는 요소를 반환한다. 

`skip()` 은 특정 요소들을 주어진 n의 크기만큼 스킵한다. `skipLast()` 는 마지막 요소부터 n개만큼 스킵한다. `skipUntil()` 은 주어진 `Predicate`가 true가 될 때까지 요소들을 건너뛴다. `skipWhile()`은 주어진 `Predicate`가 true를 반환할 때까지만 요소를 반환하고 이후 요소들은 스킵한다.

```kotlin
package me.sup2is.reactiveexam.ch04

import me.sup2is.reactiveexam.Person
import reactor.core.publisher.Flux

fun main(args: Array<String>) {

    val group1 = Flux.just(
        Person(name = "choi", age = 29),
        Person(name = "woo", age = 31),
        Person(name = "yoon", age = 30),
        Person(name = "hwang", age = 30)
    )

    println("\n### Flux.elementAt()\n")

    group1.elementAt(2)
        .log()
        .subscribe { println("subscribe: $it") }

    println("\n### Flux.last()\n")

    group1.last()
        .log()
        .subscribe { println("subscribe: $it") }

    println("\n### Flux.skip()\n")

    group1.skip(2)
        .log()
        .subscribe { println("subscribe: $it") }

    println("\n### Flux.skipLast()\n")

    group1.skipLast(1)
        .log()
        .subscribe { println("subscribe: $it") }

    println("\n### Flux.skipUntil()\n")

    group1.skipUntil {
        it.age == 30
    }
        .log()
        .subscribe { println("subscribe: $it") }

    println("\n### Flux.skipWhile()\n")

    group1.skipWhile {
        it.age != 30
    }
        .log()
        .subscribe { println("subscribe: $it") }
}

// console

### Flux.elementAt()

[ INFO] (main) | onSubscribe([Fuseable] MonoElementAt.ElementAtSubscriber)
[ INFO] (main) | request(unbounded)
[ INFO] (main) | onNext(Person(name=yoon, age=30))
subscribe: Person(name=yoon, age=30)
[ INFO] (main) | onComplete()

### Flux.last()

[ INFO] (main) | onSubscribe([Fuseable] MonoTakeLastOne.TakeLastOneSubscriber)
[ INFO] (main) | request(unbounded)
[ INFO] (main) | onNext(Person(name=hwang, age=30))
subscribe: Person(name=hwang, age=30)
[ INFO] (main) | onComplete()

### Flux.skip()

[ INFO] (main) onSubscribe(FluxSkip.SkipSubscriber)
[ INFO] (main) request(unbounded)
[ INFO] (main) onNext(Person(name=yoon, age=30))
subscribe: Person(name=yoon, age=30)
[ INFO] (main) onNext(Person(name=hwang, age=30))
subscribe: Person(name=hwang, age=30)
[ INFO] (main) onComplete()

### Flux.skipLast()

[ INFO] (main) onSubscribe(FluxSkipLast.SkipLastSubscriber)
[ INFO] (main) request(unbounded)
[ INFO] (main) onNext(Person(name=choi, age=29))
subscribe: Person(name=choi, age=29)
[ INFO] (main) onNext(Person(name=woo, age=31))
subscribe: Person(name=woo, age=31)
[ INFO] (main) onNext(Person(name=yoon, age=30))
subscribe: Person(name=yoon, age=30)
[ INFO] (main) onComplete()

### Flux.skipUntil()

[ INFO] (main) onSubscribe(FluxSkipUntil.SkipUntilSubscriber)
[ INFO] (main) request(unbounded)
[ INFO] (main) onNext(Person(name=yoon, age=30))
subscribe: Person(name=yoon, age=30)
[ INFO] (main) onNext(Person(name=hwang, age=30))
subscribe: Person(name=hwang, age=30)
[ INFO] (main) onComplete()

### Flux.skipWhile()

[ INFO] (main) onSubscribe(FluxSkipWhile.SkipWhileSubscriber)
[ INFO] (main) request(unbounded)
[ INFO] (main) onNext(Person(name=yoon, age=30))
subscribe: Person(name=yoon, age=30)
[ INFO] (main) onNext(Person(name=hwang, age=30))
subscribe: Person(name=hwang, age=30)
[ INFO] (main) onComplete()
```





<br>

***

포스팅은 여기까지 하겠습니다. 감사합니다!

예제: [https://github.com/sup2is/study/tree/master/reactive/reactive-with-kotlin](https://github.com/sup2is/study/tree/master/reactive/reactive-with-kotlin)

<br>

- **[References]**

  - [https://godekdls.github.io/Reactor%20Core/appendixawhichoperatordoineed/](https://godekdls.github.io/Reactor%20Core/appendixawhichoperatordoineed/)
  - [https://projectreactor.io/docs/core/release/reference/index.html#which-operator](https://projectreactor.io/docs/core/release/reference/index.html#which-operator)
  - [https://www.woolha.com/tutorials/project-reactor-using-mono-never-and-flux-never-examples](https://www.woolha.com/tutorials/project-reactor-using-mono-never-and-flux-never-examples)
  - [https://www.baeldung.com/flux-sequences-reactor](https://www.baeldung.com/flux-sequences-reactor)

  
