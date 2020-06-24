---
layout: post
title: "브릿지(Bridge) 패턴 Feat.Java"
tags: [Java, Design Pattern, Bridge Pattern, Structural Pattern]
date: 2020-06-24
comments: true
---



<br>

# OverView

이번시간엔 구조패턴중 하나인 브릿지 패턴에 대해서 알아보도록 하겠다.

# 브릿지(Bridge) 패턴

어떤 하나의 추상적 개념이 여러가지 구현으로 구체화 될 수 있을때 객체지향에서는 대부분 상속이라는 개념으로 문제를 해결한다. 하지만 상속 또는 구현은 추상적 개념을 구현부 클래스에 영속적으로 종속시킨다. 구현을 분리해서 재사용하거나 수정 또는 확장 하기가 쉽지 않다.

브릿지 패턴은 **추상적 개념에 해당하는 클래스와 실제 구현부를 분리함**으로써 문제를 해결한다.

![99808B445AA7B2AF02](https://user-images.githubusercontent.com/30790184/85514756-555f8000-b637-11ea-80f4-159819cae358.png)

이 브릿지 패턴을 사용하면 추상적 개념과 이에 대한 구현 사이의 지속적인 종속 관계를 피할 수 있고 런타임 환경에서 유연하게 동작할 수 있도록 도와준다. 이 브릿지 패턴을 사용하면 컴파일시점에는 어떤 구현체를 사용할지 알 수 없다. 오로지 런타임에서만 결정되기 때문이다.

일반적인 상속으로 문제를 풀어가면 수직적으로 객체들이 증가되는 구조지만 브릿지 패턴을 사용한다면 아래와 같이 수평적으로 문제를 해결할 수 있다.

![20200624_165025](https://user-images.githubusercontent.com/30790184/85518311-116e7a00-b63b-11ea-85e6-ecac237c9bf7.png)



아래 피자집 예제를 확인해보자.

# 피자집 예제

추상화 개념에서 피자는 빠지지않는 것 같다. 피자를 만드는 프로그램을 브릿지 패턴을 사용해서 작성해보자.



## 브릿지 패턴 구현하기

브릿지 패턴을 사용해서 Pizza 클래스와 PizzaImp 클래스를 작성해보자

**Pizza.java**

```java
package me.sup2is;

public abstract class Pizza {

    private PizzaImp pizzaImp;

    public Pizza(PizzaImp pizzaImp) {
        this.pizzaImp = pizzaImp;
    }

    public void taste() {
        pizzaImp.taste();
    }

}

```

**PizzaImp.java**

```java
package me.sup2is;

public interface PizzaImp {

    void taste();

}

```

<br>

보통 우리에게 익숙한 구현방식은 아래와 같다.

> ```java
> public interface Pizza {
> 	void teste()
> }
> 
> 
> ...
>     
>     
> public class ChicagoPizza implements Pizza {
>     @Override
>     public void taste() {
>         ...
>     }
> }
> ```
>
> Pizza라는 인터페이스로 ChicagoPizza를 수직적으로 확장해서 다형성을 구현한다.

그러나 위에서 보인 Pizza 클래스와와 PizzaImp 클래스는 조금 느낌이다. 위에서 브릿지 패턴을 설명하면서 **추상적 개념에 해당하는 클래스와 실제 구현부를 분리함**이라는 문장이 이에 해당한다. Pizza는 추상적 개념으로 사용되고 실제 구현부는 PizzaImp로 구현했다.

## PizzaImp의 구현체 만들기

실제 구현체는 PizzaImp을 통해서 구현한다.

**PepperoniPizza.java, MargheritaPizza.java, ChicagoPizza.java**

```java
package me.sup2is;

public class PepperoniPizza implements PizzaImp {
    @Override
    public void taste() {
        System.out.println("Pepperoni pizza taste!!");
    }
}


...


public class MargheritaPizza implements PizzaImp {
    @Override
    public void taste() {
        System.out.println("Margherita pizza taste!!");
    }
}


...


public class ChicagoPizza implements PizzaImp {
    @Override
    public void taste() {
        System.out.println("ChicagoPizza pizza taste!!");
    }
}

```



## 피자 만들기

피자의 기본이 되는 도우클래스를 작성한다.

```java
package me.sup2is;

public class Dough extends Pizza{
    public Dough(PizzaImp pizzaImp) {
        super(pizzaImp);
    }
}

```

도우 클래스는 Pizza 클래스를 상속받아서 구현하는데 인스턴스 생성 시점에는 반드시 PizzaImp 타입의 실제 구현체가 넘어와야한다.

실제 브릿지 패턴의 사용은 다음과 같다.

```java
package me.sup2is;

public class Main {

    public static void main(String[] args) {

        Dough dough = new Dough(new PepperoniPizza());
        dough.taste();

        //갑자기 고객이 주문을 변경함
        dough = new Dough(new ChicagoPizza());
        dough.taste();

        //갑자기 고객이 주문을 또 변경함
        dough = new Dough(new MargheritaPizza());
        dough.taste();
    }
}

```

런타임시점에 알맞게 PizzaImp타입의 구현체만 전달해주기 때문에 런타임에서 매우 유연하게 프로그램을 작성할 수있다.

```
Pepperoni pizza taste!!
ChicagoPizza pizza taste!!
Margherita pizza taste!!
```



# 마무리

만약 추상 팩토리 패턴을 공부하신 분이라면 브릿지패턴과 매우 유사하다는 느낌을 받을 수 있다. 두 개의 차이점에 대해서 조금이라도 도움을 얻고 싶다면 [https://stackoverflow.com/questions/7700854/abstractfactory-versus-bridge-pattern](https://stackoverflow.com/questions/7700854/abstractfactory-versus-bridge-pattern)을 참고해도 좋다.



<hr>
포스팅은 여기까지 하겠습니다. 퍼가실때는 출처를 반드시 남겨주세요!

예제: [https://github.com/sup2is/study/tree/master/design-pattern/bridge-pattern](https://github.com/sup2is/study/tree/master/design-pattern/bridge-pattern)

<br>

**References**

- GoF의 디자인 패턴 - 에릭 감마, 리처드 헬름, 랄프 존슨, 존 블리시디스
- [https://stackoverflow.com/questions/319728/when-do-you-use-the-bridge-pattern-how-is-it-different-from-adapter-pattern](https://stackoverflow.com/questions/319728/when-do-you-use-the-bridge-pattern-how-is-it-different-from-adapter-pattern)
- [https://lktprogrammer.tistory.com/35](https://lktprogrammer.tistory.com/35)