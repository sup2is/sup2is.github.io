---
layout: post
title: "Java Stream.flatMap() 간단하게 이해하기"
tags: [Java, Stream, FlatMap]
date: 2021-1-19
comments: true
---


<br>

# OverView

이번시간에는 Java8에서 도입된 Stream 인터페이스 중 `flatMap()` 메서드에 대해서 간단하게 알아보도록 하겠다.

# Stream.map() 메서드

Stream의 `map()` 메서드는 Function 타입의 함수형 인터페이스를 인자로 받는 매핑 메서드이다. Stream의 `map()`인터페이스는 아래와 같다. 

이 `map()` 메서드에서 Function 타입의 함수형 인터페이스는 아래와 같다.

```java
//Stream의 map() 메서드 인터페이스
<R> Stream<R> map(Function<? super T, ? extends R> mapper);


//map() 메서드가 인자로 받는 Function 타입
@FunctionalInterface
public interface Function<T, R> {
    R apply(T t);
}
```

간단한 예제로 `map()` 메서드를 사용해보자

```java
    @Test
    public void testMap() {
        Member choi = new Member("choi", 28);
        Member woo = new Member("woo", 25);

        List<Member> members = Arrays.asList(choi, woo);

        members.stream()
                .map(m -> m.getName() + "님의 나이는 " + m.getAge() + "입니다.")
                .forEach(System.out::println);
    }

```

위 경우는 `Member` 타입을 받아서 문자열을 붙인 후 `String` 타입으로 매핑해서 출력하게 했다. 아래의 결과물을 확인해 보자.

```
choi님의 나이는 28입니다.
woo님의 나이는 25입니다.
```

# Stream.flatMap() 메서드

`flatMap()` 메서드는 `map()` 메서드와 동일하게 매핑 메서드 역할을 하는데 2차원 또는 2단계 배열 또는 List타입에 대해서 일괄적으로 하나의 Stream에서 연산할 수 있도록 도와준다.

```java
//flatMap 메서드 인터페이스
<R> Stream<R> flatMap(Function<? super T, ? extends Stream<? extends R>> mapper);
```

예를 들어 다음과 같이 과일의 색깔 별로 2개씩 분류한 `String[][]` 타입의 이차원 배열이 있다고 가정하자.

여기에서 `Apple`인 것만 제외하고 새로운 `String[]` 타입을 얻는 요구사항을 가장 간단하게 구현하는 방법은 아래와 같다.

```java
    @Test
    public void 이차원_배열을_중첩for문으로_접근하기() {
        String[][] nested = new String[][]{ {"Apple", "Cherry"}, {"Mango", "Orange"}, {"Grape", "BlueBerry"}};

        List<String> fruits = new ArrayList<>();

        for (String[] fs : nested) {
            for (String f : fs) {
                if(!f.equals("Apple")) fruits.add(f);
            }
        }

        String[] exclude = fruits.toArray(new String[fruits.size()]);
        System.out.println(Arrays.toString(exclude));
    }
```

결과값은 아래와 같다.

```
[Cherry, Mango, Orange, Grape, BlueBerry]
```

위 실행 결과를 Stream의 `filter()` 메서드만 활용해서 최대한 비슷하게 구현하면 아래와 같다.

```java
    @Test
    public void 이차원_배열을_flatMap없이_접근하기() {
        String[][] nested = new String[][]{ {"Apple", "Cherry"}, {"Mango", "Orange"}, {"Grape", "BlueBerry"}};

        List<String[]> fruits = Arrays.stream(nested)
                .filter(fs -> { //<- nested가 2차원 배열이기 때문에 넘어오는 fs 변수는 1차원 배열 String[] 임
                    for (String f : fs) { //<- fs가 1차원 배열이기 때문에 다시 순회하기 위해 for문 적용
                        if (f.equals("Apple")) return false;
                    }
                    return true;
                })
                .collect(Collectors.toList()); //<- 결과적으로 filter()가 리턴하는 타입은 Stream<String[]> 이기 때문에 List<String>타입으로 가져올 수 없고 List<String[]> 타입으로 collecting 해야함

        fruits.forEach(fs-> System.out.println(Arrays.toString(fs)));
    }

```

코드에서 설명한대로 2차원배열-> 1차원배열 -> 개별요소로 접근해야하므로 코드의 가독성이 안좋아지고 우리가 원하는 `List<String> ` 타입이 아닌 `List<String[]>` 타입으로 가져오게 되어 별도의 처리를 해주어야한다. 그리고 가장 큰 문제점은 결과값에서 확인할 수 있다.

```
[Mango, Orange]
[Grape, BlueBerry]
```

단순히 `Apple`값을 제외하려고 했지만 `Apple`과 같은 배열에 있는 `Cherry` 역시 제거된 상태가 되는데 말그대로 위에서 설명한 `fs` 변수가 개별요소가 아닌 1차원 배열이기때문에 `Apple` 이 포함된 `[Apple, Cherry]` 배열 자체를 필터링 시킨 결과가 나온다.

위와 같은 경우에 `flatMap()`을 활용하여 쉽게 접근할 수 있다.

```java
    @Test
    public void 이차원_배열을_flatMap으로_접근하기() {
        String[][] nested = new String[][]{ {"Apple", "Cherry"}, {"Mango", "Orange"}, {"Grape", "BlueBerry"}};

        List<String> fruits = Arrays.stream(nested)
                .flatMap(fs -> Arrays.stream(fs)) //<- Stream 타입을 리턴
                .filter(f -> !f.equals("Apple"))
                .collect(Collectors.toList());

        fruits.forEach(System.out::println);
    }
```

결과값은 아래와 같다.

```
Cherry
Mango
Orange
Grape
BlueBerry
```

## 고객 주문 내역서의 총 결제금액 계산하기

List 타입으로 중첩된 객체들 또한 `flatMap()` 메서드를 사용해서 쉽게 접근할 수 있다.

간단한 예제를 위해 `Member` : `Order` : `Product`의 연관관계를 다음과 같이 구성했다.

- `Member`와 `Order`는 1:N
- `Order`와 `Product`는 1:N

**Order.java**

```java
class Order {

    private List<Product> itemList = new ArrayList<>();

    public void addItem(Product product) {
        this.itemList.add(product);
    }

	//getter...
}
```

**Product.java**

```java
class Product {

    private String name;
    private int price;

	//getter, constructor...
}
```

**Member.java**

```java

class Member {

    private List<Order> orders;
    private String name;
    private int age;
    
    public Member(String name, int age) {
        this.name = name;
        this.age = age;
    }
    
    public void addOrder(Order order) {
        this.orders.add(order);
    }
    
	//getter...
}
```

예제를 위해 도메인 로직에서는 총 금액에 대한 처리가 없다고 가정하고 `flatMap()` 을 통해 유저가 주문한 총 금액을 계산하는 로직을 작성하면 아래와 같다.

```java

    @Test
    public void flatMapTest() {
		//given
        Member choi = new Member("choi", 28);
        Member woo = new Member("woo", 25);

        Product tv = new Product("TV", 300_000);
        Product airConditioner = new Product("에어컨", 200_000);
        Product refrigerator = new Product("냉장고", 500_000);
        Product phone = new Product("핸드폰", 100_000);

        Order order1 = new Order();
        order1.addItem(tv);
        order1.addItem(airConditioner);

        Order order2 = new Order();
        order2.addItem(refrigerator);
        order2.addItem(phone);

        choi.addOrder(order1);
        woo.addOrder(order2);

        List<Member> members = Arrays.asList(choi, woo);
        
		//when
        long totalPrice = members.stream()
                .flatMap(m -> m.getOrders().stream())
                .flatMap(o -> o.getItemList().stream())
                .map(p -> p.getPrice())
                .reduce(0, (p1, p2) -> p1 + p2)
                .intValue();
		
        //then
        long expect = tv.getPrice() + airConditioner.getPrice() + refrigerator.getPrice() + phone.getPrice();
        Assert.assertEquals(expect, totalPrice);
    }
```

위와 같은 중첩구조에도 간단하게  Stream의 `flatMap()`을 사용해서 한번에 총 주문금액을 가져올 수 있는 것을 확인할 수 있다.













<br>

***

포스팅은 여기까지 하겠습니다. 퍼가실때는 출처를 반드시 남겨주세요!

<br>

**References**

- [https://mkyong.com/java8/java-8-flatmap-example/](https://mkyong.com/java8/java-8-flatmap-example/)
