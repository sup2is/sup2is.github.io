---
layout: post
title: "책임 연쇄(Chain of Responsibility) 패턴 Feat.Java"
tags: [Java, Design Pattern, Proxy Pattern, Behavioral Pattern]
date: 2020-07-02
comments: true
---



<br>

# OverView

저번시간에 프록시 패턴을 끝으로 구조패턴에 대한 정리를 마쳤고 이번시간부터 행동 패턴에 대해서 다루도록 하겠다. 행동 패턴에 첫번째로 책임 연쇄 패턴을 다룬다.

# 책임 연쇄(Chain of Responsibility) 패턴

책임 연쇄 패턴은 **메시지를 보내는 객체와 이를 받아 처리하는 객체들 간의 결합도를 낮추기 위한 패턴**중에 하나다. **수신측에는 메시지를 처리할 객체들을 연결시켜 놓고 송신측에서 메시지를 수신측에게 전달하면 요청에 적합한 객체를 찾을때까지 연결된 객체들에게 메시지를 전달**한다.

이 책임 연쇄 패턴은 하나 이상의 객체가 요청을 처리해야하고 메시지를 받을 수신측과 메시지를 송신할 송신측의 결합도를 낮추는 역할을 한다. 따라서 객체에게 책임을 할당하는데 유연성을 높일 수 있는 장점이 있지만 객체 연결이 잘 성립되지 않았다면 해당 요청이 무시 될 수 있다.

![chain_of_responsibility](https://user-images.githubusercontent.com/30790184/86211732-cc55c500-bbb1-11ea-986b-3bf2f4f93a02.png)

- Handler: 요청을 처리하는 인터페이스를 정의하고 후속 처리자와 연결을 구현함. 연결고리에 연결된 다음 객체에게 다시 메시지를 보낸다.
- ConcreateHandler: 책임져야 할 행동이 있다면 스스로 요청을 처리하여 후속 처리자에 접근할 수 있음 즉 자신이 처리할 행동이 있으면 처리하고 그렇지 않으면 후속 처리자에 다시 처리를 요청한다.
- Client: ConcreateHandler 객체에게 필요한 요청을 보낸다.

# 우편물 분류 시스템

우편물 분류 시스템을 구성한다고 가정해보자. 컨베이어 벨트에 일렬로 나란히 늘어선 우편물들은 각각 자기자신의 목적지에 해당하는 곳으로 분류되어야 한다. 이때 어떠한 처리기에 의해 자동적으로 분류될 수 있도록 책임 연쇄 패턴을 사용해서 우편물 분류 시스템을 만들어 보자.

## 인터페이스 작성하기

**ClassificationSystem.java**

```java
package me.sup2is;

public abstract class ClassificationSystem {

    protected ClassificationSystem system;

    public abstract void classify(Post post);
    public abstract void setNext(ClassificationSystem system);

}

```

ClassificationSystem 클래스는 모든 처리기에 대한 인터페이스를 제공한다. `classify()`를 구현해서 우편물은 분류하고 `setNext()` 메서드를 통해서 체이닝을 구현할 것이다.

실제 ClassificationSystem 를 구현하는 처리기들을 만들어보자

**ConveyorSystem.java**

```java
package me.sup2is;

public class ConveyorSystem extends ClassificationSystem {

    @Override
    public void classify(Post post) {
        system.classify(post);
    }

    @Override
    public void setNext(ClassificationSystem system) {
        this.system = system;
    }
}

```

ConveyorSystem은 단순히 메시지들을 체이닝시키는 역할만 한다.

**GangnamguPostSystem.java**

```java
package me.sup2is;

public class GangnamguPostSystem extends ClassificationSystem {

    @Override
    public void classify(Post post) {
        if(post.getDestination().equals("강남구")) {
            System.out.println(post.getId() + "번 포스트를 강남구로 분류!");
        }else {
            system.classify(post);
        }
    }

    @Override
    public void setNext(ClassificationSystem system) {
        this.system = system;
    }
}

```

**SeochoguPostSystem.java**

```java
package me.sup2is;

public class SeochoguPostSystem extends ClassificationSystem {

    @Override
    public void classify(Post post) {
        if(post.getDestination().equals("송파구")) {
            System.out.println(post.getId() + "번 포스트를 송파구로 분류!");
        }else {
            system.classify(post);
        }
    }

    @Override
    public void setNext(ClassificationSystem system) {
        this.system = system;
    }
}

```

GangnamguPostSystem 와 SeochoguPostSystem를 살펴보면 `classify()` 에서 포스트의 목적지가 만약 자기의 관할이면 메시지를 처리하고 아니면 다음 시스템으로 전달하는 역할을한다.

코드를 추가하지는 않았지만 송파구 우편물을 담당하는 SongpaguPostSystem 클래스도 만들어서 총 3개의 처리기가 있다.

## 객체 연결시키고 프로그램 실행하기

이제 직접 연쇄작용을 통해서 프로그램을 실행시켜보자.

**Main.java**

```java
package me.sup2is;

public class Main {

    public static void main(String[] args) {

        ConveyorSystem conveyorSystem = new ConveyorSystem();
        GangnamguPostSystem gangnamguPostSystem = new GangnamguPostSystem();
        SongpaguPostSystem songpaguPostSystem = new SongpaguPostSystem();
        SeochoguPostSystem seochoguPostSystem = new SeochoguPostSystem();

        conveyorSystem.setNext(gangnamguPostSystem);
        gangnamguPostSystem.setNext(songpaguPostSystem);
        songpaguPostSystem.setNext(seochoguPostSystem);


        Post post1 = new Post(1, "강남구", "bla bla ...");
        Post post2 = new Post(2, "서초구", "bla bla ...");
        Post post3 = new Post(3, "송파구", "bla bla ...");
        Post post4 = new Post(4, "노원구", "bla bla ...");

        conveyorSystem.classify(post1);
        conveyorSystem.classify(post2);
        conveyorSystem.classify(post3);
        conveyorSystem.classify(post4);

    }

}

```

컨베이어벨트를 시작으로 각각 시스템끼리 연결을 시켜줘서 하나의 체인을 만들었다. 그리고 대표 수신객체인 conveyorSystem의 `classify()` 메서드를 호출해서 요청에 대한 체인을 만들었다.

```
1번 포스트를 강남구로 분류!
2번 포스트를 서초구로 분류!
3번 포스트를 송파구로 분류!
Exception in thread "main" java.lang.NullPointerException
	at me.sup2is.SeochoguPostSystem.classify(SeochoguPostSystem.java:10)
	at me.sup2is.SongpaguPostSystem.classify(SongpaguPostSystem.java:11)
	at me.sup2is.GangnamguPostSystem.classify(GangnamguPostSystem.java:10)
	at me.sup2is.ConveyorSystem.classify(ConveyorSystem.java:7)
	at me.sup2is.Main.main(Main.java:26)
```

맨 마지막 노원구로 가는 우편물은 담당하는 시스템이 없기 때문에 NPE가 발생했다. 이처럼 어떤 객체가 이 처리에 대한 수신을 담당한다 라는 것이 보장되지 않기 때문에 예외처리나 객체 연결을 잘 구성해야 한다.

# 마무리

예제를 작성하면서 조금 걱정되는 부분은 체인에 대한 디버깅이다. 체인을 구성함으로써 충분한 유연함을 얻긴 했지만 연쇄작용에서 오는 부작용도 몇가지 있을 것 같다. 대표적으로 어떤 시스템이 이 요청을 담당할 수 없다는 것. 디버깅이 어려울 것 같다. 테스트를 빡세게 작성해야 할 것 같다...



<hr>
포스팅은 여기까지 하겠습니다. 퍼가실때는 출처를 반드시 남겨주세요!


예제: [https://github.com/sup2is/study/tree/master/design-pattern/chain-of-responsibility-pattern](https://github.com/sup2is/study/tree/master/design-pattern/chain-of-responsibility-pattern)

<br>

**References**

- GoF의 디자인 패턴 - 에릭 감마, 리처드 헬름, 랄프 존슨, 존 블리시디스
- [https://sexycoder.tistory.com/105](https://sexycoder.tistory.com/105)

