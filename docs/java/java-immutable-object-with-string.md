---
layout: post
title: "Java의 가변객체와 불변객체 feat.String"
tags: [Java, String, StringBuffer, StringBuilder, Mutable, Immutable]
comments: true
parent: Java
nav_order: 2
date: 2020-01-29
---



최근에 본 면접에서 아주 기본적인 질문에 대한 답변을 명확하게 하지 못했다 ...

"String, StringBuilder, StringBuffer 의 특징과 차이점에 대해서 말씀해주세요"

순간 벙 쪄서 아.. 이러다가 여차여차 StringBuilder랑 StringBuffer의 차이점만 두루뭉실하게 설명했다... 한심했다 ... 어쨋든 이번시간에는 Java의 불변객체와 String, StringBuffer, StringBuilder에 대해 파헤쳐보자!

<hr>



# Mutable vs Immutable

## OverView

Mutable은 사전적의미로 '변할 수 있는, 가변의' 을 뜻하고 Immutable은 '변할 수 없는, 불변의' 라는 뜻을 갖고 있다.  Java에서 Mutable Object를 '**가변객체**' , Immutable Object는 '**불변객체**' 로도 표현이 가능하다.

<br>

그럼 Java에서 가변객체와 불변객체의 정확한 의미는 어떤것일까?

불변객체는 **Java에서 class의 instance가 생성된 시점부터 내부 상태가 일정하게 유지되는 객체**이다. 말 그대로 변경이 불가능하다. 대표적인 불변객체중 하나인 String class는 객체가 불변객체로 생성이 되기 때문에 한번 생성된 String 객체는 + 연산자 등등의 변경을 시도하려는 행위를 해도 최초에 생성된 객체 자체가 변경되지는 않는다. 이것에 대해서는 아래에서 자세하게 살펴보도록 한다.

<br>

가변객체는 불변객체와는 반대로 **Java에서 class의 instance가 생성된 이후에도 내부 상태 변경이 가능한 객체**이다. 우리가 Spring에서 사용하는 Value Object 객체 등등 많은 객체들이 가변객체이고 위에서 언급한 String class의 가변객체는 StringBuilder와 StringBuffer가 있다. 이것 역시 아래에서 자세하게 살펴보도록 하자!



## Why?

그렇다면 왜 가변객체와 불변객체로 객체의 상태를 mutable 또는 immutable하도록 분리해놨을까?

<br>

불변객체는 위에서 설명한것처럼 class의 instance가 생성된 시점부터 변경이 불가능하기때문에 자연스럽게 multi-thread 환경에서 안전하다. 여러개의 thread가 불변객체에 접근해서 수정하려해도 수정이 불가능하기때문이다. 또 객체 자체가 변경에 폐쇄적인 상태로 설계가 되어야 할 경우에도 불변객체로 선언할 수 있다.

가변객체는 역시 반대로 생각했을때 multi-thread 환경에서 안전하지 않다. 굳이 자세하게 표현하자면 가변객체를 multi-thread 환경에서 사용하려면 별도의 동기화 처리를 해줘야한다 가 맞다. 이렇게 동기화 처리된 객체중 하나가 StringBuffer인데 아래에서 StringBuffer와 StringBuilder에 대한 차이를 자세하게 알아보도록 하겠다.



## How?

객체를 불변객체로 생성하기 위해서는 어떤 방법을 취해야 할까?

<br>

방법은 생각보다 간단하다 객체의 변경에 대해 폐쇄적으로 설계하면 된다.

1. 모든 class field 변수는 final로 선언
2. 모든 class field 변수의 setter 메서드 선언 X
3. class를 상속하지 못하도록 선언 (class를 final로 선언하거나 생성자를 private로 선언)
4. 모든 field 변수가 final이 아닐때 즉 가변객체타입의 field 변수가 있을 경우 그 가변객체 타입의 field변수에 대해 직접적으로 접근하지 못하도록 copy 객체를 생성하여 새로운 인스턴스를 반환하도록 방어적 복사본 전략을 사용



위 네가지 방법을 준수하면 객체를 불변으로 생성할 수 있다.

<hr>

위에서 언급한 불변객체와 가변객체의 차이점을 기반으로 String 과 StringBuffer, StringBuilder를 한번 파헤쳐보자

<br>

# String vs StringBuffer vs StringBuilder

## 1. String

### String Pool

Java에서 String 관련 이야기 중 String Pool에 대한 이야기가 빠질 수 없다. 간단하게 String Pool에 대해서 알아보고 넘어가자.

<br>

![string pool](https://cdn.journaldev.com/wp-content/uploads/2012/11/String-Pool-Java1-450x249.png)

출처:  https://www.journaldev.com/797/what-is-java-string-pool 

<br>

위에 있는 그림을 통해 String Pool에 대해 쉽게 이해할 수 있다. 그림에서 s1과 s2는 "Cat"이라는 문자열 리터럴을 통해 String 객체를 생성한다. 이렇게 `""`를 통해 생성된 String 객체는 String Pool 공간에 할당되고 만약 같은 값을 갖는 String 객체가 다시 생성될 경우 String Pool에 먼저 생성된 객체의 주소값을 반환한다.



그러나 s3의 경우 new String("Cat") 처럼 new 연산자를 사용할 경우 강제적으로 새로운 인스턴스 주소값을 갖도록 사용할 수 있다. 이 방법 역시 String의 intern() 메서드로 String Pool에 접근할 수 있는데 자세한 설명은 생략한다.



아래의 코드를 한번 봐 보자.

> ```java
> 
> package com.journaldev.util;
> 
> public class StringPool {
> 
>     /**
>      * Java String Pool example
>      * @param args
>      */
>     public static void main(String[] args) {
>         String s1 = "Cat";
>         String s2 = "Cat";
>         String s3 = new String("Cat");
>         
>         System.out.println("s1 == s2 :"+(s1==s2));
>         System.out.println("s1 == s3 :"+(s1==s3));
>     }
> 
> }
> ```



```
s1 == s2 :true
s1 == s3 :false
```



<hr>

### Why String is Immutable in Java?

이어서 String이 불변객체로 설계된 이유에 대해서 알아보자.

<br>

1. 첫번째 이유는 바로 위에서 언급한 String Pool의 사용이다. Java에서 가장 많이 사용하는 자료형은 String인데 Runtime 환경에서 동일한 문자열을 필요로하는 경우가 굉장히 많을때 String Pool을 이용하여 heap영역의 많은 공간을 절약할 수 있다. 만약 String이 가변객체라면 같은 주소를 참조하는 cached 된 String 역시 값이 변경되기 때문에 String Pool을 사용하기 위해서는 String 은 불변객체여야한다.
2. 만약 String이 가변객체라면 application의 보안상 치명적인 요소가 된다.  Database Connection은 database의 username, password를 통해서 얻어지는데 String 이 가변객체일 경우 참조되는 값을 변경이 가능하기 때문에 보안상 치명적일 수 있다.
3. String이 불변객체이기때문에 multi-thread 환경에서 안전하다 여러개의 thread가 동시에 접근한다 하더라도 항상 동일한 값을 보장한다.
4. String은 Java의 ClassLoader에 사용되는데 String 자체가 불변하기때문에 ClassLoader가 올바른 class를 로드하는것을 보장해준다.
5. String은 불변객체이기때문에 생성되는 시점 이후부터는 항상 동일한 hashcode를 반환한다. 새로운 hashcode를 계산할 필요가 없기 때문에 Map 객체의 key를 String으로 주는 이유가 바로 이것이다.



<hr>

이어서 말씀드릴 StringBuffer와 StringBuilder는 가변객체이다. 만약 실제 눈으로 확인하고싶다면 class 내부를 확인해보면 된다 String같은 경우는 내부 value라는 char[] field는 final로 선언되어있지만 StringBuilder와 String Buffer는 final로 선언되어 있지 않다.



## 2. StringBuilder vs StringBuffer

StringBuilder와 StringBuffer는 가변객체이기때문에 내부 char[] 배열을 자유자재로 늘려서 사용 가능하다는 점은 동일하다 그렇다면 어떤 차이가 있을까?

<br>

아래는 StringBuilder의 append() 메서드이다.

```java
@Override
public StringBuilder append(String str) {
    super.append(str);
    return this;
}
```
아래는 StringBuffer의 append() 메서드이다.

```java
@Override
public synchronized StringBuffer append(CharSequence s) {
    toStringCache = null;
    super.append(s);
    return this;
}
```


차이점이 보이는가?

StringBuffer같은 경우는 multi-thread 환경에서 사용이 가능하도록 모든 주요 메서드들이 synchronized 처리가 되어있지만 StringBuilder는 아니다. 만약 multi-thread환경에서 안전하게 String 객체를 보장하려면 StringBuffer를 사용해야한다. 만약 굳이 multi-thread 환경을 고려해야 하지 않는다면 StringBuilder를 사용하면된다.



<hr>

이렇게 Java의 불변객체, 가변객체에 대해 알아보고 추가로 String과 StringBuffer, StringBuilder에 대해 알아봤다.  

<br>

포스팅은 여기까지 하겠습니다

<br>

퍼가실때는 링크와 출처를 반드시 명시해주세요. 감사합니다.

<br>



**Reference**

1.  https://www.baeldung.com/java-immutable-object 
2.  https://www.baeldung.com/java-string-immutable 
3.  https://www.journaldev.com/129/how-to-create-immutable-class-in-java 
4.  https://javarevisited.blogspot.com/2010/10/why-string-is-immutable-or-final-in-java.html#axzz6COxVVrSH 
5.  https://www.journaldev.com/802/string-immutable-final-java 
6.  https://www.journaldev.com/797/what-is-java-string-pool 