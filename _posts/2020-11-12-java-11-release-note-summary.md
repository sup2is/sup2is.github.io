---
layout: post
title: "Java11에서 추가된 주요 기능 알아보기"
tags: [Java, Java 11]
date: 2020-11-02
comments: true
---


<br>

# OverView

java 11은 2018년 9월 25일에 정식으로 릴리즈되었다. 지금 이 글을 쓰는 시점보다 약 2년이 지났지만 아직도 제대로 아는게 없어서 한번 정리를 해야겠다고 생각했다. java 11에서 중요 업데이트와 추가된 기능에 대해서 알아보도록 하자.

> 이 글은 Oracle JDK Release Note 기반으로 작성되었다 .[https://www.oracle.com/java/technologies/javase/jdk-11-relnote.html](https://www.oracle.com/java/technologies/javase/jdk-11-relnote.html)

# Important Changes and Information

중요 변경 사항은 다음과 같다.

- **Applets 및 Web Start Applications에서 필요한 deployment stack이 jdk 9에서 deprecated 되었는데 jdk 11에서 완전하게 제거됨**
- **JDK 11 지원 설정 리스트중에서 deployment stack없이 지원되는 브라우저의 전체 섹션이 JDK 11에서 제거됨**
- **Windows, MacOS에서 JRE 설치에 사용할 수 있었던 자동 업데이트는 더이상 사용이 불가함**
- **Windows 및 MacOS에서 이전 릴리즈에 JDK를 설치시 선택적으로 JRE를 설치했는데 JDK 11에서는 제거됨**
- **JRE 또는 server JRE가 제공되지 않음. JDK만 제공되고 사용자는 jlink를 사용해서 작은 사용자 지정 런타임을 만들 수 있음 [what is jlink](https://www.baeldung.com/jlink)**
- **javaFX는 JDK에 더이상 포함되지 않고 openjfx.io에서 관리함**
- **JDK 7, 8, 9, 10에 포함된 Oracle Java Mission Control은 더이상 Oracle JDK에 포함되지 않고 별도의 다운로드를 해야함 [Oracle Java Mission Control](https://blogs.oracle.com/javakr/java-mission-control)**
- **jdk 11이상에서 제공하는 번역 releases 중 여러 나라가 제외되었는데 한국어도 제외됨**
- **Windows용 업데이트된 패키징 형식이 tar.gz에서 .zip으로 변경됨**
- **MacOS용 업데이트 패키징 형식이 .app에서 .dmg로 변경됨**

원문은 다음과 같다.

> - The deployment stack, required for Applets and Web Start Applications, was deprecated in JDK 9 and has been removed in JDK 11.
> - Without a deployment stack, the entire section of supported browsers has been removed from the list of supported configurations of JDK 11.
> - Auto-update, which was available for JRE installations on Windows and macOS, is no longer available.
> - In Windows and macOS, installing the JDK in previous releases optionally installed a JRE. In JDK 11, this is no longer an option.
> - In this release, the JRE or Server JRE is no longer offered. Only the JDK is offered. Users can use `jlink` to create smaller custom runtimes.
> - JavaFX is no longer included in the JDK. It is now available as a separate download from [openjfx.io](https://openjfx.io/).
> - Java Mission Control, which was shipped in JDK 7, 8, 9, and 10, is no longer included with the Oracle JDK. It is now a separate download.
> - Previous releases were translated into English, Japanese, and Simplified Chinese as well as French, German, Italian, Korean, Portuguese (Brazilian), Spanish, and Swedish. However, in JDK 11 and later, French, German, Italian, Korean, Portuguese (Brazilian), Spanish, and Swedish translations are no longer provided.
> - Updated packaging format for Windows has changed from `tar.gz` to `.zip`, which is more common in Windows OSs.
> - Updated package format for macOS has changed from `.app` to `.dmg`, which is more in line with the standard for macOS.



# What's New in JDK 11 - New Features and Enhancements

새롭게 적용되는 feature에서 중요하다고 생각되는것만 정리했다. 전부 자세히 확인하고싶다면 [jdk-11-relnote](https://www.oracle.com/java/technologies/javase/jdk-11-relnote.html) 를 확인하자.



## Unicode 10.0.0 지원

core-libs/java.lang
**[➜](https://www.oracle.com/java/technologies/javase/jdk-11-relnote.html#JDK-8191410) JEP 327 Unicode 10**

- JDK 11 릴리즈에는 유니코드 10.0.0에 대한 지원이 포함되 어있음
- 16,018 개의 새로운 문자, 18개의 새로운 blocks, 10개의 새로운 scripts가 추가됨



## HTTP Client 표준화

- JDK 9, 10에서 HttpClient는 패키지를 포함한 풀네임이 `jdk.incubator.http.HttpClient` 이었는데 JDK 11에서 `java.net.http.HttpClient`로 표준화됨

> java에서 incubator의 의미는 non-final api, non-final tools를 의미함 향후 릴리즈에서 제거되거나 완성된다는 의미



## Collection.toArray(IntFunction) 메서드 추가

- java.util.Collection 인터페이스에 toArray(IntFunction) 메서드가 default 메서드로 추가됨

```java
default <T> T[] toArray(IntFunction<T[]> generator) {
    return toArray(generator.apply(0));
}
```

- 이 메서드를 사용하면 다음과 같은 문법사용이 가능함

```java
 
import java.util.Arrays;
import java.util.List;
 
public class ListToArray {
 
    public static void main(String[] args) {
        List<String> list = List.of("Doc", "Grumpy", "Happy", 
                "Sleepy", "Dopey", "Bashful", "Sneezy");
        
        System.out.println("List to Array example in Java 11:");
        
        // old method
        String[] array1 = list.toArray(new String[list.size()]);
        System.out.println(Arrays.toString(array1));
        
        // new method
        String[] array2 = list.toArray(String[]::new);
        System.out.println(Arrays.toString(array2));
    }
}
```

- 객체 변환시에도 유용함

```java
 
import java.util.Arrays;
import java.util.HashSet;
import java.util.LinkedHashSet;
import java.util.List;
import java.util.Set;
import lombok.ToString;
 
public class CollectionToArray {
 
    public static void main(String[] args) {
        List<Integer> list = List.of(1, 2, 3, 4, 5);
        
        Integer[] array1 = list.toArray(Integer[]::new);
        System.out.println(Arrays.toString(array1));
        
        Set<Integer> hset1 = new LinkedHashSet<>(list);
        hset1.remove(1);
        Integer[] array2 = hset1.toArray(Integer[]::new);
        System.out.println(Arrays.toString(array2));
        
        Set<Country> hset2 = new HashSet<>();
        hset2.add(new Country("ID", "Indonesia"));
        hset2.add(new Country("SG", "Singapore"));
        hset2.add(new Country("MY", "Malaysia"));
        Country[] array3 = hset2.toArray(Country[]::new);
        System.out.println(Arrays.toString(array3));
    }
    
    @ToString
    static class Country {
        String code;
        String name;
        
        Country(String code, String name) {
            this.code = code;
            this.name = name;
        }
    }
}
```

- `Collection.toArray(IntFunction)` 메서드는 기존에 있던 `Collection.toArray(T[])`  메서드를 오버로드한건데 다음과 같은 코드에 컴파일타임 에러를 추론하지 못했음

```java
String[] array1 = list.toArray(null); // 런타임 에러
```

- `Collection.toArray(IntFunction)` 메서드가 추가되면서 컴파일타임 에러를 확인할 수 있음 



## Lazy Allocation of Compiler Threads 기능 추가

- 컴파일러 스레드를 동적으로 제어하기 위해  `-XX:+UseDynamicNumberOfCompilerThreads` 옵션이 추가됨
- 기본적으로 켜져있는 계층형 컴파일 모드에서 VM은 사용 가능한 메모리와 컴파일 요청에 관계없이 많은 수의 컴파일러 스레드를 시작함
- 스레드가 유휴 상태일 때도 메모리를 소비하기 때문이 이옵션을 사용하면 리소스를 효율적으로 사용할 수 있음



## Z Garbage Collector(ZGC) 추가

- 새롭게 추가된 ZGC의 주요 기능
  - 일시 정지 시간이 10ms를 넘어가지 않음
  - 일시 정지 시간이 힙 또는 live set size에 따라 증가하지 않음
  - 수테라바이트 크기의 힙 처리
- ZGC는 Concurrent GC에 해당함 따라서 애플리케이션 실행중에도 동작하기 때문에 응답시간에 큰 영향이 없음
- 현재는 실험기간이고 `-XX:+UnlockExperimentalVMOptions` 과 `-XX : + UseZGC`옵션으로 킬 수 있음 
- linux 64bit에서만 사용 가능
- `-XX:+UseCompressedOops` and `-XX:+UseCompressedClassPointers`  옵션은 기본적으로 비활성화, 활성화해도 효과가 전혀 없음
- `-XX:+ClassUnloading` and `-XX:+ClassUnloadingWithConcurrentMark` 옵션은 기본적으로 비활성화, 활성화해도 효과가 전혀 없음
- Graal과 함께 ZGC를 사용하는 것은 지원되지 않음
- 아래는 GC Pause Times를 ZGC, Parallel GC, G1GC와 비교한 사진임

![](https://cdn.app.compendium.com/uploads/user/e7c690e8-6ff9-102a-ac6d-e4aebca50425/34d3828b-c12e-4c7e-8942-b6b7deb02e12/Image/2a6b88308ed1dbc56465b1af8feb892b/figure2_600w.jpg)

출처 : [https://blogs.oracle.com/javamagazine/understanding-the-jdks-new-superfast-garbage-collectors](https://blogs.oracle.com/javamagazine/understanding-the-jdks-new-superfast-garbage-collectors)





## No-Op Garbage Collector 추가

- Epsilon GC는 실험적으로 등장한 가비지 컬렉터
- 메모리 할당만 처리하고 메모리 재 확보 메커니즘을 구현하지 않음
- 다른 gc의 비용 / 이점을 대조하기 위해 성능 테스트에 유용함





## Nest-Based Access Control 추가

- Java에서 클래스, 인터페이스를 중첩해서 사용 가능한데 이런 중첩유형에서는 private fields, private methods들에 대한 접근 권한을 각각 중첩 클래스마다 갖고 있어서 침범할 수 있었음
- JDK 11에는 외부 및 중첩 클래스 내에서 private access를 지원하는 Nest-Based Access Context가 도입되었음
- 자세한 내용은 블로그 참고 [https://dzone.com/articles/java-11-nest-based-access-control-via-reflection](https://dzone.com/articles/java-11-nest-based-access-control-via-reflection)



## TLS 1.3 추가

- TLS1.3은 아주 많은 개편으로 이전 버전에 비해 상당한 보안 및 성능을 제공함
- 자세한 내용은 공홈 참고 [https://www.oracle.com/java/technologies/javase/jdk-11-relnote.html#JDK-8145252](https://www.oracle.com/java/technologies/javase/jdk-11-relnote.html#JDK-8145252)



## Local-Variable Syntax for Lambda Parameters 추가

- `var` 라는 예약어를 사용해서 람다 매개변수를 선언할때와 local variable 선언시 사용 가능
- `var`를 사용하면 매개 변수의 형식을 유추함
- 복잡한 선언 없이 `var`로 선언하면 타입을 알아서 유추해줌

```java

var myList = new ArrayList<Map<String, List<Integer>>>();

for (var current : myList) {
    // current is infered to type: Map<String, List<Integer>>
    System.out.println(current);
}
```

- `var`를 사용해서 람다 매개변수 타입에 애너테이션을 적용할 수 있음
- `var`를 사용해서 람다 매개변수를 선언했으면 해당 람다의 모든 매개변수에 `var` 타입을 사용해야함

```java
Predicate<String> predicate = (@Nullable var a) -> true;
```



# 마무리

이것 외에도 새롭게 추가된 암호화 관련 기능과 제거된 feature, deprecated된 feature에 대한 정보는 [https://www.oracle.com/java/technologies/javase/jdk-11-relnote.html](https://www.oracle.com/java/technologies/javase/jdk-11-relnote.html)에서 조금 더 자세하게 확인할 수 있다.



<br>

***

포스팅은 여기까지 하겠습니다. 퍼가실때는 출처를 반드시 남겨주세요!

<br>

**References**

- [https://www.dariawan.com/tutorials/java/java-11-convert-collection-array/](https://www.dariawan.com/tutorials/java/java-11-convert-collection-array/)
- [https://winterbe.com/posts/2018/09/24/java-11-tutorial/](https://winterbe.com/posts/2018/09/24/java-11-tutorial/)
- [https://www.oracle.com/java/technologies/javase/jdk-11-relnote.html](https://www.oracle.com/java/technologies/javase/jdk-11-relnote.html)
