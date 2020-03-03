---
layout: post
title: "Spring Security로 Database 로그인 구현하기"
tags: [Spring Framework, Spring Security, Login, Database]
comments: true
date: 2019-03-03
parent: Security
grand_parent: Spring
---



<br>

[이전](https://sup2is.github.io/docs/spring/spring-security-login-exam.html)글에서 InMemory 기반의 Spring Security Login 기능을 구현했었다.

이번 글에서는 실제 Database와 연동한 Security 로그인 기능을 구성해보자 !

※이 글은 Spring Boot 기반으로 작성됐다.



# Example (Spring Boot 2.1.5)

이 예제에 간단한 시나리오 먼저 구성해보면

1. **JPA를 사용한 Member Entity 구성**
2. **Rest API를 사용해서 Member를 등록**
3. **등록된 Member 기반으로 Spring Security Login 구성**

정도가 될 것 같다. DB는 가볍게 **InMemory 기반 H2**를 사용하고 html같은경우는 [이전](https://sup2is.github.io/docs/spring/spring-security-login-exam.html)글 에서 사용한 템플릿을 그대로 사용하도록 하겠다.



먼저 간단하게 RestAPI를 구성해서 테스트용 **Member** 데이터를 몇가지 넣어주는 작업을 하도록 하겠다.

<br>

**pom.xml**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.2.5.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>me.sup2is.securitylogin2</groupId>
    <artifactId>securitylogin2</artifactId>
    <version>1.0-SNAPSHOT</version>
    <name>securitylogin2</name>


    <properties>
        <java.version>1.8</java.version>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-security</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-thymeleaf</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>com.h2database</groupId>
            <artifactId>h2</artifactId>
            <scope>runtime</scope>
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
            <groupId>org.springframework.security</groupId>
            <artifactId>spring-security-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <scope>provided</scope>
        </dependency>
    </dependencies>
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>

```



<br>

**Member.java**

```java
package me.sup2is.securitylogin2.member;

import lombok.*;

import javax.persistence.*;

@Entity
@Getter
@ToString
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class Member {

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    @Column(name = "member_id")
    private Long id;

    @Column(unique = true)
    private String email;
    private String password;

    public Member(String email, String password) {
        this.email = email;
        this.password = password;
    }

    public static Member createMember(String email, String password) {
        return new Member(email,password);
    }

}

```

<br>

**MemberController.java**

```java
package me.sup2is.securitylogin2.member;

import lombok.Data;
import lombok.RequiredArgsConstructor;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequiredArgsConstructor
public class MemberController {

    final MemberRepository memberRepository;
    final PasswordEncoder encode;

    @PostMapping("/api/member")
    public String saveMember(@RequestBody MemberDto memberDto) {
        memberRepository.save(Member.createMember(memberDto.getEmail(), encode.encode(memberDto.getPassword())));
        return memberRepository.findAll().toString();
    }

}

@Data
class MemberDto {
    private String email;
    private String password;
}

```

<br>

**MemberRepository.java**

```java
package me.sup2is.securitylogin2.member;

import org.springframework.data.repository.CrudRepository;

import java.util.Optional;

public interface MemberRepository extends CrudRepository<Member, Long> {
    Optional<Member> findByEmail(String email);
}

```



api와 jpa 관련은 위 설정으로 최소화하여 진행했다.

다음은 실제 spring security와 관련된 부분의 설정이다.



**MemberDetailService.java**

```java
package me.sup2is.securitylogin2.member;

import lombok.RequiredArgsConstructor;
import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.authority.SimpleGrantedAuthority;
import org.springframework.security.core.userdetails.User;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.core.userdetails.UsernameNotFoundException;
import org.springframework.stereotype.Service;

import java.util.HashSet;
import java.util.Set;

@Service
@RequiredArgsConstructor
public class MemberDetailService implements UserDetailsService {

    final MemberRepository memberRepository;

    @Override
    public UserDetails loadUserByUsername(String email) throws UsernameNotFoundException {
        Member member = memberRepository.findByEmail(email)
                .orElseThrow(() -> new UsernameNotFoundException(email));
        Set<GrantedAuthority> grantedAuthorities = new HashSet<>();
        grantedAuthorities.add(new SimpleGrantedAuthority(Role.USER.getValue()));
        if (email.equals("sup2is@gmail.com")) {
            grantedAuthorities.add(new SimpleGrantedAuthority(Role.ADMIN.getValue()));
        }

        return new User(member.getEmail(), member.getPassword(), grantedAuthorities);
    }
}

```



이 **UserDetailService**가 login 관련해서 핵심적인 부분이라고 생각하는데 **UserDetailService**는 사용자의 인증 및 권한 부여 정보를 검색하는데 사용되는 Spring Security 의 핵심 interface이다. 나같은경우는 **MemberDetailService** 클래스를 새로 만들어서 email 기반의 로그인 기능을 구현했다.



이렇게 커스텀 된 **UserDetailService**를 사용하기 위해서는 다음과 같은 설정이 필요하다. 





**SecurityConfig.java**

```java

@EnableWebSecurity
@RequiredArgsConstructor
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    final DataSource dataSource;
    final MemberDetailService memberDetailService;

...

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.userDetailsService(memberDetailService).passwordEncoder(passwordEncoder());
        auth.jdbcAuthentication().dataSource(dataSource);
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }

...

```



**WebSecurityConfigurerAdapter**의 메서드중 하나인 **configure(AuthenticationManagerBuilder auth)** 의  **AuthenticationManagerBuilder** 에 앞에서 **UserDetailService**를 구현한 **MemberDetailService**를 넣어줘야한다.

```java
auth.userDetailsService(memberDetailService).passwordEncoder(passwordEncoder());
```

<br>

마찬가지로 **DataSource**도 등록하여 database에 접근 가능하도록 설정해줘야한다.

간단한 database에 로그인은 위 설정만으로도 가능하다.

![녹화_2020_03_03_15_20_52_437](https://user-images.githubusercontent.com/30790184/75748840-c0710800-5d63-11ea-9b66-cbfd5a767c9b.gif)





<br>

포스팅은 여기까지 하겠습니다.  모든예제는 제 github에서 확인하실 수 있습니다.

예제 : https://github.com/sup2is/spring-example/tree/master/spring-security-login-with-database



<br>

퍼가실때는 링크와 출처를 반드시 명시해주세요. 감사합니다.

<br>





**References**

-  https://www.baeldung.com/spring-boot-configure-data-source-programmatic 
-  https://www.baeldung.com/spring-security-jdbc-authentication 