---
layout: post
title: "Spring Test Context Caching과 JUnit5의 Class Order 기능"
tags: [Spring Boot, JUnit5, Spring Test Context Caching]
date: 2022-03-06
comments: true
---

<br>

# Overview

오랜만에 쓰는 글이다. 이번 시간에는 JUnit5의 Class Order기능에 대해 알아보고 Spring Test Context Caching에서 어떻게 활용하면 좋을지에 대해 알아보도록 하자.

# JUnit5의 Method Order

JUnit5의 Class Order를 살펴보기 전에 Method Order도 확인해보자.

테스트들은 기본적으로 의존관계가 없어야하고 한 메서드는 하나의 기능만 테스트해야한다. 하지만 실전에서는 통합테스트나 순서가 필요한 기능 테스트들이 의존관계로 엮여 있는 경우가 많다.

기본적으로 JUnit5는 Method Order만 제공했고 아래와 같은 형태로 많이 사용했다.

```java
@TestMethodOrder(MethodOrderer.OrderAnnotation.class)
class OrderedTests {

  @Test
  @Order(1)
  void nullValues() {}

  @Test
  @Order(2)
  void emptyValues() {}

  @Test
  @Order(3)
  void validValues() {}
}
```

위에서 사용한 `@TestMethodOrder` 애너테이션은 아래와 같은 모습을 갖고 있다.

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@API(status = STABLE, since = "5.7")
public @interface TestMethodOrder {

	/**
	 * The {@link MethodOrderer} to use.
	 *
	 * @see MethodOrderer
	 * @see MethodOrderer.MethodName
	 * @see MethodOrderer.DisplayName
	 * @see MethodOrderer.OrderAnnotation
	 * @see MethodOrderer.Random
	 */
	Class<? extends MethodOrderer> value();

}

```

`MethodOrderer` 타입은 위에서 설명된것과 같이 `MethodOrderer.MethodName`, `MethodOrderer.DisplayName`, `MethodOrderer.OrderAnnotation`, `MethodOrderer.Random` 이고 클래스 이름이 직관적이기 때문에 따로 설명은 필요 없을 것 같다. 

실제로 `MethodOrderer` 인터페이스를 살펴보면 구현체들을 확인할 수 있다. 

```java
@API(status = STABLE, since = "5.7")
public interface MethodOrderer {

	void orderMethods(MethodOrdererContext context);

	default Optional<ExecutionMode> getDefaultExecutionMode() {
		return Optional.of(ExecutionMode.SAME_THREAD);
	}

	@API(status = DEPRECATED, since = "5.7")
	@Deprecated
	class Alphanumeric extends MethodName {

		public Alphanumeric() {
		}
	}

	@API(status = EXPERIMENTAL, since = "5.7")
	class MethodName implements MethodOrderer {

		public MethodName() {
		}

		@Override
		public void orderMethods(MethodOrdererContext context) {
			context.getMethodDescriptors().sort(comparator);
		}

		private static final Comparator<MethodDescriptor> comparator = Comparator.<MethodDescriptor, String> //
				comparing(descriptor -> descriptor.getMethod().getName())//
				.thenComparing(descriptor -> parameterList(descriptor.getMethod()));

		private static String parameterList(Method method) {
			return ClassUtils.nullSafeToString(method.getParameterTypes());
		}
	}

	@API(status = EXPERIMENTAL, since = "5.7")
	class DisplayName implements MethodOrderer {

		public DisplayName() {
		}

		@Override
		public void orderMethods(MethodOrdererContext context) {
			context.getMethodDescriptors().sort(comparator);
		}

		private static final Comparator<MethodDescriptor> comparator = Comparator.comparing(
			MethodDescriptor::getDisplayName);
	}

	class OrderAnnotation implements MethodOrderer {

		public OrderAnnotation() {
		}

		@Override
		public void orderMethods(MethodOrdererContext context) {
			context.getMethodDescriptors().sort(comparingInt(OrderAnnotation::getOrder));
		}

		private static int getOrder(MethodDescriptor descriptor) {
			return descriptor.findAnnotation(Order.class).map(Order::value).orElse(Order.DEFAULT);
		}
	}

	class Random implements MethodOrderer {

		private static final Logger logger = LoggerFactory.getLogger(Random.class);

		private static final long DEFAULT_SEED;

		static {
			DEFAULT_SEED = System.nanoTime();
			logger.config(() -> "MethodOrderer.Random default seed: " + DEFAULT_SEED);
		}

		public static final String RANDOM_SEED_PROPERTY_NAME = "junit.jupiter.execution.order.random.seed";

		public Random() {
		}

		@Override
		public void orderMethods(MethodOrdererContext context) {
			Collections.shuffle(context.getMethodDescriptors(),
				new java.util.Random(getCustomSeed(context).orElse(DEFAULT_SEED)));
		}

		private Optional<Long> getCustomSeed(MethodOrdererContext context) {
			return context.getConfigurationParameter(RANDOM_SEED_PROPERTY_NAME).map(configurationParameter -> {
				Long seed = null;
				try {
					seed = Long.valueOf(configurationParameter);
					logger.config(
						() -> String.format("Using custom seed for configuration parameter [%s] with value [%s].",
							RANDOM_SEED_PROPERTY_NAME, configurationParameter));
				}
				catch (NumberFormatException ex) {
					logger.warn(ex,
						() -> String.format(
							"Failed to convert configuration parameter [%s] with value [%s] to a long. "
									+ "Using default seed [%s] as fallback.",
							RANDOM_SEED_PROPERTY_NAME, configurationParameter, DEFAULT_SEED));
				}
				return seed;
			});
		}
	}

}

```

필요하다면 커스텀형태로 사용해도 되는데 그럴만한 경우는 따로 없을 것 같다.

클래스마다 지정하지 않고 기본값을 지정하고싶다면 `src/test/resources/junit-platform.properties`에 아래와 같은 형태로 적어주면 기본값을 설정할 수 있다.

```properties
junit.jupiter.testmethod.order.default = \
    org.junit.jupiter.api.MethodOrderer$OrderAnnotation
```

# JUnit5의 Class Order

위에서 설명한 JUnit5의 Method Order 기능은 2020년 9월 5.7버전 release note에 포함되어 있다. 

이번에 설명할 JUnit5의 Class Order 기능은 5.8 버전에 포함되었고 5.8 버전은 2021년 9월에 release되었다. (글을 쓰는 시점이 2022년 3월이므로 꽤 최근 기능이다.)

- 5.7 release note : [https://junit.org/junit5/docs/5.7.0/release-notes/index.html#release-notes-5.7.0](https://junit.org/junit5/docs/5.7.0/release-notes/index.html#release-notes-5.7.0)
- 5.8 release note : [https://junit.org/junit5/docs/current/release-notes/index.html#release-notes-5.8.0](https://junit.org/junit5/docs/current/release-notes/index.html#release-notes-5.8.0)

위에서 설명한것처럼 테스트는 기본적으로 의존 관계가 없어야하고 메서드를 포함한 클래스의 순서도 마찬가지다. 하지만 아래와 같은 경우 테스트의 순서를 의미있게 지정해서 이득을 볼 수 있다.

- **빠른 실패**
- **병렬 환경에서 수행시간이 긴 테스트를 먼저 실행**
- **Spring 개발 환경에서 Integration 테스트와 Unit 테스트(Mocking을 활용한)가 뒤죽박죽 섞인 경우 Test Context의 장점을 제대로 활용하지 못할 때**

첫번째 빠른 실패는 말 그대로 불안정한 테스트를 빠르게 실패해서 사용자가 빠른 피드백을 얻게하거나 기타 등등의 효과를 얻을 수 있을 것 같고 두번째 병렬 환경에서 수행시간이 긴 테스트를 먼저 실행하는 건 빠른 테스트를 위한 필수 요건이다.

세번째의 경우 아래에서 코드와 사례로 자세하게 설명하려고 한다.

<br>

Class Order 기능도 `ClassOrderer` 인터페이스를 구현해서 사용하면 된다. 기본 구현체는 아래와 같다. 이 또한 클래스 이름이 매우 직관적이기 때문에 자세한 설명은 생략한다.

- [ClassOrderer.ClassName](https://junit.org/junit5/docs/current/api/org.junit.jupiter.api/org/junit/jupiter/api/ClassOrderer.ClassName.html)
- [ClassOrderer.DisplayName](https://junit.org/junit5/docs/current/api/org.junit.jupiter.api/org/junit/jupiter/api/ClassOrderer.DisplayName.html)
- [ClassOrderer.OrderAnnotation](https://junit.org/junit5/docs/current/api/org.junit.jupiter.api/org/junit/jupiter/api/ClassOrderer.OrderAnnotation.html)
- [ClassOrderer.Random](https://junit.org/junit5/docs/current/api/org.junit.jupiter.api/org/junit/jupiter/api/ClassOrderer.Random.html)

```java
@API(status = EXPERIMENTAL, since = "5.8")
public interface ClassOrderer {

	void orderClasses(ClassOrdererContext context);

	class ClassName implements ClassOrderer {

		public ClassName() {
		}

		@Override
		public void orderClasses(ClassOrdererContext context) {
			context.getClassDescriptors().sort(comparator);
		}

		private static final Comparator<ClassDescriptor> comparator = Comparator.comparing(
			descriptor -> descriptor.getTestClass().getName());
	}

	class DisplayName implements ClassOrderer {

		public DisplayName() {
		}

		@Override
		public void orderClasses(ClassOrdererContext context) {
			context.getClassDescriptors().sort(comparator);
		}

		private static final Comparator<ClassDescriptor> comparator = Comparator.comparing(
			ClassDescriptor::getDisplayName);
	}

	class OrderAnnotation implements ClassOrderer {

		public OrderAnnotation() {
		}

		@Override
		public void orderClasses(ClassOrdererContext context) {
			context.getClassDescriptors().sort(comparingInt(OrderAnnotation::getOrder));
		}

		private static int getOrder(ClassDescriptor descriptor) {
			return descriptor.findAnnotation(Order.class).map(Order::value).orElse(Order.DEFAULT);
		}
	}

	class Random implements ClassOrderer {

		private static final Logger logger = LoggerFactory.getLogger(Random.class);

		private static final long DEFAULT_SEED;

		static {
			DEFAULT_SEED = System.nanoTime();
			logger.config(() -> "ClassOrderer.Random default seed: " + DEFAULT_SEED);
		}

		public static final String RANDOM_SEED_PROPERTY_NAME = MethodOrderer.Random.RANDOM_SEED_PROPERTY_NAME;

		public Random() {
		}

		@Override
		public void orderClasses(ClassOrdererContext context) {
			Collections.shuffle(context.getClassDescriptors(),
				new java.util.Random(getCustomSeed(context).orElse(DEFAULT_SEED)));
		}

		private Optional<Long> getCustomSeed(ClassOrdererContext context) {
			return context.getConfigurationParameter(RANDOM_SEED_PROPERTY_NAME).map(configurationParameter -> {
				Long seed = null;
				try {
					seed = Long.valueOf(configurationParameter);
					logger.config(
						() -> String.format("Using custom seed for configuration parameter [%s] with value [%s].",
							RANDOM_SEED_PROPERTY_NAME, configurationParameter));
				}
				catch (NumberFormatException ex) {
					logger.warn(ex,
						() -> String.format(
							"Failed to convert configuration parameter [%s] with value [%s] to a long. "
									+ "Using default seed [%s] as fallback.",
							RANDOM_SEED_PROPERTY_NAME, configurationParameter, DEFAULT_SEED));
				}
				return seed;
			});
		}
	}

}

```



적용하는 방법 역시 간단하다. `src/test/resources/junit-platform.properties` 안에 아래와 같은 형태로 설정할 수 있다.

```properties
junit.jupiter.testclass.order.default = \
    org.junit.jupiter.api.ClassOrderer$OrderAnnotation
```



# Spring Test Context Caching & JUnit 5 Class Order

사실 이게 본론이다. 현재 회사에서 테스트의 종류는 아래와 같다.

1. **Spring을 사용하지 않는 테스트 (Unit Test)**
2. **Spring + `@MockBean`을 사용하는 테스트 (Unit Test)**
3. **only Spring만 사용하는 테스트 (Integration Test)**

나는 여러 책들을 읽으면서 얻은 **"테스트는 외부 컴포넌트에 의존하면 안된다"** 라는 원칙을 갖고 있기 때문에 Database, 외부 API 서버, 프레임워크 등등 테스트와 최대한 분리시키기 위해 **Spring을 사용하지 않는 테스트**를 기본으로 작성하려고 노력한다.

하지만 Spring restdocs를 사용하면 어쩔 수 없이 Spring 위에서 테스트를 올려야하는데 이 경우엔 **Spring + `@MockBean`을 사용하는 테스트**를 작성했었다.

추가로 통합테스트가 필요한 경우에는 Spring Context에 등록된 Bean들로만 구성해야하기 때문에 **only Spring만 사용하는 테스트** 역시 필요해 졌다.

<br>

아직 경험이 많지 않아서 그런지 모르겠지만 개인적으로 생각했을때 위에서 설명한 세가지 테스트는 모두 각자의 의미가 있고 애플리케이션에서 반드시 필요한 테스트라고 생각한다.

하지만 1번의 경우는 문제가 되지 않지만 2번과 3번을 섞어 쓰는 경우 문제가 생길 수 있다. 어떤 문제가 생길지 보기 전에 Spring의 Test Context Caching 에 대해서 간단하게 알아보자.

## Spring의 Test Context Caching

Spring에서 Test Context가 ApplicationContext를 로드하면 해당 컨텍스트는 캐시된 상태로 동일한 테스트내에서 계속해서 재사용된다.  이런 특성을 잘 활용하면 무거운 Spring Test들을 효과적으로 수행해서 빠른 빌드&테스트를 수행할 수 있다.

하지만 `@MockBean` 을 사용하면 앞에서 설명한 **동일한 테스트** 라는 전제가 깨지게되는데 이는 테스트 성능에 악영향을 준다. 만약 100개의 테스트 클래스에서 100개의 `@MockBean` 을 개별적으로 사용하면 Spring Context는 100개의 Context를 로드하는 과정을 거친다. 마치 `@DirtiesContext` 를 사용한것과 같다.

하지만 `@MockBean` 의 사용이 불가피한 경우에 Spring의 Test Context Caching을 사용할 수 있는 방법이 있는데 간단하게 아래와 같이 `@MockBean` 을 나열한 추상클래스를 확장하는 테스트를 만들면 된다.

```java
abstract class AbstractMockTestConfig {

    @MockBean
    protected AService aService;

    @MockBean
    protected BService bService;

    @MockBean
    protected CService cService;

}
```

위와 같이 사용할 경우 `AbstractMockTestConfig` 은 시간이 지날수록 거대해진다는 단점이 있지만 Test Context Caching의 이점을 최대한 이용하면서 `@MockBean`을 효과적으로 사용할 수 있다.



## JUnit5 Class Order

다시 본론으로 넘어가서 Test Context Caching을 최대한 활용하려고 하더라도 여러 테스트클래스들이 있는 실무 환경에서의 테스트 순서는 아래와 같을 수 있다.



```
# 테스트 순서

1. Spring + @MockBean을 사용하는 테스트 (Unit Test)
2. only Spring만 사용하는 테스트 (Integration Test)
3. Spring + @MockBean을 사용하는 테스트 (Unit Test)
4. only Spring만 사용하는 테스트 (Integration Test)
5. Spring + @MockBean을 사용하는 테스트 (Unit Test)
6. only Spring만 사용하는 테스트 (Integration Test)
7. Spring + @MockBean을 사용하는 테스트 (Unit Test)
8. only Spring만 사용하는 테스트 (Integration Test)
```



<br>

이런 경우에 적절하게 사용할 수 있는 방법이 JUnit5의 Class Order 기능이다. 

```java
public class CustomTestClassOrder implements ClassOrderer {

    @Override
    public void orderClasses(ClassOrdererContext context) {
        context.getClassDescriptors().sort(Comparator.comparingInt(this::getOrder));
    }

    private int getOrder(ClassDescriptor classDescriptor) {
        if (classDescriptor.getTestClass().getSuperclass().equals(AbstractMockTestConfig.class)) {
            return 1;
        } else {
            return 2;
        }
    }
}
```

위와 같이 `ClassOrderer` 를 구현한 `CustomTestClassOrder` 를 만들어서 상위타입을 체크한 뒤 정렬하면 실행되는 테스트의 순서를 아래와 같이 제어할 수 있다.

```
# 테스트 순서

1. Spring + @MockBean을 사용하는 테스트 (Unit Test)
2. Spring + @MockBean을 사용하는 테스트 (Unit Test)
3. Spring + @MockBean을 사용하는 테스트 (Unit Test)
4. Spring + @MockBean을 사용하는 테스트 (Unit Test)
5. only Spring만 사용하는 테스트 (Integration Test)
6. only Spring만 사용하는 테스트 (Integration Test)
7. only Spring만 사용하는 테스트 (Integration Test)
8. only Spring만 사용하는 테스트 (Integration Test)
```

이렇게 실행되는 테스트들은 Spring Context를 단 두번만 로드하기 때문에 테스트 성능이 월등히 좋아질 수 있다.

<br>

또는 아래와 같이  `@WebMvcTest`, `@DataJpaTest` 등등 식별가능한 애너테이션이 있다면 `ClassDescriptor`의 `findAnnotation()` 메서드를 통해 특정 애너테이션 타입으로 Class Order를 지정할 수도 있다.

```java

public class SpringBootTestClassOrderer implements ClassOrderer {
    @Override
    public void orderClasses(ClassOrdererContext classOrdererContext) {
        classOrdererContext.getClassDescriptors().sort(Comparator.comparingInt(SpringBootTestClassOrderer::getOrder));
    }

    private static int getOrder(ClassDescriptor classDescriptor) {
        if (classDescriptor.findAnnotation(SpringBootTest.class).isPresent()) {
            return 4;
        } else if (classDescriptor.findAnnotation(WebMvcTest.class).isPresent()) {
            return 3;
        } else if (classDescriptor.findAnnotation(DataJpaTest.class).isPresent()) {
            return 2;
        } else {
            return 1;
        }
    }
}
```

> https://www.wimdeblauwe.com/blog/2021/02/12/junit-5-test-class-orderer-for-spring-boot/

아래는 `ClassDescriptor` 에 정의된 메서드 목록이다.

```java
@API(status = EXPERIMENTAL, since = "5.8")
public interface ClassDescriptor {

	/**
	 * Get the class for this descriptor.
	 *
	 * @return the class; never {@code null}
	 */
	Class<?> getTestClass();

	/**
	 * Get the display name for this descriptor's {@link #getTestClass() class}.
	 *
	 * @return the display name for this descriptor's class; never {@code null}
	 * or blank
	 */
	String getDisplayName();

	/**
	 * Determine if an annotation of {@code annotationType} is either
	 * <em>present</em> or <em>meta-present</em> on the {@link Class} for
	 * this descriptor.
	 *
	 * @param annotationType the annotation type to search for; never {@code null}
	 * @return {@code true} if the annotation is present or meta-present
	 * @see #findAnnotation(Class)
	 * @see #findRepeatableAnnotations(Class)
	 */
	boolean isAnnotated(Class<? extends Annotation> annotationType);

	/**
	 * Find the first annotation of {@code annotationType} that is either
	 * <em>present</em> or <em>meta-present</em> on the {@link Class} for
	 * this descriptor.
	 *
	 * @param <A> the annotation type
	 * @param annotationType the annotation type to search for; never {@code null}
	 * @return an {@code Optional} containing the annotation; never {@code null} but
	 * potentially empty
	 * @see #isAnnotated(Class)
	 * @see #findRepeatableAnnotations(Class)
	 */
	<A extends Annotation> Optional<A> findAnnotation(Class<A> annotationType);

	/**
	 * Find all <em>repeatable</em> {@linkplain Annotation annotations} of
	 * {@code annotationType} that are either <em>present</em> or
	 * <em>meta-present</em> on the {@link Class} for this descriptor.
	 *
	 * @param <A> the annotation type
	 * @param annotationType the repeatable annotation type to search for; never
	 * {@code null}
	 * @return the list of all such annotations found; neither {@code null} nor
	 * mutable, but potentially empty
	 * @see #isAnnotated(Class)
	 * @see #findAnnotation(Class)
	 * @see java.lang.annotation.Repeatable
	 */
	<A extends Annotation> List<A> findRepeatableAnnotations(Class<A> annotationType);

}
```



# 정리

Method Order 기능뿐만 아니라 Class Order 기능도 잘 활용하면 강력한 도구가 될 것 같다. 특히 Spring 환경에서 `@MockBean` + Test Context 활용에 대한 고민이 있다면 적용해 볼 것을 적극 추천한다!





<br>

***

포스팅은 여기까지 하겠습니다. 감사합니다!



<br>

**References**

-  [https://junit.org/JUnit5/docs/current/user-guide/#writing-tests-test-execution-order-classes](https://junit.org/JUnit5/docs/current/user-guide/#writing-tests-test-execution-order-classes)
-  [https://docs.spring.io/spring-framework/docs/current/reference/html/testing.html#testcontext-ctx-management-caching](https://docs.spring.io/spring-framework/docs/current/reference/html/testing.html#testcontext-ctx-management-caching)
-  https://www.wimdeblauwe.com/blog/2021/02/12/junit-5-test-class-orderer-for-spring-boot/
