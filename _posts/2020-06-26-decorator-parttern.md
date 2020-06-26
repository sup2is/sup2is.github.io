---
layout: post
title: "데코레이터(Decorator) 패턴 Feat.Java"
tags: [Java, Design Pattern, Decorator Pattern, Structural Pattern]
date: 2020-06-26
comments: true
---



<br>

# OverView

이번시간엔 데코레이터 패턴에 대해서 알아보도록 하겠다!

# 데코레이터(Decorator) 패턴

데코레이터 패턴은 **객체에 동적으로 새로운 기능 또는 책임을 추가할 수 있게 하는 패턴**이다. **정적으로 서브클래스를 생성하지 않고 동적으로 생성**하기때문에 런타임에 유연하게 동작 가능하다.

데코레이터 패턴은 전체 클래스에 새로운 기능을 추가할 필요가 없지만 개별적으로 객체에 새로운 책임 또는 기능을 추가해야할 때 사용한다. 이 데코레이터 패턴은 자신이 장식하는 인터페이스를 자기 자신도 동일하게 제공하기때문에 클라이언트에게 감춰진 형태로 사용 가능하다.

보통 상속으로 서브클래스를 계속 만드는 방법이 실질적이지 못할 때 사용하고 동적으로 투명하게 다른 객체에 영향을 주지 않고 각각의 객체에 새로운 책임 또는 기능을 추가할 수 있다. 이런 데코레이터 패턴을 적용하면 책임 또는 기능을 무한정 추가가 가능하다.



![400px-Decorator_UML_class_diagram svg](https://user-images.githubusercontent.com/30790184/85813744-06ccf580-b79f-11ea-8da7-da5888c017d1.png)



- Component : 동적으로 추가할 서비스를 가질 가능성이 있는 객체들에 대한 인터페이스이다.
- ConcreteComponent : 추가적인 서비스가 실제로 정의되어야 할 필요가 있는 객체이다.
- Decorator: Component 객체에 대한 참조자를 관리하면서 Component에 정의된 인터페이스를 만족하도록 인터페이스를 정의한다.
- ConcreateDecorator: Component 에 새롭게 추가할 서비스를 실제로 구현하는 클래스다.

아래에서 햄버거집예제를 통해 데코레이터 패턴을 이해해보자.



# 햄버거집 예제

보통 햄버거집에서는 마요네즈를 추가하거나 패티추가, 치즈추가 등등 햄버거에 대한 여러가지 추가옵션이 존재한다. 하지만 이런 경우 정적으로 서브클래스로 확장하기엔 비용이 너무 크다. 이럴때 데코레이터 패턴을 사용하면 된다.



## 인터페이스 작성하기

**Burger.java**

```java
package me.sup2is;

public interface Burger {

    int price();
    void description();
}

```

Burger 인터페이스는 `price()`와 `description()`  이라는 메서드를 갖고 있고 각각 자기 자신의 가격과 설명에 대한 print를 찍는 함수로 구현할 것이다.

Burger를 구현하는 CheeseBurger를 생성해보자.

**CheeseBurger.java**

```java
package me.sup2is;

public class CheeseBurger implements Burger {

    @Override
    public int price() {
        return 3000;
    }

    @Override
    public void description() {
        System.out.println("Cheese 버거");
    }
}

```

CheeseBurger는 Burger를 구현하고 가격은 3000원, 설명에는 'Cheese 버거'라는 문자열을 찍도록 구현했다.

이제 데코레이터를 적용해보자.

## 데코레이터 적용하기

**BurgerDecorator.java**

```java
package me.sup2is;

public abstract class BurgerDecorator implements Burger {

    protected Burger burger;

    public BurgerDecorator(Burger burger) {
        this.burger = burger;
    }

}

```

BurgerDecorator 클래스는 위에서 언급한대로 Burger 인터페이스를 구현함과 동시에 Burger 타입의 인스턴스 변수를 갖고 있다. 이 Burger타입의 인스턴스 변수는 생성자 시점에서 초기화되도록 구성했다. 이제 BurgerDecorator 클래스를 상속하는 데코레이터들은 Burger타입의 인스턴스의 초기화 + Burger 인터페이스 메서드를 구현해야한다. 실제로 ExtraCheese와 ExtraMayonnaise 클래스를 작성해보자.

**ExtraMayonnaise.java**

```java
package me.sup2is;

public class ExtraMayonnaise extends BurgerDecorator {

    public ExtraMayonnaise(Burger burger) {
        super(burger);
    }

    @Override
    public void description() {
        burger.description();
        System.out.println("+ 마요네즈 추가");
    }

    @Override
    public int price() {
        return burger.price() + 200;
    }

}

```

BurgerDecorator를 상속받은 ExtraMayonnaise는 생성자로 받은 Burger 타입 인스턴스 메서드 실행을 가로채서 '+ 마요네즈 추가' 라는 문자열, 기존 가격에 +200원을 더한 가격을 리턴하도록 구성했다.

**ExtraCheese.java**

```java
package me.sup2is;

public class ExtraCheese extends BurgerDecorator {

    public ExtraCheese(Burger burger) {
        super(burger);
    }

    @Override
    public void description() {
        burger.description();
        System.out.println("+ 치즈 추가");
    }

    @Override
    public int price() {
        return burger.price() + 300;
    }

}

```

ExtraCheese 클래스도 비슷하게 작성했다.

이제 실제 햄버거를 판매해보자

## 햄버거 판매하기

**Main.java**

```java
package me.sup2is;

public class Main {

    public static void main(String[] args) {

        Burger burger = new CheeseBurger();

        burger.description();
        System.out.println(burger.price() + "원");
        System.out.println("=============================================");

        Burger extraMayo = new ExtraMayonnaise(new CheeseBurger());

        extraMayo.description();
        System.out.println(extraMayo.price() + "원");
        System.out.println("=============================================");

        Burger doubleExtraCheese = new ExtraCheese(new ExtraCheese(extraMayo));

        doubleExtraCheese.description();
        System.out.println(doubleExtraCheese.price() + "원");
        System.out.println("=============================================");

    }

}

```

기존 오리지날 치즈버거에서 마요네즈를 추가하고, 치즈를 두장 추가하면 아래와 같은 결과가 나온다.

```
Cheese 버거
3000원
=============================================
Cheese 버거
+ 마요네즈 추가
3200원
=============================================
Cheese 버거
+ 마요네즈 추가
+ 치즈 추가
+ 치즈 추가
3800원
=============================================
```

데코레이터 패턴을 사용해서 기존 인스턴스에 책임을 추가하는 것이기 때문에 이런식으로 별도의 서브클래스 없이 동적으로 유연하게 동작하는 코드를 작성할 수 있다.



![20200626_105854](https://user-images.githubusercontent.com/30790184/85813762-18160200-b79f-11ea-8e3b-661748b749e1.png)

위 사진은 맨 마지막 3개가 중첩된 burger 인스턴스들이다.

# 마무리

데코레이터 패턴을 사용하면 단순한 상속보다 설계의 융통성을 더 많이 증대시킬 수 있고 동적으로 기능을 추가할 수 있기 때문에 중첩된 클래스들 중 최상단에 많은 기능이 누적되는 상황을 피할 수 있다. 하지만 너무 많은 데코레이터들이 존재한다면 이들을 모두 이해하는게 쉽지 않고 수정하는 과정이 복잡해질 수 있다.



<hr>
포스팅은 여기까지 하겠습니다. 퍼가실때는 출처를 반드시 남겨주세요!

예제: [https://github.com/sup2is/study/tree/master/design-pattern/decorator-pattern](https://github.com/sup2is/study/tree/master/design-pattern/decorator-pattern)

<br>

**References**

- GoF의 디자인 패턴 - 에릭 감마, 리처드 헬름, 랄프 존슨, 존 블리시디스
- [https://johngrib.github.io/wiki/decorator-pattern/](https://johngrib.github.io/wiki/decorator-pattern/)
- [https://www.tutorialspoint.com/design_pattern/decorator_pattern.htm](https://www.tutorialspoint.com/design_pattern/decorator_pattern.htm)