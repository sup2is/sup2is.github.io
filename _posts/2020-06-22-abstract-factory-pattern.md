---

layout: post
title: "추상 팩토리(Abstract Factory) 패턴 Feat.Java"
tags: [Java, Design Pattern, Abstract Factory, Creation Pattern]
date: 2020-06-22
comments: true
---



<br>

# OverView

오늘은 추상 팩토리 패턴에 대해서 알아보도록하겠다.



# 추상 팩토리(Abstract Factory) 패턴

![](https://www.researchgate.net/profile/Hironori_Washizaki2/publication/221317734/figure/fig1/AS:305566212739072@1449864070022/UML-Class-diagram-describing-the-AbstractFactory-pattern.png)

추상 팩토리 패턴은 **상세화된 서브클래스를 정의하지 않고도 서로 관련성이 있거나 독립적인 여러 객체의 군을 생성하기 위한 인터페이스를 제공하는 패턴**이다.

추상 팩토리 패턴은 객체의 생성, 표현, 구성과는 무관하게 시스템을 독립적으로 만들고자 할 때 유용한 패턴이다. 여러 클래스들 중에 하나를 선택해서 시스템을 설정해야하고 한번 구성한 제품을 다른것으로 대체할 수 있을 때 사용한다. 

추상 팩토리 패턴과 추상 메서드 패턴은 다른 패턴이다. 내가 생각했을 때 추상 팩토리 패턴의 가장 큰 장점은 관련성이 있는 독립적인 여러 객체의 군을 유연하게 변경할 수 있다는 점이다.

예제를 통해 함께 알아보도록 하자.

# 현대 자동차 생산 공장 예제

이 공장에서는 현대자동차의 아반떼와 소나타를 생산하는 공장이라고 가정한다. UML을 보면 도움이 될 수 있다.

![20200622_164211](https://user-images.githubusercontent.com/30790184/85261625-61b4d300-b4a7-11ea-809d-aa9e8bb13c7b.png)

## 부품 만들기

아반떼와 소나타의 부품이 될 겉모양 Frame과 바퀴의 Wheel을 추상화해서 각각 아반떼와 소나타의 부품을 생성했다.

먼저 Frame을 생성해보자.

**Frame.java**

```java
package me.sup2is.component;

public interface Frame {
    void shape();
}

```

Frame은 단순히 shape라는 메서드를 갖고있는 인터페이스이다. 이제 각각 아반떼와 소나타의 Frame을 생성하고 `shape()`를 재정의 해보자.

**AvanteFrame.java**

```java
package me.sup2is.component;

public class AvanteFrame implements Frame {

    @Override
    public void shape() {
        System.out.println("this is avante shape");
    }
}

```

**SonataFrame.java**

```java
package me.sup2is.component;

public class SonataFrame implements Frame {
    @Override
    public void shape() {
        System.out.println("this is sonata shape");
    }
}

```

예제를 간단하게 하기 위해서 아반떼 Frame은 "this is avante shape" 이라는 문자열을 반환. 소나타 Frame은 "this is sonata shape"을 반환하게 해놨다.

이번엔 Wheel을 생성해보고 아반떼와 소나타의 `size()`를 재정의해보자!

**Wheel.java**

```java
package me.sup2is.component;

public interface Wheel {
    void size();
}
```

**AvanteWheel**

```java
package me.sup2is.component;

public class AvanteWheel implements Wheel {
    @Override
    public void size() {
        System.out.println("this is avante wheel size");
    }
}

```

**SonataWheel.java**

```java
package me.sup2is.component;

public class SonataWheel implements Wheel {
    @Override
    public void size() {
        System.out.println("this is avante sonata size");
    }
}

```

마찬가지로 간단하게 print를 찍도록 구현했다.



## 공장 만들기

이번에는 실제로 인터페이스로 추상 팩토리 패턴을 생성해보도록 하겠다. 아반떼와 소나타의 규격은 서로 다르다. 아반떼의 Frame에 소나타의 Wheel이 맞지않고 (~~실제로 맞는지 안맞는지는 모름 ..~~) 반대로 소나타의 Frame에는 아반뗴의 Wheel이 맞지 않을 수 있다. 아반떼는 아반떼의 Frame과 Wheel을, 소나타는 소나타의 Frame과 Wheel을 생성하도록해서 자동차 공장을 만들어 보자.

**CarFactory.java**

```java
package me.sup2is;

import me.sup2is.component.Frame;
import me.sup2is.component.Wheel;

public interface CarFactory {
    Frame createFrame();
    Wheel createWheel();
}

```

CarFactory 인터페이스는 `createFrame()` 메서드와 `createWheel()`을 갖고 있다. 공장라인에서 생산에 필요한 부품에 대한 인터페이스라고 생각하면 된다. 실제 아반떼와 소나타 부품을 생산하는 공장은 이 CarFactory를 구현한다.

**AvanteFactory.java**

```java
package me.sup2is;

import me.sup2is.component.AvanteFrame;
import me.sup2is.component.AvanteWheel;
import me.sup2is.component.Frame;
import me.sup2is.component.Wheel;

public class AvanteFactory implements CarFactory {

    @Override
    public Frame createFrame() {
        return new AvanteFrame();
    }

    @Override
    public Wheel createWheel() {
        return new AvanteWheel();
    }
}

```

AvanteFactory는 CarFactory를 구현하는 클래스다. 이전에 생성했던 AvanteFrame과 AvanteWheel을 만들어서 아반떼 부품을 생산하도록 구성했다.



**SonataFactory.java**

```java
package me.sup2is;

import me.sup2is.component.Frame;
import me.sup2is.component.SonataFrame;
import me.sup2is.component.SonataWheel;
import me.sup2is.component.Wheel;

public class SonataFactory implements CarFactory {
    @Override
    public Frame createFrame() {
        return new SonataFrame();
    }

    @Override
    public Wheel createWheel() {
        return new SonataWheel();
    }
}

```

SonataFactory 역시 마찬가지로 CarFactory를 구현하고 SonataFrame과 SonataWheel을 생산한다.



## 자동차 생산하기

이제 추상 팩토리 패턴을 사용해서 실제 자동차를 생산해보자.

**Car.java**

```java
package me.sup2is;

import me.sup2is.component.Frame;
import me.sup2is.component.Wheel;

public class Car {

    private Frame frame;
    private Wheel wheel;

    public Car(Frame frame, Wheel wheel) {
        this.frame = frame;
        this.wheel = wheel;
    }

    public Frame getFrame() {
        return frame;
    }

    public Wheel getWheel() {
        return wheel;
    }
}

```

Car 클래스는 Frame과 Wheel 타입을 갖는 자동차 클래스다.



**Main.java**

```java
package me.sup2is;

import me.sup2is.component.Frame;
import me.sup2is.component.Wheel;

public class Main {

    public static void main(String[] args) {
        CarFactory avanteFactory = new AvanteFactory();
        Car avante = new Car(avanteFactory.createFrame(), avanteFactory.createWheel());
        avante.getFrame().shape();
        avante.getWheel().size();

        System.out.println("================");

        CarFactory sonataFactory = new SonataFactory();
        Car sonata = new Car(sonataFactory.createFrame(), sonataFactory.createWheel());
        sonata.getFrame().shape();
        sonata.getWheel().size();
    }
}

```

AvanteFactory 와 SonataFactory를 통해 Frame과 Wheel 타입을 얻어내서 자동차를 한뒤에 각각 `shape()` 와 `size()`를 호출하도록 프로그램을 구성했다. 아래에서 결과를 확인할 수 있다.

```
this is avante shape
this is avante wheel size
================
this is sonata shape
this is sonata wheel size
```

# 마무리

추상 팩토리 패턴의 가장 큰 장점은 새로운 클래스나 인스턴스를 쉽게 추가 및 교체할 수 있다. 만약 자동차 생산라인에 그랜져가 추가되면 어떨까? 추상 팩토리 패턴을 사용했다면 CarFactory를 구현하는 GrangerFactory를 만들어서 사용하면된다.

추상 팩토리 패턴의 단점중에 하나는 인터페이스이기 때문에 Frame이나 Wheel 이외에 자동차의 부품이 추가될 경우이다. 예를 들어 자동차 전조등의 Headlight가 추가된다고하면 이를 구현하고있는 모든 Factory들은 수정되어야하지만 이는 자바8에서 추가된 인터페이스 default 메서드로 어느정도 해결이 가능하다.







<hr>
포스팅은 여기까지 하겠습니다. 퍼가실때는 출처를 반드시 남겨주세요!

예제: [https://github.com/sup2is/study/tree/master/design-pattern/abstract-factory-pattern](https://github.com/sup2is/study/tree/master/design-pattern/abstract-factory-pattern)

<br>

**References**

- GoF의 디자인 패턴 - 에릭 감마, 리처드 헬름, 랄프 존슨, 존 블리시디스
- [https://jdm.kr/blog/192](https://jdm.kr/blog/192)