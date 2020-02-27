---
layout: post
title: "Test(TDD)"
tags: [Spring Framework, Test, TDD, JUnit]
date: 2019-04-03
comments: true
parent: Spring
nav_order: 3
---









<br>

# TDD 란?

------



TDD는 Test-Driven-Development로써 직역하면 테스트 주도 개발입니다. 네.... 말그대로 테스트를 중심으로 개발한다는 개발방법론인데요. 간단하게 정의하면 기능이되는 코드 이전에 테스트코드를 작성해서 검증한 뒤 기능코드를 작성하는 방법입니다. 저같은 초급 개발자에겐 많이 생소한 느낌이지만 이미 많은 기업에서도 TDD를 적용하고 있습니다.

SW개발에 있어서 개발시간을 가장 크게 증가시키는 요인중에 하나가 바로 버그입니다. 이 버그는 Test가 정확하게 이루어지지 않아서 발생한다고 볼 수 있죠. TDD를 사용하면 선 테스트 후 개발이기 때문에 테스트를 거치지 않은 코드가 없습니다. 버그의 발생률을 상당히 줄여주는 이유기기도 하죠. 실제로 "TDD의 적용 여부에 따라서 개발시간이 30%가량 단축된다." 라는 글도 있을 정도로 TDD는 SW개발에서 상당히 중요한 역할을 하고 있습니다.

TDD의 궁극적인 목표는 "Clean code that works" 인데요. 깔끔하고 잘 동작하는 코드입니다. TDD의 과정을 요약하면 다음과 같은데요

<br>

> 1. 테스트를 작성한다. 
> 2. 작성한 테스트를 통과할 수 있도록 가장 빠른 방법으로 코드를 작성한다. 이 과정에 중복된 코드를 만들어도 상관 없다. 
> 3. 테스트를 수행한다. 
> 4. 테스트를 통과하면 작성한 코드에서 중복을 제거한다. 아니면 2번으로 돌아간다. 
> 5. 테스트를 수행한다. 
> 6. 테스트를 통과하면 완성. 다음 테스트를 1번부터 시작한다. 실패하면 4로 돌아가서 디버깅한다. 

<br>

자세한 내용은 [이곳](https://web.archive.org/web/20070628064054/http://xper.org/wiki/xp/TestDrivenDevelopment) , [저곳](https://web.archive.org/web/20061012050617/http://xper.org/wiki/xp/TDD_bc_f6_b7_c3_b9_fd)을 확인하시기 바랍니다.

<br>

# JUnit 이란?

------

<br>

JUnit은 java 진영에서 가장 강력하고 유명하게 사용하는 테스트 프레임워크입니다. TDD에서 아주 중요한 역할을 해줍니다. 주요 메서드와 어노테이션을 알아보는 시간을 가져볼께요.



<br>

## JUnit Assert Class

------

Assert class의 메서드들은 전부 static으로 선언되어 있기 때문에 Assert.assertEquals("", ""); 이런식으로 사용해도 되지만 편의성을 높이기 위해 보통은 class파일 최상단(package 바로 밑)에 import static org.junit.Assert.*; 이런식으로 정적선언 해준 뒤 메서드명만 사용하는게 일반적입니다.

Assert class 메서드의 true,false 값은 Test 메서드의 성공여부를 나타냅니다. 

<br>

### assertEquals()

------

void assertEquals(boolean expected, boolean actual)

두개의 값이 같을경우 true 아니면 false 예상값을 첫번째 파라미터에 넣어줍니다.

<br>

### assertFalse()

------

void assertFalse(boolean condition)

넘어온 값이 false면 true , true면 false

<br>

### assertTrue()

------

void assertTrue(boolean condition)

넘어온 값이 true면 true , false면 false

<br>

### assertNotNull()

------

void assertNotNull(Object object)

넘어온 Object가 null이면 false, null이 아니면 true

<br>

### assertNull()

------

void assertNotNull(Object object)

넘어온 Object가 null이면 true, null이 아니면 false

<br>

### fail()

------

void fail()

어떠한 경우라도 fail()을 만나면 Test결과 실패



<br>

## JUnit Annotaion

------

<br>

### @Test

------

@Test를 붙임으로써 이 메서드가 테스트 케이스로 실행될 수 있음을 나타내줍니다. 반드시 public void methodName 형태여야 합니다.

<br>

### @Before

------

@Before는 @Test 가 붙은 메서드가 실행되기 이전에 동작합니다. 보통 객체를 초기화시키는 작업에서 많이 사용하고 메서드 명은 setup을 사용합니다. 

<br>

### @After

------

@After는 @Before와 비슷하게 메서드가 실행되고 난 후에 동작합니다. 주로 변수설정을 다시하거나 임시파일을 삭제하는곳에서 사용합니다.

<br>

### @Ignores

------

@Ignores는 말 그대로 테스트를 사용하지 않고 싶을때 사용합니다.

<br>

### @Test(timeout=500)

------

@Test에 timeout 필드는 이 테스트에 시간제한을 명시해주는 필드입니다. 이 필드의 값은 ms 기준이고 테스트가 0.5초가 지나면 테스트 결과는 실패로 간주됩니다.

<br>

### @Test(expected=IllegalArgumentException.class)

------

@Test에 expected 필드는 이 테스트가 던지는 예외를 지정할 수 있습니다. 만약 IllegalArgumentException.class가 테스트 도중 이 메서드로 던져지면 테스트 결과는 성공, 아니면 실패로 간주됩니다.

<br>

### @RunWith(SpringRunner.class)

------

@Runwith는 JUnit 프레임워크의 확장입니다. 이를 이용하여 자신에게 필요한 테스트 러너를 직접 만들어서 자신만의 고유한 기능을 추가해 테스트를 수행할 수 있습니다.  JUnit 5 이상 버전에서는 더이상 @Runwith가 아니라 @ExtendWith를 사용합니다.

<br>

### @SpringBootTest

------

@SpringBootTest는 Spring legacy의 @ContextConfiguration 어노테이션을 SpringBoot에서 사용할 수 있도록 대체해주는 역할을 합니다. 이 어노테이션은 ApplicationContext를 만들고 테스트할 수 있도록 해줍니다. Spring boot 기준으로 반드시@RunWith(SpringRunner.class) 도 명시해주어야하며 이 또한 JUnit5 이상일 경우 @ExtendWith로 대체할 수 있습니다.

<br>

# MockObject 란?

------

<br>

일단 Mock이란 직역하면 모조품이란 뜻을 가지고 있습니다. 객체지향 프로그래밍에서 Mock Object는 application을 테스트할 때 주로 사용되는 모의 객체입니다. spring-boot-starter-test 모듈에서도 역시 java mocking framework인 Mockito를 내장하고 있습니다.

<br>

> A mock object can be useful in place of a real object that:
>
> 1. Runs slowly or inefficiently in practical situations
> 2. Occurs rarely and is difficult to produce artificially
> 3. Produces non-deterministic results
> 4. Does not yet exist in a practical sense
> 5. Is intended mainly or exclusively for conducting tests.

<br>

이 Mock을 사용하는 이유는 실제 A객체를 테스트해야하는데 A객체가 B객체와 의존되어있는 상황에서 B객체와 의존성을 단절시키기 위해 Mock으로 대체하여 테스트를 진행할 수 있습니다. Mock은 주로 Test하기 어려운 DB가 묶인 상황에도 많이 사용됩니다.

<br>

이 Mock의 사용은 예제에서 한번 다뤄보겠습니다. 예제는 Mockito를 사용합니다.

<br>

테스트 더블에 대한 이야기는 [링크](https://repo.yona.io/files/4000) 에서 아주 정확하게 얘기해주고 있으니 참고 바랍니다. 



<br>

# Example (Spring boot 2.1.3) JUnit + Mockito

------

이번예제는 간단하게 TDD + JUnit + Mockito를 사용하여 Test해보는 시간입니다.

Spring boot 기준으로 예제를 생성하면 다음과 같이 pom.xml에 spring-boot-starter-test 모듈이 추가되는데요

<br>

\- pom.xml

```xml
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
```

<br>

spring-boot-starter-test 모듈은 아래와 같은 라이브러리를 포함하고 있습니다.

<br>

> - [JUnit](http://junit.org/): The de-facto standard for unit testing Java applications.
> - [Spring Test](https://docs.spring.io/spring/docs/5.1.5.RELEASE/spring-framework-reference/testing.html#integration-testing) & Spring Boot Test: Utilities and integration test support for Spring Boot applications.
> - [AssertJ](https://joel-costigliola.github.io/assertj/): A fluent assertion library.
> - [Hamcrest](http://hamcrest.org/JavaHamcrest/): A library of matcher objects (also known as constraints or predicates).
> - [Mockito](http://mockito.org/): A Java mocking framework.
> - [JSONassert](https://github.com/skyscreamer/JSONassert): An assertion library for JSON.
> - [JsonPath](https://github.com/jayway/JsonPath): XPath for JSON.

<br>

따라서 기존 legacy 프로젝트처럼 junit 모듈을 추가하지 않아도 돼요!



<br>

예제 시나리오는 회원가입 비지니스 로직에서 id,pw 를 가진 User.class를 db에 넣기 이전에 pw값을 암호화하여 db에 넣어주는 예제입니다. 물론 실 db는 안들어가요.

<br>

먼저 간단하게 풀어서 써보면

1. 앞단에서 사용자 id, pw를 입력하고 회원가입 요청을 한다
2. 서버는 id, pw가 유효한지 검증한다 (ex : id는 5자 이상, pw는 특수문자 포함 8자 이상 등..)
3. 검증이 끝나면 pw를 암호화 알고리즘을 통해서 암호화 된 값으로 변환한다.
4. 검증과 암호화가 끝난 id,pw를 db에 넣는다

정도가 될꺼 같습니다.

<br>

(3과 4는 join()에서 한번에 이루어지기 때문에 한번에 처리합니다.)





<br>

\- User.class

```java
package com.sup2is.demo;

public class User {
	
	private String id;
	private String pw;
    
    
    public User(String id, String pw) {
		this.id = id;
		this.pw = pw;
	}

    
    //getter...
    //setter...
    
	
}

```

<br>

먼저 필요한 VO객체 입니다. 그냥 id와 pw 필드만 있는 간단한 User객체입니다.

<br>

다음은 JoinService interface입니다.

<br>

\- JoinService.interface

```java
package com.sup2is.demo;

public interface JoinService {
	
	public User join(User user);
	
	public boolean verify(User user);

}

```



<br>

메서드는 join, verify를 갖고 있는 interface 골격이 되겠습니다.

<br>

다음은 JoinService를 구현하는 몸체가 되는 JoinServiceImpl.class 입니다

\- JoinServiceImple.class

```java
package com.sup2is.demo;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

@Service
public class JoinServiceImpl implements JoinService {

    //@Autowired 런타임 에러를 위해 cryptoService는 required false로 줌
	@Autowired(required=false)
	private CryptoService cryptoService;
	
	@Override
	public User join(User user) {
		//user pw 암호화
		user.setPw(cryptoService.encrypt(user.getPw()));
		
		//join ... db에 넣는 로직 .. 
		return user;
	}

	@Override
	public boolean verify(User user) {
		
		//verify ...
		
		return false;
	}

}

```



<br>

verify 부분에서는 id,pw의 검증을 통해서 boolean값을 리턴하게 만들었습니다. 아직은 method 몸체랑 단순 return값 밖에 없는 상태예요.

<br>

join부분은 verify가 true일 경우에 동작하는데 주입받은 cryptoService를 이용해 cryptoService.encrypt(user.getPw()); 를 이용하여 암호화하는 부분입니다. 실 db에 넣지 않기 때문에 값을 확인하기 위하여 User객체를 반환합니다.

<br>

마지막으로

\- CryptoService.interface

```java
package com.sup2is.demo;

public interface CryptoService {
	
	public String encrypt(String pw);

}

```

<br>

그냥 매우 간단하게 암호화에 필요한 encrypt() 메서드가 들어있습니다.

그리고 아직 CryptoService에 대한 스펙은 명시가 되어있지 않습니다. 아직 어떤 알고리즘을 사용하기로 얘기하지 않았기 때문이죠.



<br>

시나리오대로 Test Case를 작성해보겠습니다



<br>

**1.앞단에서 사용자 id, pw를 입력하고 회원가입 요청을 한다**

------

<br>

이제 Test 클래스를 작성해보겠습니다. 이에 앞서 Test클래스를 만들어놓고 여러개의 테스트를 작성해야하는데 메서드 네임이 도저히 생각나지 않을때는 [링크](https://dzone.com/articles/7-popular-unit-test-naming) 를 참고하시기 바랍니다!

<br>

> MethodName_StateUnderTest_ExpectedBehavior
>
> : There are arguments against this strategy that if method names change as part of code refactoring than test name like this should also change or it becomes difficult to comprehend at a later stage. Following are some of the example:
>
> - isAdult_AgeLessThan18_False
> - withdrawMoney_InvalidAccount_ExceptionThrown
> - admitStudent_MissingMandatoryFields_FailToAdmit

<br>

저는 1번 방식으로 사용해보겠습니다. 후달리는 영어지만 되도록 영어를 사용하도록 노력해봅시다 ... 

<br>

@Autowired를 사용하기 때문에 Spring context가 필요하니 @Runwith와 @SpringBootTest를 명시해줍니다.

<br>

\- JoinServiceTest.class

```java
package com.sup2is.demo;

import static org.junit.Assert.assertEquals;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

@RunWith(SpringRunner.class)
@SpringBootTest
public class JoinServiceTest {
	
	@Autowired
	private JoinService joinService;

	@Test
	public void testVerify_provideUser_verifyIsTrue() {
		//given
		User user = new User("sup2is", "12345678");
		
		//when
		boolean isValid = joinService.verify(user);
			
		//then
		assertEquals(isValid, true);
	}

}

```

<br>

모든 Test Case는 given,when,then으로 역할을 명확하게 지정해서 하시면 좀 더 읽기 좋은 Test Case를 구성하실 수 있습니다.

<br>

given에는 User객체를 생성해주고 실제 user 객체를 사용할 then에서는 verify() 메서드를 호출해줬습니다. when에서의 결과 확인을하기 위해 assertEquals() 메서드를 호출해주었습니다.

<br>

assertEquals(isValid,true); 를 사용하여 값비교를 실행했습니다. 결과는 당연히 실패겠죠? 아직 저희는 verify()의 몸체를 작성하지 않고 return false로만 해뒀으니까요. 따라서 verify() 메서드를 수정해줍니다.

<br>

**2.서버는 id, pw가 유효한지 검증한다 (ex : id는 5자 이상, pw는 특수문자 포함 8자 이상 등..)**

------

<br>

\- JoinServiceImpl.class

```java
	@Override
	public boolean verify(User user) {
		
		// id는 5자 이상 pw는 8자 이상
		if(user.getId().length() >= 5 && user.getPw().length() >= 8) {
			return true;
		}
		
		return false;
	}

```



다시 테스트를 실행해봅니다. 초록불뜨는걸 확인했습니다. verify에서 특수문자도 검사해야하지만 id pw의 길이만 검사하기로 했습니다.

<br>

이제 join()를 테스트해보도록 하겠습니다.

<br>

**3.검증이 끝나면 pw를 암호화 알고리즘을 통해서 해시값으로 변환한다.검증과 암호화가 끝난 id,pw를 db에 넣는다**

------

<br>

**4.검증과 암호화가 끝난 id,pw를 db에 넣는다**

------

<br>

\- JoinServiceTest.class

```java
	@Test
	public void testVerify_provideUser_verifyIsTrue() {
		//given
		User user = new User("sup2is", "12345678");
		
		//when
		boolean isValid = joinService.verify(user);
			
		//then
		assertEquals(isValid, true);
	}
	
	@Test
	public void testJoin_provideUser_joinSuccess() {
		
		//given
		User user = new User("sup2is", "12345678");
		
		//when
		User encryptUser = joinService.join(user);
		
		//then
		assertEquals("encrypted String", encryptUser.getPw());
		
	}
```

<br>

테스트를 작성해보니 메서드 중복이 존재합니다. user를 생성해주는 부분을 JUnit에서 제공하는 @Before를 사용해서 중복제거를 해보도록하겠습니다.

<br>

\- JoinServiceTest.class

```java
//...	
 	
	private User user;
	
	@Before
	public void setup() {
		user = new User("sup2is", "12345678");
	}


//...


	@Test
	public void testVerify_provideUser_verifyIsTrue() {
		//given
		
		//when
		boolean isValid = joinService.verify(user);
			
		//then
		assertEquals(isValid, true);
	}
	
	@Test
	public void testJoin_provideUser_joinSuccess() {
		
		//given
		
		//when
		User encryptUser = joinService.join(user);
		
		//then
		assertEquals("encrypted String", encryptUser.getPw());
		
	}
```



<br>

이제 testJoin_provideUser_joinSuccess() 를 실행해보겠습니다. 네 당연히 에러가 나겠죠? 저희는 아직 joinService에서 사용하는 cryptoService를 생성하지 않았으니까요. NullPointerException를 뿜뿜합니다.

<br>

근데 cryptoService의 상세 스펙이 아직도 정해지지않았다면 어떻게해야할까요? 아직 어떤 암호화 알고리즘을 사용할 지 결정되지 않았다면 테스트를 중지해야할까요?

<br>

이런상황에서 사용하는게 바로 mock입니다. mock객체를 사용해서 모조품을 만들고 join메서드를 완성시키도록 하겠습니다.

<br>

mockito에서 제공하는 어노테이션을 사용해서 joinservice 내부에 @Autowired로 받는 cryptoService를 mock객체로 주입해보겠습니다.

<br>

먼저 사용될 어노테이션에대해 간단하게 설명드립니다.

<br>

## @Mock

------

@Mock은 Mock 객체를 생성해줍니다. mockito에서 제공하는 mock() 메서드와 동일한 역할을 합니다.



<br>

## @InjectMocks

------

@InjectMocks 는 Mock객체가 필요한 객체에 Mock객체를 연결시켜주는 역할을 합니다. 



<br>

*이런 ... JoinService 타입이 interface이기 때문에 @InjectMocks으로 인스턴스화 할 수 없군요 .. ㅠㅠ  어쩔 수 없이 JoinServiceTest.class의 joinService 필드를 JoinServiceImpl 타입으로 변경하였습니다. 이 예제는 만들면서 진행한것이기 때문에 위에는 별도의 처리를 하지 않겠습니다.

<br>

joinService의 ctyptoService를 Autowired하기 위해 @Autowired로 주입받던 joinService를 @InjectMocks을 사용하여 Mockito에게 인스턴스화를 떠넘깁니다. 그와 동시에 @Mock을 사용하여 cryptoService를 mock객체로 생성해줍니다.

<br>

이제 마저 테스트를 실행해보겠습니다.

<br>

\- JoinServiceTest.class

```java
	

	@InjectMocks // <-- @Autowired에서 @InjectMocks로 변경
	JoinServiceImpl joinService; //<-- 구현체로 변경
	
	@Mock // <-- mock 객체로 생성
	CryptoService cryptoService; 
	

// ...


	@Test
	public void testJoin_provideUser_joinSuccess() {
		
		//given
		when(cryptoService.encrypt(user.getPw())).thenReturn("encrypted String");
		
		//when & then
		User encryptUser = joinService.join(user);
		assertEquals("encrypted String", encryptUser.getPw());
		
	}

```

<br>

Test는 성공으로 떨어집니다. mockito에서 제공하는 when().thenReturn메서드를 통해 cryptoService.encrypt() 메서드의 몸체를 구현하지 않고 단순히 "encrypted String" 을 반환하게 만들었습니다.
<br>

이렇게 JoinService의 join() 메서드가 cryptoService의 encrypt() 메서드에게 의존하고 있음에도 불구하고 Mock을 사용하여 다른 CryptoService의 구현체 없이 테스트를 통과할 수 있었습니다. 앞서 말씀드린 의존관계를 어느정도 단절시키는 아주 좋은 예가 되었으면 좋겠습니다.

<br>

사실 저도 이번기회에 TDD 와 Mock을 제대로 접한거라 틀린 정보가 있을 수도 있고 예제 구현 방식에 있어서 미흡한 부분이 있을 수 있으니 참고바랍니다.



<br>

포스팅은 여기까지 하겠습니다.  모든예제는 제 github에서 확인하실 수 있습니다.

예제 : <https://github.com/sup2is/spring-example/tree/master/springframework-3/src>



<br>

spring 주제는 일단 여기까지로 하고 다음시간부터는 관심있는 주제부터 하나하나씩 포스팅 할 예정입니다.

<br>

퍼가실때는 링크와 출처를 반드시 명시해주세요. 감사합니다.

<br>



출처: <http://agiledata.org/essays/tdd.html>

출처: <http://www.nextree.co.kr/p11104/>

출처 : <https://repo.yona.io/doortts/blog/issue/1>

출처 : <https://web.archive.org/web/20070628064054/http://xper.org/wiki/xp/TestDrivenDevelopment>

출처 : <https://junit.org/junit5/>

출처 : <https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#boot-features-testing>

출처 : <https://doortts.tistory.com/169>

출처 : <https://dzone.com/articles/7-popular-unit-test-naming>

출처 : <https://web.archive.org/web/20061012050617/http://xper.org/wiki/xp/TDD_bc_f6_b7_c3_b9_fd>

출처 : <https://web.archive.org/web/20070628064054/http://xper.org/wiki/xp/TestDrivenDevelopment>

출처 : <https://www.guru99.com/junit-annotations-api.html>

출처 : <https://codedragon.tistory.com/5507>

출처 : <https://searchsoftwarequality.techtarget.com/definition/mock-object>

출처 : <http://hyeonjae-blog.logdown.com/posts/679308>

출처 : <https://stackoverflow.com/questions/16467685/difference-between-mock-and-injectmocks>