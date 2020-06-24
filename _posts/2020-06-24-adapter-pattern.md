---

layout: post
title: "어댑터(Adapter) 패턴 Feat.Java"
tags: [Java, Design Pattern, Adapter Pattern, Structural Pattern]
date: 2020-06-24
comments: true
---



<br>

# OverView

오늘은 구조패턴의 첫번째 어댑터 패턴에 대해서 알아보도록 하겠다.

# 어댑터(Adapter) 패턴

어댑터 패턴은 클래스의 인터페이스를 사용자가 기대하는 인터페이스 형태로 적응시키고 서로 일치하지 않는 인터페이스를 갖는 클래스들을 동작시킬때 주로 사용되는 패턴이다.

이미 개발된 클래스의 인터페이스를 수정할 수 없거나 재사용하고싶다면, 즉 A와 B를 동시에 사용해야하는데 A와 B의 인터페이스가 일치하지 않을 경우에는 어댑터 패턴을 구현해서 C라는 클래스로 A와 B의 인터페이스를 맞춰줘야한다. 어댑터 패턴은 두가지 형태로 구현할 수 있는데

1. **C가 A의 인터페이스와 B의 구현을 모두 상속**  <- 클래스 어댑터
2. **B의 인스턴스를 C에 포함시키고 A 인터페이스를 C가 구현함** <- 오브젝트 어댑터

1번의 경우 다중상속이라는 개념이 언어적차원에서 필요하기때문에 자바에서는 상속과 구현을 적절하게 사용해야한다. wiki에 1번예제가 있다. [여기](https://ko.wikipedia.org/w/index.php?title=%EC%96%B4%EB%8C%91%ED%84%B0_%ED%8C%A8%ED%84%B4&action=edit&section=3)

클래스 어댑터의 uml은 다음과 같다

![다운로드](https://user-images.githubusercontent.com/30790184/85502151-b4b29580-b621-11ea-83a3-d364fafea587.gif)



나는 예제작성을 오브젝트 어댑터를 사용해서 구현했다 오브젝트 어댑터 uml은 다음과 같다. 

![다운로드](https://user-images.githubusercontent.com/30790184/85500122-db6ecd00-b61d-11ea-8d6f-e148b29ae45a.png)



# 220V to 110V 변압기 프로그램 예제

책 내용만으로 예제를 작성하기가 조금 벅차서 인터넷을 좀 뒤져봤는데 변압기 관련된 내용이 몇개 있어서 그대로 예제로 적용하면 좋을 것 같아서 작성해봤다.

객체지향에서 헤어드라이기의 변압 220V를 110V로 바꾸는건 식은죽 먹기지만 현실세계에서 220V의 헤어드라이기를 110V로 바꾸는일은 쉽지 않다. 이런 경우 보통 변압기라는 제품을 사용하는데 이것을 어댑터라고 생각하면 쉽다.

변압기 프로그램예제의 요구사항은 다음과 같다.

- **110V 플러그에 220V의 가전제품을 사용할 수 있게 하는 어댑터 만들기**



## 헤어드라이기 만들기

모든 가전제품을 Electronic으로 추상화시켜서 Volt라는 고유 전압을 갖도록 구성했다.

**Electronic.java**

```java
package me.sup2is;

public abstract class Electronic {

    protected final Volt volt;

    protected Electronic(Volt volt) {
        this.volt = volt;
    }

    abstract void use();
    
    public boolean is220V() {
        return this.volt == Volt.V220;
    }
}

```

Electronic은 `use()`와 자신의 전압을 true false로 알려주는 `is220V()` 메서드를 갖고 있다. 모든 전자제품은 자기자신의 동작방식인 `use()`를 구현해야한다.



**HairDryer.java**

```java
package me.sup2is;

public class HairDryer extends Electronic{

    public HairDryer(Volt volt) {
        super(volt);
    }

    @Override
    public void use() {
        System.out.println("위이이이이이잉");
    }

}

```

Electronic을 상속받는 HairDryer 클래스다. 인스턴스 생성시에 변압을 입력해줘야한다.

**Volt.java**

```java
package me.sup2is;

public enum Volt {
    V220, V110
}

```

## VoltageAdapter 작성하기

다음과같이 기존에는 110V 콘센트가 있다고 가정한다

**ConcentricPlug110V.java**

```java
package me.sup2is;

public interface ConcentricPlug110V {

    void apply();

}

```

실제 변압을 체크하고 변경해주는 어댑터를 아래와 같이 구현할 수 있다.

**VoltageAdapter.java**

```java
package me.sup2is;

public class VoltageAdapter implements ConcentricPlug110V{

    private Electronic electronic;

    public VoltageAdapter(Electronic electronic) {
        this.electronic = electronic;
    }

    @Override
    public void apply() {
        if(electronic.is220V()){
            System.out.println("## 110V의 변압기를 사용");
        }
        electronic.use();
    }
}

```

기존에 ConcentricPlug110V의 `apply()`를 VoltageAdapter 내부에서 사용하면 다음과 같이 220V의 헤어드라이기를 110V의 변압기를 사용해서 변압을 구현할 수 있다.

이제 클라이언트에서 사용해보자.

```java
package me.sup2is;

public class Main {

    public static void main(String[] args) {

        HairDryer hairDryer = new HairDryer(Volt.V220); //<- 변경 불가능한 third party lib

        VoltageAdapter v = new VoltageAdapter(hairDryer);
        v.apply();

    }
}

```

220V의 헤어드라이기를 생성해서 VoltageAdapter를 사용해 동작시키면 다음과 같이 프로그램의 출력이 나온다.



```
## 110V의 변압기를 사용
위이이이이이잉
```

# 마무리

비교적 쉽게 이해되고 예제도 비교적 쉽게 작성한 편이였다. 이 어댑터 패턴이 안드로이드의 XXXAdapter 와 같은 클래스와 연관이 있나 싶어서 조금 찾아봤는데 관심이 있다면 [https://stackoverflow.com/questions/41626980/are-android-adapters-an-example-of-adapter-design-pattern](https://stackoverflow.com/questions/41626980/are-android-adapters-an-example-of-adapter-design-pattern)를 참고해보자.



<hr>
포스팅은 여기까지 하겠습니다. 퍼가실때는 출처를 반드시 남겨주세요!

예제: [https://github.com/sup2is/study/tree/master/design-pattern/adapter-pattern](https://github.com/sup2is/study/tree/master/design-pattern/prototype-patternhttps://github.com/sup2is/study/tree/master/design-pattern/prototype-pattern)

<br>

**References**

- GoF의 디자인 패턴 - 에릭 감마, 리처드 헬름, 랄프 존슨, 존 블리시디스
- [https://jusungpark.tistory.com/22](https://jusungpark.tistory.com/22)