---
layout: post
title: "Spring Security의 ExceptionTranslationFilter와 AuthenticationEntryPoint"
tags: [Spring Framework, Spring Security, ExceptionTranslationFilter, AuthenticationEntryPoint]
comments: true
date: 2020-03-10
parent: Security
grand_parent: Spring
---



<br>

# ExceptionTranslationFilter

**ExceptionTranslationFilter**은 Filter Chain 내부에서 발생하는 **AccessDeniedException**과 **AuthenticationException**의 처리를 담당한다. 이 필터는 Java 의 exception과 HTTP의 bridge 역할을 해주기때문에 반드시 필요하다. 이 **ExceptionTranslationFilter**은 실제 보안을 수행하지는 않고 interface 를 맞추기 위해 존재한다.

<br>

만약 **ExceptionTranslationFilter**가 **AuthenticationException**을 감지했다면 **authenticationEntryPoint**를 실행한다. 이것을통해서 **AbstractSecurityInterceptor**의 서브클래스에서 발생하는 인증 실패를 공동으로 처리할 수 있다.

<br>

만약 **ExceptionTranslationFilter**가 **AuthenticationException**을 감지했다면 해당 사용자가 익명 사용자 인지의 여부를 판별하고 만약 익명 사용자일 경우 동일하게 **authenticationEntryPoint**을 실행한다. 만약 익명 사용자가 아닐 경우엔 **AccessDeniedHandler**을 위임하는데 기본 설정으로 **AccessDeniedHandlerImpl**을 사용하기때문에 추가 설정이 있지 않는 이상은 그냥 사용하면 된다.



<br>

# AuthenticationEntryPoint

만약 filter 인증에서 공통적으로 **AuthenticationException**에 대한 처리가 필요하다면  **AuthenticationEntryPoint**을 구현하여 공통 클래스로 생성이 가능하다

아래는 실제 Security 설정과 **AuthenticationEntryPoint**을 구현하는 부분을 추가해봤다



**SecurityConfig.java**

```java
@RequiredArgsConstructor
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    private final AuthenticationEntryPoint jwtAuthenticationEntryPoint;

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
            .authorizeRequests()
                .antMatchers(...).permitAll()
                .anyRequest().authenticated()
            .and()
                .exceptionHandling()
                .authenticationEntryPoint(jwtAuthenticationEntryPoint)
    }
```

<br>



**JwtAuthenticationEntryPoint.java**

```java
import org.springframework.security.core.AuthenticationException;
import org.springframework.security.web.AuthenticationEntryPoint;
import org.springframework.stereotype.Component;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

@Component
public class JwtAuthenticationEntryPoint implements AuthenticationEntryPoint {

    @Override
    public void commence(HttpServletRequest request,
                         HttpServletResponse response,
                         AuthenticationException e) throws IOException, ServletException {

        response.sendError(HttpServletResponse.SC_UNAUTHORIZED, "UnAuthorized");
    }

}
```

<br>

이런식으로 설정해놓으면 인가된 사용자가 아닐 경우엔**AuthenticationException** 을 탐지하여 client에게 **401 UnAuthorized** 을 return할 수 있도록 설정 가능하다.



<br>

포스팅은 여기까지 하겠습니다.  

<br

퍼가실때는 링크와 출처를 반드시 명시해주세요. 감사합니다.

<br>



**References**

-   https://docs.spring.io/spring-security/site/docs/current/api/org/springframework/security/web/access/ExceptionTranslationFilter.html
-   https://docs.spring.io/spring-security/site/docs/current/api/org/springframework/security/web/AuthenticationEntryPoint.html