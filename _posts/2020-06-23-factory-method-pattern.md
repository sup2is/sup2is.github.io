---

layout: post
title: "팩토리 메서드(Factory Method) 패턴 Feat.Java"
tags: [Java, Design Pattern, Factory Method Pattern, Creation Pattern]
date: 2020-06-23
comments: true
---



<br>

# OverView

오늘은 팩토리 메서드 패턴에 대해서 알아보도록 하겠다. 



# 팩토리 메서드(Factory Method) 패턴

팩토리 메서드 패턴은 **객체를 생성하기 위해 인터페이스를 정의하지만 어떤 클래스의 인스턴스를 생성할 지에 대한 결정은 서브클래스가 내리도록 할 때** 유용하게 사용된다. 어떤 클래스가 자신이 생성해야 하는 객체의 클래스를 예측할 수 없을때 사용한다.

나의 부족한 지식으로 이해한 팩토리 메서드 패턴의 구현은 두가지로 구성되는데 일반적으로 많이 사용하는(?) 것 같은 **매개변수화 된 팩토리 메서드 패턴**과 **abstract 클래스를 사용한 팩토리 메서드 패턴**이다. 일단 매개변수화된 팩토리 메서드 패턴은 아래와 같다

```java
class SomeFactory {

	public Something(String id) {
		if(id.equals("foo")) {
			return new Foo();
		} else if(id.equals("bar")) {
			return new Bar();
		} else {
			return new FooBar();
		}
	}
}
```

Something 타입의 구체클래스들의 생성을 SomeFactory에게 위임한 형식이다. 이런 방식은 많이 사용하기때문에 이번 예제에서는 다루지 않고 우리가 작성할 예제는 abstract 클래스를 사용한 팩토리 메서드 패턴이다.

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



## 팩토리 메서드 구현하기

팩토리 메서드 패턴은 추상 팩토리 패턴과 매우 비슷하다.

추상 팩토리 패턴에서는 CarFactory를 인터페이스로 만들었다면 팩토리 메서드 패턴에서는 CarFcatory를 abstract class로 구현한다.

```java
package me.sup2is;

import me.sup2is.component.Frame;
import me.sup2is.component.Wheel;

import java.util.LinkedList;
import java.util.List;

public abstract class CarFactory {

    private final List<Car> cars = new LinkedList<>();

    public CarFactory() {
        System.out.println(this.getClass());
        Wheel wheel = createWheel();
        Frame frame = createFrame();
        Car car = new Car(frame,wheel);
        cars.add(car);
    }

    abstract protected Frame createFrame();
    abstract protected Wheel createWheel();

    public List<Car> getCars() {
        return cars;
    }
}

```

이코드에서 `createFrame()` 메서드와 `createWheel()`가 팩토리 메서드이다.

CarFactory를 상속받는 서브클래스들은 인스턴스 생성 시점에 부모 클래스인 CarFactory의 생성자를 호출할 것이고 인스턴스 생성 시점에 서브클래스들은 각각 자신이 구현한  `createFrame()` 메서드와 `createWheel()` 호출할 것이다.

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

이제 팩토리 메서드 패턴을 사용해서 실제 자동차를 생산해보자.

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

        System.out.println("Sonata 물량 : " + sonataFactory.getCars().size());
        System.out.println("Avante 물량 : " + avanteFactory.getCars().size());

    }
}

```

AvanteFactory 와 SonataFactory를 통해 Frame과 Wheel 타입을 얻어내서 자동차를 한뒤에 각각 `shape()` 와 `size()`를 호출하도록 프로그램을 구성했다. 아래에서 결과를 확인할 수 있다

```
Call super
class me.sup2is.AvanteFactory
this is avante shape
this is avante wheel size
================
Call super
class me.sup2is.SonataFactory
this is sonata shape
this is sonata wheel size
Sonata 물량 : 1
Avante 물량 : 1

Process finished with exit code 0

```



# 마무리

사실 추상팩토리 메서드와 팩토리 메서드간의 차이점을 찾기 어려웠는데 [https://stackoverflow.com/questions/5739611/what-are-the-differences-between-abstract-factory-and-factory-design-patterns](https://stackoverflow.com/questions/5739611/what-are-the-differences-between-abstract-factory-and-factory-design-patterns) 여기에서 어느정도 설명해주는 것 같다. 내가 이해한 바로는 **팩토리 메소드 패턴은 객체를 작성하기위한 메소드를 클라이언트에 노출시키는 방법이고 추상 팩토리의 경우 이러한 팩토리 메소드로 구성 될 수있는 관련 객체들을 노출**한다는 점이다. 어렵다 ...





<hr>
포스팅은 여기까지 하겠습니다. 퍼가실때는 출처를 반드시 남겨주세요!

예제: [https://github.com/sup2is/study/tree/master/design-pattern/factory-method-parttern/](https://github.com/sup2is/study/tree/master/design-pattern/factory-method-parttern/)

<br>

**References**

- GoF의 디자인 패턴 - 에릭 감마, 리처드 헬름, 랄프 존슨, 존 블리시디스
- [https://en.wikipedia.org/wiki/Factory_method_pattern](https://en.wikipedia.org/wiki/Factory_method_pattern)