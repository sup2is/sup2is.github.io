---
layout: post
title: "플라이웨이트(Flyweight) 패턴 Feat.Java"
tags: [Java, Design Pattern, Flyweight Pattern, Structural Pattern]
date: 2020-06-30
comments: true
---



<br>

# OverView

이번시간엔 구조패턴 중 하나인 플라이웨이트 패턴에 대해서 알아보도록 하겠다!

# 플라이웨이트(Flyweight) 패턴

플라이웨이트 패턴은 **응용프래그램이 대량의 객체를 사용해야 할 때 사용**한다. 플라이웨이트 패턴을 이해하기 이전에 객체 자체의 intrinsic(본질적인 개념), extrinsic(부가적인 개념) 에 대해서 이해해야한다. 

intrinsic은 곧 객체의 본질적인 개념을 뜻하는데 이 특성들은 다른 객체들과 공유될 수 있지만 extrinsic, 부가적인 개념은 다른 객체들과 공유될 수 없다. 예를 들어 문자 파싱 시스템에서 문자나 스펠링은 고유한 개념으로 사용이 가능하고 공유 또한 가능하지만 해당문서에서 문자의 위치 index 또는 행 위치를 나타내는 row 같은 속성은 객체 자체의 상태를 부가적인 개념이기 때문에 객체끼리 공유하면 안된다.

이런 플라이웨이트 패턴을 사용하면 공유 가능한 객체별 본질적 상태의 양을 줄일 수 있고 더 많이 공유될 수록 인스턴스를 절약하는 효과를 가져온다. 

![Flyweight](https://user-images.githubusercontent.com/30790184/86082711-61ce5780-bad3-11ea-9a26-a34a4cacdd9d.png)



- Flyweight: Flyweight가 받아들일 수 있고 부가적 상태에서 동작해야 하는 인터페이스를 선언
- ConcreateFlyweight: Flyweight인터페이스를 구현하고 내부적으로 갖고 있어야 하는 본질적 상태에 대한 저장소를 정의, ConcreateFlyweight 객체는 공유할 수 있는 것이어야 한다. 관리하는 어떤 상태라도 본질적인 것이어야 함
- UnsharedConcreateFlyweight: 모든 플라이급 서브클래스들이 공유될 필요는 없음 Flyweight 인터페이스는 공유를 가능하게 하지만 그것을 가용해서는 안됨 
- FlyweightFactory: 플라이급 객체를 생성하고 관리하며 플라이급 객체가 제대로 공유되도록 보장함 사용자가 플라이급 객체를 요청하면 Flyweight-Factory 객체는 이미 존재하는 인스턴스를 제공하거나 없으면 새로 생성함
- Client: 플라이급 객체에 대한 참조자를 관리하며 플라이급 객체의 부가적 상태를 저장함



# 옷 공장 만들기

옷 공장의 요구사항은 다음과 같다.

- **클라이언트는 code값, size, color(객체의 본질적 요소) 를 기반으로 공장에 옷을 요청한다**
- **같은 code값이 있다면 기존 옷을 return하고 없다면 생성한다.**
- **location(객체의 부가적 요소)를 기반으로 Product 를 생산한다.**

##  인터페이스 작성하기

**Clothes.java**

```java
package me.sup2is;

public abstract class Clothes {

    private String code;
    private int size;
    private String color;

    public Clothes(String code, int size, String color) {
        this.code = code;
        this.size = size;
        this.color = color;
    }

}

```

Clothes는 모든 옷들의 상위 인터페이스이다. 별도의 추상 메서드는 생성하지 않았다. Clothes를 기반으로 Shirt와 Pant를 만들어보자.

**Shirt.java**

```java
package me.sup2is;

public class Shirt extends Clothes {
    public Shirt(String code, int size, String color) {
        super(code, size, color);
    }
}

```

**Pants.java**

```java
package me.sup2is;

public class Pants extends Clothes{
    public Pants(String code, int size, String color) {
        super(code, size, color);
    }
}

```



## 공장 만들기

플라이웨이트 패턴에서는 주로 팩토리 패턴을 사용한다. 다음 ClothesFactory 클래스를 확인해보자.

**ClothesFactory.java**

```java
package me.sup2is;

import java.util.HashMap;
import java.util.Map;

public class ClothesFactory {

    private static Map<String, Clothes> map = new HashMap<>();

    public static Product getClothes(ClothesType type, String code, int size, String color, String location) {

        Clothes clothes = map.computeIfAbsent(code, u -> {
            Clothes c = createClothes(type, code, size, color);
            map.put(code, c);
            return c;
        });

        return new Product(clothes, location);
    }

    private static Clothes createClothes(ClothesType type, String code, int size, String color) {
        System.out.println(code + " 옷을 생산합니다.");
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        Clothes c = null;

        switch (type) {
            case SHIRT:
                c = new Shirt(code, size, color);
                break;
            case PANT:
                c = new Pants(code, size, color);
                break;
        }

        return c;
    }

}

```

간략하게 설명하면 `getClothes()`로 요청이 들어왔을때 미리 만들어둔 옷이면 기존에 만들어둔 객체를 리턴하도록하고 만약 새로운 code면 새롭게 옷을 생성한다. 이 옷을 생성하는 시간은 약 1초가 걸리도록 구성했다.

최종적으로 리턴되는 Product는 납품될 위치의 부가적인 요소를 위해서 Clothes를 한번 더 감싸는 구조로 만들었다. Clothes 내부에는 공유되는 요소가 없도록 하기 위해서이다.



## 프로그램 실행하기

**Main.java**

```java
package me.sup2is;

public class Main {

    public static void main(String[] args) {

        long start = System.currentTimeMillis();

        ClothesFactory.getClothes(ClothesType.SHIRT,"AA0001", 100, "red", "강남점");
        ClothesFactory.getClothes(ClothesType.SHIRT,"AA0001", 100, "red", "신사점");
        ClothesFactory.getClothes(ClothesType.SHIRT,"AA0001", 100, "red", "역삼점");
        ClothesFactory.getClothes(ClothesType.SHIRT,"AA0001", 100, "red", "홍대점");
        ClothesFactory.getClothes(ClothesType.SHIRT,"AA0002", 95, "black", "강남점");
        ClothesFactory.getClothes(ClothesType.PANT, "AA0003", 100, "blue", "교대점");
        ClothesFactory.getClothes(ClothesType.PANT, "AA0003", 100, "blue", "강남점");

        long end = System.currentTimeMillis();

        System.out.println("프로그램 수행 시간: " + (end - start) / 1000 + "초");

    }

}

```

ClothesFactory에 약 7개의 옷을 요청했지만 code와 같은 key를 갖고 있는 옷이 있다면 옷을 생산하지 않고 바로 리턴하기때문에 실제로 프로그램 수행은 약 3초가 걸린다. 아래 결과를 확인해보자

```
AA0001 옷을 생산합니다.
AA0002 옷을 생산합니다.
AA0003 옷을 생산합니다.
프로그램 수행 시간: 3초
```

이런식으로 부가적인 상태를 제외하고 공유가 필요하거나 가능한 본질적인 개념들을 공유함으로써 저장비용을 크게 낮출 수 있다. 

# 마무리

현실세계에서 사용할 법한 예제를 찾기가 너무 어려웠다. 옷 생산 공장마저 현실세계와 괴리감이 드는건 여전하지만 플라이웨이트 패턴을 이해하는데 문제가 없을 것 같아서 포스팅한다.



<hr>
포스팅은 여기까지 하겠습니다. 퍼가실때는 출처를 반드시 남겨주세요!

예제: [https://github.com/sup2is/study/tree/master/design-pattern/flyweight-pattern](https://github.com/sup2is/study/tree/master/design-pattern/flyweight-pattern)

<br>

**References**

- GoF의 디자인 패턴 - 에릭 감마, 리처드 헬름, 랄프 존슨, 존 블리시디스
- [http://www.diranieh.com/DP/Flyweight.htm](http://www.diranieh.com/DP/Flyweight.htm)