---
layout: post
title: "Mono와 Flux랑 친해지기 #1 with Kotlin"
tags: [Reactive, Mono, Flux, Kotlin]
date: 2022-06-23
comments: true
---

<br>

# Overview

먼저 이 글에서 reactive core에 대한 이야기는 하지 않는다. 설명을 잘 할 자신도 없고 심지어 제대로 이해도 못한 것 같다. 만약 reactive core에 관심이 있거나 관련지식이 0인 상태라면 토비님의 [토비의 봄 TV 리액티브 프로그래밍](https://www.youtube.com/watch?v=8fenTR3KOJo&list=PLOLeoJ50I1kkqC4FuEztT__3xKSfR2fpw) 을 보는걸 매우 추천한다.

이 글에서는 Mono와 Flux의 주요 API를 한개씩 확인해보면서 어떤 기능을 갖는지에 대해 알아보도록 하겠다. 거의 완전히 [https://projectreactor.io/docs/core/release/reference/index.html#which-operator](https://projectreactor.io/docs/core/release/reference/index.html#which-operator)에 나와있는 API 위주로만 설명한다.

# 시퀀스 만들기

## just(), justOrNull()

이미 데이터가 준비되어 있고 시퀀스를 만들기만 하면 된다면 `just()` 메서드를 사용하면 된다. `just()` 메서드는 `Mono`와 `Flux` 모두 갖고 있다. 추가로 kotlin의 nullable타입 또는 자바의 Optional 타입을 위해 `justOrNull()` 이라는 메서드도 지원해준다. `justOrNull()`은 `Mono`만 갖고 있다.

```kotlin
package me.sup2is.reactiveexam.ch01

import me.sup2is.reactiveexam.Person
import reactor.core.publisher.Flux
import reactor.core.publisher.Mono

fun main(args: Array<String>) {

    println("\n###  Mono.just()\n")

    Mono.just(Person(name = "choi", age = 29))
        .log()
        .subscribe { println("subscribe: $it") }

    println("\n###  Flux.just()\n")

    Flux.just(
        Person(name = "choi", age = 29),
        Person(name = "woo", age = 31)
    ).log()
        .subscribe { println("subscribe: $it") }

    println("\n###  Mono.justOrEmpty()\n")

    val persion: Person? = null

    Mono.justOrEmpty(persion)
        .log()
        .subscribe()
}

// console

###  Mono.just()

[ INFO] (main) | onSubscribe([Synchronous Fuseable] Operators.ScalarSubscription)
[ INFO] (main) | request(unbounded)
[ INFO] (main) | onNext(Person(name=choi, age=29))
subscribe: Person(name=choi, age=29)
[ INFO] (main) | onComplete()

###  Flux.just()

[ INFO] (main) | onSubscribe([Synchronous Fuseable] FluxArray.ArraySubscription)
[ INFO] (main) | request(unbounded)
[ INFO] (main) | onNext(Person(name=choi, age=29))
subscribe: Person(name=choi, age=29)
[ INFO] (main) | onNext(Person(name=woo, age=31))
subscribe: Person(name=woo, age=31)
[ INFO] (main) | onComplete()

###  Mono.justOrEmpty()

[ INFO] (main) onSubscribe([Fuseable] Operators.EmptySubscription)
[ INFO] (main) request(unbounded)
[ INFO] (main) onComplete()
```

데이터를 supplier를 통해 lazy하게 받아올 수 있는 방법도 있다.

## fromSupplier(), defer()

`Mono` 의 `fromSupplier()` 를 사용하는 방법도 있고 `Mono<T>` 나 `Flux<T>` 를 반환하는 supplier를 사용해서 `defer()` 로도 시퀀스를 만들수 있다. 코드를 보면 이해가 쉽다.

```kotlin
package me.sup2is.reactiveexam.ch01

import me.sup2is.reactiveexam.Person
import reactor.core.publisher.Flux
import reactor.core.publisher.Mono

fun main(args: Array<String>) {

    println("\n###  Mono.fromSupplier()\n")

    val personSupplier: () -> Person = { Person(name = "choi", age = 29) }

    Mono.fromSupplier(personSupplier)
        .log()
        .subscribe { println("subscribe: $it") }

    println("\n###  Mono.defer()\n")

    val monoSupplier: () -> Mono<Person> = { Mono.just(Person(name = "choi", age = 29)) }

    Mono.defer(monoSupplier)
        .log()
        .subscribe { println("subscribe: $it") }

    println("\n###  Flux.defer()\n")

    val fluxSupplier: () -> Flux<Person> = {
        Flux.just(
            Person(name = "choi", age = 29),
            Person(name = "woo", age = 31)
        )
    }

    Flux.defer(fluxSupplier)
        .log()
        .subscribe { println("subscribe: $it") }
}

// console

###  Mono.fromSupplier()

[ INFO] (main) | onSubscribe([Fuseable] Operators.MonoSubscriber)
[ INFO] (main) | request(unbounded)
[ INFO] (main) | onNext(Person(name=choi, age=29))
subscribe: Person(name=choi, age=29)
[ INFO] (main) | onComplete()

###  Mono.defer()

[ INFO] (main) onSubscribe([Synchronous Fuseable] Operators.ScalarSubscription)
[ INFO] (main) request(unbounded)
[ INFO] (main) onNext(Person(name=choi, age=29))
subscribe: Person(name=choi, age=29)
[ INFO] (main) onComplete()

###  Flux.defer()

[ INFO] (main) onSubscribe([Synchronous Fuseable] FluxArray.ArraySubscription)
[ INFO] (main) request(unbounded)
[ INFO] (main) onNext(Person(name=choi, age=29))
subscribe: Person(name=choi, age=29)
[ INFO] (main) onNext(Person(name=woo, age=31))
subscribe: Person(name=woo, age=31)
[ INFO] (main) onComplete()
```

## fromArray(), fromIterable(), range(), fromStream()

`Flux` 한정으로 반복되는 요소를 통해 `Flux` 를 만들 수도 있다. 

1. `Flux.fromArray()`
2. `Flux.fromIterable()`
3. `Flux.range()`
4. `Flux.fromStream()`

```kotlin
package me.sup2is.reactiveexam.ch01

import me.sup2is.reactiveexam.Person
import reactor.core.publisher.Flux
import java.util.stream.Stream

fun main(args: Array<String>) {

    println("\n###  Flux.fromArray()\n")

    val personArray = arrayOf(Person(name = "choi", age = 29), Person(name = "woo", age = 31))

    Flux.fromArray(personArray)
        .log()
        .subscribe { println("subscribe: $it") }

    println("\n###  Flux.fromIterable()\n")

    val personList = listOf(Person(name = "choi", age = 29), Person(name = "woo", age = 31))

    Flux.fromIterable(personList)
        .log()
        .subscribe { println("subscribe: $it") }

    println("\n###  Flux.range()\n")

    Flux.range(1, 5)
        .log()
        .subscribe { println("subscribe: $it") }

    println("\n###  Flux.fromStream()\n")

    val stream = Stream.of(Person(name = "choi", age = 29))

    Flux.fromStream(stream)
        .log()
        .subscribe { println("subscribe: $it") }
}

// console


###  Flux.fromArray()

[ INFO] (main) | onSubscribe([Synchronous Fuseable] FluxArray.ArraySubscription)
[ INFO] (main) | request(unbounded)
[ INFO] (main) | onNext(Person(name=choi, age=29))
subscribe: Person(name=choi, age=29)
[ INFO] (main) | onNext(Person(name=woo, age=31))
subscribe: Person(name=woo, age=31)
[ INFO] (main) | onComplete()

###  Flux.fromIterable()

[ INFO] (main) | onSubscribe([Synchronous Fuseable] FluxIterable.IterableSubscription)
[ INFO] (main) | request(unbounded)
[ INFO] (main) | onNext(Person(name=choi, age=29))
subscribe: Person(name=choi, age=29)
[ INFO] (main) | onNext(Person(name=woo, age=31))
subscribe: Person(name=woo, age=31)
[ INFO] (main) | onComplete()

###  Flux.range()

[ INFO] (main) | onSubscribe([Synchronous Fuseable] FluxRange.RangeSubscription)
[ INFO] (main) | request(unbounded)
[ INFO] (main) | onNext(1)
subscribe: 1
[ INFO] (main) | onNext(2)
subscribe: 2
[ INFO] (main) | onNext(3)
subscribe: 3
[ INFO] (main) | onNext(4)
subscribe: 4
[ INFO] (main) | onNext(5)
subscribe: 5
[ INFO] (main) | onComplete()

###  Flux.fromStream()

[ INFO] (main) | onSubscribe([Synchronous Fuseable] FluxIterable.IterableSubscription)
[ INFO] (main) | request(unbounded)
[ INFO] (main) | onNext(Person(name=choi, age=29))
subscribe: Person(name=choi, age=29)
[ INFO] (main) | onComplete()
```



## error(), empty(), never()

에러를 발생하거나 바로 데이터가 없는 상태에서 `onComplete`를 호출하도록 `error()` 또는 `empty()` 메서드를 사용할 수도 있다. 추가로 `never()` 를 사용해서 어떤 `onComplete` , `onError`도 발생시키지 않는 시퀀스를 생성할 수도 있다.

```kotlin
package me.sup2is.reactiveexam.ch01

import reactor.core.publisher.Flux
import reactor.core.publisher.Mono

fun main(args: Array<String>) {

    println("\n###  Mono.empty()\n")

    Mono.empty<Any>()
        .log()
        .defaultIfEmpty("DEFAULT")
        .subscribe()

    println("\n###  Flux.empty()\n")

    Flux.empty<Any>()
        .log()
        .defaultIfEmpty("DEFAULT")
        .subscribe()

    println("\n###  Mono.error()\n")

    Mono.error<Any>(IllegalAccessException())
        .log()
        .subscribe()

    Thread.sleep(10)

    println("\n###  Flux.error()\n")

    Flux.error<Any>(IllegalAccessException())
        .log()
        .subscribe()

    Thread.sleep(10)

    println("\n###  Mono.never()\n")

    Mono.never<Any>()
        .log()
        .defaultIfEmpty("DEFAULT")
        .subscribe()

    println("\n###  Flux.never()\n")

    Flux.never<Any>()
        .log()
        .defaultIfEmpty("DEFAULT")
        .subscribe()
}

// console

###  Mono.empty()

[ INFO] (main) onSubscribe([Fuseable] Operators.EmptySubscription)
[ INFO] (main) request(unbounded)
[ INFO] (main) onComplete()

###  Flux.empty()

[ INFO] (main) onSubscribe([Fuseable] Operators.EmptySubscription)
[ INFO] (main) request(unbounded)
[ INFO] (main) onComplete()

###  Mono.error()

[ INFO] (main) onSubscribe([Fuseable] Operators.EmptySubscription)
[ INFO] (main) request(unbounded)
[ERROR] (main) onError(java.lang.IllegalAccessException)
[ERROR] (main)  - java.lang.IllegalAccessException
java.lang.IllegalAccessException
	at me.sup2is.reactiveexam.ch01.ReactiveExam4Kt.main(ReactiveExam4.kt:24)
[ERROR] (main) Operator called default onErrorDropped - reactor.core.Exceptions$ErrorCallbackNotImplemented: java.lang.IllegalAccessException
reactor.core.Exceptions$ErrorCallbackNotImplemented: java.lang.IllegalAccessException
Caused by: java.lang.IllegalAccessException
	at me.sup2is.reactiveexam.ch01.ReactiveExam4Kt.main(ReactiveExam4.kt:24)

###  Flux.error()

[ INFO] (main) onSubscribe([Fuseable] Operators.EmptySubscription)
[ INFO] (main) request(unbounded)
[ERROR] (main) onError(java.lang.IllegalAccessException)
[ERROR] (main)  - java.lang.IllegalAccessException
java.lang.IllegalAccessException
	at me.sup2is.reactiveexam.ch01.ReactiveExam4Kt.main(ReactiveExam4.kt:32)
[ERROR] (main) Operator called default onErrorDropped - reactor.core.Exceptions$ErrorCallbackNotImplemented: java.lang.IllegalAccessException
reactor.core.Exceptions$ErrorCallbackNotImplemented: java.lang.IllegalAccessException
Caused by: java.lang.IllegalAccessException
	at me.sup2is.reactiveexam.ch01.ReactiveExam4Kt.main(ReactiveExam4.kt:32)

###  Mono.never()

[ INFO] (main) onSubscribe([Fuseable] Operators.EmptySubscription)
[ INFO] (main) request(unbounded)

###  Flux.never()

[ INFO] (main) onSubscribe([Fuseable] Operators.EmptySubscription)
[ INFO] (main) request(unbounded)
```

`Mono`나 `Flux`의 `using()` 을 사용해서 함수형 스타일로 시퀀스를 만들 수도 있다. 이함수는 `Callable`, `Function`, `Consumer` 타입의 함수 세개를 받아서 동작한다. 

## using()

```kotlin
package me.sup2is.reactiveexam.ch01

import me.sup2is.reactiveexam.Person
import reactor.core.publisher.Flux
import reactor.core.publisher.Mono

fun main(args: Array<String>) {

    println("\n###  Mono.using()\n")

    val callable = {
        Person(name = "choi", age = 29)
    }

    val function: (Person) -> Mono<Person> = {
        Mono.just(Person(name = it.name, age = it.age))
    }

    val consumer: (Person) -> Unit = { println(it) }

    Mono.using(
        callable, function, consumer
    ).log()
        .subscribe()

    println("\n###  Flux.using()\n")

    val callable2 = {
        listOf(Person(name = "choi", age = 29), Person(name = "woo", age = 31))
    }

    val function2: (List<Person>) -> Flux<Person> = {
        Flux.fromIterable(it)
    }

    val consumer2: (List<Person>) -> Unit = { println(it) }

    Flux.using(
        callable2, function2, consumer2
    ).log()
        .subscribe()
}

// console

###  Mono.using()

[ INFO] (main) | onSubscribe([Fuseable] MonoUsing.MonoUsingSubscriber)
[ INFO] (main) | request(unbounded)
Person(name=choi, age=29)
[ INFO] (main) | onNext(Person(name=choi, age=29))
[ INFO] (main) | onComplete()

###  Flux.using()

[ INFO] (main) | onSubscribe([Fuseable] FluxUsing.UsingFuseableSubscriber)
[ INFO] (main) | request(unbounded)
[ INFO] (main) | onNext(Person(name=choi, age=29))
[ INFO] (main) | onNext(Person(name=woo, age=31))
[Person(name=choi, age=29), Person(name=woo, age=31)]
[ INFO] (main) | onComplete()
```

## generate(), create()

프로그래밍 방식으로 이벤트를 생성하는 Flux의 `generate()` 를 사용하거나 `create()` 를 사용할 수 있다. `generate()` 는 동기식으로 한번에 한개씩 생성하고 `create()` 는 비동기로 여러개를 방출한다. `generate()`는 상태를 유지하지만 `create()`는 상태를 유지하지 않는다. `SynchronousSink` 는 한번의 `next()`만 가능하고 `FluxSink`는 필요한만큼 `next()`를 호출할 수 있는 특징이 있다.

```kotlin
package me.sup2is.reactiveexam.ch01

import reactor.core.publisher.Flux
import reactor.core.publisher.SynchronousSink

fun main(args: Array<String>) {

    println("\n###  Flux.generate()\n")

    val callable = {
        arrayOf(0, 1)
    }

    var emitCounter = 0
    val biFunction: (Array<Int>, SynchronousSink<Int>) -> Array<Int> = { ints: Array<Int>, synchronousSink: SynchronousSink<Int> ->
        synchronousSink.next(ints[0])
        emitCounter ++
        if (emitCounter >= 10) {
            synchronousSink.complete()
        }

        val temp = ints[0]
        ints[0] = ints[1]
        ints[1] = ints[1] + temp
        ints
    }

    Flux.generate(
        callable, biFunction
    ).log()
        .subscribe()
}

// console

###  Flux.generate()

[ INFO] (main) | onSubscribe([Fuseable] FluxGenerate.GenerateSubscription)
[ INFO] (main) | request(unbounded)
[ INFO] (main) | onNext(0)
[ INFO] (main) | onNext(1)
[ INFO] (main) | onNext(1)
[ INFO] (main) | onNext(2)
[ INFO] (main) | onNext(3)
[ INFO] (main) | onNext(5)
[ INFO] (main) | onNext(8)
[ INFO] (main) | onNext(13)
[ INFO] (main) | onNext(21)
[ INFO] (main) | onNext(34)
[ INFO] (main) | onComplete()

```



# 기존 시퀀스를 변환하기

기본적인 시퀀스 생성 방법을 알았기 때문에 이제 시퀀스를 변환하는 방법을 알아보도록 하자.

## map(), cast(), index()

`map()`은 java Stream에서 제공하는 `map()`과 매우 유사하다. 다만 항상 `Mono`나 `Flux` 타입으로 감싸져있다. `cast()` 는 단순히 캐스팅 하는 용도로 사용할 수 있다. `index()`는 `Flux`에서 요소의 index가 필요할때 사용할 수 있다.

```kotlin
package me.sup2is.reactiveexam.ch02

import me.sup2is.reactiveexam.Person
import reactor.core.publisher.Flux
import reactor.core.publisher.Mono

fun main(args: Array<String>) {

    println("\n###  Mono.map()\n")

    Mono.just(Person(name = "choi", age = 29))
        .log()
        .map { it.age }
        .subscribe { println("subscribe: $it") }

    println("\n###  Flux.map()\n")

    Flux.just(
        Person(name = "choi", age = 29),
        Person(name = "woo", age = 31)
    ).log()
        .map { it.name }
        .subscribe { println("subscribe: $it") }

    println("\n###  Mono.cast()\n")

    Mono.just(Person(name = "choi", age = 29))
        .log()
        .cast(Any::class.java)
        .subscribe { println("subscribe: $it") }

    println("\n###  Flux.cast()\n")

    Flux.just(
        Person(name = "choi", age = 29),
        Person(name = "woo", age = 31)
    ).log()
        .cast(Any::class.java)
        .subscribe { println("subscribe: $it") }

    println("\n###  Flux.index()\n")

    Flux.just(
        Person(name = "choi", age = 29),
        Person(name = "woo", age = 31)
    ).log()
        .index()
        .subscribe { println("index: ${it.t1}, data: ${it.t2}") }
}

// console

###  Mono.map()

[ INFO] (main) | onSubscribe([Synchronous Fuseable] Operators.ScalarSubscription)
[ INFO] (main) | request(unbounded)
[ INFO] (main) | onNext(Person(name=choi, age=29))
subscribe: 29
[ INFO] (main) | onComplete()

###  Flux.map()

[ INFO] (main) | onSubscribe([Synchronous Fuseable] FluxArray.ArraySubscription)
[ INFO] (main) | request(unbounded)
[ INFO] (main) | onNext(Person(name=choi, age=29))
subscribe: choi
[ INFO] (main) | onNext(Person(name=woo, age=31))
subscribe: woo
[ INFO] (main) | onComplete()

###  Mono.cast()

[ INFO] (main) | onSubscribe([Synchronous Fuseable] Operators.ScalarSubscription)
[ INFO] (main) | request(unbounded)
[ INFO] (main) | onNext(Person(name=choi, age=29))
subscribe: Person(name=choi, age=29)
[ INFO] (main) | onComplete()

###  Flux.cast()

[ INFO] (main) | onSubscribe([Synchronous Fuseable] FluxArray.ArraySubscription)
[ INFO] (main) | request(unbounded)
[ INFO] (main) | onNext(Person(name=choi, age=29))
subscribe: Person(name=choi, age=29)
[ INFO] (main) | onNext(Person(name=woo, age=31))
subscribe: Person(name=woo, age=31)
[ INFO] (main) | onComplete()

###  Flux.index()

[ INFO] (main) | onSubscribe([Synchronous Fuseable] FluxArray.ArraySubscription)
[ INFO] (main) | request(unbounded)
[ INFO] (main) | onNext(Person(name=choi, age=29))
index: 0, data: Person(name=choi, age=29)
[ INFO] (main) | onNext(Person(name=woo, age=31))
index: 1, data: Person(name=woo, age=31)
[ INFO] (main) | onComplete()
```





<br>

***

포스팅은 여기까지 하겠습니다. 감사합니다!



<br>

- **[References]**

  - [https://godekdls.github.io/Reactor%20Core/appendixawhichoperatordoineed/](https://godekdls.github.io/Reactor%20Core/appendixawhichoperatordoineed/)
  - [https://projectreactor.io/docs/core/release/reference/index.html#which-operator](https://projectreactor.io/docs/core/release/reference/index.html#which-operator)
  - [https://www.woolha.com/tutorials/project-reactor-using-mono-never-and-flux-never-examples](https://www.woolha.com/tutorials/project-reactor-using-mono-never-and-flux-never-examples)
  - [https://www.baeldung.com/flux-sequences-reactor](https://www.baeldung.com/flux-sequences-reactor)

  
