---
layout: post
title: "Spring Security로 InMemory 로그인 구현하기"
tags: [Spring Framework, Spring Security, Login]
comments: true
date: 2019-03-02
parent: Spring
---



<br>

블로그를 바꾸고 아마 처음 쓰는 포스팅인것같다.

최근 토이프로젝트에서 Spring Security를 사용하기로해서 간단하게 로그인 구현을 해보자



※이 글은 Spring Boot 기반으로 작성됐다.



# Spring Security

Spring Security는 Spring 기반 Application의 보안을 위한 Spring 표준 framework이다. Spring Security는 매우 강력하고 커스터마이징이 가능하다.



Spring Security는 아래와 같은 특징을 갖고 있다.

- 인증 및 권한 부여에 대한 포괄적이고 확장 가능한 지원
-  session  fixation, clickjacking, cross site request forgery 같은 공격으로부터 의 보호
- Servlet API 통합
- Spring Web MVC와의 Optional한 통합
- 그외 등등 ..



사실 Security 자체만 놓고 보더라도 굉장히 많은 내용이 있어서 이 글에서 전부 다루기는 쉽지 않을것이다. 이 글은 전적으로 간단한 로그인만 구현한다.

<br>

# Example (Spring Boot 2.2.5)





**pom.xml**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.2.5.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>me.sup2is.securitylogin</groupId>
    <artifactId>demo</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>demo</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-security</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
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
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-thymeleaf</artifactId>
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



역시 Spring Boot 답게 아무것도 설정하지 않고 spring-starter-security 모듈을 포함하고 실행하면 다음과 같은 화면이 나온다. 이 예제는 아래의 로그인 폼을 활용한다.

<br>

![주석 2020-03-02 110811](https://user-images.githubusercontent.com/30790184/75647936-ea59fa00-5c91-11ea-8751-148d9f2b4dad.png)

<br>

이제 로그인 절차를 위한 SecurityConfig를 설정해보자

**SecurityConfig.java**

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
 
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.inMemoryAuthentication()
                .withUser("sup2is")
            	.password(passwordEncoder().encode("sup2is")).roles("USER")
                .and()
                .withUser("superuser")
                .password(passwordEncoder().encode("superuser"))
                .roles("USER").roles("ADMIN");
    }
    
    
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
    
    ...
```

<br>

먼저 코드에 사용된 `@EnableWebSecurity`와 `WebSecurityConfigurerAdapter`를 설명한다.

 @EnableWebSecurity annotation과 WebSecurityConfigurerAdapter을 같이 사용하면 웹 기반의 보안을 제공하게 해준다.WebSecurityConfigurerAdapter를 통해서 다음과 같은 설정이 가능하다



- Application 내부 URL에 접근하기 이전에 user의 인증이 필요하도록 설정 가능
- 이름이 "user", 비밀번호가 "password" 인 User를 생성가능
- HTTP 기본 및 Form 기반 인증



따라서 위에서 설명한것처럼 `WebSecurityConfigurerAdapter`의  **configure(final AuthenticationManagerBuilder auth)** 메서드를 통해 inMemory 기반의 user를 생성한다.

나의 경우 일반 유저는 sup2is로 생성하고 admin 유저는 그대로 admin으로 생성했다.



<br>

이어서 실제 매핑되는 url에 security 설정을 마저 하도록 하겠다.

**SecurityConfig.java**

```java
...

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
            .authorizeRequests()
                .antMatchers("/home").hasRole("USER")
                .antMatchers("/admin").hasRole("ADMIN")
                .antMatchers("/", "/index").permitAll()
                .anyRequest().authenticated()
            .and()
                .formLogin()
//            .loginPage("/login.html") <- Spring Boot 기본 제공
                .defaultSuccessUrl("/home", true)
            .and()
                .logout()
            .and()
                .exceptionHandling().accessDeniedPage("/403");
    }


...
```

<br>

실제 url mapping 관련한 security 처리는 `WebSecurityConfigurerAdapter`의 **configure(HttpSecurity http)** 메서드 재정의를 통해서 이루어진다

간단하게 기능에 대해 정리하면

-  **.authorizeRequests()**는 **/home** 경로에 **USER** 라는 **Role** 을 가진사람만 접근 가능하고 **/admin** 경로에는 **ADMIN Role** 을 가진사람만 접근 가능, **/index**는 누구나 접근 가능
- **.formLogin()**을 통한 로그인 기능, **.loginPage**의 경우 따로 설정된 view가 없으면 Spring Boot가 제공하는 기본 view를 제공, 만약 성공했으면 **/home**으로 리턴
- **.logout()**로그아웃 기능, 이것 또한 Spring Boot가 제공하는 **/logout** 과 기본 logout 설정 사용 가능
- **.exceptionHandling()** 만약 권한이 없는 사용자가 있다면 **/403** 으로 리턴

사실 이것 외에도 굉장히 많은 설정이 있는편이고 필요한 기능에따라 적절하게 찾아서 사용하는걸 추천한다.

<br>

지금 위에서 설정한 **SecurityConfig.class**는 아래의 xml 설정을 대체한다 (실제 대체는 아니고 아래 처럼 사용했었다는 의미임 .. )

```xml
...
<http use-expressions="true">
    <intercept-url pattern="/login*" access="isAnonymous()" />
    <intercept-url pattern="/**" access="isAuthenticated()"/>
 
    <form-login login-page='/login.html'
      default-target-url="/homepage.html"
      authentication-failure-url="/login.html?error=true" />
    <logout logout-success-url="/login.html" />
</http>
 
<authentication-manager>
    <authentication-provider>
        <user-service>
            <user name="user1" password="user1Pass" authorities="ROLE_USER" />
        </user-service>
        <password-encoder ref="encoder" />
    </authentication-provider>
</authentication-manager>

...
```

<br>

마지막으로 Controller로 실제 매핑될 view를 이어주자

**HomeController.java**

```java
package me.sup2is.securitylogin;

import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;

import java.security.Principal;

@Controller
public class HomeController {

    @GetMapping(value = {"/", "/index"})
    public String index(){
        return "index";
    }

    @GetMapping("/home")
    public String home(Principal principal, Model model){
        model.addAttribute("user", principal.getName());
        model.addAttribute("roles", ((UsernamePasswordAuthenticationToken) principal).getAuthorities());
        return "/home";
    }

    @GetMapping("/admin")
    public String admin(Principal principal, Model model){
        model.addAttribute("user", principal.getName());
        model.addAttribute("roles", ((UsernamePasswordAuthenticationToken) principal).getAuthorities());
        return "/admin";
    }

    @GetMapping("/403")
    public String forbidden() {
        return "/403";
    }

}

```



 위와 같이 Controller 내부에서 **java.security.Principal** 을 파라미터로 받으면 현재 세션의 User를 얻어올 수 있으니 참고 바란다. 아래는 위 소스를 기반으로 실행한 예제이다.

<br>



![녹화_2020_03_02_14_21_22_54](https://user-images.githubusercontent.com/30790184/75647930-e4fcaf80-5c91-11ea-9f54-9749c0bd8f35.gif)





<br>

포스팅은 여기까지 하겠습니다.  모든예제는 제 github에서 확인하실 수 있습니다.

예제 : https://github.com/sup2is/spring-example/tree/master/spring-security-login-exam



<br>

퍼가실때는 링크와 출처를 반드시 명시해주세요. 감사합니다.

<br>





**References**

-  https://www.baeldung.com/java-config-spring-security 
-  https://spring.io/projects/spring-security 
-  https://spring.io/guides/topicals/spring-security-architecture 
-  https://spring.io/blog/2013/07/03/spring-security-java-config-preview-web-security#wsca 