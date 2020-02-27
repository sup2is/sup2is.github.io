---
layout: post
title: "AOP"
tags: [Spring Framework, AOP]
comments: true
date: 2019-03-27
parent: Spring
nav_order: 2
---









네 .. 저번시간에 DI/IoC 위주로 Spring을 한번 파헤쳐보자고 했는데 역시 블로그가 쉬운게 아니네요 .. 글솜씨가 많이 없지만 양해바랍니다. ㅠㅠ



<br>



# AOP 란 ?

------



간단하게 용어설명을 하자면 Aspect-Oriented Programming인데요. 직역하면 관점지향 프로그래밍입니다. 네.. 해석한 한글이 더 어렵구요... 설명하기 쉽게 비슷한 개념으로 Java에서 사용중인 OOP가 있습니다. Object-Oriented Programming 이죠. 이건 많이 들어보셔서 아시겠지만 바로 객체지향 프로그래밍입니다. Java에서는 Object 즉 클래스 위주의 프로그래밍으로 하나의 프로그램을 구현하잖아요? AOP도 같은 개념입니다. 관점을 지향하는 프로그래밍 방식이예요.

<br>

AOP는 주로 공통관심사에 사용되는데 가장 대표적인게 logging, transaction, 인증 등의 처리가 되겠습니다. AOP에서는 cross-cutting이라는 개념이 있는데 이게 바로 logging, transaction, 인증이 아주 좋은 예가 될 수 있을꺼 같네요.

<br>



AOP를 사용하는 이유에 대해서 조금 더 설명하면 다음과같은 Foo.class는 여러개의 메서드를 갖고 있어요.

```java
class Foo {
    
    
    public void a(){ 
        //something.. 
    }
    public void a2(){ 
        //something.. 
    }
    public void a3(){ 
        //something.. 
    }
    public void b(){ 
        //something.. 
    }
    public void c(){ 
        //something.. 
    }
    public void c2(){ 
        //something.. 
    }
    public void d(){ 
        //something.. 
    }
    
}
```



<br>

만약 a로 시작하는 메서드의 시작과 끝부분에 log를 남겨 메서드의 실행시간을 체크하고 싶은 경우가 있을 수 있죠. 그렇다면 이때 a로 시작하는 메서드 시작과 끝부분에 로직을 추가해 메서드의 실행시간을 체크할 수 있습니다. 하지만 별로 좋은생각은 아니죠 만약 Foo.class의 a메서드 뿐만 아니라 수많은 클래스의 a메서드, 또는 프로그램에서 호출하는 모든 메서드에 실행시간을 체크하고 싶다면? 저 방법으로는 어느정도 한계가 있죠. 그래서 사용하는게 AOP입니다. 

<br>







# AOP 용어

------



- ## Advice

------

Advice는 제공할 서비스를 나타내는데요. aspect가 무엇을 언제 할지 정의하는 역할을 합니다.

Spring에서는 총 5개의 관점이 있습니다.

1. Before : 메소드 실행 전 Advice 실행
2. After : 메소드 실행 후 Advice 실행
3. After-returning : 메서드가 성공 후(예외 없이)  Advice 실행
4. After-throwing : 메서드가 예외발생 후 Advice 실행
5. Around : 메소드 실행 전과 후 Advice 실행 (Before + After)

<br>

- ## Joinpoint

------

JoinPoint는 AOP를 적용할 수 있는 지점을 나타냅니다. Spring AOP에서 join point는 항상 메소드 실행을 나타냅니다.

<br>

- ## Pointcut

------

Pointcut은 표현식이나 패턴들을 활용하는 AOP의 EL이라고 생각하시면 되고 하나 또는 여러개의 joinpoint 집합입니다.

<br>

1. execution(public * *(..))

   \- 모든 public 메서드에 실행

2. execution(* set*(..))

   \- set으로 시작하는 모든 메서드에 실행

3. execution(* com.sup2is.service.AccountService.*(..))

   \- com.sup2is.service.AccountService 안에 모든 메서드에 실행

4. execution(* com.sup2is.service.\*.\*(..))

   \- com.sup2is.service 패키지 안에 모든 메서드에 실행

5. execution(* com.sup2is.service..\*.\*(..))

   \- com.sup2is.service의 서브패키지를 포함한 패키지 안에 모든 메서드에 실행

6. within(com.sup2is.service.*)

   \- com.sup2is.service 패키지 안에 모든 joinpoint에 실행

7. within(com.sup2is.service..*)

   \- com.sup2is.service의 서브패키지를 포함한 패키지 안에 모든 joinpoint에 실행

<br>

이 외에도 this, target,arg,@target,@within 등등이 있습니다.

<br>

- ## Introduction

------

Introduction을 통해 기존 클래스에 새 메소드나 특성을 추가 할 수 있습니다.

<br>

- ## Aspect

------

여러 객체에 공통 관심사 cross-cutting 개념을 갖고 있습니다. 위에서 설명한 logging,transaction,인증이 아주 좋은 예입니다.

<br>

- ## Weaving

------

Weaving은 advice를 다른 application 또는 Object와 관점을 연결하여 개체를 만드는 프로세스입니다. Weaving은 Compile tile, Run time, Load time에 실행될 수 있습니다. Spring AOP 에서는 Runtime에 동작합니다.

<br>

- ## Target Object

------

Target Object는 말 그대로 Advice가 적용된 하나 또는 여러개의 관점들 입니다. Spring AOP는 Runtime Proxy를 사용하여 구현되기 때문에 Spring proxy 객체로 알려져 있습니다. 



<br>



용어 설명은 이정도로 마치고.. 예제를 통해서 직접 사용해보는게 좋을것 같아요



<br>







# Example (Spring boot 2.1.3)

------





먼저 간단한 시나리오는 덧셈,뺄셈,나눗셈,곱셈의 동작을 하는 Calculator 객체가 있는데 이 메서드들에게 적절한 pointcut EL을 사용하여 log를 찍어보는 예제입니다. 매우 간단해요.



<br>

\- MyCalculator.class

```java
package com.sup2is.demo;

import org.springframework.stereotype.Component;

@Component
public class MyCalculator {
	
	public int add(int a, int b) {
		System.out.println("### add method 실행");
		return a + b;
	}
	
	public int sub(int a, int b) {
		System.out.println("### sub method 실행");
		return a - b;
	}
	
	public int division(int a, int b) {
		System.out.println("### division method 실행");
		return a / b;
	}
	
	public int multiply(int a, int b) {
		System.out.println("### multiply method 실행");
		return a * b;
	}
}


```

네 .. 정말 간단하네요 ..

<br>

이제 cross-cutting을 적용해볼께요. 

<br>

## @Before

------

\- CalculationAspect.class

```java
package com.sup2is.demo;

import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;
import org.springframework.stereotype.Component;

@Aspect //Aspect 명시
@Component //반드시 Component 등의 Spring Bean으로 등록해야 Aspect가 제대로 적용됨
public class CalculationAspect {

    // com.sup2is.demo.MyCalculator class 내부의 모든 메서드에 실행
	@Before("execution(* com.sup2is.demo.MyCalculator.*(..))") 
	public void beforeLog(JoinPoint joinPoint) {
		System.out.println("### " + joinPoint.getSignature().getName() +
                           " : before execute");
	}
	
}

```



<br>

이 CalculationAspect.class 에서 주목해야 하는부분은 바로 beforeLog() 메서드의 파라미터인 org.aspectj.lang.Joinpoint interface인데요. @Before, @After, @AfterReturning, @AfterThrowing 에 선택적으로 파라미터를 명시하면 자동으로 메서드에 파라미터가 넘어오게됩니다. 이 joinpoint는 getSignature() 메서드 처럼 AOP가 호출된 메서드의 정보 등이 넘어오니 자세한 내용은 [링크](https://www.eclipse.org/aspectj/doc/next/runtime-api/org/aspectj/lang/JoinPoint.html)를 직접확인해보시는게 좋을 것 같아요.



<br>

\- SpringDemoApplicationTests.class

```java
package com.sup2is.demo;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

@RunWith(SpringRunner.class)
@SpringBootTest
public class SpringDemoApplicationTests {

	@Autowired
	private MyCalculator myCalculator;
	
	@Test
	public void addTest() {
		System.out.println(myCalculator.add(5, 5));
	}
	@Test
	public void subTest() {
		System.out.println(myCalculator.sub(5, 5));
	}
	@Test
	public void divisionTest() {
		System.out.println(myCalculator.division(5, 5));
	}
	@Test
	public void multiplyTest() {
		System.out.println(myCalculator.multiply(5, 5));
	}

}

```



<br>

저는 @Before 어노테이션을 사용해서 MyCalculator.class 내부에 동작하는 모든 메서드에 

\#\#\# 메서드 이름 : before execute 라는 로그를 찍게 구현해봤어요. 실행해보겠습니다.

<br>

\- console



```
...
### sub : before execute
### sub method 실행
0
### division : before execute
### division method 실행
1
### add : before execute
### add method 실행
10
### multiply : beforeexecutee
### multiply method 실행
25
```



<br>

생각보다 간단하죠? 저는 분명 MyCalculator.class 내부에 log를 넣어놓은 적이 없는데 AOP가 알아서 log를 남겨주고 있습니다.



<br>

## @After

------



마찬가지로 @After는 메서드 실행 이후에 동작합니다. 

\- CalculationAspect.class

```java
	// com.sup2is.demo.MyCalculator class 내부의 add 메서드에 실행
	@After("execution(* com.sup2is.demo.MyCalculator.add(..))") 
	public void afterLog(JoinPoint joinPoint) {
		System.out.println("### " + joinPoint.getSignature().getName() +
                           " : after execute");
	}
```



<br>

@After는 add라는 메서드에만 동작하게 해놨는데요. 실행해보면

<br>

\- console

```
### sub : before execute
### sub method 실행
0
### division : before execute
### division method 실행
1
### add : before execute
### add method 실행
### add : after execute
10
### multiply : before execute
### multiply method 실행
25
```

<br>

보시는것처럼 add 메서드에만 @After 가 동작한걸 확인하실 수 있어요.





<br>

## @AfterReturing

------

<br>

@AfterReturing은 메서드가 예외 없이 성공적으로 끝났을때 호출이 되는데요.

이번에는 add 메서드와 division 메서드에 동작시키도록 해 보겠습니다. 메서드명에 공통적으로시작하는 접두어가 있다면 com.sup2is.demo.MyCalculator.find*(..) 으로 표현식을 작성 할 수 있지만 그렇지 않다면 && 와 \|\|를 이용해 표현식을 연결 할 수 있습니다.



<br>

\- CalculationAspect.class

```java
	// com.sup2is.demo.MyCalculator class 내부의 add,division 메서드에 실행
	@AfterReturning("execution(* com.sup2is.demo.MyCalculator.add(..) || execution(* com.sup2is.demo.MyCalculator.division(..)" )
	public void afterReturning(JoinPoint joinPoint) {
		System.out.println("### " + joinPoint.getSignature().getName() +" : after returning execute");
	}
```



<br>

myCalculator.division() 메서드 파라미터에 5와 0을 넘겨서 의도적으로java.lang.ArithmeticException 발생시킵니다.



<br>

\- SpringDemoApplicationTests.class

```java
	@Test
	public void addTest() {
		System.out.println(myCalculator.add(5, 5));
	}

 ... 
     
	@Test
	public void divisionTest() {
		System.out.println(myCalculator.division(5, 0));
	}
```



<br>



\- console

```
### sub : before execute
### sub method 실행
0
### division : before execute
### division method 실행  <-- java.lang.ArithmeticException 발생

### add : before execute
### add method 실행
### add : after execute
### add : after returning execute
10
### multiply : before execute
### multiply method 실행
25
```

<br>

성공적으로 값을 반환한 add메서드와는 달리 division은 @AfterRiturning이 적용되지 않은걸 확인하실 수 있습니다.

<br>

추가적으로 @AfterReturning은 반환한 return값을 가져올 수 있는 returning 필드가 @AfterReturning 어노테이션 필드에 내장되어 있는데요. 말그대로 Aspect가 적용된 메서드의 return값을 메서드 내부에서 사용할 수 있습니다. 아주 유용하게 쓰일 수 있죠.

<br>

\- CalculationAspect.class

```java
	// com.sup2is.demo.MyCalculator class 내부의 add,division 메서드에 실행
	@AfterReturning(pointcut = "execution(* com.sup2is.demo.MyCalculator.add(..)) ||"
			 + " execution(* com.sup2is.demo.MyCalculator.division(..))" , 
                    returning="value")
	public void afterReturning(JoinPoint joinPoint, Integer value) {
		System.out.println("### " + joinPoint.getSignature().getName() +
                           " : after returning execute");
		System.out.println("### value : " + value );
	}
```

<br>

\- console

```
### add : before execute
### add method 실행
### add : after execute
### add : after returning execute
### value : 10
10
```









<br>

## @AfterThrowing

------



@AfterThrowing은 메서드실행중 예외가 발생했을때만 동작합니다.





```java
	// com.sup2is.demo.MyCalculator class 내부의 division,multiply 메서드에 실행
	@AfterThrowing(pointcut = "execution(* com.sup2is.demo.MyCalculator.division(..)) ||"
			+ " execution(* com.sup2is.demo.MyCalculator.multiply(..))", throwing="ex")
	public void afterThrowing(JoinPoint joinPoint, ArithmeticException ex) {
		System.out.println("### " + joinPoint.getSignature().getName() +
                           " : afterThrowing execute");
		System.out.println("### " + ex.getMessage() +
                           " : exception occurred");
	}
```



<br>

@AfterThrowing 어노테이션은 throwing이라는 필드를 갖고 있는데요. 예외가 발생했을때 Aspect 내부에 Exception 객체를 전달해주는 역할을 합니다. 파라미터에 Exception.class를 입력하면 Exception전부를 잡아내지만 타입을 강하게 ArithmeticException.class로 준다면 ArithmeticException이 발생된 메서드만 @AfterThrowing이 발생합니다. 



<br>

\- MyCalculator.class

```java
	public int multiply(int a, int b) {
		System.out.println("### multiply method 실행");
		
		if(a == 0) {
			throw new IllegalArgumentException();
		}
		
		return a * b;
	}
```

<br>

간단하게 multiply 메서드를 조금 수정해줬는데요. 첫번째 파라미터가 0이면 IllegalArgumentException(); 를 반환하게 수정했습니다.

<br>



\- SpringDemoApplicationTests.class

```java
	@Test
	public void divisionTest() {
		System.out.println(myCalculator.division(5, 0));
	}
	@Test
	public void multiplyTest() {
		System.out.println(myCalculator.multiply(0, 5));
	}
```



<br>

\- console

```
### sub : before execute
### sub method 실행
0
### division : before execute
### division method 실행
### division : afterThrowing execute
### java.lang.ArithmeticException: / by zero : exception occurred

### add : before execute
### add method 실행
### add : after execute
### add : after returning execute
10
### multiply : before execute
### multiply method 실행

```

<br>



보시는것처럼 division 메서드는 ArithmeticException이 발생했기 때문에 @AfterThrowing이 적절하게 발생했지만 multiply 메서드는 그렇지 않은걸 확인하실 수 있습니다.



<br>

## @Around

------



<br>

@Around는 AOP 중에서도 가장 강력하게 작용하는 Advice입니다. 이 @Around는 다른 어노테이션과는 달리 메서드의 첫번째 인자로 반드시 JoinPoint의 하위타입인 org.aspectj.lang.ProceedingJoinPoint.class 가 반드시 와야 합니다. 메서드의 흐름을 보면 ProceedingJoinPoint.proceed 메서드를 통해 Advice가 적용된 메서드의 실행여부를 @Around내부에서 직접 제어할 수 있습니다.

<br>

\- CalculationAspect.class

```java
	@Around("execution(* com.sup2is.demo.MyCalculator.sub(..))")
	public Object aroundLog(ProceedingJoinPoint joinPoint) throws Throwable {
		System.out.println("### " + joinPoint.getSignature().getName() + 
                           " : before around excute");
		try {
			Object result = joinPoint.proceed();
			return result;
		}finally {
			System.out.println("### " + joinPoint.getSignature().getName() +
                               " : after around excute");
		}
	}
```

<br>

메서드의 실행순서는 @Advice가 먼저 실행되고 그 이후에 proceed 메서드가 실행되는데 proceed 메서드의 리턴값이 바로 sub메서드의 return값이 됩니다. 이 말은 @Advice 내부에서 결과값을 제어 할 수도 있다는거죠.



<br>

\- SpringDemoApplicationTests.class



```java
	@Test
	public void subTest() {
		System.out.println(myCalculator.sub(5, 5));
	}
```



<br>

\- console



```
### sub : before around excute
### sub : before execute
### sub method 실행
### sub : after around excute
0
```

<br>



이전에 적용했던 @Before와 @Advice에 순서도 확인하실 수 있으시죠?

<br>

@Around를 조금 변형해서 만약 결과값이 0이면 -1을 return하도록 수정해보겠습니다.

<br>

\- CalculationAspect.class

```java
	// com.sup2is.demo.MyCalculator class 내부의 sub 메서드에 실행
	@Around("execution(* com.sup2is.demo.MyCalculator.sub(..))")
	public Object aroundLog(ProceedingJoinPoint joinPoint) throws Throwable {
		System.out.println("### " + joinPoint.getSignature().getName() +
                           " : before around excute");
		try {
			Object result = joinPoint.proceed();
			
			if(Integer.parseInt(result.toString()) == 0) {
				return -1;
			}
			
			return result;
		}finally {
			System.out.println("### " + joinPoint.getSignature().getName() + "
                               : after around excute");
		}
	}
```

<br>

\- MyCalculator.class

```java
	public int sub(int a, int b) {
		System.out.println("### sub method 실행");
		return a - b;
	}
```



<br>

\- console

```
### sub : before around excute
### sub : before execute
### sub method 실행
### sub : after around excute
-1

```



<br>

확인하시는것처럼 메서드 내부에는 전혀 수정이 없었지만 @Around 내부에서 값을 제어하는것을 확인하실 수 있습니다.







<br>

포스팅은 여기까지 하겠습니다.  모든예제는 제 github에서 확인하실 수 있습니다.

예제 : <https://github.com/sup2is/spring-example/tree/master/springframework-2/src>

<br>

다음시간에는 TDD 관련 해서 포스팅 예정입니다~



<br>

퍼가실때는 링크와 출처를 반드시 명시해주세요. 감사합니다.

<br>



출처 : <https://www.javatpoint.com/spring-aop-tutorial>

출처 <https://www.topjavatutorial.com/frameworks/spring/spring-aop/aspect-oriented-programming-concepts/>

출처 : <https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#aop-api>

출처 : <http://www.javajigi.net/display/OSS/Aspect-Oriented+Programming+in+Java>

출처 : <https://www.tutorialspoint.com/spring/aop_with_spring.htm>

출처 : https://docs.spring.io/spring/docs/4.0.x/spring-framework-reference/html/aop.html#aop-introduction-defn

출처 : https://howtodoinjava.com/spring-aop/aspectj-around-annotation-example/