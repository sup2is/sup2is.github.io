---
layout: post
title: "Java의 Synchronized 키워드"
tags: [Java, Synchronized]
date: 2020-11-02
comments: true
---



<br>

# OverView

이번시간에는 java에서 synchronized 키워드에 대해서 알아보도록 하겠다.

# multi-thread와 race-condition

multi-thread 환경에서 공유되는 자원을 서로 다른 thread가 동시에 접근해서 변경하려고하면 race-condition이 발생한다. 이런 race-condition은 데이터 정합성과 크게 연관되어 있기 때문에 multi-thread환경에서는 반드시 주의해야 한다.

간단한 예로 Thread A, Thread B가 있다고 가정했을때 하나의 공유되는 인스턴스의 class 변수인 `count`를 증가시키는 상황을 살펴보자.

![Screenshot-2020-03-27-at-06 53 27-1024x235](https://user-images.githubusercontent.com/30790184/97866025-7dca5a00-1d4e-11eb-990b-0ca1b0cc3247.png)

출처 : [https://www.baeldung.com/java-testing-multithreaded](https://www.baeldung.com/java-testing-multithreaded)

위와 같은 상황은 Thread A, Thread B가 격리된 상태로 동작하기 때문에 `count` 값이 2번 증가된 값으로 출력된다. 하지만 만약 동시에 같은 자원에 접근한다면 어떻게 될까?

![Screenshot-2020-03-27-at-06 54 15-1024x243](https://user-images.githubusercontent.com/30790184/97866022-7c992d00-1d4e-11eb-9766-983692fbea39.png)

출처 : [https://www.baeldung.com/java-testing-multithreaded](https://www.baeldung.com/java-testing-multithreaded)

위와 같은 상황은 Thread A, Thread B가 동시에 같은 값을 읽고 같은 값으로 증가시키기 때문에 `count` 값은 2가아닌 1로 출력이된다. 이는 데이터 정합성에 있어서 매우 주의해야하는 상황이다.

아래는 실제 `MyObject.java` 를 생성하고 테스트한 결과값이다.

**MyObject.java**

```java
package me.sup2is;

public class MyObject {
    int count = 0;

    public void increaseCnt() {
        this.count ++;
    }

    public int getCount() {
        return count;
    }

    @Override
    public String toString() {
        return "MyObject{" +
            "count=" + count +
            '}';
    }
}
```

 <br>

**MyObjectTest.java**

```java
package me.sup2is;

import org.junit.jupiter.api.RepeatedTest;
import org.junit.jupiter.api.Test;

import java.util.concurrent.CountDownLatch;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

import static org.junit.jupiter.api.Assertions.*;

class MyObjectTest {

    @RepeatedTest(5)
    public void test_concurrency_not_safe() throws Exception{
        //given
        int numberOfThreads = 1000;
        ExecutorService service = Executors.newFixedThreadPool(numberOfThreads);
        CountDownLatch latch = new CountDownLatch(numberOfThreads);
        MyObject counter = new MyObject();

        //when
        for (int i = 0; i < numberOfThreads; i++) {
            service.execute(() -> {
                counter.increaseCnt();
                latch.countDown();
            });
        }

        //then
        latch.await();
        assertEquals(numberOfThreads, counter.getCount());
    }
}
```

- **Executors.newFixedThreadPool(numberOfThreads)**: **numberOfThreads**만큼의 스레드 풀을 생성
- **new CountDownLatch(numberOfThreads)**: CountDownLatch는 thread 관련 테스트할때 유용한데 간단하게 설명하면 인스턴스 생성시점에 **int형 인자**를 주었을때 `latch.countDown();`을 주어진 **int형 인자만큼** 실행시켜야 `latch.await();` 이후 코드가 실행됨. 

<br>

위 코드를 테스트해보면 다음과 같은 결과가 나온다.

![20201102_233733](https://user-images.githubusercontent.com/30790184/97880465-8d549d80-1d64-11eb-9605-21232f2c48c9.png)

운이 좋을 경우 1~2개는 성공할 수있지만 대부분의 경우는 실패할 것이다.

<br>

java에서는 이런 multi-thread 환경에서 동시성을 제어하는 키워드인 `synchronized` 를 제공한다.

# synchronized

java의 synchronized는 총 세가지로 분류할 수 있다. 

- **Instance methods**
- **Static methods**
- **Code blocks**

## Instance Method 내부 synchronized

가장 간단한 방법은 값을 증가하는 메서드에 `synchronized` 키워드를 붙이는 방법이다. 

**MyObject.java**

```java
    public synchronized void increaseCntWithSynchronized() {
        this.count ++;
    }
```

**MyObjectTest.java**

```java
    @RepeatedTest(5)
    public void test_concurrency_sync_method() throws Exception{
        //given
        int numberOfThreads = 1000;
        ExecutorService service = Executors.newFixedThreadPool(numberOfThreads);
        CountDownLatch latch = new CountDownLatch(numberOfThreads);
        MyObject counter = new MyObject();

        //when
        for (int i = 0; i < numberOfThreads; i++) {
            service.execute(() -> {
                counter.increaseCntWithSynchronized();
                latch.countDown();
            });
        }

        //then
        latch.await();
        assertEquals(numberOfThreads, counter.getCount());
    }
```



![20201102_233747](https://user-images.githubusercontent.com/30790184/97880471-8e85ca80-1d64-11eb-81aa-deb37133c030.png)

인스턴스 메서드에 `synchronized` 를 사용하는 방법은 메서드를 소유한 클래스의 인스턴스를 통해 동기화하기 때문에 **클래스 인스턴스 당 하나의 thread 만이 메서드를 실행**할 수 있다.



## Static Method 내부 synchronized

두번째 방법은 static method에 `synchronized`를 사용하는 방법이다. 이 방법은 위에서 설명한 방법과 유사해보이지만 약간 다르다.

**MyObject.java**

```java
    static int staticCnt = 0;


...


	//static method에 synchronized 추가
    public static synchronized void increaseCntWithStaticSynchronized() {
        staticCnt ++;
    }
```

**MyObjectTest.java**

```java
    @RepeatedTest(5)
    public void test_concurrency_static_sync_method() throws Exception{
        //given
        int numberOfThreads = 1000;
        ExecutorService service = Executors.newFixedThreadPool(numberOfThreads);
        CountDownLatch latch = new CountDownLatch(numberOfThreads);

        //when
        for (int i = 0; i < numberOfThreads; i++) {
            service.execute(() -> {
                MyObject.increaseCntWithStaticSynchronized();
                latch.countDown();
            });
        }

        //then
        latch.await();
        assertEquals(numberOfThreads, MyObject.getStaticCnt());

        MyObject.staticCnt = 0;
    }
```

![20201102_233759](https://user-images.githubusercontent.com/30790184/97880472-8e85ca80-1d64-11eb-8063-3e4e78a19ffd.png)

java의 static 키워드의 특징 상 JVM 힙의 Metaspace에 저장된다. 이 static method 들은 선언된 클래스와 연관이 있기 때문에 클래스의 선언 없이 변수에 접근하거나 메서드를 호출할 수 있다. 따라서 위에서 언급한 방법과는 다르게 **선언된 클래스 당 하나의 thread 만이 메서드를 실행**할 수 있다.

## Code Blocks synchronized

 `synchronized` 블록은 메서드 전체에 `synchronized`를 적용하지 않고 싶을때 사용할 수 있다.

**MyObject.java**

```java
    public synchronized void increaseCntWithStaticBlock() {
        synchronized (this) {
            this.count ++;
        }
    }
```

**MyObjectTest.java**

```java
    @RepeatedTest(5)
    public void test_concurrency_sync_block() throws Exception{
        //given
        int numberOfThreads = 1000;
        ExecutorService service = Executors.newFixedThreadPool(numberOfThreads);
        CountDownLatch latch = new CountDownLatch(numberOfThreads);
        MyObject counter = new MyObject();

        long start = System.currentTimeMillis();
        //when
        for (int i = 0; i < numberOfThreads; i++) {
            service.execute(() -> {
                counter.increaseCntWithStaticBlock();
                latch.countDown();
            });
        }
        long end = System.currentTimeMillis();
        System.out.println((end - start));

        //then
        latch.await();
        assertEquals(numberOfThreads, counter.getCount());
    }
```


![20201102_233814](https://user-images.githubusercontent.com/30790184/97880473-8f1e6100-1d64-11eb-9b2a-316cdd1fc75d.png)

이 방법을 사용한다면 중첩해서 락을 획득할 수 있다.

```java
    public synchronized void increaseCntWithStaticBlock() {
        synchronized (this) {
            this.count ++;
            synchronized (this) {
                System.out.println("연속적으로 lock 획득 가능" + count);
            }
        }
    }
```

# synchronized와 Moniter

java의 `synchronized`는 Monitor를 이용해 Thread의 동기화를 보장한다. 모든 객체는 하나의 Monitor를 가지고 있고 Monitor는 하나의 Thread만을 소유할 수 있다. Monitor를 소유하고 있는 Thread가 Monitor를 해제할 때까지 Wait Queue에서 대기해야 한다. 관련해서 [https://www.kdata.or.kr/info/info_04_view.html?field=&keyword=&type=techreport&page=18&dbnum=183741&mode=detail&type=techreport](https://www.kdata.or.kr/info/info_04_view.html?field=&keyword=&type=techreport&page=18&dbnum=183741&mode=detail&type=techreport) 이 글을 읽어보면 도움이 될 수 있다.

# 마무리

실제로 실무에서 `synchronized`를 사용하는 경우는 많지 않은데 이 방법은 성능에 치명적일 수 있기 때문이다. 따라서 `synchronized`의 사용은 지양해야한다. 동시성을 보장해야 하는 경우에는 자바가 제공하는 thread-safe한 자료구조 (`ConcurrentHashMap`, `BlockingQueue` 등) 를 사용해야 한다.



<br>

***

포스팅은 여기까지 하겠습니다. 퍼가실때는 출처를 반드시 남겨주세요!

예제: [https://github.com/sup2is/study/tree/master/java/java-synchronized-test](https://github.com/sup2is/study/tree/master/java/java-synchronized-test)

<br>

**References**

- [https://www.baeldung.com/java-synchronized](https://www.baeldung.com/java-synchronized)
- https://www.baeldung.com/java-testing-multithreaded
- [https://www.linkedin.com/pulse/static-variables-methods-java-where-jvm-stores-them-kotlin-malisciuc](https://www.linkedin.com/pulse/static-variables-methods-java-where-jvm-stores-them-kotlin-malisciuc)
- [https://www.kdata.or.kr/info/info_04_view.html?field=&keyword=&type=techreport&page=18&dbnum=183741&mode=detail&type=techreport](https://www.kdata.or.kr/info/info_04_view.html?field=&keyword=&type=techreport&page=18&dbnum=183741&mode=detail&type=techreport)
