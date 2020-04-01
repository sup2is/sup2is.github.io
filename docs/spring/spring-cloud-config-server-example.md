---
layout: post
title: "Spring Cloud Config Server로 Configuration 관리하기 #1"
tags: [Spring Boot, Spring Cloud, Spring Cloud Config]
date: 2020-04-01
comments: true
parent: Spring
---



<br>

# OverView

Spring Cloud에서는 수십 또는 수백가지의 어플리케이션들의 Configuration을 관리할 수 있도록 도와주는 **Spring Cloud Config Server**가 존재한다. 이번시간에 간단한 예제와 함께 Spring Cloud 기반의 Config Server를 구성하는 방법을 알아보자!



<br>

# Why?

보통의 웹 어플리케이션이라면 어플리케이션과 Configuration은 함께 배포되는게 맞다. 하지만 MSA의 관점에서는 수십 또는 수백가지의 어플리케이션들의 Configuration들이 각각 개별 어플리케이션 배포에 포함되어있다면 추후 유지보수가 굉장히 힘들어질 것이다. 따라서 Spring Cloud는 Spring Cloud Config Server를 통해서 Configuration 중앙 집중화를 통하여 각각 어플리케이션들의 Configuration들을 효율적으로 관리할 수 있도록 도와준다.



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

이후에 추가되는 부분은 **ConfigurationServerApplication.java** 파일인데 설정할게 거의 없다. 사실 Spring Boot가 전부 알아서해준다 주목할점은 **@EnableConfigServer**이다.

<br>

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

**@EnableConfigServer** Spring 공홈에도 자세한 역할을 제대로 찾지는 못해서 정확하게 이런 어노테이션이다라고 설명은 불가능하나 아무래도 Spring Config Server 관련한 부분을 Spring Boot가 자동으로 설정해주도록 도와주는 어노테이션이지 않을까 싶다.

<br>

놀랍게도 Config Server의 java 코드는 위 설정만으로도 충분하다. 그럼 실제 어플리케이션의 설정들은 어디에 존재할까? 바로 GitHub에 존재한다. Spring Cloud Config Server는 GitHub과 통합이 가능하기때문에 Configuration 파일들도 쉽게 형상관리가 가능하다.

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

마지막으로 application.yml에 **spring.cloud.config.server.git**에 대한 설정을 해준다 나같은경우는 미리 준비해놓은 repository를 바라보도록 설정했다. 실제 서비스의 이름이 될 **fooservice**를 설정해 놓았다 n개의 서비스가 존재한다면 뒤에 `,`를 활용해서 추가하면된다. 실제 [config-repo](https://github.com/sup2is/config-repo) 에서 확인할 수 있다.

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

<br>

설정이 만약 제대로 됐다는 가정하에 실제 Spring Cloud Config Server를 동작시키고 postman을 이용하여 http요청을 날려보자

![주석 2020-04-01 215551](https://user-images.githubusercontent.com/30790184/78141864-128e7180-7467-11ea-8e42-fb0646bb0725.png)

놀랍게도 다음과 같이 추가적인 코드 작성 없이도 Spring Cloud Config Server가 알아서 service에 해당하는 endpoint를 알아서 만들어준다. 사용법은 **/{service-name}/{profile}** 이다. 

default 설정은 **/{service-name}/default**를 이용하여 가져올 수 있다.

<br>

만약 추가적인 사용법을 알고싶다면 Spring Boot가 최초에 로딩될때 로그를 보면 다음과 같은 endpoint를 만들어주는것을 확인할 수 있다.

![주석 2020-04-01 221947](https://user-images.githubusercontent.com/30790184/78141873-13bf9e80-7467-11ea-82bc-ee8dfb01397b.png)

<br>

아래는 그냥 재미삼아 /actuator도 한번 요청해봤다.



![주석 2020-04-01 215726](https://user-images.githubusercontent.com/30790184/78141869-13bf9e80-7467-11ea-8515-2533726dcdcd.png)





다음시간에는 이제 fooservice를 작성해서 최초에 어플리케이션 로딩시점에 config server와 통신하여 어플리케이션을 설정하는 방법과 config server에서 configuration이 변경되었을때 변경사항을 적용시키는 방법에 대해서 알아보도록 하겠다.





<br>

포스팅은 여기까지 하겠습니다.  모든예제는 제 github에서 확인하실 수 있습니다.

예제 : https://github.com/sup2is/spring-example/tree/master/spring-batch-example 



<br>

References

-  https://docs.spring.io/spring-batch/docs/4.2.x/reference/html/readersAndWriters.html#readersAndWriters 
-  https://spring.io/projects/spring-batch#learn 

