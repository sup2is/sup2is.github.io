---
layout: post
title: "Java의 Concurrency 관련 클래스들 사용하기"
tags: [Java, Thread, Concurreny, Executor, ThreadLocal, CountDownLatch, FutureTask]
date: 2021-06-27
comments: true
---

<br>



# Overview

이번시간에는 자바에서 제공하는 Executor 프레임웍과 Thread 관련해서 유용한 클래스들을 알아보는 시간을 갖도록 하겠다.



# Thread Pool 사용하기

자바를 처음 배웠을때는 다음과 같이 스레드를 생성하는 방법을 사용했다.

```java
new Thread(() -> {
    try {
	    //do something ...
    } catch (InterruptedException e) {
	    e.printStackTrace();
    }
}).start();
```

위 방법처럼 Thread를 직접생성해서 사용하는 방법은 다음과 같은 문제점이 있다. 

- **생성된 스레드를 효율적으로 관리할 수 없음**
- **하드웨어에 장착되어있는 프로세서보다 많은 수의 스레드가 만들어져 동작중이라면 실제로는 대부분의 스레드가 대기상태에 머무름. 대기하는 스레드가 많으면 많을수록 cpu를 사용하기 위한 경쟁상태가 더욱 심화되기때문에 더 많은 자원을 소모함**
- **너무 많은 스레드가 생성되어 OOM이 발생한 상황이면 이미 손 쓸 방법이 없음.**

위와 같은 문제점의 핵심은 **스레드의 수가 특정 수준을 넘어간다면 성능이 떨어진다.** 라는 것이다. 따라서 애플리케이션이 만들어 낼 수 있는 스레드의 수에 제한을 두는 것이 현병한 방법이고 높은 양의 작업 요청이 왔을때도 자원이 고갈되어 멈추는 경우가 발생하지 않도록 하는게 중요하다.

자바에서는 스레드를 조금 더 효율적으로 관리하기 위한 `Executor` 프레임웍을 제공한다. 

그중에서도 클라이언트 레벨에서 사용하기 좋은 `ThreadPoolExecutor`를 알아볼 예정인데 자세한 내용은 [https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/ThreadPoolExecutor.html](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/ThreadPoolExecutor.html) 에서 찾아볼 수 있다.

`ThreadPoolExecutor`를 사용하면 위에서 언급한 문제점들을 해결할 수 있고 그외에도 아래와 같은 기능들을 사용할 수 있다.

- **Core and maximum pool sizes**
- **On-demand construction**
- **Creating new threads**
- **Keep-alive times**
- **Queuing**
- **Rejected tasks**
- **Hook methods**
- **Queue maintenance**
- **Reclamation**

직접 `ThreadPoolExecutor`을 커스터마이징하여 사용하는 방법도 있지만 이미 자바에서는 여러가지 팩토리 메서드들을 제공해주고 있다.

`Executors` 클래스는 여러가지 `ThreadPoolExecutor` 구현체를 생성해주는데 아래와 같이 필요한경우에 따라 사용하면된다.

  - `Executors.newFixedThreadPool()`: 처리할 작업이 등록되면 그에 따라 실제 작업할 스레드를 하나씩 생성. 생성할 수 있는 스레드의 최대 개수는 제한되어 있고 제한된 개수까지 스레드를 생성하고 나면 더 이상 생성하지 않고 스레드 수를 유지함

```java
    public static ExecutorService newFixedThreadPool(int nThreads) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>());
    }
```

  - `Executors.newCachedThreadPool()`: 캐시 스레드 풀은 현재 풀에 갖고 있는 스레드의 수가 처리할 작업의 수보다 많아서 쉬는 스레드가 많이 발생할 때 쉬는 스레드를 종료시켜 훨씬 우연하게 대응할 수 있고 처리할 작업의 수가 많아지면 필요한 만큼 스레드를 새로 생성함. 반면에 스레드의 수에는 제한을 두지 않음

```java
    public static ExecutorService newCachedThreadPool() {
        return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                      60L, TimeUnit.SECONDS,
                                      new SynchronousQueue<Runnable>());
    }
```

  - `Executors.newSingleThreadExecutor()`: 단일 스레드로 동작하는 Executor로서 작업을 처리하는 스레드가 단 하나뿐 만약 작업 중에 비정상적으로 종료되면 다시 하나를 생성해 나머지 작업을 실행함 등록된 작업은 설정된 큐의 우선순위 FIFO, LIFO 에 따라 반드시 순차적으로 처리됨

```java
    public static ExecutorService newSingleThreadExecutor() {
        return new FinalizableDelegatedExecutorService
            (new ThreadPoolExecutor(1, 1,
                                    0L, TimeUnit.MILLISECONDS,
                                    new LinkedBlockingQueue<Runnable>()));
    }
```

- `Executors.newSingleThreadExecutor()`: 일정 시간 이후에 실행하거나 주기적으로 작업을 실행할 수 있고 스레드의 수가 고정되어 있는 형태

```java
    public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize) {
        return new ScheduledThreadPoolExecutor(corePoolSize);
    }


...
    

    public ScheduledThreadPoolExecutor(int corePoolSize) {
        super(corePoolSize, Integer.MAX_VALUE,
              DEFAULT_KEEPALIVE_MILLIS, MILLISECONDS,
              new DelayedWorkQueue());
    }


```

간단한 예제로 `Executors.newScheduledThreadPool()`를 사용하여 스케쥴을 구성하는 예제이다.

```java
public class ScheduledExecutorRunnable {

    public static void main(String[] args) {

        ScheduledExecutorService ses = Executors.newScheduledThreadPool(1);

        Runnable task2 = () -> System.out.println("Running task2...");

        task1();

        //run this task after 5 seconds, nonblock for task3
        ses.schedule(task2, 5, TimeUnit.SECONDS);

        task3();

        ses.shutdown();

    }

    public static void task1() {
        System.out.println("Running task1...");
    }

    public static void task3() {
        System.out.println("Running task3...");
    }

}

```

```
Running task1...
Running task3...
Running task2... //display after 5 seconds
```

> 출처 : [https://mkyong.com/java/java-scheduledexecutorservice-examples/](https://mkyong.com/java/java-scheduledexecutorservice-examples/)



# CountDownLatch

[CountDownLatch](https://docs.oracle.com/javase/7/docs/api/java/util/concurrent/CountDownLatch.html)는 다른 스레드에서 수행되는 작업 집합이 완료 될 때까지 하나 이상의 스레드가 대기 할 수 있도록 지원해주는 클래스이다. 이 Latch는 일종의 관문과 같은 형태로 동작하는데 Latch가 터미널 상태에 다다르면 관문이 열리고 모든 스레드가 통과한다고 이해한다면 된다.

`CountDownLatch`는 다음과 같은 상황에서 유용하게 사용할 수 있다.

- **특정 자원을 확보하기 전에는 시작하지 말아야할 작업이 있는 경우**
- **의존성을 갖고 있는 다른 서비스가 시작하기 전에는 특정 서비스가 실행되지 않도록 막아야하는 경우**
- **특정 작업에 필요한 모든 객체가 실행할 준비를 갖출 때까지 기다리는 경우**

아래는 `CountDownLatch`의 간단한 예제이다.

```java
package me.sup2is;

import java.util.concurrent.CountDownLatch;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class CountDownLatchExam {

    public static void main(String[] args) {

        final int nThreads = 10;
        final ExecutorService executorService = Executors.newFixedThreadPool(nThreads);
        final CountDownLatch startGate = new CountDownLatch(1);

        for (int i = 0; i < nThreads; i++) {
            executorService.submit(() -> {
                try {
                    startGate.await(); // <- 여기에서 해당 runnable은 잠금상태
                    System.out.println("run!");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            });
        }

        System.out.println("all thread ready!");
        startGate.countDown();
        executorService.shutdown();
    }

}
```

`CountDownLatch`는 초기화한 count의 개수만큼  `countDown()` 메서드가 호출이 되어야 실행이 가능한 상태가 된다. 위 예제에서는 10개의 스레드를 만들고 실행시키려하지만 `startGate.await()` 메서드에 의해 대기?상태가되고 모든 스레드의 준비가 완료되었다는 **"all thread ready!"** 이후 `startGate.countDown()`를 이용해 초기화한 count를 0으로 만들기 때문에 생성된 Runnable들이 모두 실행되는 구조이다.

```
all thread ready!
run!
run!
run!
run!
run!
run!
run!
run!
run!
run!
```



# ThreadLocal

[Java에서 Thread-Safe하게 프로그래밍하기](https://sup2is.github.io/2021/05/03/thread-safe-in-java.html)에서 소개한 방법중에 하나로 stack 한정 프로그래밍 방식이 있었다. Thread내에서 stack 공간은 다른 Thread가 침범하지 못하기 때문에 thread-safe하다고 소개한 방법인데 이 방법의 가장 큰 단점은 정말 해당 스택을 벗어나면 해당 값이 사라지게 된다는 것이다.

위 단점을 해결하기 위한 방법으로 자바는 `ThreadLocal`이라는 클래스를 제공한다.

`ThreadLocal`을 사용하면 set메서드로 저장했던 값을 현재 실행중인 스레드에서 가져올 수 있다.

자세한 사용방법은 [https://javacan.tistory.com/entry/ThreadLocalUsage](https://javacan.tistory.com/entry/ThreadLocalUsage) 에서 잘 소개하고 있다.

추가적으로 Spring Security를 사용해보았다면 `SecurityContextHolder.getContext()` 와 같은형태로 현재 유저에 저장된 Context를 꺼내오는 작업을 수행해본적이 있을텐데 이 방법도 `ThreadLocal`을 사용하는 방법이다.

`SecurityContextHolder`을 별다른 설정없이 사용한다면 security 전략에 `ThreadLocalSecurityContextHolderStrategy`라는 구현체가 들어가게 되는데 `ThreadLocalSecurityContextHolderStrategy` 내부엔 아래와 같은 `ThreadLocal` 멤버변수가 있다.

```java
package org.springframework.security.core.context;

import org.springframework.util.Assert;

final class ThreadLocalSecurityContextHolderStrategy implements SecurityContextHolderStrategy {
    private static final ThreadLocal<SecurityContext> contextHolder = new ThreadLocal();

    ThreadLocalSecurityContextHolderStrategy() {
    }

    public void clearContext() {
        contextHolder.remove();
    }

    public SecurityContext getContext() {
        SecurityContext ctx = (SecurityContext)contextHolder.get();
        if (ctx == null) {
            ctx = this.createEmptyContext();
            contextHolder.set(ctx);
        }

        return ctx;
    }

    public void setContext(SecurityContext context) {
        Assert.notNull(context, "Only non-null SecurityContext instances are permitted");
        contextHolder.set(context);
    }

    public SecurityContext createEmptyContext() {
        return new SecurityContextImpl();
    }
}

```

그렇기 때문에 현재 로그인한 유저의 `SecurityContextHolder`를 request 내에서 쉽게 가져올 수 있다.



# FutureTask

어떤 작업 A가 있고 이 작업 A가 시간이 많이 필요한 작업이라면 `FutureTask`를 사용해서 비동기적으로 미리 실행시켜 놓고 이후에 결과값을 받는 형태로 사용할 수 있다.

`FutureTask`를 사용하는 방법은 두가지가 있다.

```java
    public FutureTask(Runnable runnable, V result) {
        this.callable = Executors.callable(runnable, result);
        this.state = NEW;       // ensure visibility of callable
    }

    public FutureTask(Callable<V> callable) {
        if (callable == null)
            throw new NullPointerException();
        this.callable = callable;
        this.state = NEW;       // ensure visibility of callable
    }
```

`Runnable`과 함께 V 객체를 넣어준 뒤 특정 결과 이후에 다시 그 V 객체를 반환받거나 애초에 `Callable<V>`을 넣어줘서 특정한 객체를 받아내면 된다. `Callable<V>`은 리턴타입이 있는 `Runnable`로 생각하면 쉽다.



```java
@FunctionalInterface
public interface Callable<V> {
    /**
     * Computes a result, or throws an exception if unable to do so.
     *
     * @return computed result
     * @throws Exception if unable to compute a result
     */
    V call() throws Exception;
}

```



아래 예제를 살펴보자.

```java
package me.sup2is;

import java.util.Objects;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Future;
import java.util.concurrent.FutureTask;

public class FutureTaskExam {

    public static void main(String[] args) throws InterruptedException, ExecutionException {

        final ExecutorService executorService = Executors.newSingleThreadExecutor();

        final FutureTask<SomeJob> futureTask = new FutureTask<>(() -> {
            //work in external api ...
            Thread.sleep(4000);
            System.out.println("api call done!");
            return new SomeJob();
        });

        final Future<?> future = executorService.submit(futureTask);

        for (int i = 0; i < 3; i++) {
            Thread.sleep(1000);
            System.out.println("작업 " + (i + 1) + " 종료");
        }

        Objects.nonNull(future.get());
        executorService.shutdown();
    }

    static class SomeJob {
        public SomeJob() {
            System.out.println("created SomeJob");
        }
    }
}

```

위 예제의 결과는 아래와 같다.

```
작업 1 종료
작업 2 종료
작업 3 종료
api call done!
created SomeJob
```

만약 비동기 FutureTask를 사용하지 않았을경우 3초 + 4초 = 7초가 걸리는 작업이 될테지만 시간이 오래걸리는 작업을 미리 실행시켜놓는 방법으로 보다 빠르게 약 3초정도의 시간을 앞당길 수 있다.





# 마무리

이 글은 대부분 [자바병렬프로그래밍](http://www.yes24.com/Product/Goods/3015162)을 참조했다. 



<br>

***

포스팅은 여기까지 하겠습니다. 퍼가실때는 출처를 반드시 남겨주세요!

<br>

**References**

- 자바병렬프로그래밍 -에이콘-
- [https://mkyong.com/java/java-scheduledexecutorservice-examples/](https://mkyong.com/java/java-scheduledexecutorservice-examples/)
- [https://docs.oracle.com/en/java/javase/11/docs/api/index.html](https://docs.oracle.com/en/java/javase/11/docs/api/index.html)

