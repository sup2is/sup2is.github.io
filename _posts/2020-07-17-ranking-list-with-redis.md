---
layout: post
title: "Spring Data Redis로 구현하는 Ranking List"
tags: [NoSQL, Spring Data Redis, Redis]
date: 2020-07-17
comments: true
---

<br>

# OverView

오늘은 Redis의 Sorted Set을 사용한 Ranking API를 만들어보고 약 천만건의 player 데이터를 통해 RDB와 Sorted Set의 성능 차이가 얼마나 있는지에 대해서 알아보도록 하겠다.

# 시작하기 전에

먼저 RDB에 약 **천만개**의 더미데이터를 넣고 ORDER BY 연산자를 통해 상위 **5만개**의 player 목록을 가져오는 테스트를 진행하도록 하겠다.

테스트를 위한 더미 데이터를 넣어주고 **인덱스 설정을 해주어서 기본 order by 연산의 수행속도를 단축**시켜놓았다.

RDB는 MariaDB 5.5 를 사용할 것이고 테이블 스펙은 아래와 같다.

```sql
CREATE TABLE player(
	id bigint,
	playername varchar(50),
	score bigint
);
```

약 천만개의 데이터를 score 기준으로 select 해오면 약  **15 sec ~ 16 sec** 이 소요된다.

![20200716_155146](https://user-images.githubusercontent.com/30790184/87643684-1e881000-c786-11ea-9d7a-a94f6311d521.png)

작은 서비스라면 상관없겠지만 사용자가 점점 더 증가하는 환경에서 천만개 정도로 약 15~16초가 걸리는 상황이라면 꽤 문제가 있는 상황이다. 사용자가 늘어나면 늘어날 수록 수행시간은 더 길어질 것이다.

이제 이 서비스를 Redis의 Sorted Set을 활용해서 성능개선을 해보도록 하겠다.



# Spring Boot Redis 관련 설정하기

먼저 Spring Boot에서 Redis를 사용하기위해서 라이브러리 설정과 Configuration 설정을 해주도록 하겠다.

**pom.xml**

```xml
<?xml version="1.0" encoding="UTF-8"?>

<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.3.1.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>

    <groupId>me.sup2is</groupId>
    <artifactId>ranking-api-with-redis</artifactId>
    <version>1.0-SNAPSHOT</version>
    <name>ranking-api-with-redis</name>

    <properties>
        <java.version>1.8</java.version>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
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
            <groupId>redis.clients</groupId>
            <artifactId>jedis</artifactId>
            <version>3.3.0</version>
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

주목할만한 부분은 **spring-boot-starter-data-redis** 모듈이다. 사실 이 모듈(boot ver 2.3.1)에는 lettuce가 기본 장착되어있기 때문에 redis client로 lettuce를 사용한다면 별도의 모듈을 추가하지 않아도 괜찮다. 

그럼에도 **jedis** 모듈을 추가한건 아래 ''**Redis에 Dummy Data 넣기**''에서 이유를 확인할 수 있다.

이어서 Configuration 설정을 해보자.

**RedisConfig.java**

```java
package me.sup2is;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.serializer.StringRedisSerializer;

@Configuration
public class RedisConfig {

    @Autowired
    private RedisConnectionFactory redisConnectionFactory;

    @Bean
    public RedisTemplate<String, String> redisTemplate() {
        RedisTemplate<String, String> redisTemplate = new RedisTemplate<>();
        redisTemplate.setConnectionFactory(redisConnectionFactory);
        redisTemplate.setKeySerializer(new StringRedisSerializer());
        redisTemplate.setValueSerializer(new StringRedisSerializer());
        return redisTemplate;
    }
}

```

RedisConfig 클래스는 RedisTemplate 빈을 등록해주는 역할을 한다. RedisConnectionFactory는 Spring Boot가 알아서 등록해주기 때문에 주입받아서 사용하면 된다. 실제 Redis 관련 설정은 application.yml에 있다.



**application.yml**

```yaml
spring:
  redis:
    host: 192.168.56.107
    port: 6379

```

이제 Redis 관련 설정이 끝났다면 Redis 내부에 Dummy Data를 넣는것부터 시작해보자.

# Redis에 Dummy Data 넣기

Redis는 **Mass insert** 라는 기능을 제공하는데 데이터를 대량으로 삽입하는 api이다. 자세한 정보는 [https://redis.io/topics/mass-insert](https://redis.io/topics/mass-insert)에서 찾아볼 수 있다. 

Mass insert 에 대해서 조금 더 자세하게 알아보는 도중 Redis Client인 Jedis 라이브러리 내부에 Pipeline API를 제공하는 방법이 있어서 이 방법을 사용해서 Redis 내부에 Dummy Data를 넣도록 하겠다. (lettuce도 있다. ~~근데 무식하게 jedis를 사용했다.~~ lettuce 관련 자료는 [https://github.com/lettuce-io/lettuce-core/wiki/Pipelining-and-command-flushing](https://github.com/lettuce-io/lettuce-core/wiki/Pipelining-and-command-flushing)에서 찾을 수 있다. )

> 추가적으로 Redis의 Lua script를 활용해서 대량의 벌크작업을 수행할 수 있다. by 쿄잉

**RedisService.java**

```java
package me.sup2is;

//..import

@Service
public class RedisService {
    
    
    //...
 
    @Value("${spring.redis.host}")
    private String redisHost;

    @Value("${spring.redis.port}")
    private int redisPort;

    public void add() {
        try (Jedis jedis = new Jedis(redisHost, redisPort)) {
            Pipeline pipeline = jedis.pipelined();
            for (int i = 0; i < 10_000_000; i++) {
                pipeline.zadd(KEY,  i * (int)(Math.random() * 10_000_000) + 1, String.valueOf(i * (int)(Math.random() * 10_000_000) + 1));
            }
            pipeline.sync();
        }
    }

    
    //...

}
```

RedisService에 `add()` 메서드는 약 천만개의 데이터를 Redis server에 대량으로 push한다. 데이터 양이 적지 않기 때문에 생각보다 오래걸릴 수 있다. 실시간으로 Redis Server에서`dbsize`  명령어를 사용하면 실시간으로 데이터가 증가하는 것을 확인할 수 있다. 실제로 테스트해보자. 

**RedisServiceTest.java**

```java
package me.sup2is;

//..import

@SpringBootTest
public class RedisServiceTest {

    @Autowired
    private RedisService redisService;

    @Resource(name = "redisTemplate")
    private ZSetOperations<String, String> zSetOperations;

    private static final String KEY = "player";

    @Test
    public void add() {
        redisService.add();
        assertTrue(zSetOperations.zCard(KEY) > 9_000_000);
    }

}

```

같은 아이디가 들어갈 수도 있기 때문에 오차범위를 생각해서 약 900만개 이상이면 테스트를 통과하도록 했다. 실제로 레디스 내부에 들어간 데이터는 970만개 정도가 된다.

![20200716_163556](https://user-images.githubusercontent.com/30790184/87643681-1def7980-c786-11ea-836a-7bdce6113cb0.png)

이제 Redis 내부에서 상위 5만개 데이터를 가져오는 코드를 작성해보자.

# Redis에서 상위 5만개 데이터 가져오기

**RedisService.java**

```java
package me.sup2is;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.data.redis.core.HashOperations;
import org.springframework.data.redis.core.ZSetOperations;
import org.springframework.stereotype.Service;
import redis.clients.jedis.Jedis;
import redis.clients.jedis.Pipeline;

import javax.annotation.Resource;
import java.util.*;

@Service
public class RedisService {

    private final String KEY = "player";

    @Value("${spring.redis.host}")
    private String redisHost;

    @Value("${spring.redis.port}")
    private int redisPort;

    @Resource(name = "redisTemplate")
    private ZSetOperations<String, String> zSetOperations;

    @Resource(name = "redisTemplate")
    private HashOperations<String,String,String> hashOperations;

    public void add() {
        try (Jedis jedis = new Jedis(redisHost, redisPort)) {
            Pipeline pipeline = jedis.pipelined();
            for (int i = 0; i < 10_000_000; i++) {
                pipeline.zadd(KEY,  i * (int)(Math.random() * 10_000_000) + 1, String.valueOf(i * (int)(Math.random() * 10_000_000) + 1));
            }
            pipeline.sync();
        }
    }

    public List<String> getPlayers() {
        Set<String> range = zSetOperations.reverseRange(KEY, 0, 49_999);
        return new ArrayList<>(range);
    }

}

```

lettuce나 jedis나 api가 매우 간단하게 설계되어 있어서 Redis 와 통신하는 부분은 간단하게 처리할 수 있다. zSetOperations의 `reverseRange()`라는 메서드로 간편하게 상위 5만개 데이터를 가져올 수 있고 이 로직은 아래 redis 명령어와 같은 역할을 한다.

```
zrange player 0 49999
```

그렇다면 실제 수행시간은 어떻게 측정될까? 테스트를 작성해보자.

**RedisServiceTest.java**

```java
package me.sup2is;

//..import

@SpringBootTest
public class RedisServiceTest {

    @Autowired
    private RedisService redisService;

    @Resource(name = "redisTemplate")
    private ZSetOperations<String, String> zSetOperations;

    private static final String KEY = "player";

    @Test
    public void add() {
        redisService.add();
        assertTrue(zSetOperations.zCard(KEY) > 9_000_000);
    }

    @Test
    @RepeatedTest(3)
    public void getPlayers() {
        long start = System.currentTimeMillis();
        List<String> players = redisService.getPlayers();
        long end = System.currentTimeMillis();
        System.out.println("프로그램 수행 시간: " + (double)(end - start)/1000);

        assertTrue(players.size() == 50_000);
    }

}

```

약 테스트를 3회돌려서 나온 결과는 약 다음과 같다

```
프로그램 수행 시간: 0.089
프로그램 수행 시간: 0.088
프로그램 수행 시간: 0.089
```

Maria DB에서 15~16초가 걸리는 것과 비교했을 때 Sorted Set의 수행속도는 평균적으로 **0.1**초였다. 실제로 사용가능한 Object로 매핑한다고 하더라도 MariaDB 보다는 훨씬 더 빠른 환경에서 데이터를 가져올 수 있다.

# 마무리

초반에 예제를 작성할때 MariaDB쪽에 인덱싱을 하지 않아서 데이터를 가져오는 시간 편차가 매우매우 높았다. 예제를 좀 어거지로 작성한 느낌도 있긴 한데 Sorted Set이 얼마나 빠른지에 대한 확인을 눈으로 직접해보고 싶었다.



<hr>
포스팅은 여기까지 하겠습니다. 퍼가실때는 출처를 반드시 남겨주세요!


예제: https://github.com/sup2is/study/tree/master/db/redis/ranking-list-with-redis



**References**

- [https://stackoverflow.com/questions/30728409/how-to-do-mass-insertion-in-redis-using-java](https://stackoverflow.com/questions/30728409/how-to-do-mass-insertion-in-redis-using-java)

