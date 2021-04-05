---
layout: post
title: "Spring Boot에서 Auto-Configuration이 동작하는 방법"
tags: [Spring Boot, Auto-Configuration]
date: 2020-11-16
comments: true
---


<br>

# OverView

Spring Boot와 Spring의 가장 큰 차이는 Auto-Configuration이다. `@SpringBootApplication`내부에  `@EnableAutoConfiguration`을 포함하는데 이 애너테이션이 어떻게 동작하는지 알아보는 시간을 가져보자.

# @EnableAutoConfiguration

`@EnableAutoConfiguration` 애너테이션은 다음과 같이 `@SpringBootApplication` 내부에 있다.

```java
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = { @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
		@Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication { ... }
```

`@EnableAutoConfiguration` 애너테이션을 사용하면 classpath를 스캔해서 Spring `ApplicationContext` 의 auto-configuration을 활성화하고 Conditions에 해당하는 bean들을 등록시켜준다.

그럼 Spring `ApplicationContext` 의 auto-configuration은 어디에 있을지 생각해볼만한데 `org.springframework.boot.autoconfigure` 패키지 내부를 확인해보면 100개 이상의 익숙한 AutoConfiguration 클래스들을 확인할 수 있다. 실제 auto-configuration 목록은 [spring.factories](https://github.com/spring-projects/spring-boot/blob/v2.0.0.M3/spring-boot-autoconfigure/src/main/resources/META-INF/spring.factories)파일에서 확인할 수 있다.

![20201113_142151](https://user-images.githubusercontent.com/30790184/99200409-78f2b500-27e8-11eb-9f97-db8a52b396cd.png)


이 AutoConfiguration 클래스들은 전부 `@Configuration` 애너테이션을 갖고 있기 때문에 기본적으로 Spring bean의 대상이된다.

```java
package org.springframework.boot.autoconfigure.jdbc;

...

@Configuration(proxyBeanMethods = false) // <- 요기
@ConditionalOnClass({ DataSource.class, EmbeddedDatabaseType.class })
@ConditionalOnMissingBean(type = "io.r2dbc.spi.ConnectionFactory")
@EnableConfigurationProperties(DataSourceProperties.class)
@Import({ DataSourcePoolMetadataProvidersConfiguration.class, DataSourceInitializationConfiguration.class })
public class DataSourceAutoConfiguration { ... }
```

```java
package org.springframework.boot.autoconfigure.kafka;

...

@Configuration(proxyBeanMethods = false) // <- 요기
@ConditionalOnClass(KafkaTemplate.class)
@EnableConfigurationProperties(KafkaProperties.class)
@Import({ KafkaAnnotationDrivenConfiguration.class, KafkaStreamsAnnotationDrivenConfiguration.class })
public class KafkaAutoConfiguration { ... }
```

계속해서 `@EnableAutoConfiguration` 의 내부를 살펴보면 `@Import`를 사용해서 `AutoConfigurationImportSelector` 클래스를 import 하고있다.

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
@Import(AutoConfigurationImportSelector.class)
public @interface EnableAutoConfiguration {...}

```



## AutoConfigurationImportSelector.class

`AutoConfigurationImportSelector` 클래스 내부에는 `selectImports()`와  `getAutoConfigurationEntry()` 이라는 메서드가 있는데 이 메서드에서 실제로 AutoConfiguration의 대상이 되는 클래스들을 선별하는 작업을 한다.

**AutoConfigurationImportSelector.java**

```java
	@Override
	public String[] selectImports(AnnotationMetadata annotationMetadata) {
		if (!isEnabled(annotationMetadata)) {
			return NO_IMPORTS;
		}
		AutoConfigurationEntry autoConfigurationEntry = getAutoConfigurationEntry(annotationMetadata);
		return StringUtils.toStringArray(autoConfigurationEntry.getConfigurations());
	}

...
    protected AutoConfigurationEntry getAutoConfigurationEntry(AnnotationMetadata annotationMetadata) {
        if (!isEnabled(annotationMetadata)) {
            return EMPTY_ENTRY;
        }
        AnnotationAttributes attributes = getAttributes(annotationMetadata);
        List<String> configurations = getCandidateConfigurations(annotationMetadata, attributes);
        configurations = removeDuplicates(configurations);
        Set<String> exclusions = getExclusions(annotationMetadata, attributes);
        checkExcludedClasses(configurations, exclusions);
        configurations.removeAll(exclusions);
        configurations = getConfigurationClassFilter().filter(configurations);
        fireAutoConfigurationImportEvents(configurations, exclusions);
        return new AutoConfigurationEntry(configurations, exclusions);
    }
...
```

이 메서드를 통해 `configurations` 에 총 130개의 AutoConfiguration 클래스들이 대상이되지만 filter에 의해 실제 연관된 AutoConfiguration 클래스들을 총 38개로 선별하는 것을 확인할 수 있다.

**선별되기 전**

![20201116_102044](https://user-images.githubusercontent.com/30790184/99203351-a09c4a00-27f5-11eb-9157-707e4e2998f7.png)

**선별된 후**

![20201116_102203](https://user-images.githubusercontent.com/30790184/99203352-a1cd7700-27f5-11eb-8249-dd0a09d289c4.png)



이렇게 선별된 AutoConfiguration을 어떻게 등록시키는지 `DataSourceAutoConfiguration` 내부를 확인해보자.

```java
package org.springframework.boot.autoconfigure.jdbc;

@Configuration(proxyBeanMethods = false)
@ConditionalOnClass({ DataSource.class, EmbeddedDatabaseType.class })
@ConditionalOnMissingBean(type = "io.r2dbc.spi.ConnectionFactory")
@EnableConfigurationProperties(DataSourceProperties.class)
@Import({ DataSourcePoolMetadataProvidersConfiguration.class, DataSourceInitializationConfiguration.class })
public class DataSourceAutoConfiguration {

	@Configuration(proxyBeanMethods = false)
	@Conditional(EmbeddedDatabaseCondition.class)
	@ConditionalOnMissingBean({ DataSource.class, XADataSource.class })
	@Import(EmbeddedDataSourceConfiguration.class)
	protected static class EmbeddedDatabaseConfiguration {

	}

	@Configuration(proxyBeanMethods = false)
	@Conditional(PooledDataSourceCondition.class)
	@ConditionalOnMissingBean({ DataSource.class, XADataSource.class })
	@Import({ DataSourceConfiguration.Hikari.class, DataSourceConfiguration.Tomcat.class,
			DataSourceConfiguration.Dbcp2.class, DataSourceConfiguration.Generic.class,
			DataSourceJmxConfiguration.class })
	protected static class PooledDataSourceConfiguration {

	}

...
}
```

`DataSourceAutoConfiguration` 내부를 확인해보면 다음과 같이 `@Conditional`, `@ConditionalOnClass`, `@EnableConfigurationProperties` 애너테이션을 확인할 수 있다.

# Condition 인터페이스

다음은 `Condition` 인터페이스의 내부다

```java
@FunctionalInterface
public interface Condition {
	boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata);

}

```

**Condition 인터페이스는 bean이 등록되기 직전에 조건을 확인하고 그시점에서 결정할 수 있는 기준에 따라 등록을 할수도있고 안할수도 있게 해준다.** 위에서 살펴본 `EmbeddedDatabaseConfiguration` 은`@Conditional(EmbeddedDatabaseCondition.class)` 을 사용하는데 `EmbeddedDatabaseCondition` 클래스는 `SpringBootCondition` 클래스를 상속받았다. 다음은 `SpringBootCondition` 의 내부 구현이다.

```java
package org.springframework.boot.autoconfigure.condition;

public abstract class SpringBootCondition implements Condition {

	private final Log logger = LogFactory.getLog(getClass());

	@Override
	public final boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
		String classOrMethodName = getClassOrMethodName(metadata);
		try {
			ConditionOutcome outcome = getMatchOutcome(context, metadata);
			logOutcome(classOrMethodName, outcome);
			recordEvaluation(context, classOrMethodName, outcome);
			return outcome.isMatch();
		}
		catch (NoClassDefFoundError ex) {
			throw new IllegalStateException("Could not evaluate condition on " + classOrMethodName + " due to "
					+ ex.getMessage() + " not found. Make sure your own configuration does not rely on "
					+ "that class. This can also happen if you are "
					+ "@ComponentScanning a springframework package (e.g. if you "
					+ "put a @ComponentScan in the default package by mistake)", ex);
		}
		catch (RuntimeException ex) {
			throw new IllegalStateException("Error processing condition on " + getName(metadata), ex);
		}
	}
}
```

`SpringBootCondition` 클래스에서 `matches()` 메서드를 구현하는 모습을 확인할 수 있다. 

다음 Condition관련 애너테이션들에 대해서 조금 더 자세하게 알아보자.

## @Conditional 

- **지정된 모든 Condition 인터페이스들을 만족해야 구성 요소로 등록됨**

## @ConditionalOnClass

- **지정된 클래스가 classpath에 존재해야 구성 요소로 등록됨**

## @ConditionalOnBean

- **지정된 Bean이 BeanFactory에 포함되어야 구성 요소로 등록됨**

## @ConditionalOnMissingBean

- **지정된 Bean이 BeanFactory에 포함되어있지 않아야 구성 요소로 등록됨**

이외에도 `org.springframework.boot.autoconfigure.condition` 내부에 Condition 관련된 애너테이션들을 확인할 수 있다.

# @EnableConfigurationProperties

Spring Boot에서의 환경설정은 주로 .yml, .properties를 통해 이루어진다. 이렇게 설정할 수 있는 방법은 `@ConfigurationProperties` 애너테이션을 통해서 prefix가 `spring.datasource`인 값을 바인딩할 수 있기 때문이다.

```java
@ConfigurationProperties(prefix = "spring.datasource")
public class DataSourceProperties implements BeanClassLoaderAware, InitializingBean { 
    
    ...
        
	/**
	 * Fully qualified name of the JDBC driver. Auto-detected based on the URL by default.
	 */
	private String driverClassName;

	/**
	 * JDBC URL of the database.
	 */
	private String url;

	/**
	 * Login username of the database.
	 */
	private String username;

	/**
	 * Login password of the database.
	 */
	private String password;


    ...

}
```

**application.yml**

```yml
spring:
  datasource:
    url: jdbc:h2:tcp://localhost/~/query-dsl
    driver-class-name: org.h2.Driver
    username: choi
    password: qwer!23
```





<br>

***

포스팅은 여기까지 하겠습니다. 퍼가실때는 출처를 반드시 남겨주세요!

<br>

**References**

- [https://dzone.com/articles/how-springboot-autoconfiguration-magic-works](https://dzone.com/articles/how-springboot-autoconfiguration-magic-works)
- [https://docs.spring.io/spring-framework/docs/current/javadoc-api/](https://docs.spring.io/spring-framework/docs/current/javadoc-api/)
