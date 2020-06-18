---

layout: post
title: "비동기로 구현하는 자바의 CompletableFuture"
tags: [Java, CompletableFuture]
date: 2020-06-18
comments: true
---



<br>

# OverView

오늘은 java8에 추가된 CompletableFuture에 대해서 이야기해보도록 하겠다. 일단 CompletableFuture를 이해하기 이전에 동기와 비동기에 대한 이해가 어느정도 있어야한다. 많은 블로그들이 동기와 비동기에 대해서 설명하고 있으니 동기와 비동기에 대해서 이해가 조금 부족하다면 다른글을 먼저 참고해도 좋다.

# Future란 ?

Future는 자바5에서 추가된 비동기 관련 인터페이스이다. Future 인터페이스는 비동기 계산을 모델링하는데 사용되고 시간이 걸릴 수 있는 작업을 Future 내부에서 실행시키면 Caller Thread는 Worker Thread에게 해당 로직에 대한 실행을 전가시키고 다른 작업을 실행 할 수 있다.

아래 그림으로 조금 더 쉽게 이해할 수 있을 것이다.

![20200618_164238](https://user-images.githubusercontent.com/30790184/84992362-c0660e00-b182-11ea-9a90-c3adaa5f1a36.png)



만약 Worker Thread가 어떠한 환경에 의해 예외가 발생한다면 종료되는 스레드는 Worker뿐이다. 따라서 Caller Thread는 get() 시점에서 영원히 Worker의 응답을 기다릴 수 있기 때문에 Timeout을 적절하게 구현해야한다.

이 Future 인터페이스는 몇가지 제한이 있었는데 비동기 계산의 파이프라인 구성같은 요구사항을 해결하기에는 적합하지 않았다. 따라서 자바8에서는 CompletableFuture라는 Future 인터페이스의 구현체를 만들었다. 이제 CompletableFuture에 대해서 간단한 예제와 함께 알아보도록 하자.



# 최저가 항공권 예약 시스템



최저가 항공권예약 시스템을 풀어가면서 추가적으로 병렬스트림과 CompletableFuture의 성능차이에 대해서도 알아보도록 하자.

일단 최저가 항공권예약 시스템 플로우는 다음과 같다.



1. **n개의 항공사와 티켓을 실제로 판매해주는 판매자가 있다.**
2. **클라이언트는 서울 -> 제주로 가는 항공권을 검색하려고 한다.**
3. **n개의 항공사에 개별적으로 항공권 예약 정보를 얻어낸다. (약 1초가 걸리는 작업)**
4. **항공사에서 얻어낸 항공권 예약 정보에는 가격이 없다.**
5. **따라서 가격을 얻기 위해서는 실제 항공권을 판매하는 판매자들 중에서 해당 항공권의 최소금액을 얻어온다.(약 1초가 걸리는 작업)**
6. **클라이언트에게 완성된 항공권 예약 정보를 리턴한다.**



따라서 서비스는 총 3개로 분류된다. 하지만 예제를 간단하게 하기 위해서 각각 클래스로 구분하고 약 1초가 걸리는 작업은 `Thread.sleep()`을 사용한다. 

![20200618_162059](https://user-images.githubusercontent.com/30790184/84990324-f05fe200-b17f-11ea-9423-5aac9ee717ee.png)

예제를 작성하기 전에 프로그램에 뼈대가되는 **AirlineService**그리고 **ClientReservation**과 **ReservationInfo**에 대해서 간단하게 설명한다. **SellerService**은 비동기 파이프라인구성에서 설명한다.

**ClientReservation.java**

```java
package me.sup2is;

public class ClientReservation {
    private String from;
    private String to;

	//AllArgConstructor, Getter ...
}

```

ClientReservation은 출발지와 목적지 필드를 갖는 from, to 를 갖고 있는 엔티티 클래스다.

<br>

**AirlineTicketInfo.java**

```java
package me.sup2is;

public class AirlineTicketInfo {

    private String airline;
    private ClientReservation clientReservation;
    private String departureTime;
    private String arrivalTime;
    private int price;

    private AirlineTicketInfo(){}

    public static AirlineTicketInfo createAirlineTicketInfo(String airline, ClientReservation clientReservation, String departureTime, String arrivalTime) {
        AirlineTicketInfo info = new AirlineTicketInfo();
        info.airline = airline;
        info.clientReservation = clientReservation;
        info.departureTime = departureTime;
        info.arrivalTime = arrivalTime;
        return info;
    }

    public void assignPrice(int price) {
        this.price = price;
    }

    
    //Getter ...
}

```

AirlineTicketInfo 클래스는 클라이언트의 출발지, 도착지 정보와 항공사명 비행기 티켓의 출발시간, 도착시간, 티켓의 가격정보를 갖고 있는 엔티티 클래스다. 

<br>

**AirlineService.java**

```java
package me.sup2is;

public class AirlineService {

    public AirlineTicketInfo createTickerByAirline(String airline, ClientReservation reservation) {

        //항공사 서비스를 호출해서 티켓정보를 받아오는 로직 ...
        delay();
        
        AirlineTicketInfo airlineTicketInfo =
                AirlineTicketInfo.createAirlineTicketInfo(airline,
                                                            reservation,
                                                            "2020-16-18 09:00",
                                                            "2020-16-18 10:00");
        return airlineTicketInfo;
    }

    private String getRandomTime() {
        return "2020-16-18 "
                + String.format("%02d", random.nextInt(22) + 1)
                + ":"
                + String.format("%02d", random.nextInt(58) + 1);
    }

    
    public void delay () {
        try {
            Thread.sleep(1000);
        }catch (Exception e) {
            e.printStackTrace();
        }
    }

}

```

AirlineService는 항공사 이름과 ClientReservation타입의 고객 정보를 받고 항공사 서비스를 호출한 뒤 티켓 정보를 리턴해주는 `createTickerByAirline()` 메서드를 갖고 있다. 예제를 간단하게 하기 위해 실제 항공사 서비스 호출은 `delay()` 메서드를통해 약 1초가 걸린다고 가정한다. 예제를 간단하게 구성하기 위해 항공사 서비스에서 받아온 출발시간, 도착시간은 `getRandomTime()`을 사용했다.

<br>



이제 실제로 클라이언트의 예약정보에 따라 n개 항공사의 티켓정보를 받아오도록해보자.

## 동기 방식

티켓에 대한 정보를 동기 방식으로 약 32개의 항공사에게 요청하면 어떻게 될까?

```java
public static void main(String[] args) {

    List<String> airlines = Arrays.asList("Qatar Airways", "Singapore Airlines" ...... ); //32개의 항공사
    
    long start = System.currentTimeMillis();
    List<AirlineTicketInfo> collect = airlines.stream()
        .map(airline -> airlineService.createTickerByAirline(airline, reservation))
        .collect(Collectors.toList());

    long end = System.currentTimeMillis();
    print(collect);
    System.out.println("티켓정보를 가져오는데 걸린 시간: " + (end - start) + "ms");
    
}
```

동기방식으로 요청했을때 하나의 스레드에서 32개의 항공사에 요청하는데 걸린시간은 예상한대로 32초 이상이다. 하나의 스레드가 32개의 항공사에다 순서대로 요청하는 것이니 당연한 결과다.

```
...

AirlineTicketInfo{airline='Etihad Airways', clientReservation=me.sup2is.ClientReservation@3a03464, departureTime='2020-16-18 18:08', arrivalTime='2020-16-18 22:21', price=0}
AirlineTicketInfo{airline='Philippine Airlines', clientReservation=me.sup2is.ClientReservation@3a03464, departureTime='2020-16-18 16:03', arrivalTime='2020-16-18 22:52', price=0}
AirlineTicketInfo{airline='Air Canada', clientReservation=me.sup2is.ClientReservation@3a03464, departureTime='2020-16-18 17:30', arrivalTime='2020-16-18 15:51', price=0}
AirlineTicketInfo{airline='Finnair', clientReservation=me.sup2is.ClientReservation@3a03464, departureTime='2020-16-18 05:32', arrivalTime='2020-16-18 12:48', price=0}

티켓정보를 가져오는데 걸린 시간: 36436ms
```

아무리 생각해도 이런 서비스를 사용할 클라이언트는 없을 것 같다. 자바8에서 추가된 ParallelStream 을 사용해서 성능을 향상시켜보자.

## ParallelStream 방식

지금 내 PC의 cpu는 8코어에 16스레드다. 따라서 항공사가 32개라는 가정 하에 16개의 스레드가 싸이클을 2번돈다면 약 2초만에 모든 항공사 정보를 얻어올 수 있을 것이다. 한번 확인해보자.

```java
public static void main(String[] args) {

    List<String> airlines = Arrays.asList("Qatar Airways", "Singapore Airlines" ...... ); //32개의 항공사
    
    long start = System.currentTimeMillis();
    
    List<AirlineTicketInfo> collect = airlines.parallelStream()
        .map(airline -> airlineService.createTickerByAirline(airline, reservation))
        .collect(Collectors.toList());
    
    long end = System.currentTimeMillis();
    print(collect);
    System.out.println("티켓정보를 가져오는데 걸린 시간: " + (end - start) + "ms");
    
}
```

예상한대로 약 2초가량 걸린다.

```
...

AirlineTicketInfo{airline='Etihad Airways', clientReservation=me.sup2is.ClientReservation@3f3afe78, departureTime='2020-16-18 11:01', arrivalTime='2020-16-18 10:53', price=0}
AirlineTicketInfo{airline='Philippine Airlines', clientReservation=me.sup2is.ClientReservation@3f3afe78, departureTime='2020-16-18 18:52', arrivalTime='2020-16-18 18:24', price=0}
AirlineTicketInfo{airline='Air Canada', clientReservation=me.sup2is.ClientReservation@3f3afe78, departureTime='2020-16-18 14:45', arrivalTime='2020-16-18 17:21', price=0}
AirlineTicketInfo{airline='Finnair', clientReservation=me.sup2is.ClientReservation@3f3afe78, departureTime='2020-16-18 07:27', arrivalTime='2020-16-18 02:48', price=0}

티켓정보를 가져오는데 걸린 시간: 2110ms
```

근데 만약 항공사가 한 개 추가되면 어떤 결과가 나오게 될까?

항공사를 한 개 추가해서 33개의 항공사에 티켓정보를 가져와봤다. 

```
...

AirlineTicketInfo{airline='Etihad Airways', clientReservation=me.sup2is.ClientReservation@506e6d5e, departureTime='2020-16-18 05:53', arrivalTime='2020-16-18 22:27', price=0}
AirlineTicketInfo{airline='Philippine Airlines', clientReservation=me.sup2is.ClientReservation@506e6d5e, departureTime='2020-16-18 06:07', arrivalTime='2020-16-18 01:13', price=0}
AirlineTicketInfo{airline='Air Canada', clientReservation=me.sup2is.ClientReservation@506e6d5e, departureTime='2020-16-18 02:09', arrivalTime='2020-16-18 19:50', price=0}
AirlineTicketInfo{airline='Finnair', clientReservation=me.sup2is.ClientReservation@506e6d5e, departureTime='2020-16-18 22:24', arrivalTime='2020-16-18 18:52', price=0}
AirlineTicketInfo{airline='대한항공', clientReservation=me.sup2is.ClientReservation@506e6d5e, departureTime='2020-16-18 20:27', arrivalTime='2020-16-18 17:38', price=0}

티켓정보를 가져오는데 걸린 시간: 3112ms
```

단 한 개의 항공사를 추가해줬지만 시간대가 3초로 늘어나버렸다. 그 이유는 현재 pc의 스레드 개수는 16개이므로 한 싸이클마다 16개의 요청을 처리할 수 있다. 하지만 항공사가 33개일 경우 16 * 2 = 32 이기때문에 나머지 한 개의 작업을 끝내기 위해 싸이클을 총 세번 돌아야 작업이 끝나게 된다.

그렇다면 CompletableFuture를 사용하면 어떨까?

## CompletableFuture 방식

CompletableFuture는 자바8에서 추가된 비동기 관련 클래스이다. 자바5에서 추가된 Future 인터페이스의 구현체이고 여러 인스턴스 메서드를 통해서 비동기 파이프라이닝을 가능하게 해준다.

```java
public static void main(String[] args) {

    List<String> airlines = Arrays.asList("Qatar Airways", "Singapore Airlines" ...... ); //32개의 항공사
    
    long start = System.currentTimeMillis();
    
    List<CompletableFuture<AirlineTicketInfo>> reservationInfos = airlines.stream()
        .map(airline -> CompletableFuture.supplyAsync(() -> airlineService.createTickerByAirline(airline, reservation)))
        .collect(Collectors.toList());
    List<AirlineTicketInfo> collect = reservationInfos.stream().map(CompletableFuture::join).collect(Collectors.toList());
    
    long end = System.currentTimeMillis();
    print(collect);
    System.out.println("티켓정보를 가져오는데 걸린 시간: " + (end - start) + "ms");
    
}
```

`CompletableFuture.supplyAsync()` 메서드는 Supplier 펑션 타입을 받아 CompletableFuture 타입을 리턴해준다. 위 로직은 32개의 CompletableFuture 객체 리스트를 얻어온다. 여기에서는 아직 get을 호출하지 않았기때문에 `CompletableFuture.join()` 메서드로 실제 AirlineTicketInfo의 인스턴스를 받아와야 한다. 참고로 `CompletableFuture.get()` 과 CompletableFuture.join() 은 같은동작을하지만 `CompletableFuture.join()` 은 아무런 예외도 발생시키지 않는다.

결과값을 확인해보자.

```
...

AirlineTicketInfo{airline='Etihad Airways', clientReservation=me.sup2is.ClientReservation@2c9f9fb0, departureTime='2020-16-18 13:55', arrivalTime='2020-16-18 21:29', price=0}
AirlineTicketInfo{airline='Philippine Airlines', clientReservation=me.sup2is.ClientReservation@2c9f9fb0, departureTime='2020-16-18 01:46', arrivalTime='2020-16-18 07:12', price=0}
AirlineTicketInfo{airline='Air Canada', clientReservation=me.sup2is.ClientReservation@2c9f9fb0, departureTime='2020-16-18 04:14', arrivalTime='2020-16-18 20:45', price=0}
AirlineTicketInfo{airline='Finnair', clientReservation=me.sup2is.ClientReservation@2c9f9fb0, departureTime='2020-16-18 05:39', arrivalTime='2020-16-18 15:10', price=0}

티켓정보를 가져오는데 걸린 시간: 3118ms
```

항공사가 32개였을때 병렬스트림을 사용하면 약2초가 걸렸지만 CompletableFuture은 3초가 걸렸다. 개수는 같지만 CompletableFuture이 더 느렸다. (~~아마 CompletableFuture를 사용하면 어떤 스레드를 더 점유하나보다 ... 30개로 했을때는 똑같이 2초가 걸린다.~~) 그렇다면 CompletableFuture을 사용할 이유가 없는 것일까?



## Custom Executor 를 사용해서 CompletableFuture 성능 향상시키기

Custom Executor 을 사용해서 임의로 CompletableFuture에 스레드풀 옵션을 줄 수 있다. 예를 들어 지금 내 pc는 16스레드지만 CompletableFuture 실행 옵션에 스레드 수를 임의로 설정해서 스레드의 개수를 50개, 100개로 지정할 수 있다는 말이다. 소스를 확인해보자.

```java
public static void main(String[] args) {

    List<String> airlines = Arrays.asList("Qatar Airways", "Singapore Airlines" ...... ); //32개의 항공사
    
    long start = System.currentTimeMillis();
        
    Executor executor = Executors.newFixedThreadPool(Math.min(airlines.size(), 100), r -> {
        Thread t = new Thread(r);
        t.setDaemon(true);
        return t;
    });

    List<CompletableFuture<AirlineTicketInfo>> reservationInfos = airlines.stream()
        .map(airline -> CompletableFuture.supplyAsync(() -> airlineService.createTickerByAirline(airline, reservation), executor))
        .collect(Collectors.toList());
    List<AirlineTicketInfo> collect = reservationInfos.stream().map(CompletableFuture::join).collect(Collectors.toList());
    
    long end = System.currentTimeMillis();
    print(collect);
    System.out.println("티켓정보를 가져오는데 걸린 시간: " + (end - start) + "ms");
    
}
        

```

스레드 풀의 적정한 스레드의 개수 공식은 다음과 같다 

- **사용 가능한 코어 수 * (1+대기 시간/서비스 시간)  - Brian Goetz**

나는 `Math.min(airlines.size(), 100)` 을 사용해서 항상 `airlines.size()` or 100개 만큼의 스레드가 생성될 것이다. 그리고 `CompletableFuture.supplyAsync()`에 생성한 executor 인스턴스를 인자로 넘겼다.

실행 결과를 확인해보자. 

```
AirlineTicketInfo{airline='Etihad Airways', clientReservation=me.sup2is.ClientReservation@7ac7a4e4, departureTime='2020-16-18 19:22', arrivalTime='2020-16-18 03:37', price=0}
AirlineTicketInfo{airline='Philippine Airlines', clientReservation=me.sup2is.ClientReservation@7ac7a4e4, departureTime='2020-16-18 18:37', arrivalTime='2020-16-18 05:18', price=0}
AirlineTicketInfo{airline='Air Canada', clientReservation=me.sup2is.ClientReservation@7ac7a4e4, departureTime='2020-16-18 05:22', arrivalTime='2020-16-18 15:41', price=0}
AirlineTicketInfo{airline='Finnair', clientReservation=me.sup2is.ClientReservation@7ac7a4e4, departureTime='2020-16-18 18:14', arrivalTime='2020-16-18 19:58', price=0}

티켓정보를 가져오는데 걸린 시간: 1129ms
```

놀랍게도 약 1초의 시간이 걸렸다. 이는 CompletableFuture 의 요청을 항공사의 개수만큼 32개의 스레드가 담당했음을 의미한다. 병렬스트림을 사용할때와는 다르게 CompletableFuture 을 사용하면 커스텀 Executor 를 사용해서 성능을 더욱 높일 수 있다.



# CompletableFuture으로 파이프라인 구성하기

CompletableFuture는 다양한 인스턴스 메서드를 통해서 비동기 파이프라인을 구성할 수 있게 해준다. 아직 티켓에 대한 최저가를 측정하지 않았다.(시스템플로우 5번) 간단하게 SellerService를 소개한다.

**SellerService.java**

```java
package me.sup2is;

public class SellerService {

    public AirlineTicketInfo assignPrice(AirlineTicketInfo airlineTicketInfo) {
        
        //판매자에게 해당 티켓의 최저가를 가져오는 로직 ...
        delay();
        airlineTicketInfo.assignPrice(100_000);
        return airlineTicketInfo;
    }

    public void delay () {
        try {
            Thread.sleep(1000);
        }catch (Exception e) {
            e.printStackTrace();
        }
    }
}

```

SellerService는 AirlineTicketInfo를 통해서 시중에 나온 티켓중 가장 저렴한 티켓의 가격을 측정해서 price 값을 입력해주는 서비스다. 이것 역시 예제를 간단하게 하기 위해서 실제 판매자 서비스 호출은 delay() 메서드를통해 약 1초가 걸린다고 가정하고 가격은 임의로 측정했다.

실제로 비동기 파이프라인을 구성해보자.

```java
public static void main(String[] args) {

    List<String> airlines = Arrays.asList("Qatar Airways", "Singapore Airlines" ...... ); //32개의 항공사
    
    long start = System.currentTimeMillis();
        
    Executor executor = Executors.newFixedThreadPool(Math.min(airlines.size(), 100), r -> {
        Thread t = new Thread(r);
        t.setDaemon(true);
        return t;
    });

    List<CompletableFuture<AirlineTicketInfo>> reservationInfos = airlines.stream()
        .map(airline -> CompletableFuture.supplyAsync(() -> airlineService.createTickerByAirline(airline, reservation), executor))
        .map(future -> future.thenCompose(airlineTicketInfo -> CompletableFuture.supplyAsync(() 
                                                                                             -> sellerService.assignPrice(airlineTicketInfo), executor)))
        .collect(Collectors.toList());
    List<AirlineTicketInfo> collect = reservationInfos.stream().map(CompletableFuture::join).collect(Collectors.toList());

    
    long end = System.currentTimeMillis();
    print(collect);
    System.out.println("티켓정보를 가져오는데 걸린 시간: " + (end - start) + "ms");
    
}
        

```

CompletableFuture.thenCompose() 을 통해 비동기 파이프라인을 구성했다. 

`.map(airline -> CompletableFuture.supplyAsync(() -> airlineService.createTickerByAirline(airline, reservation), executor))` 의 리턴타입이 CompletableFuture이기때문에 

`.map(future -> future.thenCompose(airlineTicketInfo -> CompletableFuture.supplyAsync(() -> sellerService.assignPrice(airlineTicketInfo), executor)))` 을 사용해서 **항공사에서 가져온 티켓정보(비동기) + 판매자의 티켓 최저가 측정(비동기) 메서드까지 파이프라인을 구성해서 한번에 실행**시켰다.

이렇게 실행시키면 항공사의 티켓정보를 가져오는 부분에서 1초를 소요하고 판매자에게 최저가를 검색하는데에서 1초를 사용해 총 2초의 시간이 소요된다. 한번 확인해보자.

```
AirlineTicketInfo{airline='Etihad Airways', clientReservation=me.sup2is.ClientReservation@32d992b2, departureTime='2020-16-18 11:29', arrivalTime='2020-16-18 21:48', price=100000}
AirlineTicketInfo{airline='Philippine Airlines', clientReservation=me.sup2is.ClientReservation@32d992b2, departureTime='2020-16-18 05:12', arrivalTime='2020-16-18 05:08', price=100000}
AirlineTicketInfo{airline='Air Canada', clientReservation=me.sup2is.ClientReservation@32d992b2, departureTime='2020-16-18 22:54', arrivalTime='2020-16-18 12:17', price=100000}
AirlineTicketInfo{airline='Finnair', clientReservation=me.sup2is.ClientReservation@32d992b2, departureTime='2020-16-18 07:14', arrivalTime='2020-16-18 10:45', price=100000}
티켓정보를 가져오는데 걸린 시간: 2138ms
```



# 마무리

예제를 작성하다보니 항공사 서비스와 판매자 서비스에 대한 약간의 괴리감이 들지만 ... 비동기 프로그래밍을 이해하는데 상관은 없을 것 같아서 수정 없이 포스팅한다.... 예제는 예제일뿐~! 



<hr>
포스팅은 여기까지 하겠습니다. 퍼가실때는 출처를 반드시 남겨주세요!

예제: [https://github.com/sup2is/java-example/tree/master/java-completable-future-example](https://github.com/sup2is/java-example/tree/master/java-completable-future-example)

<br>

**References**

- 자바 8 인 액션 - 라울-게이브리얼 우르마, 마리오 푸스코, 앨런 마이크로프트