---
layout: post
title: "Spring Security + JWT로 Token기반 Security Login 구현하기"
tags: [Spring Framework, Spring Security, Login, JWT]
comments: true
date: 2020-03-05
parent: Security
grand_parent: Spring
---



<br>

토이프로젝트에 필요한 Spring Security + JWT 스펙을 활용해서 request 마다 token 값을 검증하여 RestAPI에 보안 기능을 넣어보자!



# JWT

먼저 JWT에 대한 간단한 설명부터 진행한다.  [jwt.io](https://jwt.io/introduction/) 에서는 JWT를 다음과 같이 소개한다

<br>

> ## What is JSON Web Token?
>
> JSON Web Token (JWT) is an open standard ([RFC 7519](https://tools.ietf.org/html/rfc7519)) that defines a compact and self-contained way for securely transmitting information between parties as a JSON object. This information can be verified and trusted because it is digitally signed. JWTs can be signed using a secret (with the **HMAC** algorithm) or a public/private key pair using **RSA** or **ECDSA**.
>
> Although JWTs can be encrypted to also provide secrecy between parties, we will focus on *signed* tokens. Signed tokens can verify the *integrity* of the claims contained within it, while encrypted tokens *hide* those claims from other parties. When tokens are signed using public/private key pairs, the signature also certifies that only the party holding the private key is the one that signed it.
>
> 
>
> ## When should you use JSON Web Tokens?
>
> Here are some scenarios where JSON Web Tokens are useful:
>
> - **Authorization**: This is the most common scenario for using JWT. Once the user is logged in, each subsequent request will include the JWT, allowing the user to access routes, services, and resources that are permitted with that token. Single Sign On is a feature that widely uses JWT nowadays, because of its small overhead and its ability to be easily used across different domains.
> - **Information Exchange**: JSON Web Tokens are a good way of securely transmitting information between parties. Because JWTs can be signed—for example, using public/private key pairs—you can be sure the senders are who they say they are. Additionally, as the signature is calculated using the header and the payload, you can also verify that the content hasn't been tampered with.

<br>

간단하게 설명하면 **Header**, **Payload**, **Signature** 를 포함한 JSON 타입의 self-contained 한 암호화 서명이다. 이 JWT에 대한 내용을 자세하게 더 알고 싶다면  [jwt.io](https://jwt.io/introduction/)를 참고하자

<br>

보통의 login처리 프로세스를 잠깐 살펴보자. 

backend와 frontend가 하나의 어플리케이션으로 올라가있을때 form login 방식으로 화면에서 작성된 유저정보를 server에 전달하고 인증절차를 거친 뒤에 해당 정보가 올바른 유저정보라면 브라우저 세션에 해당 유저정보를 넣어주는 방식이다.

<br>

근데 최근에는 backend와 frontend가 분리된 환경을 많이 구축하곤 하는데 이 경우에는 세션을 공유하지 않기때문에 현재 접속한 유저가 인증된 유저인지 알 방법이 없다. ~~세션을 공유하는 방법은 있는듯 함~~ 이때 사용하기 좋은게 바로 JWT다.

위에서 설명한것처럼 **self-contained** 한 암호화 서명이기때문에 만약 로그인 기능이 필요하다면 최초에 로그인시점에 JWT토큰을 서버에게 발급받고 이후에 클라이언트가 인증이 필요한부분에 접근하려고할때 항상 JWT토큰을 요청과 함께보내면 ~~토큰이 털리지 않는 이상~~ 해당 유저가 인증된 유저임을 보장 가능하다.

<br>

# Example (Spring Boot 2.1.5)



시나리오는 다음과 같다

1. **/api/member**를 통해 로그인에 사용할 **email**과 **password**를 실시간 RestAPI로 저장한다
2. **/authenticate**를 통해 1번에서 저장한 email과 password를 이용하여 인증요청을 보낸다
   1. 인증이 성공된다면 **인증 요청한 email이 들어간 token발급**
   2. 인증이 실패되면 **401 UnAhtorized** 리턴
3. 반환된 token을 기반으로 **/hello**로 요청한다.
4. **/hello**를 보내기 이전에 token에 대한 유효성검사와 **userdetailsservice**를  태운다
5. 모든 인증절차가 끝나면 "Hello world"라는 메세지를 전달한다.

<br>



db는 간단하게 **InMemory h2** 를 사용한다

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

    <groupId>me.sup2is.jwtsecurity</groupId>
    <artifactId>jwtsecurity</artifactId>
    <version>1.0-SNAPSHOT</version>

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
            <groupId>org.springframework.security</groupId>
            <artifactId>spring-security-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>io.jsonwebtoken</groupId>
            <artifactId>jjwt</artifactId>
            <version>0.9.1</version>
        </dependency>
        <dependency>
            <groupId>javax.xml.bind</groupId>
            <artifactId>jaxb-api</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
        <dependency>
            <groupId>com.h2database</groupId>
            <artifactId>h2</artifactId>
        </dependency>
    </dependencies>
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <configuration>
                    <source>10</source>
                    <target>10</target>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

별다른 특이점은 없는데 **javax.xml.bind** 관련해서 에러가 났었다

관련된 내용은 [여기](https://java.ihoney.pe.kr/521)에서 확인할 수 있다.

<br>

이 프로젝트에서 사용하는 전체 패키지 구조는 다음과 같다.

내용이 꽤 많아보이긴 하는데 차근차근 설명해보도록 하겠다.



![img1](https://user-images.githubusercontent.com/30790184/75956292-eed51b80-5efa-11ea-8553-71504ccf5c86.png)



<br>



먼저 **Spring Security** 설정 먼저 해보도록 하자

**WebSecurityConfig.java**

```java
package me.sup2is.jwtsecurity.config;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.authentication.AuthenticationProvider;
import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
import org.springframework.security.config.http.SessionCreationPolicy;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.security.web.authentication.UsernamePasswordAuthenticationFilter;

import javax.sql.DataSource;

@Configuration
@EnableWebSecurity
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {

    @Autowired
    private JwtAuthenticationEntryPoint jwtAuthenticationEntryPoint;

    @Autowired
    private UserDetailsService userDetailsService;

    @Autowired
    private JwtRequestFilter jwtRequestFilter;

    @Autowired
    private DataSource dataSource;

    @Autowired
    private AuthenticationProvider authenticationProvider;

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {

        auth.jdbcAuthentication().dataSource(dataSource);
        auth.authenticationProvider(authenticationProvider);
        auth
                .userDetailsService(userDetailsService)
                .passwordEncoder(passwordEncoder());
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
                .csrf().disable()
                .authorizeRequests().antMatchers("/authenticate", "/api/member").permitAll()
                .anyRequest().authenticated()
                .and()
                .exceptionHandling()
                .authenticationEntryPoint(jwtAuthenticationEntryPoint)
                .and()
                .sessionManagement()
                .sessionCreationPolicy(SessionCreationPolicy.STATELESS);

        http.addFilterBefore(jwtRequestFilter, UsernamePasswordAuthenticationFilter.class);
    }
}

```



몇가지만 살펴보면

- .exceptionHandling().authenticationEntryPoint(jwtAuthenticationEntryPoint)  : 

  \- exceptionHandling을 위해서 실제 구현한 **jwtAuthenticationEntryPoint**을 넣어준다 **jwtAuthenticationEntryPoint** 아래에서 구현한다.

- .sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS)

  \- Spring Security에서 Session을 생성하거나 사용하지 않도록 설정한다.

- auth.authenticationProvider(jwtUserDetailsService)

  \- **jwtUserDetailsService** 역시 실제로 구현해서 사용한다. 구현은 아래에 있다.

- http.addFilterBefore(jwtRequestFilter, UsernamePasswordAuthenticationFilter.class);

  \- **jwtRequestFilter**역시 아래에서 구현한다

<br>



**JwtAuthenticationEntryPoint.java**

```java
package me.sup2is.jwtsecurity.config;

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

이 **JwtAuthenticationEntryPoint**는 **AuthenticationEntryPoint**를 구현하여 인증에 실패한 사용자의 response에  **HttpServletResponse.SC_UNAUTHORIZED**를 담아주도록 구현한다.

<br>



**JwtUserDetailsService.java**

```java
package me.sup2is.jwtsecurity.service;

import me.sup2is.jwtsecurity.member.Member;
import me.sup2is.jwtsecurity.member.MemberRepository;
import me.sup2is.jwtsecurity.member.Role;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.authority.SimpleGrantedAuthority;
import org.springframework.security.core.userdetails.User;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.core.userdetails.UsernameNotFoundException;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.stereotype.Service;

import java.util.HashSet;
import java.util.Set;

@Service
public class JwtUserDetailsService implements UserDetailsService {

    @Autowired
    private PasswordEncoder encoder;

    @Autowired
    private MemberRepository memberRepository;

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

**JwtUserDetailsService**는 들어온 **email**으로 Member를 찾아서 결과적으로 User 객체를 반환해주는 역할을 한다.

<br>

**JwtTokenUtil.java**

```java
package me.sup2is.jwtsecurity.config;

import io.jsonwebtoken.Claims;
import io.jsonwebtoken.Jwts;
import io.jsonwebtoken.SignatureAlgorithm;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.stereotype.Component;

import java.util.Date;
import java.util.HashMap;
import java.util.Map;
import java.util.function.Function;

@Component
public class JwtTokenUtil {

    private static final String secret = "jwtpassword";

    public static final long JWT_TOKEN_VALIDITY = 5 * 60 * 60;

    public String getUsernameFromToken(String token) {
        return getClaimFromToken(token, Claims::getSubject);
    }

    public <T> T getClaimFromToken(String token, Function<Claims, T> claimsResolver) {
        final Claims claims = getAllClaimsFromToken(token);
        return claimsResolver.apply(claims);
    }

    private Claims getAllClaimsFromToken(String token) {
        return Jwts.parser().setSigningKey(secret).parseClaimsJws(token).getBody();
    }

    private Boolean isTokenExpired(String token) {
        final Date expiration = getExpirationDateFromToken(token);
        return expiration.before(new Date());
    }

    public Date getExpirationDateFromToken(String token) {
        return getClaimFromToken(token, Claims::getExpiration);
    }

    public String generateToken(UserDetails userDetails) {
        Map<String, Object> claims = new HashMap<>();
        return doGenerateToken(claims, userDetails.getUsername());
    }

    private String doGenerateToken(Map<String, Object> claims, String username) {
        return Jwts.builder()
                .setClaims(claims)
                .setSubject(username)
                .setIssuedAt(new Date(System.currentTimeMillis()))
                .setExpiration(new Date(System.currentTimeMillis() + JWT_TOKEN_VALIDITY * 1000))
                .signWith(SignatureAlgorithm.HS512, secret)
                .compact();
    }

    public Boolean validateToken(String token, UserDetails userDetails) {
        final String username = getUsernameFromToken(token);
        return (username.equals(userDetails.getUsername())) && !isTokenExpired(token);
    }

}

```

JWT token구현에 핵심이 되는 클래스이다. 필드변수로 **secret**을 선언해놨는데 이 secret을 통해서 토큰을 검증하기때문에 실제 서비스에서 저런식으로 사용하면 안된다. 해당 클래스에서는 token을 발급하고 token에서 username을 추출하고 token의 유효성검사 처리를 한다. 실제 사용에서는 입맛에 맞추어 Expiration 시간을 바꾼다던지 필요한 데이터를 더 넣어주던지 처리하면 될 것이다.



<br>

**jwtRequestFilter.java** 

```java
package me.sup2is.jwtsecurity.config;

import io.jsonwebtoken.ExpiredJwtException;
import me.sup2is.jwtsecurity.service.JwtUserDetailsService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;
import org.springframework.security.core.context.SecurityContextHolder;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.web.authentication.WebAuthenticationDetailsSource;
import org.springframework.stereotype.Component;
import org.springframework.web.filter.OncePerRequestFilter;

import javax.servlet.FilterChain;
import javax.servlet.ServletException;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

@Component
public class JwtRequestFilter extends OncePerRequestFilter {

    @Autowired
    private JwtUserDetailsService jwtUserDetailService;

    @Autowired
    private JwtTokenUtil jwtTokenUtil;

    @Override
    protected void doFilterInternal(HttpServletRequest request,
                                    HttpServletResponse response,
                                    FilterChain filterChain) throws ServletException, IOException {

        final String requestTokenHeader = request.getHeader("Authorization");

        String username = null;
        String jwtToken = null;

        if (requestTokenHeader != null && requestTokenHeader.startsWith("Bearer ")) {
            jwtToken = requestTokenHeader.substring(7);
            try {
                username = jwtTokenUtil.getUsernameFromToken(jwtToken);
            } catch (IllegalArgumentException e) {
                System.out.println("Unable to get JWT Token");
            } catch (ExpiredJwtException e) {
                System.out.println("JWT Token has expired");
            }
        } else {
            logger.warn("JWT Token does not begin with Bearer String");
        }

        if(username != null && SecurityContextHolder.getContext().getAuthentication() == null) {
            UserDetails userDetails = this.jwtUserDetailService.loadUserByUsername(username);

            if(jwtTokenUtil.validateToken(jwtToken, userDetails)) {
                UsernamePasswordAuthenticationToken authenticationToken =
                        new UsernamePasswordAuthenticationToken(userDetails, null ,userDetails.getAuthorities());

                authenticationToken.setDetails(new WebAuthenticationDetailsSource().buildDetails(request));
                SecurityContextHolder.getContext().setAuthentication(authenticationToken);
            }
        }
        filterChain.doFilter(request,response);
    }
}

```

**jwtRequestFilter** 역시 중요한 부분중 하나인데 **OncePerRequestFilter** 를 상속받은 클래스로써 요청당 한번의 filter를 수행하도록 **doFilterInternal()** 메서드를 구현해주면 된다. 

로직에서는 헤더에서 **Authorization**값을 꺼내어 토큰을 검사하고 해당 유저가 실제 DB에 있는지 검사하는 등의 전반적인 인증처리를 여기서 진행한다.

<br>

**JwtAuthenticationProvider.java**

```java
package me.sup2is.jwtsecurity.config;

import me.sup2is.jwtsecurity.member.Member;
import me.sup2is.jwtsecurity.member.MemberRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.security.authentication.AuthenticationProvider;
import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;
import org.springframework.security.core.Authentication;
import org.springframework.security.core.AuthenticationException;
import org.springframework.security.core.userdetails.UsernameNotFoundException;
import org.springframework.stereotype.Component;

@Component
public class JwtAuthenticationProvider implements AuthenticationProvider {

    @Autowired
    private MemberRepository memberRepository;

    @Override
    public Authentication authenticate(Authentication authentication) throws AuthenticationException {
        String email = authentication.getName();
        String password = authentication.getCredentials().toString();

        Member member = memberRepository.findByEmail(email).orElseThrow(() -> new UsernameNotFoundException(email));

        if(!member.getPassword().equals(password)) {
            throw new RuntimeException("UnAuthorized");
        }
        return new UsernamePasswordAuthenticationToken(email, password);
    }

    @Override
    public boolean supports(Class<?> authentication) {
        return authentication.equals(UsernamePasswordAuthenticationToken.class);
    }
}

```

**JwtAuthenticationProvider** 에서는 **AuthenticationProvider**의 **authenticate()** 메서드를 구현하여 인증처리를 도와준다 실제 존재하는 email인지 확인하고 요청된 password와 대조하는 역할을 한다



<br>

**JwtAuthenticationController.java**

```java
package me.sup2is.jwtsecurity.controller;

import lombok.AllArgsConstructor;
import lombok.Data;
import me.sup2is.jwtsecurity.service.JwtUserDetailsService;
import me.sup2is.jwtsecurity.config.JwtTokenUtil;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.ResponseEntity;
import org.springframework.security.authentication.*;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.web.bind.annotation.CrossOrigin;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RestController;

@RestController
@CrossOrigin
public class JwtAuthenticationController {

    @Autowired
    private AuthenticationProvider authenticationProvider;

    @Autowired
    private JwtTokenUtil jwtTokenUtil;

    @Autowired
    private JwtUserDetailsService userDetailService;

    @Autowired
    private PasswordEncoder passwordEncoder;

    @PostMapping("/authenticate")
    public ResponseEntity<?> createAuthenticationToken(@RequestBody JwtRequest authenticationRequest) throws Exception {
        authenticate(authenticationRequest.getEmail(), authenticationRequest.getPassword());

        final UserDetails userDetails = userDetailService.loadUserByUsername(authenticationRequest.getEmail());
        final String token = jwtTokenUtil.generateToken(userDetails);

        return ResponseEntity.ok(new JwtResponse(token));
    }

    private void authenticate(String username, String password) throws Exception {
        try {
            authenticationProvider.authenticate(new UsernamePasswordAuthenticationToken(username,password));
        } catch (DisabledException e) {
            throw new Exception("USER_DISABLED", e);
        } catch (BadCredentialsException e) {
            throw new Exception("INVALID_CREDENTIALS", e);
        }
    }


}

@Data
class JwtRequest {

    private String email;
    private String password;

}

@Data
@AllArgsConstructor
class JwtResponse {

    private String token;

}


```

마지막으로 **JwtAuthenticationController** 실제 endpoint를 지정해주는 controller를 작성해준다.

※ 블로그를 작성하다보니 authenticationProvider에서 findMember()를 사용하고 다시 아래 userDetailService에서 findMember()를 사용하는 부분이 있다. 이부분은 추후 리팩토링...



<br>

이렇게 구성하면 다음과같이 동작 가능하다



![녹화_2020_03_05_16_00_01_846](https://user-images.githubusercontent.com/30790184/75956286-eaa8fe00-5efa-11ea-938b-d33e87a0c879.gif)





<br>

포스팅은 여기까지 하겠습니다.  모든예제는 제 github에서 확인하실 수 있습니다.

예제 : https://github.com/sup2is/spring-example/tree/master/spring-security-login-with-jwt



<br>

퍼가실때는 링크와 출처를 반드시 명시해주세요. 감사합니다.

<br>





**References**

-   https://dzone.com/articles/spring-boot-security-json-web-tokenjwt-hello-world 
-   https://jwt.io/ 
-   https://docs.spring.io/spring-security/site/docs/4.2.13.RELEASE/apidocs/