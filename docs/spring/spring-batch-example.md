---
layout: post
title: "Spring Batch 맛보기"
tags: [Spring Boot, Spring Batch]
date: 2020-03-24
comments: true
parent: Spring
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

위 시나리오를 토대로 간단한  Spring Batch 프로그램을 작성해보자!



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

 Spring Batch에는 database, file 등등을 읽거나 쓸 수 있는 **ItemReader**와 **ItemWriter**들이 존재한다. 그리고 그 사이에 존재하는 **ItemProcessor**를 통해 읽어들인 데이터에 비지니스 로직을 삽입할 수 있다. 간단하게 개념만 알아보자

## ItemReader

 Spring Batch의 ItemReader 다양한 유형의 입력에서 데이터를 제공해주는 interface가 된다.

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

 Spring Batch의 ItemWriter는 ItemReader와 유사하지만 주로 쓰는작업에 사용한다. Database에 insert하거나 update할 수 있다.

> ```java
> public interface ItemWriter<T> {
> 
>     void write(List<? extends T> items) throws Exception;
> 
> }
> ```

## ItemProcessor

실제 비지니스로직이 들어가는 부분은 **ItemProcessor**다

> ```java
> public interface ItemProcessor<I, O> {
> 
>     O process(I item) throws Exception;
> }
> ```

I타입의 객체가 들어오면 O타입으로 반환하는데 제공되는 객체가 반드시 동일하지 않아도 된다. ItemProcessor를 step에 직접 연결할 수 있지만 이번 예제에서는 간단하게 ItemProcessor를 연결하는 부분만 적용시켜본다.

<br>

## FieldSet

우리는 debug.log라는 파일을 읽어야하기 때문에 FieldSet이라는 개념에대해 먼저 숙지해야한다. FieldSet은  Spring Batch에서 파일 타입의 작업을 할 때 입력 또는 출력에 관계없이 가장 중요한 클래스가 된다. FieldSet은 파일에서 필드를 파인딩 할 수 있도록 하는 추상화된  Spring Batch의 interfcae이다. FieldSet은 JDBC의 ResultSet과 매우 유사한 형태로 사용이 가능하다

> ```java
> String[] tokens = new String[]{"foo", "1", "true"};
> FieldSet fs = new DefaultFieldSet(tokens);
> String name = fs.readString(0);
> int value = fs.readInt(1);
> boolean booleanValue = fs.readBoolean(2);
> ```

<br>

## FlatFileItemReader

플랫 파일은 최대 2차원 표 형식의 데이터를 포함하는 모든 유형의 파일이다.  Spring Batch에서는 플랫 파일을 읽기위한 Reader로 **FlatFileItemReader**를 사용한다. 이 FlatFileItemReader에서 가장 중요한 요소는 **Resource**와 **LineMapper**인데 아래에서 자세하게 설명한다.



## LineMapper

**LineMapper**는 문자열 라인을 객체로 변환하기 위해서 필요한 interfcae이다

> ```java
> public interface LineMapper<T> {
> 
>     T mapLine(String line, int lineNumber) throws Exception;
> 
> }
> ```

일단 읽어온 라인을 T타입으로 리턴해주는것만 기억하자

<br>

## LineTokenizer

FieldSet으로 변환해야하는 많은 플랫 파일 데이터 형식이 있을 수 있으므로 **LineTokenizer**를 이용해서 FieldSet으로 변환시켜서 사용할 수 있다.

> ```java
> public interface LineTokenizer {
> 
>     FieldSet tokenize(String line);
> 
> }
> ```

위 설명대로 읽어들인 라인을 FieldSet으로 반환시켜주는데 나중에 등장할 **FieldSetMapper**로 전달 할 수 있다.  Spring Batch에서는 기본적으로 세개의 LineTokenizer를 구현하고있다.

- DelimitedLineTokenizer: 레코드의 필드가 구분자에 의해 token화 된다. 기본적으로 `,` 를 사용하지만 `:` 이나 `;`을 사용할 수도 있다.
- FixedLengthTokenizer: 레코드의 필드가 각각 **fixed width**에 의해 정의된다. 각 필드의 너비는 각 레코드 유형애 데해 정의되어야 한다.
- PatternMatchingCompositeLineTokenizer: 패턴을 확인하여 특정 라인에서 사용할 tokenizer 목록중 어떤  LineTokenizer를 사용할 지 결정한다.

## FieldSetMapper

이 FieldSetMapper는 말그대로 mapper역할을 해 준다.

> ```java
> public interface FieldSetMapper<T> {
> 
>     T mapFieldSet(FieldSet fieldSet) throws BindException;
> 
> }
> ```

mapFieldSet을 사용하여 넘어온 FieldSet객체를 T타입으로 반환시켜서 사용하면 된다.

<br>

## DefaultLineMapper

이제 어느정도 숙지가 되었다면 아래의 클래스가 눈에 조금들어올것이다.

> ```java
> public class DefaultLineMapper<T> implements LineMapper<>, InitializingBean {
> 
>     private LineTokenizer tokenizer;
> 
>     private FieldSetMapper<T> fieldSetMapper;
> 
>     public T mapLine(String line, int lineNumber) throws Exception {
>         return fieldSetMapper.mapFieldSet(tokenizer.tokenize(line));
>     }
> 
>     public void setLineTokenizer(LineTokenizer tokenizer) {
>         this.tokenizer = tokenizer;
>     }
> 
>     public void setFieldSetMapper(FieldSetMapper<T> fieldSetMapper) {
>         this.fieldSetMapper = fieldSetMapper;
>     }
> }
> ```

Spring Batch에서 기본적으로 제공하는 **DefaultLineMapper** 를 보면 앞에서 설명한 LineTokenizer, FieldSetMapper, LineMapper 등등의 interfcae가 어떻게 활용되는지 확인할 수 있다. 이 DefaultLineMapper를 사용해서 앞에서 설명한 시나리오를 완성해보자





<hr>





## Batch Configuration 설정하기

**LogBatchConfiguration.java**

```java
import org.springframework.batch.core.Job;
import org.springframework.batch.core.Step;
import org.springframework.batch.core.configuration.annotation.EnableBatchProcessing;
import org.springframework.batch.core.configuration.annotation.JobBuilderFactory;
import org.springframework.batch.core.configuration.annotation.StepBuilderFactory;
import org.springframework.batch.core.launch.support.RunIdIncrementer;
import org.springframework.batch.item.ItemProcessor;
import org.springframework.batch.item.database.BeanPropertyItemSqlParameterSourceProvider;
import org.springframework.batch.item.database.JdbcBatchItemWriter;
import org.springframework.batch.item.database.builder.JdbcBatchItemWriterBuilder;
import org.springframework.batch.item.file.FlatFileItemReader;
import org.springframework.batch.item.file.builder.FlatFileItemReaderBuilder;
import org.springframework.batch.item.file.transform.DelimitedLineTokenizer;
import org.springframework.batch.item.file.transform.LineTokenizer;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.io.ClassPathResource;

import javax.sql.DataSource;

@Configuration
public class LogBatchConfiguration {

    @Bean
    public FlatFileItemReader<Log> reader() {
        return new FlatFileItemReaderBuilder<Log>()
                .name("logItemReader")
                .resource(new ClassPathResource("debug.log"))
                .lineTokenizer(delimitedLineTokenizer())
                .fieldSetMapper(new LogFieldSetMapper())
                .build();
    }

    @Bean
    public LineTokenizer tokenizer() {
        DelimitedLineTokenizer delimitedLineTokenizer = new DelimitedLineTokenizer();
        delimitedLineTokenizer.setDelimiter(":");
        return delimitedLineTokenizer;
    }

    @Bean
    public ItemProcessor<Log, Log> processor() {
        return new LogBatchItemProcessor();
    }

    @Bean
    public JdbcBatchItemWriter<Log> writer(DataSource dataSource) {
        return new JdbcBatchItemWriterBuilder<Log>()
                .itemSqlParameterSourceProvider(new BeanPropertyItemSqlParameterSourceProvider<>())
                .sql("INSERT INTO system_log " +
                        "(status, message) " +
                        "VALUES (?, ?)")
                .itemPreparedStatementSetter(new LogBatchItemPreparedStatementSetter())
                .dataSource(dataSource)
                .build();
    }
    
    ...


}
```

먼저 설정한 부분은 크게 4가지인데 `reader()` , `processor()`, `write()`이다. 하나씩 살펴보면

1. `reader()` : batch job에서 reader역할을 한다 나의 경우 플랫파일이기 때문에 FlatFileItemReader를 사용해서 설정했다. FlatFileItemReaderBuilder를 사용해서 FlatFileItemReader를 만드는데 `lineTokenizer()`를 사용해서 구분자가 `:` 인 DelimitedLineTokenizer를 넣어줬고 실제 Log타입으로 mapping해줄 LogFieldSetMapper를 `fieldSetMapper()`를 통해서 넣어줬다 

**LogFieldSetMapper.java**

```java
package me.sup2is.logbatch;

import org.springframework.batch.item.file.mapping.FieldSetMapper;
import org.springframework.batch.item.file.transform.FieldSet;
import org.springframework.validation.BindException;

public class LogFieldSetMapper implements FieldSetMapper<Log> {

    @Override
    public Log mapFieldSet(FieldSet fieldSet) throws BindException {
        Log log = new Log();
        log.setStatus(LogStatus.valueOf(fieldSet.readString(0)));
        log.setMessage(fieldSet.readString(1));
        return log;
    }
}

```

다음과 같이 LogFieldSetMapper를 생성해서 넘어오는 FieldSet을 Log 객체로 변환시켜서 리턴하도록 작성한다. FieldSetMapper를 구현했다.

<br>

2. `processor()` : 중재자가 되어줄 processor를 등록해준다. LogBatchItemProcessor 인스턴스를 등록해준다.

**LogBatchItemProcessor.java**

```java
package me.sup2is.logbatch;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.batch.item.ItemProcessor;

public class LogBatchItemProcessor implements ItemProcessor<Log, Log> {

    private static final Logger logger = LoggerFactory.getLogger(LogBatchItemProcessor.class);

    @Override
    public Log process(Log log) throws Exception {
        logger.info("this is LogBatchItemProcessor Log : " + log.getClass());
        return log;
    }
}

```

예제에서는 딱히 ItemProcessor의 역할을 할만한게 없어서 그냥 로그만찍어보도록 했다.

<br>

3.`writer()` : JdbcBatchItemWriter를 이용하여 batch writer를 작성했다. 실제 write에 필요한  datasource, sql 등등을 지정하는데 **ItemSqlParameterSourceProvider** 또는 **ItemPreparedStatementSetter** 를 반드시 이용해서 실행될 sql의 파라미터에 매개변수를 매핑해준다 이 예제에서는 ItemPreparedStatementSetter를 사용했다

**LogBatchItemPreparedStatementSetter.java**

```java
package me.sup2is.logbatch;

import org.springframework.batch.item.database.ItemPreparedStatementSetter;

import java.sql.PreparedStatement;
import java.sql.SQLException;

public class LogBatchItemPreparedStatementSetter implements ItemPreparedStatementSetter<Log> {
    @Override
    public void setValues(Log log, PreparedStatement ps) throws SQLException {
        ps.setString(1, log.getStatus().name());
        ps.setString(2, log.getMessage());
    }
}

```



위에서 등록한 Configuration만으로 reader, processor, writer의 역할 테스트가 가능하다. 간단하게 테스트를 작성해서 확인해 보자. 

**LogBatchApplicationTest.java**

```java
package me.sup2is.logbatch;

import org.junit.jupiter.api.Test;
import org.springframework.batch.item.ExecutionContext;
import org.springframework.batch.item.file.FlatFileItemReader;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;

import static org.junit.jupiter.api.Assertions.assertEquals;

@SpringBootTest
class LogBatchApplicationTest {

    @Autowired
    private FlatFileItemReader<Log> flatFileItemReader;

    @Test
    public void reader_oneline_test() throws Exception {

        flatFileItemReader.open(new ExecutionContext());
        Log log = flatFileItemReader.read();

        System.out.println(log);

        assertEquals(LogStatus.DEBUG, log.getStatus());
        assertEquals("Lorem Ipsum is simply dummy text of the printing and typesetting industry 1."
                , log.getMessage());
    }
    
}
```

<br>

결과는 성공!!

```
Log(status=DEBUG, message=Lorem Ipsum is simply dummy text of the printing and typesetting industry 1.)
```

<br>

이제 실제 batch job과 step을 등록해서 debug.log에 있는 데이터를 모두 데이터화 시키는 작업을 진행해 보자.

## Batch Job 등록하기

**LogBatchConfiguration.java**

```java
package me.sup2is.logbatch;

@Configuration
@EnableBatchProcessing
public class LogBatchConfiguration {

    @Autowired
    public JobBuilderFactory jobBuilderFactory;

    @Autowired
    public StepBuilderFactory stepBuilderFactory;

    @Bean
    public Job logJob(JobCompletionNotificationListener listener, Step step1) {
        return jobBuilderFactory.get("logJob")
                .listener(listener)
                .flow(step1)
                .end()
                .build();
    }

    @Bean
    public Step step1(JdbcBatchItemWriter<Log> writer) {
        return stepBuilderFactory.get("step1")
                .<Log, Log> chunk(10)
                .reader(reader())
                .processor(processor())
                .writer(writer)
                .build();
    }
}

```

아까 설정한 LogBatchConfiguration에 위에 내용을 추가해준다. 몇가지 변한부분은 클래스에 **@EnableBatchProcessing**을 사용하고 Job과 Step을 등록해줬다.

1. @EnableBatchProcessing : 실제 Spring Batch에서 필요하고 중요한 Bean들을 사용가능하게 도와주는 Annotation이므로 반드시 추가해야한다.
2.  `logJob()` : jobBuilderFactory를 사용해서 수행될 job들을 지정해준다 예제의 경우 inmemory 기반의 database를 사용했기때문에 application이 종료되는 순간 데이터도 날아간다. 따라서 job이 끝나는시점에 listener를 지정해줘서 application이 종료되기 전에 database에 있는 log 데이터들을 전부 꺼내오도록 지정해줬다.

**JobCompletionNotificationListener.java**

```java
package me.sup2is.logbatch;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.batch.core.BatchStatus;
import org.springframework.batch.core.JobExecution;
import org.springframework.batch.core.listener.JobExecutionListenerSupport;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.stereotype.Component;

@Component
public class JobCompletionNotificationListener extends JobExecutionListenerSupport {

    private static final Logger logger = LoggerFactory.getLogger(JobCompletionNotificationListener.class);

    private final JdbcTemplate jdbcTemplate;

    @Autowired
    public JobCompletionNotificationListener(JdbcTemplate jdbcTemplate) {
        this.jdbcTemplate = jdbcTemplate;
    }

    @Override
    public void afterJob(JobExecution jobExecution) {
        if(jobExecution.getStatus() == BatchStatus.COMPLETED) {
            logger.info("!!! JOB FINISHED! Time to verify the results");

            jdbcTemplate.query("SELECT status, message FROM system_log",
                    (rs, row) -> new Log(
                            LogStatus.valueOf(rs.getString(1)),
                            rs.getString(2))
            ).forEach(findLog -> logger.info("Found <" + findLog + "> in the database."));
        }
    }
}
```



3. `step1()` : stepBuilderFactory를 사용해서 수행될 step들을 지정해준다. 앞에서 지정해놓은 `reader()`, `processor()` 와 파라미터로 넘어오는 writer를 지정해주고 실제 chunk될 사이즈도 `chunk()`를 사용하여 지정해줬다.



## 실행하기

그냥 간단하게 Spring Application을 실행하면 다음과같은 로그를 확인할 수 있다.

```
...

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v2.2.5.RELEASE)

...

2020-03-25 13:56:25.855  INFO 34244 --- [  restartedMain] m.sup2is.logbatch.LogBatchItemProcessor  : this is LogBatchItemProcessor Log status : DEBUG
2020-03-25 13:56:25.856  INFO 34244 --- [  restartedMain] m.sup2is.logbatch.LogBatchItemProcessor  : this is LogBatchItemProcessor Log status : DEBUG
2020-03-25 13:56:25.856  INFO 34244 --- [  restartedMain] m.sup2is.logbatch.LogBatchItemProcessor  : this is LogBatchItemProcessor Log status : DEBUG
2020-03-25 13:56:25.856  INFO 34244 --- [  restartedMain] m.sup2is.logbatch.LogBatchItemProcessor  : this is LogBatchItemProcessor Log status : DEBUG
2020-03-25 13:56:25.856  INFO 34244 --- [  restartedMain] m.sup2is.logbatch.LogBatchItemProcessor  : this is LogBatchItemProcessor Log status : DEBUG
2020-03-25 13:56:25.857  INFO 34244 --- [  restartedMain] m.sup2is.logbatch.LogBatchItemProcessor  : this is LogBatchItemProcessor Log status : INFO
2020-03-25 13:56:25.857  INFO 34244 --- [  restartedMain] m.sup2is.logbatch.LogBatchItemProcessor  : this is LogBatchItemProcessor Log status : INFO
2020-03-25 13:56:25.857  INFO 34244 --- [  restartedMain] m.sup2is.logbatch.LogBatchItemProcessor  : this is LogBatchItemProcessor Log status : INFO
2020-03-25 13:56:25.857  INFO 34244 --- [  restartedMain] m.sup2is.logbatch.LogBatchItemProcessor  : this is LogBatchItemProcessor Log status : INFO
2020-03-25 13:56:25.857  INFO 34244 --- [  restartedMain] m.sup2is.logbatch.LogBatchItemProcessor  : this is LogBatchItemProcessor Log status : INFO
2020-03-25 13:56:25.867  INFO 34244 --- [  restartedMain] m.sup2is.logbatch.LogBatchItemProcessor  : this is LogBatchItemProcessor Log status : ERROR
2020-03-25 13:56:25.868  INFO 34244 --- [  restartedMain] m.sup2is.logbatch.LogBatchItemProcessor  : this is LogBatchItemProcessor Log status : ERROR
2020-03-25 13:56:25.868  INFO 34244 --- [  restartedMain] m.sup2is.logbatch.LogBatchItemProcessor  : this is LogBatchItemProcessor Log status : ERROR
2020-03-25 13:56:25.868  INFO 34244 --- [  restartedMain] m.sup2is.logbatch.LogBatchItemProcessor  : this is LogBatchItemProcessor Log status : ERROR
2020-03-25 13:56:25.868  INFO 34244 --- [  restartedMain] m.sup2is.logbatch.LogBatchItemProcessor  : this is LogBatchItemProcessor Log status : ERROR
2020-03-25 13:56:25.871  INFO 34244 --- [  restartedMain] o.s.batch.core.step.AbstractStep         : Step: [step1] executed in 113ms
2020-03-25 13:56:25.878  INFO 34244 --- [  restartedMain] m.s.l.JobCompletionNotificationListener  : !!! JOB FINISHED! Time to verify the results
2020-03-25 13:56:25.882  INFO 34244 --- [  restartedMain] m.s.l.JobCompletionNotificationListener  : Found <Log(status=DEBUG, message=Lorem Ipsum is simply dummy text of the printing and typesetting industry 1.)> in the database.
2020-03-25 13:56:25.882  INFO 34244 --- [  restartedMain] m.s.l.JobCompletionNotificationListener  : Found <Log(status=DEBUG, message=Lorem Ipsum is simply dummy text of the printing and typesetting industry 2.)> in the database.
2020-03-25 13:56:25.882  INFO 34244 --- [  restartedMain] m.s.l.JobCompletionNotificationListener  : Found <Log(status=DEBUG, message=Lorem Ipsum is simply dummy text of the printing and typesetting industry 3.)> in the database.
2020-03-25 13:56:25.882  INFO 34244 --- [  restartedMain] m.s.l.JobCompletionNotificationListener  : Found <Log(status=DEBUG, message=Lorem Ipsum is simply dummy text of the printing and typesetting industry 4.)> in the database.
2020-03-25 13:56:25.882  INFO 34244 --- [  restartedMain] m.s.l.JobCompletionNotificationListener  : Found <Log(status=DEBUG, message=Lorem Ipsum is simply dummy text of the printing and typesetting industry 5.)> in the database.
2020-03-25 13:56:25.882  INFO 34244 --- [  restartedMain] m.s.l.JobCompletionNotificationListener  : Found <Log(status=INFO, message=Lorem ipsum dolor sit amet 1.)> in the database.
2020-03-25 13:56:25.882  INFO 34244 --- [  restartedMain] m.s.l.JobCompletionNotificationListener  : Found <Log(status=INFO, message=Lorem ipsum dolor sit amet 2.)> in the database.
2020-03-25 13:56:25.882  INFO 34244 --- [  restartedMain] m.s.l.JobCompletionNotificationListener  : Found <Log(status=INFO, message=Lorem ipsum dolor sit amet 3.)> in the database.
2020-03-25 13:56:25.882  INFO 34244 --- [  restartedMain] m.s.l.JobCompletionNotificationListener  : Found <Log(status=INFO, message=Lorem ipsum dolor sit amet 4.)> in the database.
2020-03-25 13:56:25.882  INFO 34244 --- [  restartedMain] m.s.l.JobCompletionNotificationListener  : Found <Log(status=INFO, message=Lorem ipsum dolor sit amet 5.)> in the database.
2020-03-25 13:56:25.882  INFO 34244 --- [  restartedMain] m.s.l.JobCompletionNotificationListener  : Found <Log(status=ERROR, message=Application throws Exception !!)> in the database.
2020-03-25 13:56:25.882  INFO 34244 --- [  restartedMain] m.s.l.JobCompletionNotificationListener  : Found <Log(status=ERROR, message=Application throws Exception !!)> in the database.
2020-03-25 13:56:25.882  INFO 34244 --- [  restartedMain] m.s.l.JobCompletionNotificationListener  : Found <Log(status=ERROR, message=Application throws Exception !!)> in the database.
2020-03-25 13:56:25.882  INFO 34244 --- [  restartedMain] m.s.l.JobCompletionNotificationListener  : Found <Log(status=ERROR, message=Application throws Exception !!)> in the database.
2020-03-25 13:56:25.882  INFO 34244 --- [  restartedMain] m.s.l.JobCompletionNotificationListener  : Found <Log(status=ERROR, message=Application throws Exception !!)> in the database.
2020-03-25 13:56:25.884  INFO 34244 --- [  restartedMain] o.s.b.c.l.support.SimpleJobLauncher      : Job: [FlowJob: [name=logJob]] completed with the following parameters: [{}] and the following status: [COMPLETED] in 168ms
2020-03-25 13:56:26.104  WARN 34244 --- [extShutdownHook] o.s.b.f.support.DisposableBeanAdapter    : Invocation of destroy method failed on bean with name 'inMemoryDatabaseShutdownExecutor': org.h2.jdbc.JdbcSQLNonTransientConnectionException: Database is already closed (to disable automatic closing at VM shutdown, add ";DB_CLOSE_ON_EXIT=FALSE" to the db URL) [90121-200]
2020-03-25 13:56:26.105  INFO 34244 --- [extShutdownHook] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Shutdown initiated...
2020-03-25 13:56:26.108  INFO 34244 --- [extShutdownHook] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Shutdown completed.
Disconnected from the target VM, address: '127.0.0.1:50084', transport: 'socket'

Process finished with exit code 0

```



debug.log 를 reader를 이용해서 읽고, processor를 이용해서 로그를 출력하고, writer를 이용해서 database에 insert 시킨 뒤 job이 끝나는시점에 모든 데이터를 조회하도록 한 결과를 확인할 수 있다.



<br>

포스팅은 여기까지 하겠습니다.  모든예제는 제 github에서 확인하실 수 있습니다.

예제 : https://github.com/sup2is/spring-example/tree/master/spring-batch-example 



<br>

References

-  https://docs.spring.io/spring-batch/docs/4.2.x/reference/html/readersAndWriters.html#readersAndWriters 
-  https://spring.io/projects/spring-batch#learn 

