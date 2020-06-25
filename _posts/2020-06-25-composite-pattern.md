---
layout: post
title: "컴포지트(Composite) 패턴 Feat.Java"
tags: [Java, Design Pattern, Composite Pattern, Structural Pattern]
date: 2020-06-25
comments: true
---



<br>

# OverView

이번시간엔 구조패턴중 하나인 컴포지트 패턴에 대해서 알아보도록 하겠다.

# 컴포지트(Composite Pattern) 패턴

컴포지트 패턴은 전체의 계층을 하나의 인터페이스로 통합해서 트리구조로 구성하는 구조 패턴중 하나다. 개별객체와 그 개별객체를 포함하는 복합객체를 모두 동일하게 다룬다.

이런 컴포지트 패턴의 가장 중요한 요소는 개별객체와 복합객체 모두를 표현할 수 있는 하나의 추상화 클래스를 정의하는 것이다.

이 컴포지트 패턴은 파일 - 폴더와 같이 전체의 객체 계통을 표현하고싶을 때 사용하면 된다. 

![1280px-Composite_UML_class_diagram_(fixed) svg](https://user-images.githubusercontent.com/30790184/85658668-88187f80-b6ee-11ea-9ea6-eb6d57e80bb0.png)

- Component: 집합 관계에 정의될 모든 객체에 대한 인터페이스를 정의한다. 모든 클래스에 해당하는 공통의 행동을 구현한다.
- Leaf: 가장 말단 객체, 자식이 없고 객체 합성에 가장 기본이 되는 객체의 행동을 정의한다.
- Composite: 자식이 있는 구성 요소에 대한 행동을 정의함 자신이 복합하는 요소들을 저장하면서, Component 인터페이스에 정의된 자식 관련 연산을 구현한다.



직접 예제를 구성하면서 컴포지트 패턴에 대해서 알아보자.

# 파일 - 폴더 구조 구성하기

위에서 설명한 파일 - 폴더 구조를 직접 구성하면서 컴포지트 패턴을 설명하도록 하겠다. 파일 - 폴더를 추상화한 인터페이스를 먼저 작성해보자.



## 인터페이스 구성하기

**FileSystem.java**

```java
package me.sup2is;

public interface FileSystem {

    void print();

}

```

파일과 폴더를 추상화한 FileSystem 인터페이스는 `print()` 라는 추상메서드를 가졌다. 간단히 print 만 찍는 메서드로 구현할 것이다.

이어서 FileSystem을 구현하는 File 클래스와 Folder 클래스를 구현해보자.

**File.java**

```java
package me.sup2is;

public class File implements FileSystem {

    String name;

    @Override
    public void print() {
        System.out.println(this.getClass().getSimpleName() + "(" + name + ")");
    }

    public File(String name) {
        this.name = name;
    }
}

```

**Folder.java**

```java
package me.sup2is;

import java.util.ArrayList;
import java.util.List;

public class Folder implements FileSystem {

    String name;

    private List<FileSystem> files = new ArrayList<>();

    @Override
    public void print() {
        System.out.println(this.getClass().getSimpleName() + "(" + name + ")");
        files.forEach(fileSystem -> fileSystem.print());
    }

    public void addFile(FileSystem file) {
        files.add(file);
    }

    public Folder(String name) {
        this.name = name;
    }
}

```

여기에서 File 클래스는 단일 객체(Leaf)로 표현된다. 따라서 자신의 이름을 갖는 name 이라는 필드만 존재하고 Folder 클래스가 복합 객체(Composite)로 사용된다. 이 복합객체는 FileSystem 타입의 List를 갖고 있다. 인스턴스 메서드로 갖고 있는 `addFile()`은 FileSystem 타입의 인스턴스를 저장하는데 File과 Folder 모두 FileSystem 타입이기때문에 모두 들어올 수 있다.

## 파일 -폴더 구현하기

이제 직접 파일 - 폴더 구조를 구성해보도록 하겠다.

**Main.java**

```java
package me.sup2is;

public class Main {

    public static void main(String[] args) {

        Folder folder1 = new Folder("최상위 폴더");
        File myFile1 = new File("myFile1");
        File myFile2 = new File("myFile2");
        folder1.addFile(myFile1);
        folder1.addFile(myFile2);

        Folder folder2 = new Folder( "1depth");
        File myFile3 = new File("myFile3");
        File myFile4 = new File("myFile4");
        folder2.addFile(myFile3);
        folder2.addFile(myFile4);

        Folder folder3 = new Folder("2depth");
        File myFile5 = new File("myFile5");
        File myFile6 = new File("myFile6");
        folder3.addFile(myFile5);
        folder3.addFile(myFile6);

        Folder folder4 = new Folder("3depth");
        File myFile7 = new File("myFile7");
        File myFile8 = new File("myFile8");
        folder4.addFile(myFile7);
        folder4.addFile(myFile8);

        folder1.addFile(folder2);
        folder2.addFile(folder3);
        folder3.addFile(folder4);

        folder1.print();
        System.out.println("====================");
        folder2.print();
        System.out.println("====================");
        folder3.print();
        System.out.println("====================");

    }
}

```

4개의 Folder 인스턴스와 각각 여러개의 File 인스턴스를 넣어서 파일 - 폴더 구조를 형성했다. File과 Folder는 모두 하나의 인터페이스로 통일되었기 때문에 `folder1.addFile(folder2);`형태로 연결이 가능하다.

프로그램의 실행 결과는 아래와 같다.

```
Folder(최상위 폴더)
	File(myFile1)
	File(myFile2)
	Folder(1depth)
		File(myFile3)
		File(myFile4)
		Folder(2depth)
			File(myFile5)
			File(myFile6)
			Folder(3depth)
				File(myFile7)
				File(myFile8)
====================
Folder(1depth)
	File(myFile3)
	File(myFile4)
	Folder(2depth)	
		File(myFile5)
		File(myFile6)
		Folder(3depth)
			File(myFile7)
			File(myFile8)
====================
Folder(2depth)
	File(myFile5)
	File(myFile6)
		Folder(3depth)
		File(myFile7)
		File(myFile8)
====================

```

조금 더 보기 좋은 결과를 나타내기 위해 별도의 인덴트를 넣었다. 이렇게 단일 객체와 복합 객체로 구성된 하나의 일관된 구조를 구성할때는 컴포지트 패턴을 사용하면 된다.

# 마무리

컴포지트 패턴의 장점은 사용자 입장에서는 이게 단일 객체인지 복합 객체인지 신경쓰지 않고 사용할 수 있다는 장점이 있지만 설계가 지나치게 범용성을 갖기 때문에 새로운 요소를 추가할 때 복합 객체에서 구성 요소에 제약을 갖기가 힘들다.



<hr>
포스팅은 여기까지 하겠습니다. 퍼가실때는 출처를 반드시 남겨주세요!

예제: [https://github.com/sup2is/study/tree/master/design-pattern/composite-pattern](https://github.com/sup2is/study/tree/master/design-pattern/composite-pattern)

<br>

**References**

- GoF의 디자인 패턴 - 에릭 감마, 리처드 헬름, 랄프 존슨, 존 블리시디스
- [https://www.baeldung.com/java-composite-pattern](https://www.baeldung.com/java-composite-pattern)
- [https://stackoverflow.com/questions/5334353/when-should-i-use-composite-design-pattern](https://stackoverflow.com/questions/5334353/when-should-i-use-composite-design-pattern)