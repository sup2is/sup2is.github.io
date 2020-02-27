---
layout: post
title: "Spring Data JPA"
tags: [Spring Framework, JPA]
comments: true
date: 2019-03-21
nav_order: 4
parent: Spring
---





 최근에 알고리즘 + 스터디로 너무 바쁘게 보내고 있어서 사실 spring 관련한 공부하기가 여간 쉬운게 아니였다 .. 회사일이 다시 러프해질꺼같아서 업무시간에 틈틈히 spring 공부중이다 화이팅!

<br>

# JPA

------

<br>

JPA는 Java Persistence API의 약자다. IT쪽에서 Persistence는 주로 영속성이라는 단어로 해석이된다. Persistence는 Application에서 Java 객체가 Application이 종료된 이후에도 계속 유지되는 메커니즘을 의미하기때문에 Persistence라는 단어를 포함하여 JPA라고 통칭한다.

<br>

JPA가 어떤 tool이나 framework는 아니다. 혼자서 작동할 수도 없기 때문에 반드시 구현체가 필요하다.

JPA는 Java의 POJO 객체가 RDB에 매핑되는 방식을 Annotaion 또는 xml방식으로 관계형 매핑을 정의할 수 있게 해준다.



# Repository Interface

------



Spring JPA에서 Repository는 가장 핵심적인 인터페이스가 되는데 도메인 class와 도메인의 id 타입을 제네릭 타입인터페이스로 받는다



<br>

> ```java
> public interface CrudRepository<T, ID extends Serializable> extends Repository<T, ID> {
> 
>     <S extends T> S save(S entity); 
> 
>     T findOne(ID primaryKey);       
> 
>     Iterable<T> findAll();          
> 
>     Long count();                   
> 
>     void delete(T entity);          
> 
>     boolean exists(ID primaryKey);  
> 
>     // … more functionality omitted.
> }
> ```



<br>

위에서 볼 수 있는 CrudRepository는 Repository를 상속받은 인터페이스로써 Crud 기능을 제공해준다

<br>

CrudRepository 이외에도 PagingAndSortingRepository, ReacitiveCrudRepository 등등이 있는데 전부 Repository 인터페이스를 상속받는다

<br>

> org.springframework.data.repository
>
> ## Interface Repository<T,ID>
>
> - - **Type Parameters:**
>
>     `T` - the domain type the repository manages
>
>     `ID` - the type of the id of the entity the repository manages
>
>   - All Known Subinterfaces:
>
>     [CrudRepository](https://docs.spring.io/spring-data/commons/docs/current/api/org/springframework/data/repository/CrudRepository.html)<T,ID>, [PagingAndSortingRepository](https://docs.spring.io/spring-data/commons/docs/current/api/org/springframework/data/repository/PagingAndSortingRepository.html)<T,ID>, [ReactiveCrudRepository](https://docs.spring.io/spring-data/commons/docs/current/api/org/springframework/data/repository/reactive/ReactiveCrudRepository.html)<T,ID>, [ReactiveSortingRepository](https://docs.spring.io/spring-data/commons/docs/current/api/org/springframework/data/repository/reactive/ReactiveSortingRepository.html)<T,ID>, [RevisionRepository](https://docs.spring.io/spring-data/commons/docs/current/api/org/springframework/data/repository/history/RevisionRepository.html)<T,ID,N>, [RxJava2CrudRepository](https://docs.spring.io/spring-data/commons/docs/current/api/org/springframework/data/repository/reactive/RxJava2CrudRepository.html)<T,ID>, [RxJava2SortingRepository](https://docs.spring.io/spring-data/commons/docs/current/api/org/springframework/data/repository/reactive/RxJava2SortingRepository.html)<T,ID>
>
>   ------
>
>   ```
>   @Indexed
>   public interface Repository<T,ID>
>   ```
>
>   Central repository marker interface. Captures the domain type to manage as well as the domain type's id type. General purpose is to hold type information as well as being able to discover interfaces that extend this one during classpath scanning for easy Spring bean creation.
>
>   Domain repositories extending this interface can selectively expose CRUD methods by simply declaring methods of the same signature as those declared in [`CrudRepository`](https://docs.spring.io/spring-data/commons/docs/current/api/org/springframework/data/repository/CrudRepository.html).
>
>   - **Author:**
>
>     Oliver Gierke
>
>   - **See Also:**
>
>     [`CrudRepository`](https://docs.spring.io/spring-data/commons/docs/current/api/org/springframework/data/repository/CrudRepository.html)



<br>

Repository 인터페이스를 까보면 그냥 marker 인터페이스 역할만 해주고 Spring에서 제공하는 CrudRepository, PagingAndSortingRepository 이외에 따로 필요한 인터페이스가 있으면 Repository 인터페이스를 상속받는 인터페이스를 새로 작성하면 된다.

<br>

\- example

> ```java
> interface PersonRepository extends Repository<User, Long> {
>   List<Person> findByLastname(String lastname);
> }
> ```

<br>

# Query Method

------

이제 CustomRepository를 만들어서 메서드명을 실제 db에 엑세스하는 쿼리로 만드는 부분이다.

<br>

> ```java
> public interface PersonRepository extends Repository<User, Long> {
> 
>   List<Person> findByEmailAddressAndLastname(EmailAddress emailAddress, String lastname);
> 
>   // Enables the distinct flag for the query
>   List<Person> findDistinctPeopleByLastnameOrFirstname(String lastname, String firstname);
>   List<Person> findPeopleDistinctByLastnameOrFirstname(String lastname, String firstname);
> 
>   // Enabling ignoring case for an individual property
>   List<Person> findByLastnameIgnoreCase(String lastname);
>   // Enabling ignoring case for all suitable properties
>   List<Person> findByLastnameAndFirstnameAllIgnoreCase(String lastname, String firstname);
> 
>   // Enabling static ORDER BY for a query
>   List<Person> findByLastnameOrderByFirstnameAsc(String lastname);
>   List<Person> findByLastnameOrderByFirstnameDesc(String lastname);
> }
> ```



쿼리 빌더 메커니즘은 findBy .. readBy 등등의 접두어들을 떼어내고 나머지 부분을 파싱한다. (사실 findBy ... 등등 OOOBy로 시작하는 접두어를 jpa에서 사용하는지 처음알았다 .. 쩝)

And나 Or, OrderBy, IgnoreCase 등등을 메서드명에서 직접 설정할 수 있다. 



> | Keyword             | Sample                                                       | JPQL snippet                                                 |
> | :------------------ | :----------------------------------------------------------- | :----------------------------------------------------------- |
> | `And`               | `findByLastnameAndFirstname`                                 | `… where x.lastname = ?1 and x.firstname = ?2`               |
> | `Or`                | `findByLastnameOrFirstname`                                  | `… where x.lastname = ?1 or x.firstname = ?2`                |
> | `Is,Equals`         | `findByFirstname`,`findByFirstnameIs`,`findByFirstnameEquals` | `… where x.firstname = ?1`                                   |
> | `Between`           | `findByStartDateBetween`                                     | `… where x.startDate between ?1 and ?2`                      |
> | `LessThan`          | `findByAgeLessThan`                                          | `… where x.age < ?1`                                         |
> | `LessThanEqual`     | `findByAgeLessThanEqual`                                     | `… where x.age <= ?1`                                        |
> | `GreaterThan`       | `findByAgeGreaterThan`                                       | `… where x.age > ?1`                                         |
> | `GreaterThanEqual`  | `findByAgeGreaterThanEqual`                                  | `… where x.age >= ?1`                                        |
> | `After`             | `findByStartDateAfter`                                       | `… where x.startDate > ?1`                                   |
> | `Before`            | `findByStartDateBefore`                                      | `… where x.startDate < ?1`                                   |
> | `IsNull`            | `findByAgeIsNull`                                            | `… where x.age is null`                                      |
> | `IsNotNull,NotNull` | `findByAge(Is)NotNull`                                       | `… where x.age not null`                                     |
> | `Like`              | `findByFirstnameLike`                                        | `… where x.firstname like ?1`                                |
> | `NotLike`           | `findByFirstnameNotLike`                                     | `… where x.firstname not like ?1`                            |
> | `StartingWith`      | `findByFirstnameStartingWith`                                | `… where x.firstname like ?1`(parameter bound with appended `%`) |
> | `EndingWith`        | `findByFirstnameEndingWith`                                  | `… where x.firstname like ?1`(parameter bound with prepended `%`) |
> | `Containing`        | `findByFirstnameContaining`                                  | `… where x.firstname like ?1`(parameter bound wrapped in `%`) |
> | `OrderBy`           | `findByAgeOrderByLastnameDesc`                               | `… where x.age = ?1 order by x.lastname desc`                |
> | `Not`               | `findByLastnameNot`                                          | `… where x.lastname <> ?1`                                   |
> | `In`                | `findByAgeIn(Collection<Age> ages)`                          | `… where x.age in ?1`                                        |
> | `NotIn`             | `findByAgeNotIn(Collection<Age> ages)`                       | `… where x.age not in ?1`                                    |
> | `True`              | `findByActiveTrue()`                                         | `… where x.active = true`                                    |
> | `False`             | `findByActiveFalse()`                                        | `… where x.active = false`                                   |
> | `IgnoreCase`        | `findByFirstnameIgnoreCase`                                  | `… where UPPER(x.firstame) = UPPER(?1)`                      |

<br>

Repository에 선언되는 메서드 명이 jpa에서는 굉장히 중요한 역할을 하는데 자세한 알고리즘은 직접 찾아보는게 좋을 듯 하고 underscore(_) 와 camel기법을 지원하는데 나는 java진영이니까 camel표기법을 사용해야한다.

<br>

메서드 명 이외에 직접 쿼리를 작성하는것도 다음과 같이 가능하다

> ```java
> @Entity
> @NamedQuery(name = "User.findByEmailAddress",
>   query = "select u from User u where u.emailAddress = ?1")
> public class User {
> 
> }
> ```



도메인 클래스와 메서드명의 구분을 위하여 .를 사용하여 이름을 정해주고 매핑될 메서드에 query 필드로 실제 쿼리를 작성해준다. 



## @Query

------

메서드명, 도메인클래스에 @NamedQuery를 작성하지 않고 Repository에 직접 @Query를 사용하여 실제 호출 될 쿼리를 작성하는 방법도 있다. (이게 가장 많이 사용하는 방식인듯)

<br>

> ```java
> public interface UserRepository extends JpaRepository<User, Long> {
> 
>   @Query("select u from User u where u.firstname like %?1")
>   List<User> findByFirstnameEndsWith(String firstname);
> }
> ```

<br>

사실 위에서 봤던 메서드명으로 db에 엑세스하는 방법은 생각보다 메서드명 짜기가 까다로울꺼같아서 걱정됐는데 @Query를 사용하면 굉장히 편리하게 사용할 듯 하다!!

<br>

위에서 

@Query("select u from User u where u.firstname like %?1") 는 사실 우리가 평소에 쓰던 쿼리랑 형태가 조금 다른데 다음과 같이 해석한다.



> | 변수         | 사용                             | 설명                                                         |
> | :----------- | :------------------------------- | :----------------------------------------------------------- |
> | `entityName` | `select x from #{#entityName} x` | 주어진 repository에 관련된 도메인 타입의 `entityName` 를 삽입하세요. `entityName`는 다음과 같이 해석됩니다 : 만약 `@Entity`어노테이션에서 도메인 타입 이름을 정한다면 , 그것은 사용될 것입니다. 그렇지 않으면 도메인 타입의 간단한 class-name이 사용될 것입니다. (역주: 그냥 뒤의 예제들을 조금 살펴보자^^; ) |

<br>

이 방법이 맘에 들지 않으면 다음과 같이 사용한다

> ```java
> public interface UserRepository extends JpaRepository<User, Long> {
> 
>   @Query(value = "SELECT * FROM USERS WHERE EMAIL_ADDRESS = ?0", nativeQuery = true)
>   User findByEmailAddress(String emailAddress);
> }
> ```



<br>

@Query 어노테이션의 nativeQuery 옵션을 true로 변경해주면 실제 db에 액세스하는 쿼리처럼 사용 할 수 있다.

추가적으로 파라미터에 대한 매핑도

> ```java
> public interface UserRepository extends JpaRepository<User, Long> {
> 
>   @Query("select u from User u where u.firstname = :firstname or u.lastname = :lastname")
>   User findByLastnameOrFirstname(@Param("lastname") String lastname,
>                                  @Param("firstname") String firstname);
> }
> ```

@Param 어노테이션을 통해서 사용 가능하다.



<br>



# Annotation

------

jpa에서 사용하는 annotation들을 간단하게 정리해본다.



<br>

## @Entity

------

domain클래스에 주로 선언되며 이 domain객체가 entity객체임을 지정하며 테이블과 직접 매핑되는 역할을 해준다.

<br>

## @Table

------

@Entitiy 객체와 매핑할 db table을 지정해준다 name속성이 실제 table과 매칭되는 부분이고 만약 entity 클래스와 table 명이 다르다면 name속성에 실제 table 이름을 써주면 된다.

<br>

## @Column

------

@Table과 마찬가지로 매핑할 컬럼명이 실제 액세스할 db table의 컬럼명과 다르다면 name속성으로 지정해서 사용할 수 있다. nullable 과 unique 속성도 사용 가능하다.

<br>

## @Enumerated

------

java의 enum타입을 컬럼에 매핑할때 사용한다 value 속성은 EnumType.ORDINAL (enum 순서를 값으로 db에 저장), EnumType.STRING(enum 이름을 값으로 db에 저장) 이 있고 default는 EnumType.ORDINAL이다.

<br>

## @Temporal

------

java의 java.util.Date 또는 java.util.Calendar 타입을 컬럼에 매핑할때 사용한다. TemporalType.DATE TemporalType.TIME TemporalType.TIMESTAMP가 있다 default 값이 없으므로 반드시 한개를 지정해줘야 한다.

<br>

## @Lob

------

db에서 BLOB, CLOB, TEXT 타입과 매핑된다.

<br>

## @Transient

------

@Transient가 지정된 필드는 매핑하지 않는다. 객체에 임시로 어떤 값을 보관하고 싶을 때 사용한다.

<br>

## @DynamicUpdate

------

수정된 데이터만 동적으로 Update 해준다.

<br>

## @DynamicInsert

------

데이터를 저장할 때 entity 객체에 존재하는 필드만으로 Insert SQL을 동적으로 생성해준다

<br>

## @SequenceGenerator

------

Identity 전략 중 Sequence를 사용하기위해 Sequence를 설정 및 생성한다. 식별자 필드인 name은 필수값으로 설정되고 sequenceName, initialValue 등의 필드가 있다.

<br>

## @Transient

------

@Transient가 지정된 필드는 매핑하지 않는다. 객체에 임시로 어떤 값을 보관하고 싶을 때 사용한다.

<br>

## @DynamicUpdate

------

수정된 데이터만 동적으로 Update 해준다.

<br>

## @DynamicInsert

------

데이터를 저장할 때 entity 객체에 존재하는 필드만으로 Insert SQL을 동적으로 생성해준다

<br>

## @SequenceGenerator

------

Identity 전략 중 Sequence를 사용하기위해 Sequence를 설정 및 생성한다. 식별자 필드인 name은 필수값으로 설정되고 sequenceName, initialValue 등의 필드가 있다.

<br>

## @TableGenerator

------

Identity 전략 중 테이블을 사용하기 위해 시퀀스 테이블을 설정 및 생성한다. 식별자 필드인 name은 필수값으로 설정되고 table, pkColumnName, valueColumnName 등의 필드가 있다.

<br>

## @ManyToOne

***



테이블 연관관계를 매핑할 때 다중성을 나타내는 설정값으로 이용되고 N:1의 관계를 매핑할 때 설정한다. 필드로는 optional, fetch, cascade가 있다.

<br>

## @OneToMany

***



@ManyToOne과는 반대로 1:N의 관계를 매핑할 때 사용한다. 속성은 @ManyToOne과 동일하다.

<br>

## @OneToMany

***

마찬가지로 1:1의 관계를 나타내고 속성은 @ManyToOne, @OneToMany와 동일하다.

<br>

## @JoinColum

***

테이블의 연관관계를 매핑할때 사용되며 name필드는 foreign key의 이름으로 동작한다. 기본값은 필드명 + '_' + 컬럼명이고 필드는 referencedColumnName, foreignKey 가 있다.

<br>



# Example (Spring Boot 2.13)

***

실제 연습해보는 시간이다. 간단하게 시나리오 부터 설정하면 Entity가 될 User.class는 id, password, name, phone(전화번호) 정도를 받는다. 회원가입(validation은 제외) 기능을 제공하고 db에 데이터가 들어가면 이름으로 찾기, name으로 찾기, phone으로 찾기, 전체 유저목록 나타내기를 전부 jpa로 구현하는 예제다.

db도 mysql 등등 따로 들어가면 더더욱 좋겠지만 가볍게 하기 위해서 h2 db를 사용한다. h2에 대한 자세한 내용은 [여기](<https://projectjt.tistory.com/4>) 를 확인해보자

예제에서 비지니스로직은 tdd로 구현하고 실제 테스트는 간단한 폼화면을 붙여서 테스트해보도록 하겠다!!



<br>

먼저 이번 예제에서 사용할 lib를 먼저 셋팅한다. 나는 maven을 좋아하기때문에 pom.xml의 dependencies를 다음과 같이 적용한다.

<br>

\- pom.xml

```

	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
		
		<dependency>
			<groupId>javax.servlet</groupId>
			<artifactId>jstl</artifactId>
		</dependency>

		<dependency>
			<groupId>org.apache.tomcat.embed</groupId>
			<artifactId>tomcat-embed-jasper</artifactId>
		</dependency>
		
		<dependency>
		    <groupId>org.springframework.boot</groupId>
		    <artifactId>spring-boot-starter-data-jpa</artifactId>
		</dependency>
		
        <dependency>
			<groupId>org.projectlombok</groupId>
			<artifactId>lombok</artifactId>
			<optional>true</optional>
		</dependency>
		
		<dependency>
    	    <groupId>com.h2database</groupId>
	        <artifactId>h2</artifactId>
	    </dependency>
	</dependencies>

```



<br>

나는 실제 테스트에서 jsp를 사용할 것이므로 javax.servlet과 org.apache.tomcat.embed 따로 추가해 주었다 h2같은경우는 이미 spring boot jpa 모듈에서 버전관리를 해주고 있으니 따로 버전 명시는 필요 없다.



<br>



추가로 h2 db 설정이 끝났다면 console에서 다음 스크립트를 실행한다

<br>

\- user table script

```
CREATE TABLE USER (
  id varchar2(10) primary key,
  password varchar2(200) not null,
  name varchar2(10) not null,
  phone varchar2(14) ,
  email varchar2(30)
)
```

<br>



이제부터 본격적으로 jpa에 연관된 부분이다 먼저 db와 같은방식으로 entity가 될 User이라는 도메인 객체를 생성한다.

<br>

\- User.class

```java
package com.example.demo.model;

import javax.persistence.Entity;
import javax.persistence.Id;

import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Getter;
import lombok.NoArgsConstructor;
import lombok.Setter;
import lombok.ToString;


@Getter
@Setter
@ToString
@Builder
@Entity
@AllArgsConstructor
@NoArgsConstructor
public class User {

	@Id
	private String id;
	private String password;
	private String name;
	private String phone;
	private String email;
	
}

```


lombok의 @Getter, @Setter, @ToString과 함께 빌더패턴을 사용하기 위해 @Builder도 domain클래스에 선언해줬다 그리고 가장 중요한 @Entity 를 적용해서 User.class가 Entity 객체임을 명시해준다. @AllArgsConstructor 는 User.class의 필드를 전부 매개변수로 받는 생성자고 @NoArgsConstructor는 기본생성자다 @NoArgsConstructor 인 기본생성자가 Entity클래스에 없으면 Jpa에서 기본생성자가 없다고 에러가나니 조심하자

<br>



이제 실제 화면에서 넘어온 User 객체를 db에 저장하는 비지니스 로직을 작성한다. UserRepository interface는 Jpa에서 제공하는 CrudRepository를 사용하여 기본적인 Crud기능에 이름으로 찾기, email로 찾기 등의 기능을 확장한다.



\- UserRepository.interface

```java
package com.example.demo.repositories;

import org.springframework.data.repository.CrudRepository;

import com.example.demo.model.User;

public interface UserRepository extends CrudRepository<User, Long> {

	User findByName(String name);

	User findByPhone(String phone);
	
	User findByEmail(String email);
	
}



```




각각 이름, 전화번호, email로 User Entity를 db에서 꺼내오는 부분이다.

<br>




간단하게 테스트해보자

\- UserRepositoryTest.class

```java
package com.example.demo.repositories;

import static org.junit.Assert.assertEquals;

import org.junit.Before;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

import com.example.demo.model.User;	

@SpringBootTest
@RunWith(SpringRunner.class)
public class UserRepositoryTest {
	
	@Autowired
	private UserRepository userRepository;
	
	@Before
	public void test_유저를_db에_넣기() {
		User user = User.builder()
				.id("sup2is")
				.name("sup2")
				.password("qwer!23")
				.phone("010-0000-0000")
				.email("dev.sup2is@gmail.com")
				.build();
		
		//when
		userRepository.save(user);
	}
}

```

먼저 테스트 데이터를 위한 User Object를 위해 @Before로 더미데이터를 셋업한다. 우리는 사실 userRepository.save() 메서드를 작성한 적이 없지만 상속받은 CrudRepository에 save() 이외에도 여러가지 메서드가 있으니 반드시 document를 확인하는게 좋을 것 같다.



<br>

findByName(), findByPhone, findByEmail을 테스트한다.

\- UserRepositoryTest.class

```java
	@Test
	public void test_유저를_db에_저장후_name값으로_찾기() {
		
		//given
		
		//when
		User dbUser = userRepository.findByName("sup2");
		
		//then
		assertEquals("sup2is", dbUser.getId());
		System.out.println(dbUser.toString());
	}
	@Test
	public void test_유저를_db에_저장후_phone값으로_찾기() {
		
		//given
		
		//when
		User dbUser = userRepository.findByPhone("010-0000-0000");
		
		//then
		assertEquals("sup2is", dbUser.getId());
		System.out.println(dbUser.toString());
	}
	
	@Test
	public void test_유저를_db에_저장후_email값으로_찾기() {
		
		//given
		
		//when
		User dbUser = userRepository.findByEmail("dev.sup2is@gmail.com");
		
		//then
		assertEquals("sup2is", dbUser.getId());
		System.out.println(dbUser.toString());
	}
```



결과는 성공으로 떨어진다. 재미있는 사실은 우리는 UserRepository를 따로 Bean으로 등록하지 않았는데 @Autowired로 bean주입을 받는다. jpa에서 제공하는 Repository interface를 상속받으면 Spring bean으로 알아서 올려준다 마치 @Service, @Repository를 사용한 것과 동일하다. (legacy는 따로 jpa:repositories base-package 를 설정해주는듯? boot는 그냥 적용됨 ..)



<br>

너무 String 값만 불러오는 느낌이 있어서  gmail, naver 등의 사이트로 분리해서 User를 찾아오는 메서드도 하나 만들어봤다.



<br>



\- UserRepository.interface

```java
List<User> findByEmailContaining(String site);

@Query("select u from User u where u.email like %:site%")
List<User> findByEmailContaining(@Param("site") String site);
```



<br>



같은 메서드지만 하나는 Jpa에서 제공하는 "Containing"을 사용한 검색을 했고 나머지 하나는 네이티브 쿼리로 날려봤다





\- UserRepositoryTest.class

```java
	@Test
	public void test_유저의_email을_site별로_검색하기() {
		
		//given
		User user = User.builder()
				.id("chlcc")
				.name("chlcc")
				.password("qwer!23")
				.phone("010-1111-1111")
				.email("dev.sup2is@naver.com")
				.build();
		
		userRepository.save(user);
		
		//when
		List<User> users = userRepository.findByEmailContaining("naver");
		
		//then
		assertEquals(1, users.size());
		System.out.println(users.toString());
		
		
	}
```



결과는 성공

<br>



사실 화면을 붙여서 실제 db에 insert되고 select하는거까지 테스트하려고했으나 ... 너무 길어질꺼같아서 그냥 controller에서 api형식으로 불러들이는 방법으로 예제를 테스트해보겠다 !! 

<br>



\- ApiController.class

```java
package com.example.demo.controller;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import com.example.demo.model.User;
import com.example.demo.repositories.UserRepository;

@RestController
@RequestMapping("/api")
public class ApiController {

	@Autowired
	private UserRepository userRepository;

	@PostMapping("/add")
	public String save(@RequestBody User user) {
		try {
			userRepository.save(user);
			return "success";
		}catch (Exception e) {
			e.printStackTrace();
			return "failed";
		}
	}
}

```



postman을 사용하여 controller에 access한다. 그냥 간단하게 /api/add 로 요청시 db에 넘어온 user 객체가 들어가게만 한다.







![1](https://user-images.githubusercontent.com/30790184/58395277-5c0d7380-8082-11e9-9775-337d24af112d.png)





<br>



![2](https://user-images.githubusercontent.com/30790184/58395280-5d3ea080-8082-11e9-9b1d-36ed51ee49c8.png)



잘 들어갔다. 단순 select를 위해 더미데이터를 몇개 더 추가해준다.

\- ApiContoller.class

```java
	@GetMapping("/users")
	public List<User> findAll() {
		List<User> users = new ArrayList<>();
		try {
			Iterable<User> it = userRepository.findAll();
			for (User user : it) {
				users.add(user);
			}
			return users;
		}catch (Exception e) {
			e.printStackTrace();
			return users;
		}
	}
	
	
	@GetMapping("/user/containing/{site}")
	public List<User> findByEmailContaining(@PathVariable("site")String site) {
		List<User> users = new ArrayList<>();
		try {
			Iterable<User> it = userRepository.findByEmailContaining(site);
			for (User user : it) {
				users.add(user);
			}
			return users;
		}catch (Exception e) {
			e.printStackTrace();
			return users;
		}
	}
```



다해보면 좋겠지만 두가지만 추가해본다. 하나는 CrudRepository의 findAll과 내가 작성한 findByEmailContaining 을 테스트해본다.



![3](https://user-images.githubusercontent.com/30790184/58395887-464d7d80-8085-11e9-91e8-7caac24c3488.png)





<br>

![4](https://user-images.githubusercontent.com/30790184/58395888-477eaa80-8085-11e9-91c0-f2794451ed6e.png)



<br>

findAll에는 내가추가해준 data가 전부나오고 /user/containing에서는 파라미터로 넘어오는 site가 포함된 email을 갖고 있는 user객체들을 성공적으로 불러오는 모습이다





사실 mybatis 이외에 다른 lib나 framework를 도입한적이없는데 Jpa를 써본 느낌은 굉장히 편할꺼같은 느낌이다 실제로 쿼리를 작성한적이 없는데 db에서 기본이되는 crud를 간단한 메서드명으로도 지원해주니 말이다. 근데 현업에서는 어떻게 사용할 지 조금 궁금한 생각도 든다 현업에서는 단순히 crud 이외의 복잡한 쿼리를 어떻게 관리하는지도 한번 알아봐야겠다

또 하나 더 느낀점은 Jpa는 java의 vo(jpa에서의 entity 또는 domain)와 db table간의 모델링이 굉장히 중요한듯하다. 역시 결국은 설계를 잘해야 프로그래밍을 잘하는듯 ...



<br>

포스팅은 여기까지 하겠습니다.  모든예제는 제 github에서 확인하실 수 있습니다.

예제 : <https://github.com/sup2is/spring-example/tree/master/springframework-4>



<br>

퍼가실때는 링크와 출처를 반드시 명시해주세요. 감사합니다.

<br>







출처 : <https://docs.spring.io/spring-data/jpa/docs/current/reference/html/>

출처 : <https://www.javaworld.com/article/3379043/what-is-jpa-introduction-to-the-java-persistence-api.html>

출처 : <https://en.wikibooks.org/wiki/Java_Persistence/What_is_JPA%3F>

출처 : <https://arahansa.github.io/docs_spring/jpa.html#repositories.core-concepts>

출처 : <https://projectjt.tistory.com/4>

출처 : <https://m.blog.naver.com/PostView.nhn?blogId=dimigozzang&logNo=220510288775&proxyReferer=https%3A%2F%2Fwww.google.com%2F>
출처 : <https://spring.io/guides/gs/accessing-data-jpa/>


