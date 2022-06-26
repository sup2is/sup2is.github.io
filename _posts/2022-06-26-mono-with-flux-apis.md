---
layout: post
title: "Mono와 Flux랑 친해지기 #1 with Kotlin"
tags: [Reactive, Mono, Flux, Kotlin]
date: 2022-06-26
comments: true
---

<br>

# Overview

먼저 이 글에서 reactive core에 대한 이야기는 하지 않는다. 설명을 잘 할 자신도 없고 심지어 제대로 이해도 못한 것 같다. 만약 reactive core에 관심이 있거나 관련지식이 0인 상태라면 토비님의 [토비의 봄 TV 리액티브 프로그래밍](https://www.youtube.com/watch?v=8fenTR3KOJo&list=PLOLeoJ50I1kkqC4FuEztT__3xKSfR2fpw) 을 보는걸 매우 추천한다.

이 글에서는 Mono와 Flux의 주요 API를 한개씩 확인해보면서 어떤 기능을 갖는지에 대해 알아보도록 한다. 거의 완전히 [https://projectreactor.io/docs/core/release/reference/index.html#which-operator](https://projectreactor.io/docs/core/release/reference/index.html#which-operator)에 나와있는 API 위주로만 설명한다. 따로 편집하기 힘들어서 넣진 않았지만 ... 이해가 어렵다면 docs에 있는 마블다이어그램과 같이 보는걸 매우매우 추천한다.

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

## flatMap(), flatMapSequential(), handle(), flatMapMany()

`Flux`의 요소별로 1:N 으로 변형이 필요하다면 `flatMap()`을 사용할 수 있다. `flatMap()` 은 비동기적으로 실행되기 때문에 순서가 필요하다면 `flatMapSequential()` 을 사용하면 된다.

`Flux`의  `handle()` 을 사용하면 넘어오는 요소에 대한 식을 작성해서 유연하게 동작하도록 할 수 있다. `Mono`의 `flatMapMany()`를 사용해서 `Flux`로 방출할 수도 있다. 

```kotlin
package me.sup2is.reactiveexam.ch02

import me.sup2is.reactiveexam.Person
import reactor.core.publisher.Flux
import reactor.core.publisher.Mono
import reactor.core.publisher.SynchronousSink

fun main(args: Array<String>) {

    println("\n###  Flux.flatmap()\n")

    Flux.just(
        Person(name = "choi", age = 29),
        Person(name = "woo", age = 31)
    ).log()
        .flatMap {
            Flux.just(*it.name.toList().toTypedArray())
        }
        .subscribe { println("subscribe: $it") }

    println("\n###  Flux.flatMapSequential()\n")

    Flux.just(
        Person(name = "choi", age = 29),
        Person(name = "woo", age = 31)
    ).log()
        .flatMapSequential {
            Flux.just(*it.name.toList().toTypedArray())
        }
        .subscribe { println("subscribe: $it") }

    println("\n###  Flux.handle()\n")

    Flux.just(
        Person(name = "choi", age = 29),
        Person(name = "woo", age = 31)
    ).log()
        .handle { person: Person, synchronousSink: SynchronousSink<String> ->
            if (person.age == 29) {
                synchronousSink.next(person.name)
            } else {
                synchronousSink.complete()
            }
        }
        .subscribe { println("subscribe: $it") }

    println("\n###  Mono.flatMapMany()\n")

    Mono.just(
        Person(name = "choi", age = 29)
    ).log()
        .flatMapMany {
            Flux.just(*it.name.toList().toTypedArray())
        }
        .subscribe { println("subscribe: $it") }
}

// console

###  Flux.flatmap()

[ INFO] (main) | onSubscribe([Synchronous Fuseable] FluxArray.ArraySubscription)
[ INFO] (main) | request(256)
[ INFO] (main) | onNext(Person(name=choi, age=29))
subscribe: c
subscribe: h
subscribe: o
subscribe: i
[ INFO] (main) | request(1)
[ INFO] (main) | onNext(Person(name=woo, age=31))
subscribe: w
subscribe: o
subscribe: o
[ INFO] (main) | request(1)
[ INFO] (main) | onComplete()

###  Flux.flatMapSequential()

[ INFO] (main) | onSubscribe([Synchronous Fuseable] FluxArray.ArraySubscription)
[ INFO] (main) | request(256)
[ INFO] (main) | onNext(Person(name=choi, age=29))
subscribe: c
subscribe: h
subscribe: o
subscribe: i
[ INFO] (main) | request(1)
[ INFO] (main) | onNext(Person(name=woo, age=31))
subscribe: w
subscribe: o
subscribe: o
[ INFO] (main) | request(1)
[ INFO] (main) | onComplete()

###  Flux.handle()

[ INFO] (main) | onSubscribe([Synchronous Fuseable] FluxArray.ArrayConditionalSubscription)
[ INFO] (main) | request(unbounded)
[ INFO] (main) | onNext(Person(name=choi, age=29))
subscribe: choi
[ INFO] (main) | onNext(Person(name=woo, age=31))
[ INFO] (main) | cancel()

###  Mono.flatMapMany()

[ INFO] (main) | onSubscribe([Synchronous Fuseable] Operators.ScalarSubscription)
[ INFO] (main) | request(unbounded)
[ INFO] (main) | onNext(Person(name=choi, age=29))
subscribe: c
subscribe: h
subscribe: o
subscribe: i
[ INFO] (main) | onComplete()
```



## startWith(), concatWith()

기존 시퀀스에 요소를 추가하고 싶을때는 `startWith()`와 `concatWith()`를 사용할 수 있다. `startWith()`는 시퀀스의 처음에 추가하고 `concatWith()`는 시퀀스의 뒤에 추가한다. `concatWith()`는 `Publisher` 타입을 받는 특징이 있고 두 메서드 모두 `Flux`에만 있는 메서드다.

```kotlin
package me.sup2is.reactiveexam.ch02

import me.sup2is.reactiveexam.Person
import reactor.core.publisher.Flux
import reactor.core.publisher.Mono

fun main(args: Array<String>) {

    println("\n###  Flux.startWith()\n")

    Flux.just(
        Person(name = "choi", age = 29),
        Person(name = "woo", age = 31)
    ).log()
        .startWith(Person(name = "yoon", age = 30))
        .subscribe { println("subscribe: $it") }

    println("\n###  Flux.concatWith()\n")

    Flux.just(
        Person(name = "choi", age = 29),
        Person(name = "woo", age = 31)
    ).log()
        .concatWith(Mono.just(Person(name = "yoon", age = 30)))
        .subscribe { println("subscribe: $it") }
}

// console

###  Flux.startWith()

subscribe: Person(name=yoon, age=30)
[ INFO] (main) | onSubscribe([Synchronous Fuseable] FluxArray.ArraySubscription)
[ INFO] (main) | request(unbounded)
[ INFO] (main) | onNext(Person(name=choi, age=29))
subscribe: Person(name=choi, age=29)
[ INFO] (main) | onNext(Person(name=woo, age=31))
subscribe: Person(name=woo, age=31)
[ INFO] (main) | onComplete()

###  Flux.concatWith()

[ INFO] (main) | onSubscribe([Synchronous Fuseable] FluxArray.ArraySubscription)
[ INFO] (main) | request(unbounded)
[ INFO] (main) | onNext(Person(name=choi, age=29))
subscribe: Person(name=choi, age=29)
[ INFO] (main) | onNext(Person(name=woo, age=31))
subscribe: Person(name=woo, age=31)
[ INFO] (main) | onComplete()
subscribe: Person(name=yoon, age=30)
```

## collectList(), collectSortedList(), collectMap(), collectMultiMap(), collect(), count()

이 메서드들은 모두 집계와 관련된 메서드들이다. `collectList()` 는 `MutableList` 타입으로 요소들을 묶어준다. 나처럼 reactive에 익숙하지 않은 사람들은 주의해야 할 점이 있는게 단순히  `collectXXX()` 라고해서 특정 `List` 또는 `Map`으로 반환하는게 아니라 Mono에 담겨서 온다는 점이다. 앞으로 설명할 메서드 모두 `Mono`타입으로 리턴한다.

`collectSortedList()`은  `collectList()` 와 기본적인 기능은 같지만 정렬을 시켜준다. 이 때 요소들이 `Comparable` 인터페이스를 구현하고 있어야하고 이건 컴파일타임이 아니라 런타임에 에러를 발생한다는 점을 기억해야 한다. 

`collectMap()` 은 요소들을 특정 key로 묶어주는데 중복된 값이 있다면 가장 마지막으로 내보낸 요소가 값이된다. 우리가 기본적으로 예상하는 기능은 `collectMultiMap()` 이다. 이 메서드는  1:N 형태로 묶어준다.

`collect()`는 `java.util.stream.Collector` 타입을 받는 메서드인데 보통 java8의 스트림에서 사용하는 `collect()` 메서드와 같다고 생가하면 편할 것 같다.

마지막으로 `count()`는 요소들의 개수를 반환해준다.

```kotlin
package me.sup2is.reactiveexam.ch02

import me.sup2is.reactiveexam.Person
import reactor.core.publisher.Flux
import java.util.stream.Collectors

fun main(args: Array<String>) {

    println("\n###  Flux.collectList()\n")

    Flux.just(
        Person(name = "choi", age = 29),
        Person(name = "woo", age = 31)
    ).log()
        .collectList()
        .subscribe { println("subscribe: $it") }

    println("\n###  Flux.collectSortedList()\n")

    Flux.just(
        'b', 'd', 'c', 'a'
    ).log()
        .collectSortedList()
        .subscribe { println("subscribe: $it") }

    println("\n###  Flux.collectMap()\n")

    Flux.just(
        Person(name = "choi", age = 29),
        Person(name = "woo", age = 31),
        Person(name = "choi", age = 31),
        Person(name = "woo", age = 34)
    ).log()
        .collectMap { it.name }
        .subscribe { println("subscribe: $it") }

    println("\n###  Flux.collectMultiMap()\n")

    Flux.just(
        Person(name = "choi", age = 29),
        Person(name = "woo", age = 31),
        Person(name = "choi", age = 31),
        Person(name = "woo", age = 34)
    ).log()
        .collectMultimap { it.name }
        .subscribe { println("subscribe: $it") }

    println("\n###  Flux.collect()\n")

    Flux.just(
        Person(name = "choi", age = 29),
        Person(name = "woo", age = 31)
    ).log()
        .collect(Collectors.toList())
        .subscribe { println("subscribe: $it") }

    println("\n###  Flux.count()\n")

    Flux.just(
        Person(name = "choi", age = 29),
        Person(name = "woo", age = 31)
    ).log()
        .count()
        .subscribe { println("subscribe: $it") }
}

// console

###  Flux.collectList()

[ INFO] (main) | onSubscribe([Synchronous Fuseable] FluxArray.ArraySubscription)
[ INFO] (main) | request(unbounded)
[ INFO] (main) | onNext(Person(name=choi, age=29))
[ INFO] (main) | onNext(Person(name=woo, age=31))
[ INFO] (main) | onComplete()
subscribe: [Person(name=choi, age=29), Person(name=woo, age=31)]

###  Flux.collectSortedList()

[ INFO] (main) | onSubscribe([Synchronous Fuseable] FluxArray.ArraySubscription)
[ INFO] (main) | request(unbounded)
[ INFO] (main) | onNext(b)
[ INFO] (main) | onNext(d)
[ INFO] (main) | onNext(c)
[ INFO] (main) | onNext(a)
[ INFO] (main) | onComplete()
subscribe: [a, b, c, d]

###  Flux.collectMap()

[ INFO] (main) | onSubscribe([Synchronous Fuseable] FluxArray.ArraySubscription)
[ INFO] (main) | request(unbounded)
[ INFO] (main) | onNext(Person(name=choi, age=29))
[ INFO] (main) | onNext(Person(name=woo, age=31))
[ INFO] (main) | onNext(Person(name=choi, age=31))
[ INFO] (main) | onNext(Person(name=woo, age=34))
[ INFO] (main) | onComplete()
subscribe: {choi=Person(name=choi, age=31), woo=Person(name=woo, age=34)}

###  Flux.collectMultiMap()

[ INFO] (main) | onSubscribe([Synchronous Fuseable] FluxArray.ArraySubscription)
[ INFO] (main) | request(unbounded)
[ INFO] (main) | onNext(Person(name=choi, age=29))
[ INFO] (main) | onNext(Person(name=woo, age=31))
[ INFO] (main) | onNext(Person(name=choi, age=31))
[ INFO] (main) | onNext(Person(name=woo, age=34))
[ INFO] (main) | onComplete()
subscribe: {choi=[Person(name=choi, age=29), Person(name=choi, age=31)], woo=[Person(name=woo, age=31), Person(name=woo, age=34)]}

###  Flux.collect()

[ INFO] (main) | onSubscribe([Synchronous Fuseable] FluxArray.ArraySubscription)
[ INFO] (main) | request(unbounded)
[ INFO] (main) | onNext(Person(name=choi, age=29))
[ INFO] (main) | onNext(Person(name=woo, age=31))
[ INFO] (main) | onComplete()
subscribe: [Person(name=choi, age=29), Person(name=woo, age=31)]

###  Flux.count()

[ INFO] (main) | onSubscribe([Synchronous Fuseable] FluxArray.ArraySubscription)
[ INFO] (main) | request(unbounded)
[ INFO] (main) | onNext(Person(name=choi, age=29))
[ INFO] (main) | onNext(Person(name=woo, age=31))
[ INFO] (main) | onComplete()
subscribe: 2
```



## reduce(), scan()

요소들을 특정 함수로 합치고싶으면 `reduce()`와 `scan()` 을 사용할 수 있다. `reduce()` 는 모든 요소의 최종 값 하나만 방출하지만 `scan()`은 중간중간 요소들마다 방출한다.

```kotlin
package me.sup2is.reactiveexam.ch02

import me.sup2is.reactiveexam.Person
import reactor.core.publisher.Flux

fun main(args: Array<String>) {

    println("\n###  Flux.reduce()\n")

    Flux.just(
        Person(name = "choi", age = 29),
        Person(name = "woo", age = 31),
        Person(name = "choi", age = 31),
        Person(name = "woo", age = 34)
    ).log()
        .map { it.age }
        .reduce { age1, age2 ->
            age1 + age2
        }
        .subscribe { println("subscribe: $it") }

    println("\n###  Flux.scan()\n")

    Flux.just(
        Person(name = "choi", age = 29),
        Person(name = "woo", age = 31),
        Person(name = "choi", age = 31),
        Person(name = "woo", age = 34)
    ).log()
        .map { it.age }
        .scan { age1, age2 ->
            age1 + age2
        }
        .subscribe { println("subscribe: $it") }

}

// console

###  Flux.reduce()

[ INFO] (main) | onSubscribe([Synchronous Fuseable] FluxArray.ArraySubscription)
[ INFO] (main) | request(unbounded)
[ INFO] (main) | onNext(Person(name=choi, age=29))
[ INFO] (main) | onNext(Person(name=woo, age=31))
[ INFO] (main) | onNext(Person(name=choi, age=31))
[ INFO] (main) | onNext(Person(name=woo, age=34))
[ INFO] (main) | onComplete()
subscribe: 125

###  Flux.scan()

[ INFO] (main) | onSubscribe([Synchronous Fuseable] FluxArray.ArraySubscription)
[ INFO] (main) | request(unbounded)
[ INFO] (main) | onNext(Person(name=choi, age=29))
subscribe: 29
[ INFO] (main) | onNext(Person(name=woo, age=31))
subscribe: 60
[ INFO] (main) | onNext(Person(name=choi, age=31))
subscribe: 91
[ INFO] (main) | onNext(Person(name=woo, age=34))
subscribe: 125
[ INFO] (main) | onComplete()
```

## all(), any(), hasElements(), hasElement()

이 메

```kotlin
package me.sup2is.reactiveexam.ch02

import me.sup2is.reactiveexam.Person
import reactor.core.publisher.Flux

fun main(args: Array<String>) {

    println("\n###  Flux.all()\n")

    Flux.just(
        Person(name = "choi", age = 29),
        Person(name = "woo", age = 31),
        Person(name = "choi", age = 31),
        Person(name = "woo", age = 34)
    ).log()
        .all { it.name == "choi" }
        .subscribe { println("subscribe: $it") }

    println("\n###  Flux.any()\n")

    Flux.just(
        Person(name = "choi", age = 29),
        Person(name = "woo", age = 31),
        Person(name = "choi", age = 31),
        Person(name = "woo", age = 34)
    ).log()
        .any { it.name == "choi" }
        .subscribe { println("subscribe: $it") }

    println("\n###  Flux.hasElements()\n")

    Flux.just(
        Person(name = "choi", age = 29),
        Person(name = "woo", age = 31),
        Person(name = "choi", age = 31),
        Person(name = "woo", age = 34)
    ).log()
        .hasElements()
        .subscribe { println("subscribe: $it") }

    println("\n###  Flux.hasElement()\n")

    Flux.just(
        Person(name = "choi", age = 29),
        Person(name = "woo", age = 31),
        Person(name = "choi", age = 31),
        Person(name = "woo", age = 34)
    ).log()
        .map { it.name }
        .hasElement("choi")
        .subscribe { println("subscribe: $it") }
}

// console

###  Flux.all()

[ INFO] (main) | onSubscribe([Synchronous Fuseable] FluxArray.ArraySubscription)
[ INFO] (main) | request(unbounded)
[ INFO] (main) | onNext(Person(name=choi, age=29))
[ INFO] (main) | onNext(Person(name=woo, age=31))
[ INFO] (main) | cancel()
subscribe: false

###  Flux.any()

[ INFO] (main) | onSubscribe([Synchronous Fuseable] FluxArray.ArraySubscription)
[ INFO] (main) | request(unbounded)
[ INFO] (main) | onNext(Person(name=choi, age=29))
[ INFO] (main) | cancel()
subscribe: true

###  Flux.hasElements()

[ INFO] (main) | onSubscribe([Synchronous Fuseable] FluxArray.ArraySubscription)
[ INFO] (main) | request(unbounded)
[ INFO] (main) | onNext(Person(name=choi, age=29))
[ INFO] (main) | cancel()
subscribe: true

###  Flux.hasElement()

[ INFO] (main) | onSubscribe([Synchronous Fuseable] FluxArray.ArraySubscription)
[ INFO] (main) | request(unbounded)
[ INFO] (main) | onNext(Person(name=choi, age=29))
[ INFO] (main) | cancel()
subscribe: true
```







## all(), any(), hasElements(), hasElement()

이 네개의 메서드들은 모두 `Mono<Boolean>` 형태로 리턴하는 메서드다. `all()`은 주어진 `Predicate`에 모든 요소가 일치하는지 확인한다. `any()`는 한 개라도 있으면 true를 반환한다.

`hasElements()`는 요소가 0인지 확인하고 `hasElement()`는 인자로 넘긴 특정 값이 존재하는지를 확인해준다.

```kotlin
package me.sup2is.reactiveexam.ch02

import me.sup2is.reactiveexam.Person
import reactor.core.publisher.Flux

fun main(args: Array<String>) {

    println("\n###  Flux.all()\n")

    Flux.just(
        Person(name = "choi", age = 29),
        Person(name = "woo", age = 31),
        Person(name = "choi", age = 31),
        Person(name = "woo", age = 34)
    ).log()
        .all { it.name == "choi" }
        .subscribe { println("subscribe: $it") }

    println("\n###  Flux.any()\n")

    Flux.just(
        Person(name = "choi", age = 29),
        Person(name = "woo", age = 31),
        Person(name = "choi", age = 31),
        Person(name = "woo", age = 34)
    ).log()
        .any { it.name == "choi" }
        .subscribe { println("subscribe: $it") }

    println("\n###  Flux.hasElements()\n")

    Flux.just(
        Person(name = "choi", age = 29),
        Person(name = "woo", age = 31),
        Person(name = "choi", age = 31),
        Person(name = "woo", age = 34)
    ).log()
        .hasElements()
        .subscribe { println("subscribe: $it") }

    println("\n###  Flux.hasElement()\n")

    Flux.just(
        Person(name = "choi", age = 29),
        Person(name = "woo", age = 31),
        Person(name = "choi", age = 31),
        Person(name = "woo", age = 34)
    ).log()
        .map { it.name }
        .hasElement("choi")
        .subscribe { println("subscribe: $it") }
}

// console

###  Flux.all()

[ INFO] (main) | onSubscribe([Synchronous Fuseable] FluxArray.ArraySubscription)
[ INFO] (main) | request(unbounded)
[ INFO] (main) | onNext(Person(name=choi, age=29))
[ INFO] (main) | onNext(Person(name=woo, age=31))
[ INFO] (main) | cancel()
subscribe: false

###  Flux.any()

[ INFO] (main) | onSubscribe([Synchronous Fuseable] FluxArray.ArraySubscription)
[ INFO] (main) | request(unbounded)
[ INFO] (main) | onNext(Person(name=choi, age=29))
[ INFO] (main) | cancel()
subscribe: true

###  Flux.hasElements()

[ INFO] (main) | onSubscribe([Synchronous Fuseable] FluxArray.ArraySubscription)
[ INFO] (main) | request(unbounded)
[ INFO] (main) | onNext(Person(name=choi, age=29))
[ INFO] (main) | cancel()
subscribe: true

###  Flux.hasElement()

[ INFO] (main) | onSubscribe([Synchronous Fuseable] FluxArray.ArraySubscription)
[ INFO] (main) | request(unbounded)
[ INFO] (main) | onNext(Person(name=choi, age=29))
[ INFO] (main) | cancel()
subscribe: true
```



## concat(), concatWith(), concatDelayError(), mergeSequential(), merge(), mergeWith()

시퀀스들을 하나로 묶을 때 사용할 수 있는 메서드들이다. 

`concat()`은 인자로 넘어오는 모든 `Publisher` 를 하나로 묶어주는 기능이다. 인자의 순서대로 구독하고 구독이 완료되어야 다음 인자를 구독하는 특징이 있다. 오류가 발생하면 시퀀스를 즉시 중단한다. 인스턴스 메서드로 `concatWith()`를 사용할 수 있다.

만약 오류가 발생하더라도 계속해서 시퀀스를 구독하고싶으면 `concatDelayError()` 를 사용하면 된다. 맨 마지막에 오류가 발생한다.

만약 동기식? 으로 `concat()` 을 사용하는게 아니라 바로바로 시퀀스들을 구독하고싶으면 `mergeSequential()`을 사용해볼 수 있다. `mergeSequential()`은 모든 시퀀스들에 대해 즉시 subscribe를 시도하고 `concat()` 과 동일하게 구독순서에 최종적으로 따라 병합시켜준다.

`merge()`를 사용하면 실제 subscribe 시점에 넘어오는 데이터들에 대해 비동기적으로 병합을 할 수 있게 해준다. `mergeSequential()` 은 최종적으로 구독 순서에 따라 시퀀셜하게 병합하지만 `merge()`는 즉시 병합한다. 인스턴스 메서드로 `merge()`를 사용할 수 있다.



```kotlin
package me.sup2is.reactiveexam.ch02

import me.sup2is.reactiveexam.Person
import reactor.core.publisher.Flux

fun main(args: Array<String>) {

    val group1 = Flux.just(
        Person(name = "choi", age = 29),
        Person(name = "woo", age = 31)
    )

    val group2 = Flux.just(
        Person(name = "yoon", age = 30),
        Person(name = "hwang", age = 30)
    )

    println("\n###  Flux.concat()\n")

    Flux.concat(group1, group2)
        .log()
        .subscribe { println("subscribe: $it") }

    println("\n###  Flux.concatWith()\n")

    group1.concatWith(group2)
        .log()
        .subscribe { println("subscribe: $it") }

    println("\n###  Flux.concatDelayError()\n")

    Flux.concatDelayError(group1, group2)
        .log()
        .subscribe { println("subscribe: $it") }

    println("\n###  Flux.mergeSequential()\n")

    Flux.mergeSequential(group1, group2)
        .log()
        .subscribe { println("subscribe: $it") }

    println("\n###  Flux.merge()\n")

    Flux.merge(group1)
        .log()
        .subscribe { println("subscribe: $it") }

    println("\n###  Flux.mergeWith()\n")

    group1.mergeWith(group2)
        .log()
        .subscribe { println("subscribe: $it") }
}

// console

###  Flux.concat()

[ INFO] (main) onSubscribe(FluxConcatArray.ConcatArraySubscriber)
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

###  Flux.concatWith()

[ INFO] (main) onSubscribe(FluxConcatArray.ConcatArraySubscriber)
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

###  Flux.concatDelayError()

[ INFO] (main) onSubscribe(FluxConcatArray.ConcatArrayDelayErrorSubscriber)
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

###  Flux.mergeSequential()

[ INFO] (main) onSubscribe(FluxMergeSequential.MergeSequentialMain)
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

###  Flux.merge()

[ INFO] (main) | onSubscribe([Synchronous Fuseable] FluxArray.ArraySubscription)
[ INFO] (main) | request(unbounded)
[ INFO] (main) | onNext(Person(name=choi, age=29))
subscribe: Person(name=choi, age=29)
[ INFO] (main) | onNext(Person(name=woo, age=31))
subscribe: Person(name=woo, age=31)
[ INFO] (main) | onComplete()

###  Flux.mergeWith()

[ INFO] (main) onSubscribe(FluxFlatMap.FlatMapMain)
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
```



## zip(), zipWith()

이 메서드들도 시퀀스를 하나로 묶을때 사용 가능하다.

`zip()`은 넘어오는 모든 소스들이 요소를 방출할 때까지 기다렸다가 미리 정의된 `Function`에 의해 하나의 값으로 방출될때 사용할 수 있다. 인스턴스 메서드는 `zipWith()`를 사용하면 된다. `Mono`와 `Flux` 모두 갖고 있다.



```kotlin
package me.sup2is.reactiveexam.ch02

import me.sup2is.reactiveexam.Person
import reactor.core.publisher.Flux
import reactor.core.publisher.Mono

fun main(args: Array<String>) {

    val group1 = Flux.just(
        Person(name = "choi", age = 29),
        Person(name = "woo", age = 31)
    )

    println("\n###  Flux.zip()\n")

    val biFunction: (Person, Int) -> String = {
            person, int ->
        "index: $int = ${person.name}"
    }

    Flux.zip(group1, Flux.just(1, 2, 3, 4, 5), biFunction)
        .log()
        .subscribe { println("subscribe: $it") }

    println("\n###  Flux.zipWith()\n")

    group1.zipWith(Flux.just(1, 2, 3, 4, 5), biFunction)
        .log()
        .subscribe { println("subscribe: $it") }

    val biFunction2: (Person, Person) -> String = {
            person1, person2 ->
        "${person1.name} and ${person2.name}"
    }

    println("\n###  Mono.zip()\n")

    val choi = Mono.just(Person(name = "choi", age = 29))

    val woo = Mono.just(Person(name = "woo", age = 31))

    Mono.zip(choi, woo, biFunction2)
        .log()
        .subscribe { println("subscribe: $it") }

    println("\n###  Mono.zipWith()\n")

    choi.zipWith(woo, biFunction2)
        .log()
        .subscribe { println("subscribe: $it") }
}

// console

###  Flux.zip()

[ INFO] (main) onSubscribe(FluxZip.ZipCoordinator)
[ INFO] (main) request(unbounded)
[ INFO] (main) onNext(index: 1 = choi)
subscribe: index: 1 = choi
[ INFO] (main) onNext(index: 2 = woo)
subscribe: index: 2 = woo
[ INFO] (main) onComplete()

###  Flux.zipWith()

[ INFO] (main) onSubscribe(FluxZip.ZipCoordinator)
[ INFO] (main) request(unbounded)
[ INFO] (main) onNext(index: 1 = choi)
subscribe: index: 1 = choi
[ INFO] (main) onNext(index: 2 = woo)
subscribe: index: 2 = woo
[ INFO] (main) onComplete()

###  Mono.zip()

[ INFO] (main) onSubscribe([Fuseable] MonoZip.ZipCoordinator)
[ INFO] (main) request(unbounded)
[ INFO] (main) onNext(choi and woo)
subscribe: choi and woo
[ INFO] (main) onComplete()

###  Mono.zipWith()

[ INFO] (main) onSubscribe([Fuseable] MonoZip.ZipCoordinator)
[ INFO] (main) request(unbounded)
[ INFO] (main) onNext(choi and woo)
subscribe: choi and woo
[ INFO] (main) onComplete()
```

## and(), when(), firstWithSignal()

`Mono`의 `and()`는 이 모노와 다른 `Publisher`의 종료 신호를 `Mono<Void>` 에 결합해서 리턴해준다. `when()`은 N개의 Publisher가 모두 종료되어야 `Mono<Void>` 으로 리턴해준다. 모든 소스들의 종료신호를 받아야 할 때 사용하는 것 같다.

`firstWithSignal()` 은 인자로 넘어간 `Publisher`들 중 가장 먼저 값을 방출하는 시퀀스만 구독이 필요할때 사용한다.

```kotlin

package me.sup2is.reactiveexam.ch02

import me.sup2is.reactiveexam.Person
import reactor.core.publisher.Flux
import reactor.core.publisher.Mono

fun main(args: Array<String>) {

    val group1 = Flux.just(
        Person(name = "choi", age = 29),
        Person(name = "woo", age = 31)
    )

    val group2 = Flux.just(
        Person(name = "yoon", age = 30),
        Person(name = "hwang", age = 30)
    )

    val choi = Mono.just(Person(name = "choi", age = 29))

    val woo = Mono.just(Person(name = "woo", age = 31))

    println("\n###  Mono.and()\n")

    choi.and(woo)
        .log()
        .subscribe { println("subscribe: $it") }

    println("\n###  Mono.when()\n")

    Mono.`when`(choi, woo)
        .log()
        .subscribe { println("subscribe: $it") }

    println("\n###  Mono.firstWithSignal()\n")

    Mono.firstWithSignal(choi, woo)
        .log()
        .subscribe { println("subscribe: $it") }

    println("\n###  Flux.firstWithSignal()\n")

    Flux.firstWithSignal(group1, group2)
        .log()
        .subscribe { println("subscribe: $it") }
}

// console

###  Mono.and()

[ INFO] (main) onSubscribe([Fuseable] MonoWhen.WhenCoordinator)
[ INFO] (main) request(unbounded)
[ INFO] (main) onComplete()

###  Mono.when()

[ INFO] (main) onSubscribe([Fuseable] MonoWhen.WhenCoordinator)
[ INFO] (main) request(unbounded)
[ INFO] (main) onComplete()

###  Mono.firstWithSignal()

[ INFO] (main) onSubscribe(FluxFirstWithSignal.RaceCoordinator)
[ INFO] (main) request(unbounded)
[ INFO] (main) onNext(Person(name=choi, age=29))
subscribe: Person(name=choi, age=29)
[ INFO] (main) onComplete()

###  Flux.firstWithSignal()

[ INFO] (main) onSubscribe(FluxFirstWithSignal.RaceCoordinator)
[ INFO] (main) request(unbounded)
[ INFO] (main) onNext(Person(name=choi, age=29))
subscribe: Person(name=choi, age=29)
[ INFO] (main) onNext(Person(name=woo, age=31))
subscribe: Person(name=woo, age=31)
[ INFO] (main) onComplete()
```



## switchMap(), switchOnNext()

`switchMap()`은 인자로 넘어간 `Function`를 통해 생성된 새로운 `Publisher` 타입으로 전환해주고 이런 요소는 다시 `Flux` 타입으로 방출된다.

`switchOnNext()`는 인자의 순서대로 `Publisher`를 미러링하는 Flux를 만들고 다음 `Publisher`가 값을 올릴때까지 이전 `Publisher`가 데이터를 전달한다. 마지막 `Publisher`의 데이터 전달이 끝나면 완료된다.

```kotlin

package me.sup2is.reactiveexam.ch02

import me.sup2is.reactiveexam.Person
import reactor.core.publisher.Flux
import reactor.core.publisher.Mono

fun main(args: Array<String>) {

    val group1 = Flux.just(
        Person(name = "choi", age = 29),
        Person(name = "woo", age = 31)
    )

    val group2 = Flux.just(
        Person(name = "yoon", age = 30),
        Person(name = "hwang", age = 30)
    )

    println("\n###  Flux.switchMap()\n")

    group1.switchMap {
        Mono.just(it.name)
    }.log()
        .subscribe { println("subscribe: $it") }

    println("\n###  Flux.switchOnNext()\n")

    Flux.switchOnNext(Flux.just(group1, group2))
        .log()
        .subscribe { println("subscribe: $it") }
}

// console

###  Flux.switchMap()

[ INFO] (main) onSubscribe(FluxSwitchMap.SwitchMapMain)
[ INFO] (main) request(unbounded)
[ INFO] (main) onNext(choi)
subscribe: choi
[ INFO] (main) onNext(woo)
subscribe: woo
[ INFO] (main) onComplete()

###  Flux.switchOnNext()

[ INFO] (main) onSubscribe(FluxSwitchMap.SwitchMapMain)
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
```



## repeat(), interval()

`repeat()`은 `Publisher`를 얼마나 반복시킬지 정할 수 있고 `interval()`로 간격을 둘 수 있다.

```kotlin

package me.sup2is.reactiveexam.ch02

import me.sup2is.reactiveexam.Person
import reactor.core.publisher.Flux
import java.time.Duration

fun main(args: Array<String>) {

    val group1 = Flux.just(
        Person(name = "choi", age = 29),
        Person(name = "woo", age = 31)
    )

    println("\n###  Flux.repeat()\n")

    group1.repeat(2)
        .log()
        .subscribe { println("subscribe: $it") }

    println("\n###  Flux.interval()\n")

    Flux.interval(Duration.ofSeconds(1))
        .flatMap {
            group1
        }
        .repeat(2)
        .log()
        .subscribe { println("subscribe: $it") }

    Thread.sleep(3000)
}

// console

###  Flux.repeat()

[ INFO] (main) onSubscribe(FluxRepeat.RepeatSubscriber)
[ INFO] (main) request(unbounded)
[ INFO] (main) onNext(Person(name=choi, age=29))
subscribe: Person(name=choi, age=29)
[ INFO] (main) onNext(Person(name=woo, age=31))
subscribe: Person(name=woo, age=31)
[ INFO] (main) onNext(Person(name=choi, age=29))
subscribe: Person(name=choi, age=29)
[ INFO] (main) onNext(Person(name=woo, age=31))
subscribe: Person(name=woo, age=31)
[ INFO] (main) onNext(Person(name=choi, age=29))
subscribe: Person(name=choi, age=29)
[ INFO] (main) onNext(Person(name=woo, age=31))
subscribe: Person(name=woo, age=31)
[ INFO] (main) onComplete()

###  Flux.interval()

[ INFO] (main) onSubscribe(FluxRepeat.RepeatSubscriber)
[ INFO] (main) request(unbounded)
[ INFO] (parallel-1) onNext(Person(name=choi, age=29))
subscribe: Person(name=choi, age=29)
[ INFO] (parallel-1) onNext(Person(name=woo, age=31))
subscribe: Person(name=woo, age=31)
[ INFO] (parallel-1) onNext(Person(name=choi, age=29))
subscribe: Person(name=choi, age=29)
[ INFO] (parallel-1) onNext(Person(name=woo, age=31))
subscribe: Person(name=woo, age=31)
[ INFO] (parallel-1) onNext(Person(name=choi, age=29))
subscribe: Person(name=choi, age=29)
[ INFO] (parallel-1) onNext(Person(name=woo, age=31))
subscribe: Person(name=woo, age=31)
```



## defaultIfEmpty(), switchIfEmpty()

`defaultIfEmpty()`, `switchIfEmpty()` 모두 `Mono`, `Flux` 에 있는 메서드다. `defaultIfEmpty()` 는 만약 값이 비어있을때 T 타입의 default값 데이터를 정의할 수 있고 `switchIfEmpty()` 는 `Publisher<T>` 타입의 default 값을 정의할 수 있다.

```kotlin

package me.sup2is.reactiveexam.ch02

import me.sup2is.reactiveexam.Person
import reactor.core.publisher.Flux
import reactor.core.publisher.Mono

fun main(args: Array<String>) {

    val group1 = Flux.just(
        Person(name = "choi", age = 29),
        Person(name = "woo", age = 31)
    )

    val choi = Mono.just(Person(name = "choi", age = 29))

    println("\n###  Mono.defaultIfEmpty()\n")

    Mono.empty<Person>()
        .defaultIfEmpty(Person(name = "choi", age = 29))
        .log()
        .subscribe { println("subscribe: $it") }

    println("\n###  Flux.defaultIfEmpty()\n")

    Flux.empty<Person>()
        .defaultIfEmpty(Person(name = "woo", age = 31))
        .log()
        .subscribe { println("subscribe: $it") }

    println("\n###  Mono.switchIfEmpty()\n")

    Mono.empty<Person>()
        .switchIfEmpty(choi)
        .log()
        .subscribe { println("subscribe: $it") }

    println("\n###  Flux.switchIfEmpty()\n")

    Flux.empty<Person>()
        .switchIfEmpty(group1)
        .log()
        .subscribe { println("subscribe: $it") }
}

// console

###  Mono.defaultIfEmpty()

[ INFO] (main) | onSubscribe([Synchronous Fuseable] Operators.ScalarSubscription)
[ INFO] (main) | request(unbounded)
[ INFO] (main) | onNext(Person(name=choi, age=29))
subscribe: Person(name=choi, age=29)
[ INFO] (main) | onComplete()

###  Flux.defaultIfEmpty()

[ INFO] (main) onSubscribe([Fuseable] FluxDefaultIfEmpty.DefaultIfEmptySubscriber)
[ INFO] (main) request(unbounded)
[ INFO] (main) onNext(Person(name=woo, age=31))
subscribe: Person(name=woo, age=31)
[ INFO] (main) onComplete()

###  Mono.switchIfEmpty()

[ INFO] (main) onSubscribe(FluxSwitchIfEmpty.SwitchIfEmptySubscriber)
[ INFO] (main) request(unbounded)
[ INFO] (main) onNext(Person(name=choi, age=29))
subscribe: Person(name=choi, age=29)
[ INFO] (main) onComplete()

###  Flux.switchIfEmpty()

[ INFO] (main) onSubscribe(FluxSwitchIfEmpty.SwitchIfEmptySubscriber)
[ INFO] (main) request(unbounded)
[ INFO] (main) onNext(Person(name=choi, age=29))
subscribe: Person(name=choi, age=29)
[ INFO] (main) onNext(Person(name=woo, age=31))
subscribe: Person(name=woo, age=31)
[ INFO] (main) onComplete()
```



## ignoreElements(), then(), thenEmpty(), thenReturn(), thenMany(), delayUntil()

`ignoreElements()` 는 주어진 데이터들은 무시하고 종료신호를 얻을때 사용한다.

인자가 없는`then()`은 `ignoreElements()` 와 비슷하게 종료신호를 얻을때 사용하고 추가로 오류 신호도 얻을 수 있다. 인자가 있는 `then()`은 `A.then(B)` 일 때 A는 완성하지만 값은 무시하고 B의 값을 반환하고 완료한다.

`A.thenEmpty(B)` 도 인자가 있는 `then()` 과 비슷하지만 B가 `Mono<Void>` 타입이어야 한다. A의 값은 무시하고 최종적으로 반환하는 타입도 `Mono<Void>` 타입이다.

`A.thenReturn(B)` 도 A가 성공적으로 완료되도록 한 다음 인자로 넘어간 B 값을 내보낸다. 이때 다음 인자값은 `T` 타입이다.

`A.thenMany(B)`도 마찬가지로 A가 성공적으로 완료되도록 한 다음 A의 값은 무시하고 인자로 넘어온 `Flux`(B) 를 방출해준다.

`A.delayUntil(B)` 는 A가 방출될 때까지 기다리고 방출되더라도 주어진 B가 완료될 때까지 기다린 뒤에 최종적으로 완료된다.

```kotlin

package me.sup2is.reactiveexam.ch02

import me.sup2is.reactiveexam.Person
import reactor.core.publisher.Flux
import reactor.core.publisher.Mono

fun main(args: Array<String>) {

    val group1 = Flux.just(
        Person(name = "choi", age = 29),
        Person(name = "woo", age = 31)
    )

    val choi = Mono.just(Person(name = "choi", age = 29))

    val woo = Mono.just(Person(name = "woo", age = 31))

    println("\n###  Mono.ignoreElements()\n")

    Mono.ignoreElements(choi)
        .log()
        .subscribe { println("subscribe: $it") }

    println("\n###  Mono.then()\n")

    choi.then()
        .log()
        .subscribe { println("subscribe: $it") }

 
    println("\n###  Mono.then()\n")

    choi.then(woo)
        .log()
        .subscribe { println("subscribe: $it") }
  
    println("\n###  Mono.thenEmpty()\n")

    choi.thenEmpty(Mono.empty())
        .log()
        .subscribe { println("subscribe: $it") }

    println("\n###  Mono.thenReturn()\n")

    choi.thenReturn(Person(name = "woo", age = 31))
        .log()
        .subscribe { println("subscribe: $it") }

    println("\n###  Mono.thenMany()\n")

    choi.thenMany(group1)
        .log()
        .subscribe { println("subscribe: $it") }

    println("\n###  Mono.delayUntil()\n")

    choi.delayUntil { woo }
        .log()
        .subscribe { println("subscribe: $it") }
}

// console

###  Mono.ignoreElements()

[ INFO] (main) onSubscribe(MonoIgnoreElements.IgnoreElementsSubscriber)
[ INFO] (main) request(unbounded)
[ INFO] (main) onComplete()

###  Mono.then()

[ INFO] (main) onSubscribe(MonoIgnoreElements.IgnoreElementsSubscriber)
[ INFO] (main) request(unbounded)
[ INFO] (main) onComplete()

###  Mono.then()

[ INFO] (main) onSubscribe(MonoIgnoreThen.ThenIgnoreMain)
[ INFO] (main) request(unbounded)
[ INFO] (main) onNext(Person(name=woo, age=31))
subscribe: Person(name=woo, age=31)
[ INFO] (main) onComplete()

###  Mono.thenEmpty()

[ INFO] (main) onSubscribe(MonoIgnoreThen.ThenIgnoreMain)
[ INFO] (main) request(unbounded)
[ INFO] (main) onComplete()

###  Mono.thenReturn()

[ INFO] (main) onSubscribe(MonoIgnoreThen.ThenIgnoreMain)
[ INFO] (main) request(unbounded)
[ INFO] (main) onNext(Person(name=woo, age=31))
subscribe: Person(name=woo, age=31)
[ INFO] (main) onComplete()

###  Mono.thenMany()

[ INFO] (main) onSubscribe(FluxConcatArray.ConcatArraySubscriber)
[ INFO] (main) request(unbounded)
[ INFO] (main) onNext(Person(name=choi, age=29))
subscribe: Person(name=choi, age=29)
[ INFO] (main) onNext(Person(name=woo, age=31))
subscribe: Person(name=woo, age=31)
[ INFO] (main) onComplete()

###  Mono.delayUntil()

[ INFO] (main) onSubscribe(MonoDelayUntil.DelayUntilCoordinator)
[ INFO] (main) request(unbounded)
[ INFO] (main) onNext(Person(name=choi, age=29))
subscribe: Person(name=choi, age=29)
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

  
