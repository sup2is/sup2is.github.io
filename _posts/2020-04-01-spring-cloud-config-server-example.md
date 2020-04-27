---
layout: post
title: "Spring Cloud Config Server로 Configuration 관리하기"
tags: [Spring Boot, Spring Cloud, Spring Cloud Config]
date: 2020-04-01
comments: true
grand_parent: Spring
parent: Cloud
---



<br>

# OverView

Spring Cloud에서는 수십 또는 수백가지의 어플리케이션들의 Configuration을 관리할 수 있도록 도와주는 **Spring Cloud Config Server**가 존재한다. 이번시간에 간단한 예제와 함께 Spring Cloud 기반의 Config Server를 구성하는 방법을 알아보자!



<br>

# Why?

보통의 웹 어플리케이션이라면 어플리케이션과 Configuration은 함께 배포되는게 맞다. 하지만 MSA의 관점에서는 수십 또는 수백가지의 어플리케이션들의 Configuration들이 각각 개별 어플리케이션 배포에 포함되어있다면 추후 유지보수가 굉장히 힘들어질 것이다. 따라서 Spring Cloud는 Spring Cloud Config Server를 통해서 Configuration 중앙 집중화를 구현한다. Spring Cloud Config Server는 각각 어플리케이션들의 Configuration들을 효율적으로 관리할 수 있도록 도와준다.



<br>

# Spring Cloud Config Server 작성하기

먼저 Spring Cloud Config Server를 작성한다 pom.xml은 다음과 같이 설정한다.

**pom.xml**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>

    <artifactId>config-server</artifactId>
    <groupId>me.sup2is</groupId>
    <version>1.0-SNAPSHOT</version>
    <name>demo</name>
    <description>Demo project for Spring Boot</description>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.0.3.RELEASE</version>
    </parent>

    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-config-server</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-config</artifactId>
        </dependency>
    </dependencies>
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>Finchley.RELEASE</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
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

**spring-cloud-config-server** 모듈과 **spring-cloud-starter-config**을 추가해주고 **dependencyManagement** 태그를 이용하여 **org.springframework.cloud** 모듈들의 버전들을 통합시켜준다.

<br>

이후에 추가되는 부분은 **ConfigurationServerApplication.java** 파일인데 설정할게 거의 없다. 사실 Spring Boot가 전부 알아서해준다 주목할점은 **@EnableConfigServer**이다.

**ConfigurationServerApplication.java**

```java
package me.sup2is.configserver;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.config.server.EnableConfigServer;

@SpringBootApplication
@EnableConfigServer
public class ConfigurationServerApplication {

    public static void main(String[] args) {
        SpringApplication.run(ConfigurationServerApplication.class, args);
    }
}

```

**@EnableConfigServer** 어노테이션은 Spring 공홈에도 자세한 역할을 제대로 찾지는 못해서 정확하게 이런 어노테이션이다라고 설명은 불가능하나 아무래도 Spring Config Server 관련한 부분을 Spring Boot가 자동으로 설정해주도록 도와주는 어노테이션이지 않을까 싶다. 자동으로 설정해주는 부분은 아래 실습에서 확인할 수 있다.

<br>

놀랍게도 Config Server의 java 코드는 위 설정만으로도 충분하다. 그럼 실제 어플리케이션의 설정들은 어디에 존재할까? 바로 GitHub에 존재한다. Spring Cloud Config Server는 GitHub과 통합이 가능하기때문에 Configuration 파일들도 쉽게 형상관리가 가능하다. 물론 절대경로 등의 파일설정도 가능하다.

<br>

**application.yml**

```properties
server:
  port: 8888
spring:
  cloud:
    config:
      server:
        git:
          uri: https://github.com/sup2is/config-repo
          searchPaths: fooservice
```

마지막으로 application.yml에 **spring.cloud.config.server.git**에 대한 설정을 해준다 나같은경우는 미리 준비해놓은 repository를 바라보도록 설정했다. 실제 서비스의 이름이 될 **fooservice**를 설정해 놓았는데 n개의 서비스가 존재한다면 뒤에 `,`를 활용해서 추가하면된다. 실제 파일들의 내용이나 구조를 확인하고싶다면 [config-repo](https://github.com/sup2is/config-repo) 에서 확인할 수 있다.

<br>

간단하게 fooservice의 dev profile 설정의 내용을 확인해 보자

**fooservice-dev.yml**

```properties
example.property: "this is dev profile"
spring.database.driverClassName: "org.h2.Driver"
spring.datasource.url: "jdbc:h2:mem:"
spring.datasource.username: "dev_name"
spring.datasource.password: "password"
```

example.property 라는 사용자 설정과 spring에서 사용하는 spring.database.driverClassName 등등의 다양한 설정이 가능하다. 아래 fooservice는 실제로 위 Configuration을 이용하여 설정하도록 구성할 것이다.



<br>

설정이 만약 제대로 됐다는 가정하에 실제 Spring Cloud Config Server를 동작시키고 postman을 이용하여 **http://127.0.0.1:8088//fooservice/dev** 로 http요청을 날려보자



![주석 2020-04-01 215551](https://user-images.githubusercontent.com/30790184/78141864-128e7180-7467-11ea-8e42-fb0646bb0725.png)

놀랍게도 다음과 같이 추가적인 코드 작성 없이도 Spring Cloud Config Server가 알아서 service에 해당하는 endpoint를 알아서 만들어준다. 사용법은 **http://127.0.0.1:8088/{service-name}/{profile}** 이다. 

default 설정은 **http://127.0.0.1:8088/{service-name}/default**를 이용하여 가져올 수 있다.

<br>

위에서 살펴본 설정 외에도 다른 endpoint를 Spring Boot에서 자동으로 만들어주는데 사용법을 알고싶다면 Spring Boot가 최초에 로딩될때 로그를 보면 다음과 같은 endpoint를 만들어주는것을 확인할 수 있다.

![주석 2020-04-01 221947](https://user-images.githubusercontent.com/30790184/78141873-13bf9e80-7467-11ea-82bc-ee8dfb01397b.png)

<br>



# Spring Cloud Config Server를 사용하는 Application 만들기



이제 다음과 같이 Spring Cloud Config Server가 작성되었다면 실제 어플리케이션이 로딩되는 시점에 Spring Cloud Config Server와 통신하여 Configuration 설정을 하도록 구성해보자!



**pom.xml**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.0.3.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>me.sup2is</groupId>
    <version>1.0-SNAPSHOT</version>
    <artifactId>fooservice</artifactId>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-config</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-config-client</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
    </dependencies>
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>Finchley.RELEASE</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
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

별다른 설정은 없고  **spring-cloud-starter-config** 모듈과 **spring-cloud-config-client**모듈을 추가해줬다. 추가로 **spring-boot-starter-actuator** 모듈을 추가해줬는데 아래에서 **@RefreshScope**사용을 위해 추가했다 @RefreshScope은 아래에서 소개한다.

<br>



**FooServiceApplication.java**

```java
package me.sup2is.fooservice;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.context.config.annotation.RefreshScope;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@SpringBootApplication
@RestController
@RefreshScope
public class FooServiceApplication {

    public static void main(String[] args) {
        SpringApplication.run(FooServiceApplication.class, args);
    }

    @Value("${example.property}")
    private String exampleProperty;

    @GetMapping("/my-property")
    public String getProperty() {
        return exampleProperty;
    }

}

```



위에 있는 FooServiceApplication 클래스는 Spring Cloud Config Server에게 받은 Configuration에서 **${example.property}**의 key를 가진 값을 불러오도록 설정했다. /my-property로 요청했을때 그 값을 반환하도록 설정했다.



<br>

**application.yml**

```properties
spring:
  application:
    name: fooservice
  profiles:
    active:
      default
  cloud:
    config:
      uri: http://127.0.0.1:8888


management:
  endpoints:
    web:
      exposure:
        include: refresh
```

위 설정에는 spring config server가 해당 서비스를 인식할 수 있도록 **spring.application.name**에 현재 서비스의 이름인 **fooservice**를 입력했다. 이후에 profiles 속성도 dev나 default 등의 다양한 설정이 가능하다. 마지막으로 이전에 설정해 놓은 Spring Cloud Config Server와 매핑 될 수 있도록 **http://127.0.0.1:8888**을 설정해 놓았다.

추가적으로 **management.endpoints:web:exposure.include**로 refresh를 설정해 놓았는데 FooServiceApplication 클래스에 설정해놓은 **@RefreshScope**와 연관이 매우 깊다. 이렇게 설정해놓고 만약 ${example.property} 같은 사용자 정의 프로퍼티값에 변경이 필요하다면 github에 있는 configuration repository에서 변경 후 POST로 http://127.0.0.1:8080/actuator/refresh로 요청하면 변경사항이 어플리케이션의 재시작 없이 자동적으로 이루어진다. 하지만 이런 기능은 사용자 정의 프로퍼티만 가능하며 데이터베이스설정이나 스프링 데이터에서 정의된 내용은 반영되지 않는다.

<br>



# Example

위에 설정해놓은 상태로 Spring Cloud Config Server를 구동시켜놓고 fooservice를 동작시키면 아래와 같이 어플리케이션이 최초 로딩되는 시점에 Spring Cloud Config Server와 통신하여 Configuration을 가져온다.

![주석 2020-04-02 144058](https://user-images.githubusercontent.com/30790184/78214351-01d60e00-74f0-11ea-9ed5-3d31a8ed91dd.png)

<br>





![주석 2020-04-16 170423](https://user-images.githubusercontent.com/30790184/79430734-5a370080-8004-11ea-9b86-845a816a9b68.png)





<br>

포스팅은 여기까지 하겠습니다.  모든예제는 제 github에서 확인하실 수 있습니다.

예제 : https://github.com/sup2is/spring-example/tree/master/spring-cloud-config-example



<br>

References

-  Spring 마이크로서비스 코딩 공작소 -존 카넬 (길벗출판사)
-  https://spring.io/projects/spring-cloud-config#overview

