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



## sample(), sampleFirst(), single(), singleOrEmpty()

만약 주기적으로 `Flux`가 요소를 방출하고 window 사이의 값들을 얻으려면 `sample()` 을 사용하면 된다. 인자로 넘긴 `Duration` 만큼 (ex: 1초라면 0~1초 사이) sampling을 한다. `sample()`은 window 구간 사이 가장 마지막 값을 가져오고 `sampleFirst()` 은 구간 사이 가장 첫번째 값을 가져온다.

`single()` 는 요소를 한 개만 가져오는 메서드인데 `Flux` 내에 요소가 두 개 이상이거나 요소가 없다면 각각 `IndexOutOfBoundsException`과 `NoSuchElementException`을 발생시킨다. 요소가 nullable 하다면 `singleOrEmpty()` 을 사용할 수 있다.

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

    println("\n### Flux.sample()\n")

    group1.sample(Duration.ofMillis(500))
        .log()
        .subscribe { println("subscribe: $it") }

    Thread.sleep(2000)

    println("\n### Flux.sampleFirst()\n")

    group1.sampleFirst(Duration.ofMillis(500))
        .log()
        .subscribe { println("subscribe: $it") }

    Thread.sleep(2000)

    println("\n### Flux.single()\n")

    group1.single()
        .log()
        .subscribe { println("subscribe: $it") }

    println("\n### Flux.singleOrEmpty()\n")

    group1.singleOrEmpty()
        .log()
        .subscribe { println("subscribe: $it") }
}

// console

### Flux.sample()

[ INFO] (main) onSubscribe(FluxSample.SampleMainSubscriber)
[ INFO] (main) request(unbounded)
[ INFO] (main) onNext(Person(name=hwang, age=30))
subscribe: Person(name=hwang, age=30)
[ INFO] (main) onComplete()

### Flux.sampleFirst()

[ INFO] (main) onSubscribe(FluxSampleFirst.SampleFirstMain)
[ INFO] (main) request(unbounded)
[ INFO] (main) onNext(Person(name=choi, age=29))
subscribe: Person(name=choi, age=29)
[ INFO] (main) onComplete()

### Flux.single()

[ INFO] (main) | onSubscribe([Synchronous Fuseable] Operators.ScalarSubscription)
[ INFO] (main) | request(unbounded)
[ INFO] (main) | onNext(1)
subscribe: 1
[ INFO] (main) | onComplete()

### Flux.singleOrEmpty()

[ INFO] (main) | onSubscribe([Synchronous Fuseable] Operators.ScalarSubscription)
[ INFO] (main) | request(unbounded)
[ INFO] (main) | onNext(1)
subscribe: 1
[ INFO] (main) | onComplete()
```

# 에러 핸들링하기

## error(), concat(), then(), timeout()

`error()` 는 인자로 넘긴 오류를 방출하는 시퀀스를 만들 수 있다. 또는 인자로 `Supplier` 타입으로 넘길 수도 있다.

`Flux`의 `concat()`에 `Flux#error()`를 넘기면 시퀀스가 성공적으로 완료하면 에러를 발생시키게 할 수 있다. `Mono`의 `then()` 도 마찬가지로 방출에 성공하면 에러를 발생시키게 할 수 있다.

시퀀스가 요소를 너무 늦게 준다면 `timeout()` 을 사용해서 에러를 발생시키게 할 수 있다.

```kotlin
package me.sup2is.reactiveexam.ch05

import me.sup2is.reactiveexam.Person
import reactor.core.publisher.Flux
import reactor.core.publisher.Mono
import java.time.Duration

fun main(args: Array<String>) {

    val group1 = Flux.just(
        Person(name = "choi", age = 29),
        Person(name = "woo", age = 31),
        Person(name = "yoon", age = 30),
        Person(name = "hwang", age = 30)
    )

    val choi = Mono.just(Person(name = "choi", age = 29))

    println("\n### Flux.error()\n")

    Flux.error<RuntimeException>(RuntimeException())
        .log()
        .doOnError { println("doOnError: $it") }
        .subscribe()

    Thread.sleep(100)

    println("\n### Flux.concat()\n")

    Flux.concat(
        Flux.error<RuntimeException>(RuntimeException())
    )
        .log()
        .doOnError { println("doOnError: $it") }
        .subscribe()

    Thread.sleep(100)

    println("\n### Mono.then()\n")

    choi.then(
        Mono.error<RuntimeException>(RuntimeException())
    )
        .log()
        .doOnError { println("doOnError: $it") }
        .subscribe()

    Thread.sleep(100)

    println("\n### Flux.timeout()\n")

    group1.timeout(Duration.ofSeconds(1))
        .log()
        .doOnError { println("doOnError: $it") }
        .subscribe()

    Thread.sleep(100)
}

// console

### Flux.error()

[ INFO] (main) onSubscribe([Fuseable] Operators.EmptySubscription)
[ INFO] (main) request(unbounded)
doOnError: java.lang.RuntimeException
[ERROR] (main) onError(java.lang.RuntimeException)
[ERROR] (main)  - java.lang.RuntimeException
java.lang.RuntimeException
	at me.sup2is.reactiveexam.ch05.ReactiveExam1Kt.main(ReactiveExam1.kt:21)
[ERROR] (main) Operator called default onErrorDropped - reactor.core.Exceptions$ErrorCallbackNotImplemented: java.lang.RuntimeException
reactor.core.Exceptions$ErrorCallbackNotImplemented: java.lang.RuntimeException
Caused by: java.lang.RuntimeException
	at me.sup2is.reactiveexam.ch05.ReactiveExam1Kt.main(ReactiveExam1.kt:21)

### Flux.concat()

[ INFO] (main) onSubscribe([Fuseable] Operators.EmptySubscription)
[ INFO] (main) request(unbounded)
doOnError: java.lang.RuntimeException
[ERROR] (main) onError(java.lang.RuntimeException)
[ERROR] (main)  - java.lang.RuntimeException
java.lang.RuntimeException
	at me.sup2is.reactiveexam.ch05.ReactiveExam1Kt.main(ReactiveExam1.kt:31)
[ERROR] (main) Operator called default onErrorDropped - reactor.core.Exceptions$ErrorCallbackNotImplemented: java.lang.RuntimeException
reactor.core.Exceptions$ErrorCallbackNotImplemented: java.lang.RuntimeException
Caused by: java.lang.RuntimeException
	at me.sup2is.reactiveexam.ch05.ReactiveExam1Kt.main(ReactiveExam1.kt:31)

### Mono.then()

[ INFO] (main) onSubscribe(MonoIgnoreThen.ThenIgnoreMain)
[ INFO] (main) request(unbounded)
doOnError: java.lang.RuntimeException
[ERROR] (main) onError(java.lang.RuntimeException)
[ERROR] (main)  - java.lang.RuntimeException
java.lang.RuntimeException
	at me.sup2is.reactiveexam.ch05.ReactiveExam1Kt.main(ReactiveExam1.kt:42)
[ERROR] (main) Operator called default onErrorDropped - reactor.core.Exceptions$ErrorCallbackNotImplemented: java.lang.RuntimeException
reactor.core.Exceptions$ErrorCallbackNotImplemented: java.lang.RuntimeException
Caused by: java.lang.RuntimeException
	at me.sup2is.reactiveexam.ch05.ReactiveExam1Kt.main(ReactiveExam1.kt:42)

### Flux.timeout()

[ INFO] (main) onSubscribe(SerializedSubscriber)
[ INFO] (main) request(unbounded)
[ INFO] (main) onNext(Person(name=choi, age=29))
[ INFO] (main) onNext(Person(name=woo, age=31))
[ INFO] (main) onNext(Person(name=yoon, age=30))
[ INFO] (main) onNext(Person(name=hwang, age=30))
[ INFO] (main) onComplete()
```

## onErrorReturn(), onErrorResume(), onErrorMap(), doFinally()

`onErrorReturn()`은 오류가 발생했을 때 지정된 fallback값을 내보내는 방법이다. 특정 에러타입을 지정할 수 있다. `onErrorResume()` 으로 시퀀스를 보낼 수도 있고 에러가 발생하면 이 시퀀스를 구독한다.

`onErrorMap()`은 이 시퀀스내부에서 발생한 모든 오류를 다른 오류타입으로 반환시키게 할 수 있다.

`doFinally()` 는 시퀀스가 성공 또는 종료, 취소되었을때 동작 트리거링을 할 수 있게 해준다. 종료뿐만 아니라 성공, 취소에도 똑같이 적용됨을 기억해야 한다.

```kotlin
package me.sup2is.reactiveexam.ch05

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

    println("\n### Mono.onErrorReturn()\n")

    group1.single()
        .onErrorReturn(
            Person(name = "error", age = 999)
        )
        .log()
        .doOnError { println("doOnError: $it") }
        .subscribe()

    Thread.sleep(100)

    println("\n### Mono.onErrorResume()\n")

    group1.single()
        .onErrorResume {
            Mono.just(Person(name = "error", age = 999))
        }
        .log()
        .doOnError { println("doOnError: $it") }
        .subscribe()

    Thread.sleep(100)

    println("\n### Mono.onErrorMap()\n")

    Mono.error<RuntimeException>(IllegalStateException())
        .onErrorMap {
            RuntimeException(it)
        }
        .log()
        .doOnError { println("doOnError: $it") }
        .subscribe()

    Thread.sleep(100)

    println("\n### Mono.doFinally()\n")

    Mono.error<RuntimeException>(IllegalStateException())
        .onErrorMap {
            RuntimeException(it)
        }
        .doFinally { println("doFinally: $it") }
        .log()
        .doOnError { println("doOnError: $it") }
        .subscribe()

    Thread.sleep(100)

}

// console

### Mono.onErrorReturn()

[ INFO] (main) onSubscribe(FluxOnErrorResume.ResumeSubscriber)
[ INFO] (main) request(unbounded)
[ INFO] (main) onNext(Person(name=error, age=999))
[ INFO] (main) onComplete()

### Mono.onErrorResume()

[ INFO] (main) onSubscribe(FluxOnErrorResume.ResumeSubscriber)
[ INFO] (main) request(unbounded)
[ INFO] (main) onNext(Person(name=error, age=999))
[ INFO] (main) onComplete()

### Mono.onErrorMap()

[ INFO] (main) onSubscribe(FluxOnErrorResume.ResumeSubscriber)
[ INFO] (main) request(unbounded)
doOnError: java.lang.RuntimeException: java.lang.IllegalStateException
[ERROR] (main) onError(java.lang.RuntimeException: java.lang.IllegalStateException)
[ERROR] (main)  - java.lang.RuntimeException: java.lang.IllegalStateException
java.lang.RuntimeException: java.lang.IllegalStateException
	at me.sup2is.reactiveexam.ch05.ReactiveExam2Kt.main$lambda-3(ReactiveExam2.kt:44)
	at reactor.core.publisher.Mono.lambda$onErrorMap$31(Mono.java:3730)
	at reactor.core.publisher.FluxOnErrorResume$ResumeSubscriber.onError(FluxOnErrorResume.java:94)
	at reactor.core.publisher.Operators.error(Operators.java:198)
	at reactor.core.publisher.MonoError.subscribe(MonoError.java:53)
	at reactor.core.publisher.Mono.subscribe(Mono.java:4397)
	at reactor.core.publisher.Mono.subscribeWith(Mono.java:4512)
	at reactor.core.publisher.Mono.subscribe(Mono.java:4229)
	at me.sup2is.reactiveexam.ch05.ReactiveExam2Kt.main(ReactiveExam2.kt:48)
Caused by: java.lang.IllegalStateException
	at me.sup2is.reactiveexam.ch05.ReactiveExam2Kt.main(ReactiveExam2.kt:42)
[ERROR] (main) Operator called default onErrorDropped - reactor.core.Exceptions$ErrorCallbackNotImplemented: java.lang.RuntimeException: java.lang.IllegalStateException
reactor.core.Exceptions$ErrorCallbackNotImplemented: java.lang.RuntimeException: java.lang.IllegalStateException
Caused by: java.lang.RuntimeException: java.lang.IllegalStateException
	at me.sup2is.reactiveexam.ch05.ReactiveExam2Kt.main$lambda-3(ReactiveExam2.kt:44)
	at reactor.core.publisher.Mono.lambda$onErrorMap$31(Mono.java:3730)
	at reactor.core.publisher.FluxOnErrorResume$ResumeSubscriber.onError(FluxOnErrorResume.java:94)
	at reactor.core.publisher.Operators.error(Operators.java:198)
	at reactor.core.publisher.MonoError.subscribe(MonoError.java:53)
	at reactor.core.publisher.Mono.subscribe(Mono.java:4397)
	at reactor.core.publisher.Mono.subscribeWith(Mono.java:4512)
	at reactor.core.publisher.Mono.subscribe(Mono.java:4229)
	at me.sup2is.reactiveexam.ch05.ReactiveExam2Kt.main(ReactiveExam2.kt:48)
Caused by: java.lang.IllegalStateException
	at me.sup2is.reactiveexam.ch05.ReactiveExam2Kt.main(ReactiveExam2.kt:42)

### Mono.doFinally()

[ INFO] (main) onSubscribe(FluxDoFinally.DoFinallyConditionalSubscriber)
[ INFO] (main) request(unbounded)
doOnError: java.lang.RuntimeException: java.lang.IllegalStateException
doFinally: onError
[ERROR] (main) onError(java.lang.RuntimeException: java.lang.IllegalStateException)
[ERROR] (main)  - java.lang.RuntimeException: java.lang.IllegalStateException
java.lang.RuntimeException: java.lang.IllegalStateException
	at me.sup2is.reactiveexam.ch05.ReactiveExam2Kt.main$lambda-5(ReactiveExam2.kt:56)
	at reactor.core.publisher.Mono.lambda$onErrorMap$31(Mono.java:3730)
	at reactor.core.publisher.FluxOnErrorResume$ResumeSubscriber.onError(FluxOnErrorResume.java:94)
	at reactor.core.publisher.Operators.error(Operators.java:198)
	at reactor.core.publisher.MonoError.subscribe(MonoError.java:53)
	at reactor.core.publisher.Mono.subscribe(Mono.java:4397)
	at reactor.core.publisher.Mono.subscribeWith(Mono.java:4512)
	at reactor.core.publisher.Mono.subscribe(Mono.java:4229)
	at me.sup2is.reactiveexam.ch05.ReactiveExam2Kt.main(ReactiveExam2.kt:61)
Caused by: java.lang.IllegalStateException
	at me.sup2is.reactiveexam.ch05.ReactiveExam2Kt.main(ReactiveExam2.kt:54)
[ERROR] (main) Operator called default onErrorDropped - reactor.core.Exceptions$ErrorCallbackNotImplemented: java.lang.RuntimeException: java.lang.IllegalStateException
reactor.core.Exceptions$ErrorCallbackNotImplemented: java.lang.RuntimeException: java.lang.IllegalStateException
Caused by: java.lang.RuntimeException: java.lang.IllegalStateException
	at me.sup2is.reactiveexam.ch05.ReactiveExam2Kt.main$lambda-5(ReactiveExam2.kt:56)
	at reactor.core.publisher.Mono.lambda$onErrorMap$31(Mono.java:3730)
	at reactor.core.publisher.FluxOnErrorResume$ResumeSubscriber.onError(FluxOnErrorResume.java:94)
	at reactor.core.publisher.Operators.error(Operators.java:198)
	at reactor.core.publisher.MonoError.subscribe(MonoError.java:53)
	at reactor.core.publisher.Mono.subscribe(Mono.java:4397)
	at reactor.core.publisher.Mono.subscribeWith(Mono.java:4512)
	at reactor.core.publisher.Mono.subscribe(Mono.java:4229)
	at me.sup2is.reactiveexam.ch05.ReactiveExam2Kt.main(ReactiveExam2.kt:61)
Caused by: java.lang.IllegalStateException
	at me.sup2is.reactiveexam.ch05.ReactiveExam2Kt.main(ReactiveExam2.kt:54)
```

## retry(), retryWhen()

오류가 있을때 이 시퀀스를 다시 구독하고싶다면 `retry()` 를 사용하면 된다. 단 인자가 없는 `retry()` 는 무기한 구독하므로 주의하자.

조금 더 세밀하게 재시도를 하고 싶으면 `retryWhen()` 을 사용할 수 있다. 인자의 타입이 `Retry`인데 자세한 설명은 생략한다. docs를 보자. https://projectreactor.io/docs/core/release/api/reactor/util/retry/Retry.html](https://projectreactor.io/docs/core/release/api/reactor/util/retry/Retry.html)

```kotlin
package me.sup2is.reactiveexam.ch05

import reactor.core.publisher.Mono
import reactor.util.retry.Retry
import java.time.Duration

fun main(args: Array<String>) {

    println("\n### Mono.retry()\n")

    Mono.error<RuntimeException>(IllegalStateException())
        .log()
        .retry(1)
        .subscribe()

    Thread.sleep(100)

    println("\n### Mono.retryWhen()\n")

    Mono.error<RuntimeException>(IllegalStateException())
        .log()
        .retryWhen(
            Retry.fixedDelay(2, Duration.ofMillis(500))
        )
        .subscribe()

    Thread.sleep(2000)
}

// console

### Mono.retry()

[ INFO] (main) onSubscribe([Fuseable] Operators.EmptySubscription)
[ INFO] (main) request(unbounded)
[ INFO] (main) onSubscribe([Fuseable] Operators.EmptySubscription)
[ INFO] (main) request(unbounded)
[ERROR] (main) onError(java.lang.IllegalStateException)
[ERROR] (main)  - java.lang.IllegalStateException
java.lang.IllegalStateException
	at me.sup2is.reactiveexam.ch05.ReactiveExam3Kt.main(ReactiveExam3.kt:11)
[ERROR] (main) onError(java.lang.IllegalStateException)
[ERROR] (main)  - java.lang.IllegalStateException
java.lang.IllegalStateException
	at me.sup2is.reactiveexam.ch05.ReactiveExam3Kt.main(ReactiveExam3.kt:11)
[ERROR] (main) Operator called default onErrorDropped - reactor.core.Exceptions$ErrorCallbackNotImplemented: java.lang.IllegalStateException
reactor.core.Exceptions$ErrorCallbackNotImplemented: java.lang.IllegalStateException
Caused by: java.lang.IllegalStateException
	at me.sup2is.reactiveexam.ch05.ReactiveExam3Kt.main(ReactiveExam3.kt:11)

### Mono.retryWhen()

[ INFO] (main) onSubscribe([Fuseable] Operators.EmptySubscription)
[ INFO] (main) request(unbounded)
[ERROR] (main) onError(java.lang.IllegalStateException)
[ERROR] (main)  - java.lang.IllegalStateException
java.lang.IllegalStateException
	at me.sup2is.reactiveexam.ch05.ReactiveExam3Kt.main(ReactiveExam3.kt:20)
[ INFO] (parallel-1) onSubscribe([Fuseable] Operators.EmptySubscription)
[ INFO] (parallel-1) request(unbounded)
[ERROR] (parallel-1) onError(java.lang.IllegalStateException)
[ERROR] (parallel-1)  - java.lang.IllegalStateException
java.lang.IllegalStateException
	at me.sup2is.reactiveexam.ch05.ReactiveExam3Kt.main(ReactiveExam3.kt:20)
[ INFO] (parallel-2) onSubscribe([Fuseable] Operators.EmptySubscription)
[ INFO] (parallel-2) request(unbounded)
[ INFO] (parallel-2) cancel()
[ERROR] (parallel-2) onError(java.lang.IllegalStateException)
[ERROR] (parallel-2)  - java.lang.IllegalStateException
java.lang.IllegalStateException
	at me.sup2is.reactiveexam.ch05.ReactiveExam3Kt.main(ReactiveExam3.kt:20)
[ERROR] (parallel-2) Operator called default onErrorDropped - reactor.core.Exceptions$ErrorCallbackNotImplemented: reactor.core.Exceptions$RetryExhaustedException: Retries exhausted: 2/2
reactor.core.Exceptions$ErrorCallbackNotImplemented: reactor.core.Exceptions$RetryExhaustedException: Retries exhausted: 2/2
Caused by: reactor.core.Exceptions$RetryExhaustedException: Retries exhausted: 2/2
	at reactor.core.Exceptions.retryExhausted(Exceptions.java:290)
	at reactor.util.retry.RetryBackoffSpec.lambda$static$0(RetryBackoffSpec.java:67)
	at reactor.util.retry.RetryBackoffSpec.lambda$generateCompanion$4(RetryBackoffSpec.java:557)
	at reactor.core.publisher.FluxConcatMap$ConcatMapImmediate.drain(FluxConcatMap.java:376)
	at reactor.core.publisher.FluxConcatMap$ConcatMapImmediate.innerComplete(FluxConcatMap.java:296)
	at reactor.core.publisher.FluxConcatMap$ConcatMapInner.onComplete(FluxConcatMap.java:887)
	at reactor.core.publisher.Operators$MonoSubscriber.complete(Operators.java:1817)
	at reactor.core.publisher.MonoFlatMap$FlatMapInner.onNext(MonoFlatMap.java:249)
	at reactor.core.publisher.MonoIgnoreThen$ThenIgnoreMain.complete(MonoIgnoreThen.java:292)
	at reactor.core.publisher.MonoIgnoreThen$ThenIgnoreMain.onNext(MonoIgnoreThen.java:187)
	at reactor.core.publisher.MonoIgnoreThen$ThenIgnoreMain.subscribeNext(MonoIgnoreThen.java:236)
	at reactor.core.publisher.MonoIgnoreThen.subscribe(MonoIgnoreThen.java:51)
	at reactor.core.publisher.MonoFlatMap$FlatMapMain.onNext(MonoFlatMap.java:157)
	at reactor.core.publisher.MonoIgnoreThen$ThenIgnoreMain.complete(MonoIgnoreThen.java:292)
	at reactor.core.publisher.MonoIgnoreThen$ThenIgnoreMain.onNext(MonoIgnoreThen.java:187)
	at reactor.core.publisher.MonoDelay$MonoDelayRunnable.propagateDelay(MonoDelay.java:271)
	at reactor.core.publisher.MonoDelay$MonoDelayRunnable.run(MonoDelay.java:286)
	at reactor.core.scheduler.SchedulerTask.call(SchedulerTask.java:68)
	at reactor.core.scheduler.SchedulerTask.call(SchedulerTask.java:28)
	at java.base/java.util.concurrent.FutureTask.run$$$capture(FutureTask.java:264)
	at java.base/java.util.concurrent.FutureTask.run(FutureTask.java)
	at java.base/java.util.concurrent.ScheduledThreadPoolExecutor$ScheduledFutureTask.run(ScheduledThreadPoolExecutor.java:304)
	at java.base/java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1128)
	at java.base/java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:628)
	at java.base/java.lang.Thread.run(Thread.java:834)
Caused by: java.lang.IllegalStateException
	at me.sup2is.reactiveexam.ch05.ReactiveExam3Kt.main(ReactiveExam3.kt:20)
```



# 시간값 다루기

## timed(), timestamp(), delayElements()

```kotlin
package me.sup2is.reactiveexam.ch06

import me.sup2is.reactiveexam.Person
import reactor.core.publisher.Flux
import java.time.Duration

fun main(args: Array<String>) {

    val group1 = Flux.just(
        Person(name = "choi", age = 29),
        Person(name = "woo", age = 31)
    )

    println("\n### Flux.timed()\n")

    group1.timed()
        .log()
        .subscribe { println("subscribe: $it") }

    println("\n### Flux.timestamp()\n")

    group1.timed()
        .timestamp()
        .log()
        .subscribe { println("subscribe: $it") }

    println("\n### Flux.delayElements()\n")

    group1.delayElements(Duration.ofMillis(500))
        .log()
        .subscribe { println("subscribe: $it") }

    Thread.sleep(2000)
}

// console

### Flux.elapsed()

[ INFO] (main) onSubscribe(FluxTimed.TimedSubscriber)
[ INFO] (main) request(unbounded)
[ INFO] (main) onNext(Timed(Person(name=choi, age=29)){eventElapsedNanos=28552650, eventElapsedSinceSubscriptionNanos=28552650,  eventTimestampEpochMillis=1656372127034})
subscribe: Timed(Person(name=choi, age=29)){eventElapsedNanos=28552650, eventElapsedSinceSubscriptionNanos=28552650,  eventTimestampEpochMillis=1656372127034}
[ INFO] (main) onNext(Timed(Person(name=woo, age=31)){eventElapsedNanos=1098779, eventElapsedSinceSubscriptionNanos=29651429,  eventTimestampEpochMillis=1656372127035})
subscribe: Timed(Person(name=woo, age=31)){eventElapsedNanos=1098779, eventElapsedSinceSubscriptionNanos=29651429,  eventTimestampEpochMillis=1656372127035}
[ INFO] (main) onComplete()

### Flux.timestamp()

[ INFO] (main) onSubscribe(FluxMap.MapSubscriber)
[ INFO] (main) request(unbounded)
[ INFO] (main) onNext([1656372127038,Timed(Person(name=choi, age=29)){eventElapsedNanos=778372, eventElapsedSinceSubscriptionNanos=778372,  eventTimestampEpochMillis=1656372127038}])
subscribe: [1656372127038,Timed(Person(name=choi, age=29)){eventElapsedNanos=778372, eventElapsedSinceSubscriptionNanos=778372,  eventTimestampEpochMillis=1656372127038}]
[ INFO] (main) onNext([1656372127039,Timed(Person(name=woo, age=31)){eventElapsedNanos=465446, eventElapsedSinceSubscriptionNanos=1243818,  eventTimestampEpochMillis=1656372127039}])
subscribe: [1656372127039,Timed(Person(name=woo, age=31)){eventElapsedNanos=465446, eventElapsedSinceSubscriptionNanos=1243818,  eventTimestampEpochMillis=1656372127039}]
[ INFO] (main) onComplete()

### Flux.delayElements()

[ INFO] (main) onSubscribe(FluxConcatMap.ConcatMapImmediate)
[ INFO] (main) request(unbounded)
[ INFO] (parallel-1) onNext(Person(name=choi, age=29))
subscribe: Person(name=choi, age=29)
[ INFO] (parallel-2) onNext(Person(name=woo, age=31))
subscribe: Person(name=woo, age=31)
[ INFO] (parallel-2) onComplete()
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

  
