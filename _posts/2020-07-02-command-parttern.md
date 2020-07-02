---
layout: post
title: "커맨드(Command) 패턴 Feat.Java"
tags: [Java, Design Pattern, Command Pattern, Behavioral Pattern]
date: 2020-07-02
comments: true
---



<br>

# OverView

오늘은 행동 패턴 중 하나인 커맨드 패턴을 다룬다.

# 커맨드(Command) 패턴

커맨드 패턴은 **연산을 호출하는 객체와 수행하는 객체를 분리하는 패턴**이다. 이 패턴에서 핵심은 통상적으로 `execute()` 라는 추상 메서드를 가진 Command 인터페이스를 통해 기능을 확장해 나간다. Command 객체의 서브클래스들은 실제 수행에 관련된 객체를 인스턴수 변수로 저장하여 `execute()` 내부에서 실제 수행될 객체의 명령을 실행시킨다.



![Command_Design_Pattern_Class_Diagram](https://user-images.githubusercontent.com/30790184/86317868-75142b00-bc6b-11ea-8dcd-6a13586490b9.png)

- Command: 연산 수행에 필요한 인터페이스를 선언
- ConcreateCommand : 객체와 액션 간의 연결성읠 정의함 또한 처리 객체에 정의된 연산을 호출하도록 execute를 구현함 
- client: 객체를 생성하고 처리 객체로 정의함
- Invoker: 명령어에 처리를 수행할 것을 요청함
- Receiver: 요청에 관련된 연산 수행 방법을 알고있음

사실 모든 패턴이 그렇듯이 코드를 보면 조금 더 이해가 쉽다. 에어컨을 만드는 예제를 확인해보자

# 에어컨 만들기 

여기서 만들 에어컨은 작은 컨트롤러가 있다. 기본기능에만 충실해서 다음과 같은 버튼이 있다.

- 전원 버튼 (toggle)
- 온도 올리기 버튼
- 온도 내리기 버튼

세가지 기능을 갖는 컨트롤러를 통해 에어컨을 조작해보자



## 인터페이스 작성하기

위에서 언급했듯이 일단 Command 객체를 만들어야한다. 

```java
package me.sup2is;

public interface Command {
    void execute();
}

```

Command 인터페이스는 단순히 `execute()` 라는 추상메서드를 갖고 있다. 이 Command를 확장해서 실제 기능을 수행하는 ConcreateCommand 객체를 만들어보자.

**DecTemperatureCommand.java**

```java
package me.sup2is;

public class DecTemperatureCommand implements Command{

    private AirConditioner airConditioner;

    public DecTemperatureCommand(AirConditioner airConditioner) {
        this.airConditioner = airConditioner;
    }

    @Override
    public void execute() {
        airConditioner.decrease();
    }
}

```

**IncTemperatureCommand.java**

```java
package me.sup2is;

public class IncTemperatureCommand implements Command{

    private AirConditioner airConditioner;

    public IncTemperatureCommand(AirConditioner airConditioner) {
        this.airConditioner = airConditioner;
    }

    @Override
    public void execute() {
        airConditioner.increase();
    }
}

```

**PowerCommand.java**

```java
package me.sup2is;

public class PowerCommand implements Command {

    private AirConditioner airConditioner;

    public PowerCommand(AirConditioner airConditioner) {
        this.airConditioner = airConditioner;
    }

    @Override
    public void execute() {
        airConditioner.togglePower();
    }
}

```

Command 인터페이스를 확장한 XXXCommand 클래스들은 `execute()`를 구현하지만 실제 메서드 내부에서 구현하는게 아니라 Receiver의 메서드만 수행하도록 구현해야한다. 

이제 실제 로직을 수행하는 Receiver를 구현해보자.



## Receiver 구현하기

**AirConditioner.java**

```java
package me.sup2is;

public class AirConditioner {

    private boolean operation;
    private int windStrength;


    public AirConditioner() {
        windStrength = 27;
        operation = false;
    }

    public void togglePower() {
        operation = !operation;
        System.out.println("현재 에어컨 동작 상태 : " + (operation ? "실행중" : "종료"));
    }

    public void increase() {
        if(!operation) {
            System.out.println("에어컨이 동작하지 않습니다!");
            return;
        }

        if(windStrength < 30) {
            windStrength ++;
        }

        System.out.println("현재 온도 : " + windStrength);
    }

    public void decrease() {
        if(!operation) {
            System.out.println("에어컨이 동작하지 않습니다!");
            return;
        }

        if(windStrength > 18) {
            windStrength --;
        }

        System.out.println("현재 온도 : " + windStrength);
    }


}

```

AirConditioner 클래스는 실제 자기 자신의 동작을 나타낸다.

이제 AirConditioner 클래스와 에어컨을 어떻게 조종할지에 대한 커맨드 명령을 만들었으니 실제 커맨드를 클라이언트에게 노출하는 Invoker를 생성해보자.



## Invoker 구현하기 

**RemoteController.java**

```java
package me.sup2is;

public class RemoteController {

    private Command command;

    public RemoteController(Command command) {
        this.command = command;
    }

    public void setCommand(Command command) {
        this.command = command;
    }

    public void pressButton() {
        command.execute();
    }


}

```

RemoteController 클래스는 Command 타입의 인스턴스 변수를 갖고 있고 생성자시점과 `setCommand()` 라는 메서드로 명령을 변경할 수 있다. 이제 에어컨을 직접 동작시켜보자.

## 에어컨 동작시키기

**Main.java**

```java
package me.sup2is;

public class Main {

    public static void main(String[] args) {

        AirConditioner airConditioner = new AirConditioner();

        Command decTemperatureCommand = new DecTemperatureCommand(airConditioner);
        Command incTemperatureCommand = new IncTemperatureCommand(airConditioner);
        Command powerCommand = new PowerCommand(airConditioner);

        RemoteController remoteController = new RemoteController(powerCommand);
        remoteController.pressButton();

        remoteController.setCommand(incTemperatureCommand);

        remoteController.pressButton();
        remoteController.pressButton();
        remoteController.pressButton();
        remoteController.pressButton();
        remoteController.pressButton();

        remoteController.setCommand(decTemperatureCommand);

        remoteController.pressButton();
        remoteController.pressButton();
        remoteController.pressButton();
        remoteController.pressButton();

        remoteController.setCommand(powerCommand);
        remoteController.pressButton();

    }

}

```

이렇게 실행시킨 프로그램의 결과는 다음과 같다.

```
현재 에어컨 동작 상태 : 실행중
현재 온도 : 28
현재 온도 : 29
현재 온도 : 30
현재 온도 : 30
현재 온도 : 30
현재 온도 : 29
현재 온도 : 28
현재 온도 : 27
현재 온도 : 26
현재 에어컨 동작 상태 : 종료
```



# 마무리

추가적으로 RemoteController 내부에 Command 타입 인스턴수 변수의 타입을 `Stack<Command>` 이나 `List<Command>` 타입으로 두고 `setCommand()` 시점이나 `pressButton()` 시점에 실행하는 명령을 저장함으로써 명령에 대한 기록을 남길 수 있다. 이런식으로 구현하면 기존에 실행했던 명령들을 전부 취소하는 등의 행위를 할 수 있다.



<hr>
포스팅은 여기까지 하겠습니다. 퍼가실때는 출처를 반드시 남겨주세요!


예제: [https://github.com/sup2is/study/tree/master/design-pattern/command-pattern](https://github.com/sup2is/study/tree/master/design-pattern/command-pattern)

<br>

**References**

- GoF의 디자인 패턴 - 에릭 감마, 리처드 헬름, 랄프 존슨, 존 블리시디스
- [https://johngrib.github.io/wiki/command-pattern/](https://johngrib.github.io/wiki/command-pattern/)

