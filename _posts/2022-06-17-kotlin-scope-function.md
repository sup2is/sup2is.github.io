---
layout: post
title: "Kotlin Scope Function 알아보기"
tags: [Kotlin, Scope Function]
date: 2022-06-17
comments: true
---

<br>

# Table of Contents

* [Overview](#overview)
* [Scope Function](#scope-function)
   * [let](#let)
   * [with](#with)
   * [run](#run)
   * [apply](#apply)
   * [also](#also)
   * [takeIf 와 takeUnless](#takeif-와-takeunless)
* [Scope Function 별 주요 차이점 요약](#scope-function-별-주요-차이점-요약)
* [요약](#요약)

# Overview

이번 시간에는 kotlin에서 제공하는 scope 함수들을 확인해보고 각각 어떤 특징을 갖고 있는지에 대해 확인해보는 시간을 갖도록 하겠다. 이 글은 모두 [https://kotlinlang.org/docs/scope-functions.html](https://kotlinlang.org/docs/scope-functions.html) 를 참고해서 쓴 글이다.

# Scope Function

코틀린 표준 라이브러리에는 객체 컨텍스트 내에서 실행 가능한 여러 함수가 포함되어 있다. 함수의 시그니처들은 람다로 되어있고 이로 인해 함수형 프로그래밍을 구현할 수 있다. 코틀린에서는 이런 함수를 scope function 이라고 부른다.

코틀린 표준 라이브러리에서 제공해주는 scope function은 아래와 같다.

1. let
2. run
3. with
4. apply
5. also (1.1+)
6. takeIf (1.1+)
7. takeUnless (1.1+)

이런 scope함수를 확인하고싶으면 IDE를 사용해서 코드를 찾아도되고 [https://github.com/JetBrains/kotlin/blob/master/libraries/stdlib/src/kotlin/util/Standard.kt](https://github.com/JetBrains/kotlin/blob/master/libraries/stdlib/src/kotlin/util/Standard.kt) 에서 확인할 수도 있다.

scope 함수를 사용하면 코드를 더 간결하고 쉽게 사용할 수 있다.하지만 scope 함수마다 적절한 사용법이 있기 때문에 이런 관례들을 지키는것이 코드를 읽는 사람들에게 더 좋은 영향을 끼칠 수 있을 것이라고 생각한다.

뭐든지 과유불급이다. 이런 유용한 scope 함수들이라도 남용하면 가독성이 떨어지고 이는 곧 시스템 오류로 발전한다. scope 함수의 중첩을 지양해야한다.

기본적으로 scope 함수들은 두가지로 먼저 나눠볼 수 있다.

- 함수(`{블록}`)에서 컨텍스트 객체를 참조하는 방법
  - `this`: `run`, `with`, `apply`
  - `it`: `let`, `also`

`this` 를 사용하는 scope 함수들은 객체 멤버에 대한 기능을 호출하거나 프로퍼티를 할당하는 경우에 사용하는 것을 권장한다.

```kotlin
val adam = Person("Adam").apply { 
    age = 20                       // same as this.age = 20
    city = "London"
}
println(adam)
```

`it`을 사용하는 scope 함수들은 또다른 함수의 파라미터로 사용하는 경우에 사용하는 것을 권장한다.

```kotlin
fun getRandomInt(): Int {
    return Random.nextInt(100).also {
        writeToLog("getRandomInt() generated value $it")
    }
}

val i = getRandomInt()
println(i)
```

- scope 함수의 반환 결과
  - 객체 자신을 리턴한다: `apply`, `also`
  - 함수 타입을 리턴한다: `let`, `run`, `with`

객체 자신을 리턴하는 `apply`와 `also`는 아래와 같이 사용할 수 있다.

```kotlin
val numberList = mutableListOf<Double>()
numberList.also { println("Populating the list") }
    .apply {
        add(2.71)
        add(3.14)
        add(1.0)
    }
    .also { println("Sorting the list") }
    .sort()
```

함수 타입을 리턴하는 `let`, `run`, `with`는 아래와 같이 사용할 수 있다.

```kotlin
val numbers = mutableListOf("one", "two", "three")
val countEndsWithE = numbers.run { 
    add("four")
    add("five")
    count { it.endsWith("e") }
}
println("There are $countEndsWithE elements that end with e.")
```

```kotlin
val numbers = mutableListOf("one", "two", "three")
with(numbers) {
    val firstItem = first()
    val lastItem = last()        
    println("First item: $firstItem, last item: $lastItem")
}
```

위에서 확인한 것처럼 기본적으로 scope 함수들은 모두 객체에서 코드 블록을 실행한다.  모든 함수의 파라미터 시그니처가 함수를 받게 되어있기 때문이다. 이제 각각 함수별로 어떤점이 다르고 어떻게 사용하면 좋을지에 대해 알아보도록 하자.



## let

`let`은 객체를 `it` 라는 이름으로 사용하고 리턴타입은 함수의 반환 타입이다.

```kotlin
@kotlin.internal.InlineOnly
public inline fun <T, R> T.let(block: (T) -> R): R {
    contract {
        callsInPlace(block, InvocationKind.EXACTLY_ONCE)
    }
    return block(this)
}
```

`let`은 주로 non-null 인 값일 경우 실행시키는 방식으로 자주 사용된다.

```kotlin
val str: String? = "Hello"   
//processNonNullString(str)       // compilation error: str can be null
val length = str?.let { 
    println("let() called on $it")        
    processNonNullString(it)      // OK: 'it' is not null inside '?.let { }'
    it.length
}
```

다른 case로 코드의 가독성 향상을 위해 제한된 범위의 scope를 만드는 경우 아래와 같이 사용할 수 있다.

```kotlin
val numbers = listOf("one", "two", "three", "four")
val modifiedFirstItem = numbers.first().let { firstItem ->
    println("The first item of the list is '$firstItem'")
    if (firstItem.length >= 5) firstItem else "!" + firstItem + "!"
}.uppercase()
println("First item after modifications: '$modifiedFirstItem'")
```



## with

`with`는 객체를 `this` 형태로 사용할 수 있고 리턴 타입은 함수의 반환 타입이다.

```kotlin
@kotlin.internal.InlineOnly
public inline fun <T, R> with(receiver: T, block: T.() -> R): R {
    contract {
        callsInPlace(block, InvocationKind.EXACTLY_ONCE)
    }
    return receiver.block()
}
```

`with`는 확장하지 않는 함수 (일반 함수 처럼) 처럼 사용할 수도 있고 scope 함수로도 사용할 수 있다. 

`with`는 리턴타입을 제공하지 않고 블록 내에서 다른 함수를 호출할 때 사용하는것을 권장한다.

```kotlin
val numbers = mutableListOf("one", "two", "three")
with(numbers) {
    println("'with' is called with argument $this")
    println("It contains $size elements")
}
```

물론 객체의 속성을 사용해서 함수의 리턴 타입에서 사용할 수도 있다.

```kotlin
val numbers = mutableListOf("one", "two", "three")
val firstAndLast = with(numbers) {
    "The first element is ${first()}," +
    " the last element is ${last()}"
}
println(firstAndLast)
```





## run

`run`은 객체를 `this` 형태로 사용할 수 있고 리턴타입은 함수의 반환 타입이다.

```kotlin
@kotlin.internal.InlineOnly
public inline fun <R> run(block: () -> R): R {
    contract {
        callsInPlace(block, InvocationKind.EXACTLY_ONCE)
    }
    return block()
}
```

`run`은 객체 초기화와 함수의 반환값을 동시에 구현해야 하는 경우 유용하게 사용할 수 있다.

```kotlin
val service = MultiportService("https://example.kotlinlang.org", 80)

val result = service.run {
    port = 8080
    query(prepareRequest() + " to port $port")
}

// the same code written with let() function:
val letResult = service.let {
    it.port = 8080
    it.query(it.prepareRequest() + " to port ${it.port}")
}
```

다른 방법으로 확장 함수가 아니라 일반 함수를 사용하는 것처럼 `run` 을 사용할 수도 있다.

```kotlin
val hexNumberRegex = run {
    val digits = "0-9"
    val hexDigits = "A-Fa-f"
    val sign = "+-"

    Regex("[$sign]?[$digits$hexDigits]+")
}

for (match in hexNumberRegex.findAll("+123 -FFFF !%*& 88 XYZ")) {
    println(match.value)
}
```



## apply

`apply`는 객체를 `this` 형태로 사용할 수 있고 리턴 타입은 자기 자신(`this`)이다.

```kotlin
@kotlin.internal.InlineOnly
public inline fun <T> T.apply(block: T.() -> Unit): T {
    contract {
        callsInPlace(block, InvocationKind.EXACTLY_ONCE)
    }
    block()
    return this
}
```

`apply`는 `this`타입으로 넘어오는 객체에 대한 속성할당을 할때 유용하게 사용할 수 있다.

```kotlin
val adam = Person("Adam").apply {
    age = 32
    city = "London"        
}
println(adam)

```



## also

`also`는 객체를 `it` 라는 이름으로 사용하고 리턴 타입은 자기 자신(`this`)이다.

```kotlin
@kotlin.internal.InlineOnly
@SinceKotlin("1.1")
public inline fun <T> T.also(block: (T) -> Unit): T {
    contract {
        callsInPlace(block, InvocationKind.EXACTLY_ONCE)
    }
    block(this)
    return this
}
```

`also`는 `it` 로 넘어온 객체에 대한 참조가 필요하거나 파라미터로 사용하는 경우 유용하다. 

```kotlin
val numbers = mutableListOf("one", "two", "three")
numbers
    .also { println("The list elements before adding new one: $it") }
    .add("four")
```





## takeIf 와 takeUnless

`also` 역시 코틀린 1.1에 추가되었지만 그 외에도 `takeIf`와 `takeUnless`가 추가되었다.

`takeIf`는 Predicate를 받아서 만약 결과가 true라면 해당 객체를 리턴해준다. false라면 null을 리턴한다.

```kotlin
@kotlin.internal.InlineOnly
@SinceKotlin("1.1")
public inline fun <T> T.takeIf(predicate: (T) -> Boolean): T? {
    contract {
        callsInPlace(predicate, InvocationKind.EXACTLY_ONCE)
    }
    return if (predicate(this)) this else null
}
```

반대로 `takeUnless`는 Predicate를 받아서 결과가 false라면 해당 객체를 리턴해주고 true라면 null을 리턴한다.

```kotlin
@kotlin.internal.InlineOnly
@SinceKotlin("1.1")
public inline fun <T> T.takeUnless(predicate: (T) -> Boolean): T? {
    contract {
        callsInPlace(predicate, InvocationKind.EXACTLY_ONCE)
    }
    return if (!predicate(this)) this else null
}
```

아래 예제를 확인해보면 쉽게 이해할 수 있다.

```kotlin
val number = Random.nextInt(100)

val evenOrNull = number.takeIf { it % 2 == 0 }
val oddOrNull = number.takeUnless { it % 2 == 0 }
println("even: $evenOrNull, odd: $oddOrNull")
```

`takeIf`와 `takeUnless`는 nullable 타입을 리턴하기 때문에 반드시 안전한 호출 (ex `.?`)을 해야 컴파일이 가능하고 nullable한 값이 올 수 있기 때문에 위에서 확인했던 `let`과 섞어서 사용할 경우 아주 좋다.

```kotlin
fun displaySubstringPosition(input: String, sub: String) {
    input.indexOf(sub).takeIf { it >= 0 }?.let {
        println("The substring $sub is found in $input.")
        println("Its start position is $it.")
    }
}

displaySubstringPosition("010000011", "11")
displaySubstringPosition("010000011", "12")

//same
fun displaySubstringPosition(input: String, sub: String) {
    val index = input.indexOf(sub)
    if (index >= 0) {
        println("The substring $sub is found in $input.")
        println("Its start position is $index.")
    }
}

displaySubstringPosition("010000011", "11")
displaySubstringPosition("010000011", "12")
```



# Scope Function 별 주요 차이점 요약

scope 함수별로 간단하게 차이점을 요약해보면 아래와 같다.

| Function | Object reference | Return value   | Is extension function                        |
| -------- | ---------------- | -------------- | -------------------------------------------- |
| `let`    | `it`             | Lambda result  | Yes                                          |
| `run`    | `this`           | Lambda result  | Yes                                          |
| `run`    | -                | Lambda result  | No: called without the context object        |
| `with`   | `this`           | Lambda result  | No: takes the context object as an argument. |
| `apply`  | `this`           | Context object | Yes                                          |
| `also`   | `it`             | Context object | Yes                                          |

**[short guide]**

- Executing a lambda on non-null objects: `let`
- Introducing an expression as a variable in local scope: `let`
- Object configuration: `apply`
- Object configuration and computing the result: `run`
- Running statements where an expression is required: non-extension `run`
- Additional effects: `also`
- Grouping function calls on an object: `with`



# 요약

- 코틀린 std에서는 scope function을 제공해준다. 종류는 `let`, `run`, `with`, `apply`, `also`, `takeIf`, `takeUnless` 가 있다.
- scope function은 각각 적합한 상황이 있기 때문에 일관성있게 사용하는게 좋을 것 같다.

<br>

***

포스팅은 여기까지 하겠습니다. 감사합니다!



<br>



**[References]**

- [https://kotlinlang.org/docs/scope-functions.html](https://kotlinlang.org/docs/scope-functions.html)
