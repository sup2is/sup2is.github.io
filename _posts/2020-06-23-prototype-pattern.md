---

layout: post
title: "프로토타입(Prototype) 패턴 Feat.Java"
tags: [Java, Design Pattern, Prototype Pattern, Creation Pattern]
date: 2020-06-23
comments: true
---



<br>

# OverView

이번엔 프로토타입에 대해서 알아보도록 하겠다



# 프로토타입 (Prototype) 패턴

![](https://www.baeldung.com/wp-content/uploads/2019/10/Prototype-Pattern.png)

프로토타입 패턴은 기본 원형이 되는 인스턴스를 사용해서 생성할 객체의 종류를 명시하고 이렇게 만들어진 **객체를 복사해서 새로운 객체를 생성하는 패턴**이다. 프로토타입 패턴을 사용하면 프로토타입으로 생성되는 인스턴스를 복사하는 것 만으로도 마치 새로운 인스턴스를 생성하는 효과를 얻을 수 있다. 

팩토리 관련 패턴에서는 새로운 제품이 추가됨에 따라 서브클래스 및 팩토리 메서드가 증가하는 반면 프로토타입 패턴은 원형을 복제하기 때문에 서브클래스를 상속하거나 팩토리 메서드를 추가할 필요가 없다.

프로토타입 패턴을 구현하기 위해서는 반드시 `clone()` 메서드를 구현해야 한다.

예제를 확인해 보자.



# 현대 자동차 생산 공장 예제

이 공장에서의 요구사항은 다음과 같다.

- **여러가지 버전의 외형 Frame을 갖는 Avante 자동차를 생성**



## Prototype 패턴 구성하기

먼저 객체 프로토타입 인터페이스인 Car 클래스를 선언한다.

**Car.java**

```java
package me.sup2is;

public abstract class Car implements Cloneable {

    public Frame frame;
    public Wheel wheel;

    public Car(Frame frame, Wheel wheel) {
        this.frame = frame;
        this.wheel = wheel;
    }

    @Override
    public String toString() {
        return "Car{" +
                "frame=" + frame +
                ", wheel=" + wheel +
                '}';
    }

    @Override
    protected Object clone() {
        Object obj = null;
        try {
            obj = super.clone();
        }catch (Exception e) {
            e.printStackTrace();
        }
        return obj;
    }
}

```

Car클래스는 Cloneable을 구현한 복제 가능한 추상클래스이다.

다음은 Car를 상속받는 Avante 클래스이다.

**Avante.java**

```java
package me.sup2is;

public class Avante extends Car{

    public Avante(Frame frame, Wheel wheel) {
        super(frame, wheel);
    }

    public void changeFrame(Frame frame) {
        this.frame = frame;
    }

    @Override
    public String toString() {
        return "Avante{" +
                "frame=" + frame +
                ", wheel=" + wheel +
                '}';
    }
}

```

Avante는 생성 시점에 Frame과 Wheel이라는 객체타입을 받아서 초기화된다. 인스턴스 메서드로 `changeFrame()`을 갖고 있어서 원하는 Frame으로 변경할 수 있다.



## Frame, Wheel 부품 구현

마지막으로 Frame과 Wheel타입의 부품 클래스를 구현한다.

**Frame.java**

```java
package me.sup2is;

public class Frame {

    private String name;
    private String color;

    public Frame(String name, String color) {
        this.name = name;
        this.color = color;
    }

    @Override
    public String toString() {
        return "Frame{" +
                "name='" + name + '\'' +
                ", color='" + color + '\'' +
                '}';
    }
}

```

<br>

**Wheel.java**

```java
package me.sup2is;

public class Wheel {

    private String name;
    private int size;

    public Wheel(String name, int size) {
        this.name = name;
        this.size = size;
    }

    @Override
    public String toString() {
        return "Wheel{" +
                "name='" + name + '\'' +
                ", size=" + size +
                '}';
    }
}

```

<br>

준비가 끝났다면 실제로 Avante 객체를 생성하고 Frame을 변경하는 코드를 작성해보자. 단 이때 새로운 Avante 객체는 기존 Avante객체에서 `clone()` 메서드를 사용해서 프로토타입으로 구현한다.

## Prototype 패턴 사용하기

**Main.java**

```java
package me.sup2is;

public class Main {

    public static void main(String[] args) {

        Frame frame = new Frame("avante V1", "red");
        Wheel wheel = new Wheel("avante V1", 18);

        Avante redAvante = new Avante(frame, wheel);

        Avante newAvante = (Avante) redAvante.clone(); // <- 객체 복사
        Frame newFrame = new Frame("avante V2", "white");
        newAvante.changeFrame(newFrame);

        System.out.println(redAvante);
        System.out.println(newAvante);

    }
}

```

아래는 결과다.

```
Avante{frame=Frame{name='avante V1', color='red'}, wheel=Wheel{name='avante V1', size=18}}
Avante{frame=Frame{name='avante V2', color='white'}, wheel=Wheel{name='avante V1', size=18}}
```

위와 같이 새로운 서브클래스 없이도 객체 복사 후 새로운 Frame인스턴스를 전달하는 것 만으로도 새로운 객체를 얻는 효과를 얻을 수 있다. 팩토리 관련 패턴보다 조금 더 유연하게 구성 가능하다.

# 마무리

사실 GoF에서는 프로토타입을 사용하는 클라이언트쪽에서는 clone() 연산의 반환 값이 자신이 원하는 타입으로 다운캐스트할 필요가 절대 없어야 한다. 라는 조건이 있는데 예제를 작성하면서 이부분이 가장 오래걸린 부분이 아닌가 생각해본다. Car 클래스에  `public abstract Car copy()`라는 추상메서드를 생성해서 서브클래스 쪽에 `copy()` 메서드를 구현하는식으로 구성했었는데 언어차원에서 지원해주는 Clonable 인터페이스를 사용하는게 맞다고 생각해서 예제를 다시 변경했다.

뭐가 맞는걸까 ... GoF 책이 참 어렵다.



<hr>
포스팅은 여기까지 하겠습니다. 퍼가실때는 출처를 반드시 남겨주세요!

예제: [https://github.com/sup2is/study/tree/master/design-pattern/prototype-pattern](https://github.com/sup2is/study/tree/master/design-pattern/prototype-pattern)

<br>

**References**

- GoF의 디자인 패턴 - 에릭 감마, 리처드 헬름, 랄프 존슨, 존 블리시디스
- [https://m.blog.naver.com/scw0531/221465259625](https://m.blog.naver.com/scw0531/221465259625)
- [https://www.baeldung.com/java-pattern-prototype](https://www.baeldung.com/java-pattern-prototype)
- [https://github.com/iluwatar/java-design-patterns/tree/master/prototype/src/main/java/com/iluwatar/prototype](https://github.com/iluwatar/java-design-patterns/tree/master/prototype/src/main/java/com/iluwatar/prototype)