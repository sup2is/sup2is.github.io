---
layout: post
title: "Spring의 선언적 트랜잭션 Feat.Spring Boot"
tags: [Spring Boot, Transaction]
date: 2021-11-11
comments: true
---

<br>

# Overview

이번시간에는 Spring 트랜잭션과 관련된 주요 인터페이스들을 살펴보고 Spring Boot에서 어떤식으로 선언적 트랜잭션을 적용하고 있는지 확인해보는 시간을 갖도록 해보자

# Programmatic 트랜잭션

Programmatic 트랜잭션, 즉 프로그래밍에 의한 트랜잭션은 소스코드 내부에 직접 트랜잭션 관련 로직을 넣어두는 방식이다.

이 방식을 사용하면 모든 메서드마다 새롭게 트랜잭션 관련 코드를 넣어주기 때문에 유연하게 코드를 관리할 수 있다. 하지만 트랜잭션 코드 중복 문제와 소스코드 유지보수가 매우 힘들어지는 단점이 있기 때문에 권장하는 방식은 아니다. 이 방식은 많은 양의 비지니스 코드를 포함하는 프로젝트에서는 비효율적이고 상대적으로 적은 트랜잭션 로직이 도입될 때 적용해볼 만 하다.

Spring을 사용하고 있고 프로그래밍에 의한 트랜잭션을 적용하고싶다면 Spring에서 제공하는 `TransactionTemplate`을 사용하면 된다. 이 글에서는 `TransactionTemplate` 에 대한 자세한 내용은 다루지 않는다.



# Declarative 트랜잭션

Declarative 트랜잭션, 즉 선언적 트랜잭션은 소스코드에 직접 트랜잭션 관련 로직을 넣어두지 않고 비지니스 로직에서 완전히 분리하는 방식이다.

이 방식을 사용하면 프로그래밍에 의한 트랜잭션에서 나온 단점인 트랜잭션 코드 중복 문제, 소스코드 유지보수의 문제를 모두 해결할 수 있다. 이 트랜잭션이라는 횡단 관심사를 비지니스 로직에서 완전히 분리하기 때문에 SRP 관점에서 봤을때도 적합하고 많은 양의 트랜잭션 로직을 적용하기에도 합리적이다.

Spring에서는 AOP를 사용해서 선언적 트랜잭션을 구현하고 있다.

Spring에서 트랜잭션과 관련된 주요 인터페이스들을 살펴보자.

# PlatformTransactionManager

`PlatformTransactionManager`는 Spring 트랜잭션의 근간이되는 인터페이스이다. 실제로 인터페이스에 선언된 추상메서드들을 확인해보면 아래와 같다

```java
public interface PlatformTransactionManager extends TransactionManager {

   TransactionStatus getTransaction(@Nullable TransactionDefinition definition)
         throws TransactionException;

   void commit(TransactionStatus status) throws TransactionException;

   void rollback(TransactionStatus status) throws TransactionException;

}
```

우리가 이 `PlatformTransactionManager`를 확장할 일은 거의 없지만 만약 확장해야한다면 `PlatformTransactionManager`를 직접 확장하지 말고 `AbstractPlatformTransactionManager` 를 확장하는 것이 좋다.

`AbstractPlatformTransactionManager`는 추상클래스이고 `PlatformTransactionManager` 에서 제공하는 세가지 메서드를 전부 구현하고 있다. `getTransaction()` 메서드만 살펴보면 아래와 같은 형태로 구현되어 있다.

`AbstractPlatformTransactionManager.getTransaction()`

```java
@Override
public final TransactionStatus getTransaction(@Nullable TransactionDefinition definition)
      throws TransactionException {

   // Use defaults if no transaction definition given.
   TransactionDefinition def = (definition != null ? definition : TransactionDefinition.withDefaults());

   Object transaction = doGetTransaction();
   boolean debugEnabled = logger.isDebugEnabled();

   if (isExistingTransaction(transaction)) {
      // Existing transaction found -> check propagation behavior to find out how to behave.
      return handleExistingTransaction(def, transaction, debugEnabled);
   }

   // Check definition settings for new transaction.
   if (def.getTimeout() < TransactionDefinition.TIMEOUT_DEFAULT) {
      throw new InvalidTimeoutException("Invalid transaction timeout", def.getTimeout());
   }

   // No existing transaction found -> check propagation behavior to find out how to proceed.
   if (def.getPropagationBehavior() == TransactionDefinition.PROPAGATION_MANDATORY) {
      throw new IllegalTransactionStateException(
            "No existing transaction found for transaction marked with propagation 'mandatory'");
   }
   else if (def.getPropagationBehavior() == TransactionDefinition.PROPAGATION_REQUIRED ||
         def.getPropagationBehavior() == TransactionDefinition.PROPAGATION_REQUIRES_NEW ||
         def.getPropagationBehavior() == TransactionDefinition.PROPAGATION_NESTED) {
      SuspendedResourcesHolder suspendedResources = suspend(null);
      if (debugEnabled) {
         logger.debug("Creating new transaction with name [" + def.getName() + "]: " + def);
      }
      try {
         return startTransaction(def, transaction, debugEnabled, suspendedResources);
      }
      catch (RuntimeException | Error ex) {
         resume(null, suspendedResources);
         throw ex;
      }
   }
   else {
      // Create "empty" transaction: no actual transaction, but potentially synchronization.
      if (def.getIsolationLevel() != TransactionDefinition.ISOLATION_DEFAULT && logger.isWarnEnabled()) {
         logger.warn("Custom isolation level specified but no actual transaction initiated; " +
               "isolation level will effectively be ignored: " + def);
      }
      boolean newSynchronization = (getTransactionSynchronization() == SYNCHRONIZATION_ALWAYS);
      return prepareTransactionStatus(def, null, true, newSynchronization, debugEnabled, null);
   }
}
```

간단하게 `doGetTransaction()` 메서드에서 트랜잭션을 가져오는것으로 시작해서 트랜잭션 타임아웃, 트랜잭션 전파 속성에 따른 적절한 로직을 수행하는 모습을 확인할 수 있다.

여기서 핵심은 `doGetTransaction()` 인데 이 `doGetTransaction()` 은 아래와 같이 추상메서드로 선언되어 있다.

`AbstractPlatformTransactionManager.doGetTransaction()`

```java
protected abstract Object doGetTransaction() throws TransactionException;
```

`AbstractPlatformTransactionManager` 을 상속받으면 `doGetTransaction()` 을 구현함으로써 추상화되어있는 `getTransaction()` 메서드에 어느정도 기본적인 구조를 재활용함과 동시에 구현체별로 다른 동작을 할 수 있도록 도와주는 템플릿 메서드 패턴을 사용할 수 있다.  `PlatformTransactionManager` 의 `commit()`, `rollback()` 도 같은 형태로 동작한다.

`AbstractPlatformTransactionManager`의 구현체중 가장 대표적인 `DataSourceTransactionManager` 를 살펴보면 아래와 같이 `doGetTransaction()` 의 구현을 확인할 수 있다

`DataSourceTransactionManager.doGetTransaction()`

```java
	@Override
	protected Object doGetTransaction() {
		DataSourceTransactionObject txObject = new DataSourceTransactionObject();
		txObject.setSavepointAllowed(isNestedTransactionAllowed());
		ConnectionHolder conHolder =
				(ConnectionHolder) TransactionSynchronizationManager.getResource(obtainDataSource());
		txObject.setConnectionHolder(conHolder, false);
		return txObject;
	}
```

`DataSourceTransactionManager.doCommit()`

```java
	@Override
	protected void doCommit(DefaultTransactionStatus status) {
		DataSourceTransactionObject txObject = (DataSourceTransactionObject) status.getTransaction();
		Connection con = txObject.getConnectionHolder().getConnection();
		if (status.isDebug()) {
			logger.debug("Committing JDBC transaction on Connection [" + con + "]");
		}
		try {
			con.commit();
		}
		catch (SQLException ex) {
			throw translateException("JDBC commit", ex);
		}
	}
```

`DataSourceTransactionManager.doCommit()`

```java
	@Override
	protected void doRollback(DefaultTransactionStatus status) {
		DataSourceTransactionObject txObject = (DataSourceTransactionObject) status.getTransaction();
		Connection con = txObject.getConnectionHolder().getConnection();
		if (status.isDebug()) {
			logger.debug("Rolling back JDBC transaction on Connection [" + con + "]");
		}
		try {
			con.rollback();
		}
		catch (SQLException ex) {
			throw translateException("JDBC rollback", ex);
		}
	}

```

`DataSourceTransactionManager` 외에도 다양한 구현체가 있으니 [docs](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/transaction/support/AbstractPlatformTransactionManager.html) 를 직접확인해보면 된다.



# TransactionStatus

`PlatformTransactionManager`의 `getTransaction()` 메서드는 `TransactionStatus` 타입을 리턴하도록 되어있다. 

`TransactionStatus`은 말 그대로 트랜잭션의 상태를 나타내는 인터페이스이다. `AbstractPlatformTransactionManager` 에서 사용하는 기본 구현체는 `DefaultTransactionStatus` 인데 `DefaultTransactionStatus` 의 생성자는 단 한개만 정의되어 있다.

`DefaultTransactionStatus의 생성자`

```java
	public DefaultTransactionStatus(
			@Nullable Object transaction, boolean newTransaction, boolean newSynchronization,
			boolean readOnly, boolean debug, @Nullable Object suspendedResources) {

		this.transaction = transaction;
		this.newTransaction = newTransaction;
		this.newSynchronization = newSynchronization;
		this.readOnly = readOnly;
		this.debug = debug;
		this.suspendedResources = suspendedResources;
	}

```

nullable하지만 transaction 오브젝트를 포함하여 여러 필드를 받아 저장하고 있기 때문에`AbstractPlatformTransactionManager`가 내부적으로 필요로하는 모든 상태 정보를 보유한다.

# TransactionDefinintion

또 이어서 `PlatformTransactionManager`의 `getTransaction()` 메서드는 `TransactionDefinintion` 타입의 파라미터를 받도록 되어 있다.

`TransactionDefinintion` 는 트랜잭션 속성을 정의하는 인터페이스이다. 여기에서 트랜잭션 전파속성, 격리수준, timeout 시간, readonly 에 대한 속성을 정의할 수 있다.

기본 구현체인 `DefaultTransactionAttribute`를 사용하는데 `DefaultTransactionAttribute` 의 생성자를 살펴보면 아래와 같이 되어있다.

`DefaultTransactionDefinition의 생성자들과 초기화 필드`

```java
	private int propagationBehavior = PROPAGATION_REQUIRED;
	private int isolationLevel = ISOLATION_DEFAULT;
	private int timeout = TIMEOUT_DEFAULT;
	private boolean readOnly = false;

	public DefaultTransactionDefinition() {
	}

	public DefaultTransactionDefinition(TransactionDefinition other) {
		this.propagationBehavior = other.getPropagationBehavior();
		this.isolationLevel = other.getIsolationLevel();
		this.timeout = other.getTimeout();
		this.readOnly = other.isReadOnly();
		this.name = other.getName();
	}
```

`TransactionDefinition` 타입을 받더라도 `TransactionDefinition` 의 추상메서드들이 아래와 같은 형식으로 되어있어서 아래의 값이 기본값이라고 보면 된다

`TransactionDefinition의 default 메서드들`

```java
	default int getPropagationBehavior() {
		return PROPAGATION_REQUIRED;
	}
	
	...
	
	default int getIsolationLevel() {
		return ISOLATION_DEFAULT;
	}

  ..

	default int getTimeout() {
		return TIMEOUT_DEFAULT;
	}
	
  ..

	default boolean isReadOnly() {
		return false;
	}
```



사실상 `DefaultTransactionDefinition` 을 직접적으로 사용하는 부분은 거의 없고 `TransactionAttribute` 라는 인터페이스와 함께 사용되기 때문에 `TransactionAttribute` 도 같이 보는게 좋다.

# TransactionAttribute

`TransactionAttribute` 은 `TransactionDefinition` 에 `rollbackOn()` 이라는 메서드를 추가해서 사용하기 위해 존재하는 인터페이스이다.

`rollbackOn()` 이라는 속성 자체가 Spring AOP에만 사용되는 개념이기 때문에? `TransactionDefinition`을 새롭게 확장해서 사용하는 것 같다.

`DefaultTransactionAttribute 클래스`

```java

public interface TransactionAttribute extends TransactionDefinition {

	@Nullable
	String getQualifier();

	Collection<String> getLabels();

	boolean rollbackOn(Throwable ex);

}

public class DefaultTransactionAttribute extends DefaultTransactionDefinition implements TransactionAttribute {

	...
    
	@Override
	public boolean rollbackOn(Throwable ex) {
		return (ex instanceof RuntimeException || ex instanceof Error);
	}
   
}
```

기본 구현체인 `DefaultTransactionAttribute` 의 `rollbackOn()` 메서드를 확인해보면 많이 알고 있는 내용인 unchecked 예외만 롤백하는 모습을 확인할 수 있다.

이 `DefaultTransactionAttribute` 마저도 직접적으로 잘 사용되지는 않고 않고 한단계 저수준인 `RuleBasedTransactionAttribute`을 사용하는 것 같다.

`RuleBasedTransactionAttribute.rollbackOn()`

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
   if (winner == null) {
      logger.trace("No relevant rollback rule found: applying default rules");
      return super.rollbackOn(ex);
   }

   return !(winner instanceof NoRollbackRuleAttribute);
}
```





# TransactionAspectSupport

위에서 확인한 4개의 인터페이스를 통해 기본적인 트랜잭션에 대한 준비를 끝냈다면 실제로 트랜잭션 Aspect에 대한 처리가 필요하다.

Spring은 `TransactionInterceptor`를 통해 트랜잭션 Aspect를 설정하는데 `TransactionAspectSupport`는 `TransactionInterceptor`의 기본이되는 기본 Aspect 클래스다.

`TransactionAspectSupport` 는 추상클래스이기 때문에 실제 구현체인 `TransactionInterceptor` 를 보는게 맞을 수 있지만 사실 `TransactionAspectSupport` 에는 추상 메서드가 없는 형태로 되어있다.

그리고 `TransactionInterceptor`  의 역할은 `MethodInterceptor` 의 `invoke()` 메서드를 구현해서 Spring 기본 트랜잭션 API와 통합하게 해주는 역할을 한다고 생각하면 된다.

`TransactionAspectSupport` 의 주요 메서드인 `invokeWithinTransaction()` 를 살펴보면 큰 단락에서 `CallbackPreferringPlatformTransactionManager`이나 `ReactiveTransactionManager`이 아니라면 `AbstractPlatformTransactionManager.getTransaction()`  메서드를 호출해서 트랜잭션을 시작하고 에러가 났을 때는 `rollback()`, 정상적일때는 `commit()` 시키는 로직을 확인할 수 있다.

`TransactionAspectSupport.invokeWithinTransaction() 메서드 구현 일부`

```java

...

			// Standard transaction demarcation with getTransaction and commit/rollback calls.
			TransactionInfo txInfo = createTransactionIfNecessary(ptm, txAttr, joinpointIdentification);

			Object retVal;
			try {
				// This is an around advice: Invoke the next interceptor in the chain.
				// This will normally result in a target object being invoked.
				retVal = invocation.proceedWithInvocation();
			}
			catch (Throwable ex) {
				// target invocation exception
				completeTransactionAfterThrowing(txInfo, ex);
				throw ex;
			}
			finally {
				cleanupTransactionInfo(txInfo);
			}

			if (retVal != null && vavrPresent && VavrDelegate.isVavrTry(retVal)) {
				// Set rollback-only in case of Vavr failure matching our rollback rules...
				TransactionStatus status = txInfo.getTransactionStatus();
				if (status != null && txAttr != null) {
					retVal = VavrDelegate.evaluateTryFailure(retVal, txAttr, status);
				}
			}

			commitTransactionAfterReturning(txInfo);
			return retVal;

...
  
  
```

코드가 너무 많아져서 생략했지만 `createTransactionIfNecessary()`에서 `getTransaction()` 호출, `completeTransactionAfterThrowing()` 에서 `rollback()` 호출, `commitTransactionAfterReturning()` 에서 `commit()`을 호출한다.



# SpringBoot의 선언적 트랜잭션

이제 실제로 우리가 사용중인 `@Transactional`을 어떻게 Spring Boot가 `TransactionInterceptor`을 통해 트랜잭션을 처리하는지만 확인하면 된다.

Spring Boot에서 [spring.factories](https://github.com/spring-projects/spring-boot/blob/v2.0.0.M3/spring-boot-autoconfigure/src/main/resources/META-INF/spring.factories) 파일은 AutoConfiguration 관련 설정을한다. 자세한이야기는 [Spring Boot에서 Auto-Configuration이 동작하는 방법](https://sup2is.github.io/2020/11/16/how-spring-auto-configuration-works.html) 에서 확인할 수 있다.

spring.factories 파일에는 `TransactionAutoConfiguration` 를 빈으로 등록하는데 이 설정 안에서는 `@EnableTransactionManagement` 이라는 애너테이션이 있다. 

```java
@Configuration(proxyBeanMethods = false)
@ConditionalOnClass(PlatformTransactionManager.class)
@AutoConfigureAfter({ JtaAutoConfiguration.class, HibernateJpaAutoConfiguration.class,
		DataSourceTransactionManagerAutoConfiguration.class, Neo4jDataAutoConfiguration.class })
@EnableConfigurationProperties(TransactionProperties.class)
public class TransactionAutoConfiguration {

...

	@ConditionalOnMissingBean(AbstractTransactionManagementConfiguration.class)
	public static class EnableTransactionManagementConfiguration {

		@Configuration(proxyBeanMethods = false)
		@EnableTransactionManagement(proxyTargetClass = false) // <- 요기
		@ConditionalOnProperty(prefix = "spring.aop", name = "proxy-target-class", havingValue = "false",
				matchIfMissing = false)
		public static class JdkDynamicAutoProxyConfiguration {

		}

		@Configuration(proxyBeanMethods = false)
		@EnableTransactionManagement(proxyTargetClass = true) // <- 요기
		@ConditionalOnProperty(prefix = "spring.aop", name = "proxy-target-class", havingValue = "true",
				matchIfMissing = true)
		public static class CglibAutoProxyConfiguration {

		}

...

}

```



`@EnableTransactionManagement`

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Import(TransactionManagementConfigurationSelector.class)
public @interface EnableTransactionManagement {

	boolean proxyTargetClass() default false;

	AdviceMode mode() default AdviceMode.PROXY;

	int order() default Ordered.LOWEST_PRECEDENCE;

}

```

`@EnableTransactionManagement` 는 `TransactionManagementConfigurationSelector` 를 import 하고 있고  `AdviceMode.PROXY` 가 기본값으로 설정되어 있기 때문에 결론적으로 아래 `ProxyTransactionManagementConfiguration ` 설정이 로드된다.

`TransactionManagementConfigurationSelector`

```java
public class TransactionManagementConfigurationSelector extends AdviceModeImportSelector<EnableTransactionManagement> {

	@Override
	protected String[] selectImports(AdviceMode adviceMode) {
		switch (adviceMode) {
			case PROXY:
				return new String[] {AutoProxyRegistrar.class.getName(),
						ProxyTransactionManagementConfiguration.class.getName()};
			case ASPECTJ:
				return new String[] {determineTransactionAspectClass()};
			default:
				return null;
		}
	}

	private String determineTransactionAspectClass() {
		return (ClassUtils.isPresent("javax.transaction.Transactional", getClass().getClassLoader()) ?
				TransactionManagementConfigUtils.JTA_TRANSACTION_ASPECT_CONFIGURATION_CLASS_NAME :
				TransactionManagementConfigUtils.TRANSACTION_ASPECT_CONFIGURATION_CLASS_NAME);
	}

}

```



`ProxyTransactionManagementConfiguration`

```java
@Configuration(proxyBeanMethods = false)
@Role(BeanDefinition.ROLE_INFRASTRUCTURE)
public class ProxyTransactionManagementConfiguration extends AbstractTransactionManagementConfiguration {

	@Bean(name = TransactionManagementConfigUtils.TRANSACTION_ADVISOR_BEAN_NAME)
	@Role(BeanDefinition.ROLE_INFRASTRUCTURE)
	public BeanFactoryTransactionAttributeSourceAdvisor transactionAdvisor(
			TransactionAttributeSource transactionAttributeSource, TransactionInterceptor transactionInterceptor) {

		BeanFactoryTransactionAttributeSourceAdvisor advisor = new BeanFactoryTransactionAttributeSourceAdvisor();
		advisor.setTransactionAttributeSource(transactionAttributeSource);
		advisor.setAdvice(transactionInterceptor);
		if (this.enableTx != null) {
			advisor.setOrder(this.enableTx.<Integer>getNumber("order"));
		}
		return advisor;
	}

	@Bean
	@Role(BeanDefinition.ROLE_INFRASTRUCTURE)
	public TransactionAttributeSource transactionAttributeSource() {
		return new AnnotationTransactionAttributeSource();
	}

	@Bean
	@Role(BeanDefinition.ROLE_INFRASTRUCTURE)
	public TransactionInterceptor transactionInterceptor(TransactionAttributeSource transactionAttributeSource) {
		TransactionInterceptor interceptor = new TransactionInterceptor();
		interceptor.setTransactionAttributeSource(transactionAttributeSource);
		if (this.txManager != null) {
			interceptor.setTransactionManager(this.txManager);
		}
		return interceptor;
	}

}
```

`ProxyTransactionManagementConfiguration` 에서는 트랜잭션 경계를 담당하는 `TransactionInterceptor`도 설정하는 모습도 확인할 수 있고 `AnnotationTransactionAttributeSource` 도 만드는데 이때 `SpringTransactionAnnotationParser` 가 등록된다.

`SpringTransactionAnnotationParser`

```java
public class SpringTransactionAnnotationParser implements TransactionAnnotationParser, Serializable {

   @Override
   public boolean isCandidateClass(Class<?> targetClass) {
      return AnnotationUtils.isCandidateClass(targetClass, Transactional.class);
   }

   @Override
   @Nullable
   public TransactionAttribute parseTransactionAnnotation(AnnotatedElement element) {
      AnnotationAttributes attributes = AnnotatedElementUtils.findMergedAnnotationAttributes(
            element, Transactional.class, false, false);
      if (attributes != null) {
         return parseTransactionAnnotation(attributes);
      }
      else {
         return null;
      }
   }

   public TransactionAttribute parseTransactionAnnotation(Transactional ann) {
      return parseTransactionAnnotation(AnnotationUtils.getAnnotationAttributes(ann, false, false));
   }

   protected TransactionAttribute parseTransactionAnnotation(AnnotationAttributes attributes) {
      RuleBasedTransactionAttribute rbta = new RuleBasedTransactionAttribute();

      Propagation propagation = attributes.getEnum("propagation");
      rbta.setPropagationBehavior(propagation.value());
      Isolation isolation = attributes.getEnum("isolation");
      rbta.setIsolationLevel(isolation.value());

      rbta.setTimeout(attributes.getNumber("timeout").intValue());
      String timeoutString = attributes.getString("timeoutString");
      Assert.isTrue(!StringUtils.hasText(timeoutString) || rbta.getTimeout() < 0,
            "Specify 'timeout' or 'timeoutString', not both");
      rbta.setTimeoutString(timeoutString);

      rbta.setReadOnly(attributes.getBoolean("readOnly"));
      rbta.setQualifier(attributes.getString("value"));
      rbta.setLabels(Arrays.asList(attributes.getStringArray("label")));

      List<RollbackRuleAttribute> rollbackRules = new ArrayList<>();
      for (Class<?> rbRule : attributes.getClassArray("rollbackFor")) {
         rollbackRules.add(new RollbackRuleAttribute(rbRule));
      }
      for (String rbRule : attributes.getStringArray("rollbackForClassName")) {
         rollbackRules.add(new RollbackRuleAttribute(rbRule));
      }
      for (Class<?> rbRule : attributes.getClassArray("noRollbackFor")) {
         rollbackRules.add(new NoRollbackRuleAttribute(rbRule));
      }
      for (String rbRule : attributes.getStringArray("noRollbackForClassName")) {
         rollbackRules.add(new NoRollbackRuleAttribute(rbRule));
      }
      rbta.setRollbackRules(rollbackRules);

      return rbta;
   }


  ...

}
```

`SpringTransactionAnnotationParser` 에서 `@Transactional`에 정의된 속성들을 파싱하는 역할을 한다.

그 외에도 `AopUtils`, `TransactionAttributeSourcePointcut` `AbstractFallbackTransactionAttributeSource`  클래스 등등을 사용해서 최종적으로 `AbstractAutoProxyCreator` 이라는 빈 후처리기에 의해 Proxy를 생성하는 모습을 확인할 수 있다 (생각보다 엮여있는 코드가 너무 많아서 자세한 설명은 생략. 빈 후처리기를 통해 포인트컷 + 어드바이스를 충족하는 타겟 클래스가 Proxy 클래스를 자동적으로 등록시켜서 프록시 역할을 수행한다는 것을 기억하면 된다.)

`AbstractAutoProxyCreator.postProcessAfterInitialization() 메서드`

```java


	@Override
	public Object postProcessAfterInitialization(@Nullable Object bean, String beanName) {
		if (bean != null) {
			Object cacheKey = getCacheKey(bean.getClass(), beanName);
			if (this.earlyProxyReferences.remove(cacheKey) != bean) {
				return wrapIfNecessary(bean, beanName, cacheKey);
			}
		}
		return bean;
	}

	protected Object wrapIfNecessary(Object bean, String beanName, Object cacheKey) {
		if (StringUtils.hasLength(beanName) && this.targetSourcedBeans.contains(beanName)) {
			return bean;
		}
		if (Boolean.FALSE.equals(this.advisedBeans.get(cacheKey))) {
			return bean;
		}
		if (isInfrastructureClass(bean.getClass()) || shouldSkip(bean.getClass(), beanName)) {
			this.advisedBeans.put(cacheKey, Boolean.FALSE);
			return bean;
		}

		// Create proxy if we have advice.
		Object[] specificInterceptors = getAdvicesAndAdvisorsForBean(bean.getClass(), beanName, null);
		if (specificInterceptors != DO_NOT_PROXY) {
			this.advisedBeans.put(cacheKey, Boolean.TRUE);
			Object proxy = createProxy(
					bean.getClass(), beanName, specificInterceptors, new SingletonTargetSource(bean));
			this.proxyTypes.put(cacheKey, proxy.getClass());
			return proxy;
		}

		this.advisedBeans.put(cacheKey, Boolean.FALSE);
		return bean;
	}
```





# 요약

1. 특별한 경우가 아니라면 선언적 트랜잭션을 사용하자.
2. `AbstractPlatformTransactionManager`에서 팩토리 메서드 패턴을 사용해서 여러 `PlatformTransactionManager`을 확장한다.
3. `TransactionInterceptor` 를 통해 트랜잭션 경계를 설정하고 여기에서 주입된 `PlatformTransactionManager` 를 사용한다.
4. `SpringTransactionAnnotationParser` 를 통해 `@Transactional` 관련 속성을 파싱한다.
5. `AbstractAutoProxyCreator` 에 의해 Proxy로 생성되고 실제 클라이언트가 타깃에 접근할때는 Proxy를 거쳐 `TransactionInterceptor` 를 사용해 트랜잭션을 열고 타깃의 메서드를 호출하고 커밋, 롤백을 수행한다.



# 마무리

거의 문서 & 코드만보고 블로그를 작성한거라 틀린 내용은 없을것이라 예상되나 충분히 다른 방식으로 스프링이 트랜잭션을 설정할 수 있을것이라 생각한다. 그래도 주요 인터페이스에 대한 설명은 문서만 봤으니 틀린게 없겠지 뭐 ...



<br>

***

포스팅은 여기까지 하겠습니다. 감사합니다!



<br>

**References**

-  [https://docs.spring.io/spring-framework/docs/4.2.x/spring-framework-reference/html/transaction.html](https://docs.spring.io/spring-framework/docs/4.2.x/spring-framework-reference/html/transaction.html)
- [https://docs.spring.io/spring-framework/docs/current/reference/html/data-access.html#transaction-declarative](https://docs.spring.io/spring-framework/docs/current/reference/html/data-access.html#transaction-declarative)
- [https://docs.spring.io/spring-framework/docs/current/reference/html/data-access.html#transaction](https://docs.spring.io/spring-framework/docs/current/reference/html/data-access.html#transaction)
