---
layout: post
title: "QueryDSL JPA 알아보기 Feat.Spring Data"
tags: [Spring Boot, JPA, QueryDSL]
date: 2020-08-12
comments: true
---



<br>

# OverView

이번시간에는 QueryDSL JPA에 대해서 알아보도록 하겠다.



# JPA와 동적쿼리

JPA에는 동적쿼리를 사용하는 방법이 몇가지 있다. 

- JPQL 사용하기
- Criteria api 사용하기
- Native Query 사용하기
- QueryDSL 사용하기

**JPQL, Native Query**

- 애플리케이션 로딩 시점에 타입체크가 가능하지만 컴파일 시점에 타입 체크가 불가능함

**Criteria API**

- JPQL과 Native Query 보다 컴파일 타임 오류와 동적 쿼리를 비교적 안전하게 생성해줌
- api가 장황하고 복잡함

**QueryDSL**

- 컴파일 타임 오류 체크 가능
- 동적쿼리를 Criteria API보다 직관적으로 표현 가능
- JPA가 공식적으로 지원하지는 않음 따라서 별도의 의존성을 추가해야함

# QueryDSL - JPA

QueryDSL은 앞에서 설명한 것처럼 JPA 표준이 아니기 때문에 별도로 라이브러리를 추가해주어야한다.

```xml
<dependency>
  <groupId>com.mysema.querydsl</groupId>
  <artifactId>querydsl-apt</artifactId>
  <version>${querydsl.version}</version>
  <scope>provided</scope>
</dependency>    
    
<dependency>
  <groupId>com.mysema.querydsl</groupId>
  <artifactId>querydsl-jpa</artifactId>
  <version>${querydsl.version}</version>
</dependency>
```

<br>

보통 QueryDSL이라 하면 JPA만 생각할 수 있는데 현재까지 생각보다 개발된 부분이 많이 있다. mongodb, lucene 등등 다양한 언어 및 라이브러리를 지원하고 있다.

> QueryDSL github : [https://github.com/querydsl/querydsl](https://github.com/querydsl/querydsl)

<br>

친절하게 번역된 한글 문서도 존재하기 때문에 참고가 필요하다면 [http://www.querydsl.com/static/querydsl/3.4.0/reference/ko-KR/html_single/](http://www.querydsl.com/static/querydsl/3.4.0/reference/ko-KR/html_single/) 에서 확인할 수 있다.

여기에서는 튜토리얼과 일반 사용법 등등 다양한 QueryDSL 활용법이 있다.

<br>

QueryDSL을 사용하기 전에 먼저 Q라는 접두사가 붙은 쿼리타입을 생성해야 한다. 예를 들어 `Member` 라는 엔티티가 있을 경우 `QMember`라는 쿼리타입을 사용해서 쿼리를 생성해야 한다.



## Intellij에서 쿼리타입을 생성하는 방법

maven 기준으로 설명한다. 먼저 apt 플러그인을 설정한다.

```xml
    <build>
        <plugins>
            <plugin>
                <groupId>com.mysema.maven</groupId>
                <artifactId>apt-maven-plugin</artifactId>
                <version>1.1.3</version>
                <executions>
                    <execution>
                        <goals>
                            <goal>process</goal>
                        </goals>
                        <configuration>
                            <outputDirectory>target/generated-sources/java</outputDirectory>
                            <processor>com.querydsl.apt.jpa.JPAAnnotationProcessor</processor>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
```

<br>

이후에 `mvn compile` 과 같은 빌드관리 도구의 빌드를 실행시키면 target/generated-sources/java 디렉토리에 `Q{엔티티명}.java` 가 생성된다. **JPAAnnotationProcessor는 javax.persistence.Entity 애너테이션을 가진 도메인 타입을 찾아서 쿼리 타입을 생성한다.**

<br>

intellij에서는 Project Strucure라는 도구에서 Sources 디렉토리로 설정해야 쿼리타입을 ide에서 활용할 수 있게 해준다.

Project Strucure는 `File -> Product Structure` 또는 window 기준 `Ctrl + Alt + Shift + S` 에서 확인할 수 있다.

![20201020_104900](https://user-images.githubusercontent.com/30790184/96530343-378ee880-12c2-11eb-81c4-926259666b64.png)

위와 같이 target/generated-sources/java 디렉토리를 Sources 타입으로 설정하면 ide에서 쿼리타입을 사용할 수 있다.

# Example

예제에서 사용할 엔티티는 다음과 같다

-  `Team` (1) : (N) `Member`
- 팀은 여러명의 회원을 가질 수 있다.

예제는 spring boot + maven을 사용했다.

**pom.xml**

```xml


...

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
            <exclusions>
                <exclusion>
                    <groupId>org.junit.vintage</groupId>
                    <artifactId>junit-vintage-engine</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <dependency>
            <groupId>com.querydsl</groupId>
            <artifactId>querydsl-jpa</artifactId>
            <version>4.1.4</version>
        </dependency>
        <dependency>
            <groupId>com.querydsl</groupId>
            <artifactId>querydsl-apt</artifactId>
            <version>4.1.4</version>
            <scope>provided</scope>
        </dependency>
        <dependency>
            <groupId>com.h2database</groupId>
            <artifactId>h2</artifactId>
            <version>1.4.200</version>
            <scope>test</scope>
        </dependency>


    </dependencies>
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
            <plugin>
                <groupId>com.mysema.maven</groupId>
                <artifactId>apt-maven-plugin</artifactId>
                <version>1.1.3</version>
                <executions>
                    <execution>
                        <goals>
                            <goal>process</goal>
                        </goals>
                        <configuration>
                            <outputDirectory>target/generated-sources/java</outputDirectory>
                            <processor>com.querydsl.apt.jpa.JPAAnnotationProcessor</processor>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>

```

<br>

## Where

실제 쿼리를 만들때는 `JPAQueryFactory` 클래스를 활용해서 질의한다. 기본적인 사용법은 아래와 같다.

```java
    @PersistenceContext
    private EntityManager em;

...
	@Test
    public void basic_test() {
        //given
        JPAQueryFactory jpaQueryFactory = new JPAQueryFactory(em);
        memberRepository.save(Member.createMember("choi", "서울시 강남구"));

        //when
        Member findMember = jpaQueryFactory.selectFrom(QMember.member)
                .where(QMember.member.name.eq("choi"))
                .fetchOne();

        //then
        assertNotNull(findMember);
    }

...
```

`JPAQueryFactory` 생성 시점에 `EntityManager` 객체를 주입해줘야하는데 `EntityManager` 객체는 `@PersistenceContext` 를 활용해서 spring bean을 통해 주입받을 수 있다.

위 테스트로 실제 전송되는 db 쿼리는 다음과 같다

```sql
    select
        member0_.member_id as member_i1_0_,
        member0_.address as address2_0_,
        member0_.name as name3_0_ 
    from
        member member0_ 
    where
        member0_.name=?
```

<br>

where() 메서드는 다음과 같이 두개 이상의 조건을 넣을 수 있다.

```java
        //when
        List<Member> fetch = jpaQueryFactory.selectFrom(QMember.member)
                .where(QMember.member.name.eq("choi"),
                        QMember.member.address.contains("강남구"))
                .fetch();
```

위 코드는 아래의 sql로 변환되어 db에 전달된다

```sql
    select
        member0_.member_id as member_i1_0_,
        member0_.address as address2_0_,
        member0_.name as name3_0_ 
    from
        member member0_ 
    where
        member0_.name=? 
        and (
            member0_.address like ? escape '!'
        )
```





<br>

## Join

join도 비교적 손쉽게 가능하다.

```java
    @Test
    public void join() {
        //given
        JPAQueryFactory jpaQueryFactory = new JPAQueryFactory(em);
        Member choi = Member.createMember("choi", "서울시 강남구");
        Member woo = Member.createMember("woo", "서울시 강남구");
        memberRepository.save(choi);
        memberRepository.save(woo);

        Team teamA = Team.createTeam("A team");
        Team teamB = Team.createTeam("B team");
        teamRepository.save(teamA);
        teamRepository.save(teamB);
        teamA.addMember(choi);
        teamA.addMember(woo);

        //when
        List<?> fetch = jpaQueryFactory.from(QMember.member)
                .join(QMember.member.team, QTeam.team)
                .where(QMember.member.address.eq("서울시 강남구"))
                .fetch();

        //then
        assertEquals(2, fetch.size());
    }
```

위 코드는 아래의 sql로 변환되어 db에 전달된다

```sql
    select
        member0_.member_id as member_i1_0_,
        member0_.address as address2_0_,
        member0_.name as name3_0_,
        member0_.team_id as team_id4_0_ 
    from
        member member0_ 
    inner join
        team team1_ 
            on member0_.team_id=team1_.team_id 
    where
        member0_.address=?
```

QueryDSL은 이너 조인, 조인, 레프트 조인, 풀조인을 지원한다. 물론 테이블 3개를 한번에 조인하는것도 가능하다.

## Order

정렬도 아주 손쉽게 가능하다.

```java
@Test
public void ordering() {
    //given
    JPAQueryFactory jpaQueryFactory = new JPAQueryFactory(em);
    Member choi = Member.createMember("choi", "서울시 강남구");
    Member woo = Member.createMember("woo", "서울시 강남구");
    Member kim = Member.createMember("kim", "서울시 서초구");
    memberRepository.save(choi);
    memberRepository.save(woo);
    memberRepository.save(kim);

    //when
    List<?> fetch = jpaQueryFactory.from(QMember.member)
            .orderBy(QMember.member.name.asc())
            .fetch();

    //then
    assertEquals(3, fetch.size());
    assertEquals(choi, fetch.get(0));
}
```
위 코드는 아래의 sql로 변환되어 db에 전달된다

```sql
    select
        member0_.member_id as member_i1_0_,
        member0_.address as address2_0_,
        member0_.name as name3_0_,
        member0_.team_id as team_id4_0_ 
    from
        member member0_ 
    order by
        member0_.name asc
```

<br>

## SubQuery

서브쿼리가 필요할 때는 `JPAExpressions` 클래스를 사용해서 질의하면 된다. (아직 한국 문서에는 `JPASubQuery` 를 사용하라고 되어 있는데 버전업이 되면서 `JPAExpressions`를 사용해야 한다.)

```java
    @Test
    public void sub_query() {
        //given
        JPAQueryFactory jpaQueryFactory = new JPAQueryFactory(em);
        Member choi = Member.createMember("choi", "서울시 강남구");
        Member woo = Member.createMember("woo", "서울시 강남구");
        Member kim = Member.createMember("kim", "서울시 서초구");
        memberRepository.save(choi);
        memberRepository.save(woo);
        memberRepository.save(kim);

        //when
        List<?> fetch = jpaQueryFactory.from(QMember.member)
                .where(QMember.member.name.in(
                        JPAExpressions.select(QMember.member.name)
                        .from(QMember.member)
                ))
                .fetch();


        //then
        assertEquals(3, fetch.size());
    }
```

위 코드는 아래의 sql로 변환되어 db에 전달된다

```sql
    select
        member0_.member_id as member_i1_0_,
        member0_.address as address2_0_,
        member0_.name as name3_0_,
        member0_.team_id as team_id4_0_ 
    from
        member member0_ 
    where
        member0_.name in (
            select
                member1_.name 
            from
                member member1_
        )


```

> 별 쓸모없는 쿼리지만 sub query를 이런식으로 날릴 수 있다면 확인하면 된다.

# Spring에서 QueryDSL 사용하기

`JPAQueryFactory`를 활용해서 QueryDSL을 간단하게 사용 가능하지만 실제로 spring 에서 사용할때는 조금 더 구조화된 모습으로 사용한다. 이부분은 [https://jojoldu.tistory.com/372](https://jojoldu.tistory.com/372)를 많이 참조했다.

<br>

QueryDSL을 사용하기 위해서는 개발자가 직접 `JPAQueryFactory`를 사용해서 코드를 입력해야 한다. 그러나 기존에 JPA를 사용하던 방식은

```java
public interface MemberRepository extends JpaRepository<Member, Long> {
    List<Member> findByName(String name) // <- 이 메서드만 구현 불가능
}
```

JPA가 제공하는 Repositry를 확장하는 인터페이스 만들기 때문에 `MemberRepository`의 구현체를 만드는 순간 `JpaRepository`의 메서드 역시 전부 구현해야 하는 제약사항이 발생한다.

이런 경우에는 별도로 Custom 인터페이스를 구현해서 메서드를 확장하는 방법을 사용하면 된다.

**1.아래와 같이 Custom 인터페이스를 선언하고 필요한 메서드를 나열한다.**

```java
package me.sup2is.repository;

import me.sup2is.domain.Member;

public interface MemberRepositoryCustom {
    Member findByName(String name);
}

```

**2.위에서 선언한 인터페이스를 구현하는 구현체를 만든다**

```java
package me.sup2is.repository;

import com.querydsl.jpa.impl.JPAQueryFactory;
import lombok.RequiredArgsConstructor;
import me.sup2is.domain.Member;
import me.sup2is.domain.QMember;

@RequiredArgsConstructor
public class MemberRepositoryImpl implements MemberRepositoryCustom {
    private final JPAQueryFactory jpaQueryFactory;

    @Override
    public Member findByName(String name) {
        return jpaQueryFactory.selectFrom(QMember.member)
                .where(QMember.member.name.eq("choi"))
                .fetchOne();
    }
}

```

**3.기존에 사용하던 Repository에 Custom 인터페이스를 확장한다**

```java
package me.sup2is.repository;

import me.sup2is.domain.Member;
import org.springframework.data.jpa.repository.JpaRepository;

public interface MemberRepository extends JpaRepository<Member, Long>, MemberRepositoryCustom {
}

```

위와 같은 구조로 사용하면 `MemberRepository` 만 사용해도 별도로 구현한 QueryDSL의 구현체 메서드를 이용할 수 있다.

```java
package me.sup2is.repository;

import me.sup2is.domain.Member;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.transaction.annotation.Transactional;

import static org.junit.jupiter.api.Assertions.assertEquals;

@SpringBootTest
@Transactional
class MemberRepositoryTest {

    @Autowired
    MemberRepository memberRepository;

    @Test
    public void find_by_name() {
        //given
        Member member = Member.createMember("choi", "서울시 강남구");
        memberRepository.save(member);

        //when
        Member findMember = memberRepository.findByName("choi");

        //then
        assertEquals(member, findMember);
    }

}
```



<br>

# 마무리

QueryDSL을 사용해서 동적쿼리를 보다 조금 더 쉽게 사용하고 ide 자동완성 기능, 컴파일 타임 체크와 같은 기능을 사용해봤다. 실무에 적용한다면 아주 좋은 성과를 이룰 수 있을 것 같다.



<br>

***

포스팅은 여기까지 하겠습니다. 퍼가실때는 출처를 반드시 남겨주세요!

예제: [https://github.com/sup2is/study/tree/master/query-dsl/query-dsl-jpa-exam](https://github.com/sup2is/study/tree/master/query-dsl/query-dsl-jpa-exam)

<br>

**References**

- [https://www.baeldung.com/intro-to-querydsl](https://www.baeldung.com/intro-to-querydsl)
- 자바 표준 JPA 프로그래밍 (김영한)
- [http://www.querydsl.com/static/querydsl/latest/reference/html/](http://www.querydsl.com/static/querydsl/latest/reference/html/)
- [https://jojoldu.tistory.com/372](https://jojoldu.tistory.com/372)