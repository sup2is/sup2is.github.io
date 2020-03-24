---
layout: post
title: "Spring Batch 맛보기"
tags: [Spring Boot, Spring Batch]
date: 2020-03-24
comments: true
parent: Spring
exclude: true
---



<br>

# OverView

보통 대용량 벌크작업이나 반복되는 작업이 필요할 때 **배치**라는 단어를 많이 사용한다. **Spring Batch Project** 역시 마찬가지다 로깅, 트랜잭션 관리 등등의 **대량의 레코드를 처리하는데 필수적인 재사용 가능한 기능을 제공해준다.** 그리고 현업에서는 생각보다 이런 배치작업이 비일비재하게 일어나는편이라 알고 있으면 굉장히 편리할 것이다. 이 글은 [spring.io](https://spring.io/projects/spring-batch#overview) 에 있는 batch example을 보고 작성한 글이라서 간단한 맛보기 예제가 될 것 같다.

그럼 시작!

<br>



먼저 간단한 시나리오 먼저 작성해보자

1. debug.log 에는 DEBUG, INFO, ERROR 라는 세개의 상태를 갖는 상태 메세지가 있다 예를 들면 아래와 같은 형식이다.

```
DEBUG : Bla Bla... 
INFO : Bla Bla...
WANR : Bla Bla...
```

2. DB에는 system_log라는 테이블이 있고 status, message 라는 컬럼을 갖는다.
3. Spring batch를 사용해서 debug.log의 형식을 데이터화 시켜서 DB로 관리한다.

<br>

위 시나리오를 토대로 간단한 spring batch 프로그램을 작성해보자!



# Example (Spring Boot 2.2.5)

예제는 역시 Spring Boot 로 진행한다. https://start.spring.io/ 

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
	<groupId>me.sup2is</groupId>
	<artifactId>spring-batch-example</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<name>spring-batch-example</name>
	<description>Demo project for Spring Boot</description>

	<properties>
		<java.version>1.8</java.version>
	</properties>

	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-batch</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-devtools</artifactId>
			<scope>runtime</scope>
			<optional>true</optional>
		</dependency>
		<dependency>
			<groupId>com.h2database</groupId>
			<artifactId>h2</artifactId>
			<scope>runtime</scope>
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
			<groupId>org.springframework.batch</groupId>
			<artifactId>spring-batch-test</artifactId>
			<scope>test</scope>
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



database는 h2 inmemory db를 사용한다!



<br>

먼저 객체지향의 세계에서는 log라는 논리적인 개념도 객체로 변환할 수 있으니 Log라는 클래스를 생성해보자

**Log.java**

```java
package me.sup2is.logbatch;

import lombok.Data;

@Data
public class Log {
    private Status status;
    private String message;
}

enum Status{
    DEBUG, INFO, ERROR
}
```

Log는 status와 message를 갖고 status는 DEBUG, INFO, ERROR를 갖도록 설계했다.

<br>

그리고 실제 log 데이터도 함께 생성해주자

```
DEBUG : Lorem Ipsum is simply dummy text of the printing and typesetting industry 1.
DEBUG : Lorem Ipsum is simply dummy text of the printing and typesetting industry 2.
DEBUG : Lorem Ipsum is simply dummy text of the printing and typesetting industry 3.
DEBUG : Lorem Ipsum is simply dummy text of the printing and typesetting industry 4.
DEBUG : Lorem Ipsum is simply dummy text of the printing and typesetting industry 5.
ERROR : Application throws Exception !!
ERROR : Application throws Exception !!
INFO : Lorem ipsum dolor sit amet 1.
INFO : Lorem ipsum dolor sit amet 2.
INFO : Lorem ipsum dolor sit amet 3.
INFO : Lorem ipsum dolor sit amet 4.
INFO : Lorem ipsum dolor sit amet 5.
ERROR : Application throws Exception !!
ERROR : Application throws Exception !!
ERROR : Application throws Exception !!

...

```

현업이라면 비교도안될만큼 간단하지만 감잡기엔 충분하다고 생각한다 ....

<br>

spring batch에는 database, file 등등을 읽거나 쓸 수 있는 **ItemReader**와 **ItemWriter**들이 존재한다. 그리고 그 사이에 존재하는 **ItemProcessor**를 통해 읽어들인 데이터에 비지니스 로직을 삽입할 수 있다. 간단하게 개념만 알아보자

## ItemReader

spring batch의 ItemReader 다양한 유형의 입력에서 데이터를 제공해주는 interface가 된다.

- Flat File : 플랫 파일에서 일반적으로 파일의 고정 위치에 의해 정의되거나 일부 특수 문자 (예 : 쉼표)로 구분 된 데이터 필드가있는 레코드를 설명하는 데이터 행을 읽을 수 있다.
- XML : XML ItemReader는 객체 구문 분석, 매핑 및 유효성 검사에 사용되는 기술과 독립적으로 XML을 처리한다. 입력 데이터를 사용하면 XSD 스키마에 대한 XML 파일의 유효성을 검증 할 수 있다.
- DataBase : 데이터베이스 리소스에 액세스하여 처리 할 개체에 매핑 할 수있는 결과 집합을 반환한다. 기본 SQL ItemReader 구현은 RowMapper를 호출하여 오브젝트를 리턴하고, 재시작이 필요한 경우 현재 행을 추적하고, 기본 통계를 저장하며, 나중에 설명 할 트랜잭션 개선 사항을 제공한다.

> ```java
> public interface ItemReader<T> {
> 
>     T read() throws Exception, UnexpectedInputException, ParseException, NonTransientResourceException;
> 
> }
> ```

read()를 통해 하나의 항목을 반환하거나 더이상 읽어올것이 없으면 null을 반환한다. 우리가 위에 만들어놓은 Log 객체와 매핑된다

<br>

## ItemWriter

spring batch의 ItemWriter는 ItemReader와 유사하지만 주로 쓰는작업에 사용한다. Database에 insert하거나 update할 수 있다.

> ```java
> public interface ItemWriter<T> {
> 
>     void write(List<? extends T> items) throws Exception;
> 
> }
> ```

## ItemProcessor

실제 비지니스로직이 들어가는 부분은 **ItemProcessor**다





<br>

포스팅은 여기까지 하겠습니다.  모든예제는 제 github에서 확인하실 수 있습니다.

예제 :


<br>

References

- 
