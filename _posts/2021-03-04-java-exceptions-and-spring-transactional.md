---
layout: post
title: "Java의 Exception 그리고 Spring의 @Transactional"
tags: [Java, Spring, Exception, CheckedException, UnCheckedException, Transactional]
date: 2021-03-04
comments: true
---

<br>



# Overview

이번시간에는 자바의 Exception 종류인 `CheckedException`, `UnCheckedException`, `Error`에 대해서 알아보고 Spring `@Transactional`에서 각각 에러들이 어떤식으로 처리되는지 알아보는 시간을 갖도록 하겠다.





# Java의 Exceptions

자바에는 크게 3가지 종류의 Exception이 있다. `CheckedException`, `UnCheckedException`, `Error` 이 Exception들을 알아보기 전에 최상위 클래스가 되는 `Throwable`을 먼저 알아보도록 하자.



## Throwable

이 `Throwable` 클래스는 자바의 모든 에러, 예외들의 최상위 클래스이다. 큰 분류로 나누었을때, 하위 클래스들은 다음과 같은 관계도를 가진다.

![20210303_155912](https://user-images.githubusercontent.com/30790184/109766400-74d49100-7c39-11eb-93b8-3cd169c44c2f.png)

`Throwable`은 생성 당시에 스레드의 실행 스냅샷을 포함하고 오류에 대한 자세한 메시지,  중첩된 `Throwable` 형태로 설계되었다. 실제 `Throwable` 생성자 목록은 아래와 같다.

- ### Constructor Summary

  | Modifier    | Constructor                                                  | Description                                                  |
  | :---------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
  | ` `         | `Throwable()`                                                | Constructs a new throwable with `null` as its detail message. |
  | ` `         | `Throwable(String message)`                                  | Constructs a new throwable with the specified detail message. |
  | ` `         | `Throwable(String message, Throwable cause)`                 | Constructs a new throwable with the specified detail message and cause. |
  | `protected` | `Throwable(String message, Throwable cause, boolean enableSuppression, boolean writableStackTrace)` | Constructs a new throwable with the specified detail message, cause, [suppression](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Throwable.html#addSuppressed(java.lang.Throwable)) enabled or disabled, and writable stack trace enabled or disabled. |
  | ` `         | `Throwable(Throwable cause)`                                 | Constructs a new throwable with the specified cause and a detail message of `(cause==null ? null : cause.toString())` (which typically contains the class and detail message of `cause`). |



## CheckedExecption

`CheckedExecption`은 Java 컴파일러가 처리해야 하는 예외이다. `throw` 키워드를 사용해서 선언적으로 예외를 던지거나 `try-catch` 형태로 예외를 직접 처리해야 한다는 의미이다. Java에서 `CheckedExecption`은 대부분 `Exception` 클래스를 상속하는 클래스들이고 사용하는  대표적으로 `IOException`, `ServletException` 등이 있다. 

`Exception`클래스를 상속받는 다양한 `CheckedExecption`은 [https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Exception.html](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Exception.html)에서 직접 확인할 수 있다.

이 `CheckedExecption`은 클라이언트가 예외를 직접 처리하고 예외를 복구할 것으로 예측할수 있을때 사용하기 적합한 Exception 타입이다.

![20210303_153613](https://user-images.githubusercontent.com/30790184/109765198-dac01900-7c37-11eb-9000-1aef5cbe7501.png)

> 예외를 처리하지 않으면 컴파일러가 에러를 보여줌.



## UnCheckedException

`UnCheckedException`은 Java 컴파일러가 처리할 필요가 없는 예외이다. 간단하게 설명하면 컴파일러가 신경쓰지 않기 때문에 별도의 예외처리를 해주지 않아도 된다. Java에서 `UnCheckedException`은 `RuntimeException` 또는 `Error` 클래스를 상속하는 클래스들이고 대표적으로 `NullPointerException`, `IllegalArgumentException` 등이 있다. 

`RuntimeException`을 상속받는 클래스들은 [https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/RuntimeException.html](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/RuntimeException.html)에서, `Error` 클래스를 상속받는 클래스들은 [https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Error.html](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Error.html) 에서 확인할 수 있다.

이 `UnCheckedException`은 클라이언트가 예외를 만난다고 하더라도 이 예외를 복구하기위해 아무것도 할 수 없을때 사용하기 적합한 Exception 타입이다.



![20210303_153626](https://user-images.githubusercontent.com/30790184/109765195-d98eec00-7c37-11eb-91d6-681f272ade68.png)

> 별도의 예외 처리 없이 컴파일 할 수 있음



## Error

`Error`는 라이브러리 비 호환성, 무한 재귀 또는 메모리 누수와 같은 심각하고 일반적으로 복구 할 수없는 상태를 나타낸다. 위에서 설명했듯이 `Error`를 상속하는 모든 클래스들은 `UnCheckedException` 형태로 동작한다.

대표적인 `Error`클래스들로 `OutOfMemoryError`, `StackOverflowError`  등이 있다.





> Java에서는 `try-catch` 또는 `throws` 관련 예약어를 통해서 예외를 처리할 수 있다. 예외를 처리하는 자세한 정보는 [https://www.baeldung.com/java-exceptions#handling-exceptions](https://www.baeldung.com/java-exceptions#handling-exceptions) 에서 찾아볼 수 있고 그 외에도 안티패턴, `finally`에 대한 내용도 있으니 확인해보면 좋을 것 같다.



# Spring @Transactional과 Exception

이제 위에서 알아본 Java의 Exception 종류들 중 `UnCheckedException`과 `CheckedException`에 따라 다르게 동작하는 `@Transactional`애너테이션에 대해서 알아보도록 하자.



## @Transactional

Spring에서는 `@Transactional`이란 애너테이션을 사용해서 트랜잭션 처리를 한다. 내부적으로 프록시 객체를 생성해서 실행되는 메서드 전후에 트랜잭션과 관련된 로직을 삽입하여 Exception이 발생하면 rollback을 시키고 예상대로 잘 동작한 경우 commit을 한다.

하지만 여기서 정확하게 짚고 넘어가면 모든 Exception 타입이 아닌 `UnCheckedException`만 rollback을 시킨다.

> #### rollbackFor
>
> ```
> public abstract Class<? extends Throwable>[] rollbackFor
> ```
>
> Defines zero (0) or more exception [`classes`](https://docs.oracle.com/javase/8/docs/api/java/lang/Class.html?is-external=true), which must be subclasses of [`Throwable`](https://docs.oracle.com/javase/8/docs/api/java/lang/Throwable.html?is-external=true), indicating which exception types must cause a transaction rollback.
>
> By default, a transaction will be rolling back on [`RuntimeException`](https://docs.oracle.com/javase/8/docs/api/java/lang/RuntimeException.html?is-external=true) and [`Error`](https://docs.oracle.com/javase/8/docs/api/java/lang/Error.html?is-external=true) but not on checked exceptions (business exceptions). See [`DefaultTransactionAttribute.rollbackOn(Throwable)`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/transaction/interceptor/DefaultTransactionAttribute.html#rollbackOn-java.lang.Throwable-) for a detailed explanation.
>
> This is the preferred way to construct a rollback rule (in contrast to [`rollbackForClassName()`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/transaction/annotation/Transactional.html#rollbackForClassName--)), matching the exception class and its subclasses.
>
> Similar to [`RollbackRuleAttribute(Class clazz)`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/transaction/interceptor/RollbackRuleAttribute.html#RollbackRuleAttribute-java.lang.Class-).
>
> - **See Also:**
>
>   [`rollbackForClassName()`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/transaction/annotation/Transactional.html#rollbackForClassName--), [`DefaultTransactionAttribute.rollbackOn(Throwable)`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/transaction/interceptor/DefaultTransactionAttribute.html#rollbackOn-java.lang.Throwable-)
>
> - Default:
>
>   {}

위 문서에도 알 수 있듯이 `RuntimeException`과 `Error`만 롤백하는데 그이유는 EJB의 기본동작에 따라 결정된다.



`@Transactional`은 내부적으로 [cglib](https://github.com/cglib/cglib) Java 바이트코드를 생성하고 변환해서 트랜잭션 메서드를 포함하는 Proxy 객체를 생성한다. 실제로 코드를 확인하면서 어떤식으로 동작하는지 확인해보자.



## 예제

간단한 요구사항으로 `MemberService`에 저장시에 멤버의 이름앞에 `#`이라는 접두어가 붙어야한다 라는 가정을 하고 시작하겠다.

```java

    @Test
    public void save_all(){
        //given
        List<Member> members = Arrays.asList(Member.createMember("#choi", 28),
                Member.createMember("#woo", 30),
                Member.createMember("park", 35));

        //when
        try {
            memberService.saveAll(members);
        } catch (Exception e) {
            e.printStackTrace();
        }

        //then
        List<Member> all = memberRepository.findAll();
        assertEquals(0, all.size());
        assertTrue(all.isEmpty());
    }
```

유저의 이름앞에는 `#`이라는 접두어가 있어야 성공적으로 데이터베이스에 들어가고 아닌경우 롤백되어야 테스트가 성공한다. 



## 1.UnCheckedException을 throw로 던질때 

일반 `@Transactional`을 사용해서 `RuntimeException`을 `throw`시켰을때 어떤식으로 동작하는지 알아보자.

**MemberService.java**

```java
    @Transactional
    public void saveAll(List<Member> members) throws Exception {
        for (Member member : members) {
		   validUsernameThrowRuntimeException(member.getName());
            memberRepository.save(member);
        }
    }

    private void validUsernameThrowRuntimeException(String name) {
        if(!name.startsWith("#")) {
            throw new RuntimeException(); //UnChecked Exception
        }
    }
```

break 걸어 메서드 콜 스택을 확인해보면 결과적으로 `TransactionAspectSupport`클래스의 `invokeWithinTransaction()` 메서드내부 `try-catch` 부분에서 트랜잭션 롤백처리를 한다.

**TransactionAspectSupport.invokeWithinTransaction()**

```java
...
			try {
				// This is an around advice: Invoke the next interceptor in the chain.
				// This will normally result in a target object being invoked.
				retVal = invocation.proceedWithInvocation();
			}
			catch (Throwable ex) {
				// target invocation exception
				completeTransactionAfterThrowing(txInfo, ex); // <- 요기서 rollback 수행
				throw ex;
			}
...
```

 `MemberService`에서 `RumtimeException`을 발생시키면 위 `completeTransactionAfterThrowing()` 를 수행하는 구조이다.

여기서 `completeTransactionAfterThrowing()`은 throwable을 처리하고 트랜잭션을 완료하는 메서드인데 구성에 따라 커밋하거나 롤백할 수 있게 제공되는 메서드이다.

**TransactionAspectSupport.completeTransactionAfterThrowing()**

```java
	protected void completeTransactionAfterThrowing(@Nullable TransactionInfo txInfo, Throwable ex) {
		if (txInfo != null && txInfo.getTransactionStatus() != null) {
			if (logger.isTraceEnabled()) {
				logger.trace("Completing transaction for [" + txInfo.getJoinpointIdentification() +
						"] after exception: " + ex);
			}
			if (txInfo.transactionAttribute != null && txInfo.transactionAttribute.rollbackOn(ex)) { //<- 요기서 rollback 여부 판단
				try {
					txInfo.getTransactionManager().rollback(txInfo.getTransactionStatus());
				}
				catch (TransactionSystemException ex2) {
					logger.error("Application exception overridden by rollback exception", ex);
					ex2.initApplicationException(ex);
					throw ex2;
				}
				catch (RuntimeException | Error ex2) {
					logger.error("Application exception overridden by rollback exception", ex);
					throw ex2;
				}
			}
			else {
				// We don't roll back on this exception.
				// Will still roll back if TransactionStatus.isRollbackOnly() is true.
				try {
					txInfo.getTransactionManager().commit(txInfo.getTransactionStatus());
				}
				catch (TransactionSystemException ex2) {
					logger.error("Application exception overridden by commit exception", ex);
					ex2.initApplicationException(ex);
					throw ex2;
				}
				catch (RuntimeException | Error ex2) {
					logger.error("Application exception overridden by commit exception", ex);
					throw ex2;
				}
			}
		}
	}
```

`completeTransactionAfterThrowing()` 내부에서는 `txInfo.transactionAttribute.rollbackOn(ex))`이라는 메서드로 주어진 예외에서 롤백해야하는지 판단하는 처리를 한다. 여기서 `TransactionAttribute`의 구현체는 `RuleBasedTransactionAttribute`가 들어가게 된다.

**RuleBasedTransactionAttribute.rollbackOn()**

```java
	@Override
	public boolean rollbackOn(Throwable ex) {
		if (logger.isTraceEnabled()) {
			logger.trace("Applying rules to determine whether transaction should rollback on " + ex);
		}

		RollbackRuleAttribute winner = null;
		int deepest = Integer.MAX_VALUE;

		if (this.rollbackRules != null) {
			for (RollbackRuleAttribute rule : this.rollbackRules) {
				int depth = rule.getDepth(ex);
				if (depth >= 0 && depth < deepest) {
					deepest = depth;
					winner = rule;
				}
			}
		}

		if (logger.isTraceEnabled()) {
			logger.trace("Winning rollback rule is: " + winner);
		}

		// User superclass behavior (rollback on unchecked) if no rule matches.
		if (winner == null) { // <- winner 가 null일경우 DefaultTransactionAttribute.rollBackOn() 호출
			logger.trace("No relevant rollback rule found: applying default rules");
			return super.rollbackOn(ex);
		}

		return !(winner instanceof NoRollbackRuleAttribute);
	}
```

우리는 아무것도 처리하지 않은 일반 `@Transactional`을 사용했으므로 메서드 내부에서 사용하는 `winner == null` 조건에 의해 `DefaultTransactionAttribute.rollBackOn()`을 호출한다.

**DefaultTransactionAttribute.rollBackOn()**

```java

	@Override
	public boolean rollbackOn(Throwable ex) {
		return (ex instanceof RuntimeException || ex instanceof Error);
	}

```

 따라서 위와 같이 넘어온 `Exception` 타입이 `RuntimeException` 이거나 `Error`일 경우엔 롤백시키는 코드가 동작하게된다. 따라서 테스트는 통과한다.

**결론적으로 `@Transactional`을 사용하면 RuntimeException 을 throw 했을때 롤백시킨다.**



## 2.UnCheckedException을 try-catch로 잡을때

이번에는 일반 `@Transactional`을 사용해서 `RuntimeException`을 `try-catch`로 잡았을때 어떤식으로 동작하는지 알아보자.

```java
    
    @Transactional
    public void saveAll(List<Member> members) throws Exception {
        for (Member member : members) {
            validUsernameTryCatchRuntimeException(member.getName());
            memberRepository.save(member);
        }
    }
    
    private void validUsernameTryCatchRuntimeException(String name) {
        if(!name.startsWith("#")) {
            try {
                throw new RuntimeException(); //UnChecked Exception
            }catch (RuntimeException e) {
                e.printStackTrace(); //예외를 복구할것으로 예측할 수 있음
            }
        }
    }
```

1번과 동일하게 `@Transactional`이 생성한 bytecode가 실행되면서 `TransactionAspectSupport`클래스의 `invokeWithinTransaction()` 내부에서 메서드를 실행한다.

**TransactionAspectSupport.invokeWithinTransaction()**

```java
...
			try {
				// This is an around advice: Invoke the next interceptor in the chain.
				// This will normally result in a target object being invoked.
				retVal = invocation.proceedWithInvocation();
			}
			catch (Throwable ex) { //<- catch할 방법이 없음 로직 내부에서 이미 catch했기 때문
				// target invocation exception
				completeTransactionAfterThrowing(txInfo, ex);
				throw ex;
			}
...
```

하지만 위에서 확인했듯이 `RuntimeException`을 `try-catch`로 미리 처리했기 때문에 `TransactionAspectSupport.invokeWithinTransaction()` 내부 `try-catch` 에서는 `RumtimeException`을 처리할 방법이 없다.

따라서 정상적으로 `TransactionAspectSupport.commitTransactionAfterReturning()`이 동작하기 때문에 롤백이 되지 않고 커밋이 되는 상황이 발생한다. 따라서 테스트는 실패한다.

**결론적으로 `@Transactional`을 사용하면 RuntimeException 을 try-catch 했을때 롤백되지 않고 커밋시킨다.**



## 3.CheckedException을 throw로 던질때

이번에는 일반 `@Transactional`을 사용해서 `DataFormatException`을 `throw`시켰을때 어떤식으로 동작하는지 알아보자. `DataFormatException`은 `CheckedException`이다.

```java
    @Transactional
    public void saveAll(List<Member> members) throws Exception {
        for (Member member : members) {
            validUsernameThrowDataFormatException(member.getName());
            memberRepository.save(member);
        }
    }
    
    private void validUsernameThrowDataFormatException(String name) throws DataFormatException {
        if(!name.startsWith("#")) {
            throw new DataFormatException(); //Checked Exception
        }
    }
```

1, 2번과 동일하게 `@Transactional`이 생성한 bytecode가 실행되면서 `TransactionAspectSupport`클래스의 `invokeWithinTransaction()` 내부에서 메서드를 실행한다. 그리고 `DataFormatException`이 `invokeWithinTransaction()`메서드 내부 `try-catch`에 잡혔기때문에 `completeTransactionAfterThrowing()` 을 호출한다.

**TransactionAspectSupport.invokeWithinTransaction()**

```java
...
			try {
				// This is an around advice: Invoke the next interceptor in the chain.
				// This will normally result in a target object being invoked.
				retVal = invocation.proceedWithInvocation();
			}
			catch (Throwable ex) { 
				// target invocation exception
				completeTransactionAfterThrowing(txInfo, ex); //<- 넘어오는 ex는 DataFormatException
				throw ex;
			}
...
```

결론적으로 `@Transactional`에 `rollbackFor` 설정을 하지 않았기 때문에 `RuleBasedTransactionAttribute.rollbackRules` 도 아무것도 없어서 1번과 동일하게 `DefaultTransactionAttribute.rollBackOn()` 을 호출한다.

![20210304_121256](https://user-images.githubusercontent.com/30790184/109907307-62179600-7ce5-11eb-9b0c-0f30c04b1116.png)

**DefaultTransactionAttribute.rollBackOn()**

```java

	@Override
	public boolean rollbackOn(Throwable ex) { //<- 넘어오는 ex는 DataFormatException
		return (ex instanceof RuntimeException || ex instanceof Error); // false
	}

```

하지만 `DataFormatException`은 `RuntimeException`이나 `Error` 타입이 아니기 때문에 결국 롤백 로직을 수행하지 못하고 정상적으로 커밋된다.

**결론적으로 `@Transactional`을 사용하면 DataFormatException 을 throw 했을때 롤백이 되지 않고 커밋시킨다.**



## 4.CheckedException을 throw로 던질때 Feat.@Transactional.rollbackFor

이번에는 `@Transactional(rollbackFor = DataFormatException.class)`을 사용해서 `DataFormatException`을 `throw`시켰을때 어떤식으로 동작하는지 알아보자.

`@Transactional`에서는 `rollbackFor`라는 필드로 `Throwable`의 모든 하위 예외 클래스들의 롤백 유발을 정의할 수있도록 한다.

```java
    @Transactional(rollbackFor = DataFormatException.class)
    public void saveAll(List<Member> members) throws Exception {
        for (Member member : members) {
            validUsernameTryCatchDataFormatException(member.getName());
            memberRepository.save(member);
        }
    }

    private void validUsernameThrowDataFormatException(String name) throws DataFormatException {
        if(!name.startsWith("#")) {
            throw new DataFormatException(); //Checked Exception
        }
    }
```

1, 2, 3번과 동일하게 `@Transactional`이 생성한 bytecode가 실행되면서 `TransactionAspectSupport`클래스의 `invokeWithinTransaction()` 내부에서 메서드를 실행한다. 그리고 `DataFormatException`이 `invokeWithinTransaction()`메서드 내부 `try-catch`에 잡혔기때문에 `completeTransactionAfterThrowing()` 을 호출한다.

**TransactionAspectSupport.invokeWithinTransaction()**

```java
...
			try {
				// This is an around advice: Invoke the next interceptor in the chain.
				// This will normally result in a target object being invoked.
				retVal = invocation.proceedWithInvocation();
			}
			catch (Throwable ex) { 
				// target invocation exception
				completeTransactionAfterThrowing(txInfo, ex); //<- 넘어오는 ex는 DataFormatException
				throw ex;
			}
...
```

이번에는 `@Transactional`에 `rollbackFor` 설정했기 때문에 `RuleBasedTransactionAttribute.rollbackRules` 에는 `DataFormatException`이 들어가 있기 때문에 결론적으로 `TransactionAspectSupport.completeTransactionAfterThrowing()` 메서드 내부에서 롤백 로직을 수행하게된다.



**RuleBasedTransactionAttribute.rollbackOn()**

```java
	@Override
	public boolean rollbackOn(Throwable ex) {
		if (logger.isTraceEnabled()) {
			logger.trace("Applying rules to determine whether transaction should rollback on " + ex);
		}

		RollbackRuleAttribute winner = null;
		int deepest = Integer.MAX_VALUE;

		if (this.rollbackRules != null) { // <- 요기에 DataFormatException.class가 있음
			for (RollbackRuleAttribute rule : this.rollbackRules) {
				int depth = rule.getDepth(ex);
				if (depth >= 0 && depth < deepest) {
					deepest = depth;
					winner = rule;
				}
			}
		}

		if (logger.isTraceEnabled()) {
			logger.trace("Winning rollback rule is: " + winner);
		}

		// User superclass behavior (rollback on unchecked) if no rule matches.
		if (winner == null) { // <- winner 가 null일경우 DefaultTransactionAttribute.rollBackOn() 호출
			logger.trace("No relevant rollback rule found: applying default rules");
			return super.rollbackOn(ex);
		}

		return !(winner instanceof NoRollbackRuleAttribute); // true
	}
```

![20210304_122537](https://user-images.githubusercontent.com/30790184/109907310-6348c300-7ce5-11eb-8d97-0d6b831d8bb5.png)
![20210304_122518](https://user-images.githubusercontent.com/30790184/109907311-6348c300-7ce5-11eb-9bcc-445ef12a6c98.png)

**결론적으로 `@Transactional(rollbackFor = DataFormatException.class)`을 사용하면 DataFormatException 을 throw 했을때 롤백이 된다.**

# 마무리

이번 시간에는 Java의 예외 종류, 그리고 Spring `@Transactional`에서 어떻게 롤백하고 커밋하는지에 대해서 알아봤다.

트랜잭션 관련해서 [https://woowabros.github.io/experience/2019/01/29/exception-in-transaction.html](https://woowabros.github.io/experience/2019/01/29/exception-in-transaction.html) 이 글도 재미있게 볼 수 있기 때문에 추천한다.



<br>

***

포스팅은 여기까지 하겠습니다. 퍼가실때는 출처를 반드시 남겨주세요!

예제: [https://github.com/sup2is/study/tree/master/spring/spring-transaction-and-exception-example](https://github.com/sup2is/study/tree/master/spring/spring-transaction-and-exception-example)

<br>

**References**

- [https://www.baeldung.com/java-exceptions](https://www.baeldung.com/java-exceptions)
- [https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Throwable.html](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Throwable.html)
- [https://docs.oracle.com/javase/tutorial/essential/exceptions/runtime.html](https://docs.oracle.com/javase/tutorial/essential/exceptions/runtime.html)
- [https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/transaction/annotation/Transactional.html](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/transaction/annotation/Transactional.html)
- [https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/transaction/interceptor/TransactionAspectSupport.html](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/transaction/interceptor/TransactionAspectSupport.html)
