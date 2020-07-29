---

layout: post
title: "Kafka Connect로 데이터 허브 구축하기"
tags: [Apache Kafka, Kafka Connect]
date: 2020-06-08
comments: true
---



<br>

# OverView

이번시간에는 Kafka Connect에 대해서 알아보고 Kafka Connect를 기반으로 도커 컨테이너로 올린 Maria DB와 CentOS 서버 사이에 데이터 허브를 구축하는 예제를 다뤄보도록 하겠다. 

# Kafka Connect란 ?

초기에 LinkedIn의 카프카 요구사항중에 하나로 **다양한 제품과 시스템에 쉽게 연동** 이라는 내용이 있었다. 결론부터 말하면 Kafka Connect가 이걸 가능하게 해준다. database 또는 key-value store, aws S3 등등 의 매우매우 다양한 커넥터가 존재하고 필요에 맞게 알아서 사용하면 된다.

![Dy6Fr](https://user-images.githubusercontent.com/30790184/83999437-b3386a80-a99d-11ea-99d7-aea038f6256f.png)

카프카 커넥트는 프로듀서와 컨슈머 사이에 배치될 수 있는데 위에 그림처럼 JDBC 를 사용해서 Database와  Kafka를 연결시킬 수 있고 Elasticsearch 또는 FlatFile 형식으로도 연결시킬 수 있다.

카프카 커넥트에서는 프로듀서/컨슈머의 용어가 조금 바뀌는데 데이터를 Kafka로 보내는 쪽을 **Source**라고 부르고 데이터를 Kafka로부터 받아내는쪽을 **Sink**라고 부른다.

이 카프카 커넥트는 카프카 브로커 개수만큼으로 클러스터를 구성할 수도 있다.

이번시간에는 Maria DB 컨테이너와 CentOS7 사이에 데이터 허브를 구축해서 카프카 커넥트가 어떻게 동작하는지에 대해서 알아보도록 하겠다!

# 시작하기 전에

1. kafka와 mariaDB는 도커 컨테이너로 올려서 진행할 예정이다.
2. 기본적으로 카프카 서버가 한대 이상 있다고 가정하고 예제를 진행하도록 하겠다. 나같은경우는 도커 컨테이너를 구성해서 클러스터를 구성했다. docker-compose 사용이 가능하다면  [https://github.com/simplesteph/kafka-stack-docker-compose](https://github.com/simplesteph/kafka-stack-docker-compose) 을 참고해서 아주 간단하게 클러스터를 구성할 수 있다. 
3.  Kafka Connect를 사용하기 위해 관련 파일들을 [https://www.confluent.io/download에서](https://www.confluent.io/download) 받는다. (이 예제는 Confluent Platform 기반으로 작성한다.) 적절한 위치에 위치시키고 압축을 풀자. 물론 Kafka Connect를 컨테이너 기반으로도 올릴 수 있다.[https://hub.docker.com/r/confluentinc/cp-kafka-connect](https://hub.docker.com/r/confluentinc/cp-kafka-connect) 참고



# MariaDB 컨테이너 올리기

앞에서 설명했듯이 MariaDB는 도커 컨테이너 기반으로 올린다. 다음 명령어를 사용해서 컨테이너를 올려보자.

```
docker run --name mariadb \
  -e MYSQL_ROOT_PASSWORD=passwd\
  -e MYSQL_DATABASE=testdb\
  -p 3306:3306\
  -d mariadb
```

컨테이너 이미지가 없다면 docker hub에서 받아올 것이다. 만약 조금 더 상세하게 컨테이너 설정을 하고싶거나 이미지에 대해 궁금한 사항이 있으면 [https://hub.docker.com/\_/mariadb](https://hub.docker.com/_/mariadb) 을 확인하도록 하자.

성공적으로 컨테이너가 올라갔다면 다음 명령어를 통해서 컨테이너에 접속하고 테이블을 생성해보자.

```

#컨테이너 접속
docker exec -it mariadb /bin/bash

#root 계정으로 로그인 MYSQL_ROOT_PASSWORD 입력
mysql -p

#MYSQL_DATABASE로 접근
use testdb

#테이블 생성
CREATE TABLE IF NOT EXISTS test (
  id int NOT NULL PRIMARY KEY,
  name varchar(100),
  email varchar(200),
  department varchar(200)
);

#데이터 초기화
INSERT INTO test(id, name, email, department) values (1, 'choi', 'dev.sup2is@gmail.com', 'A');
INSERT INTO test(id, name, email, department) values (2, 'kim', 'kim@gmail.com', 'A');
INSERT INTO test(id, name, email, department) values (3, 'woo', 'woo@gmail.com', 'B');
INSERT INTO test(id, name, email, department) values (4, 'park', 'park@gmail.com', 'B');
INSERT INTO test(id, name, email, department) values (5, 'lee', 'lee@gmail.com', 'A');

```

몇가지 명령어와 스크립트로 test 테이블을 초기화시켜준다.

![20200605_083143](https://user-images.githubusercontent.com/30790184/83997422-5044d480-a999-11ea-9983-68ff96af9362.png)

이렇게 구성이되면 일단 Source로서의 준비는 끝이다. 다음은 Kafka Connect 설정을 해보자.

# Kafka Connect 시작하기

먼저 컨플루언트 카프카 홈 디렉토리의 **{confluent kafka home}/etc/kafka/connect-distributed.properties** 파일을 수정한다. 만약 패키지 도구로 받았으면 /etc/kafka/.. 등등의 디렉토리에 있다. 수정할부분은 다음과같다. 만약 테스트 환경등에 의해 카프카 클러스터를 구성하지 못했다면 브로커 한개로 돌릴 수 있는 **connect-standalone.properties** 파일을 사용하면 된다. connect-standalone 환경은 주로 개발 및 테스트환경에서만 진행하고 실제 운영에서는 사용하지 않는게 좋다.

**※** 참고로 confluent 에서 제공하는 cp-kafka-connect 라는 도커 이미지도 있다. 카프카 커넥트를 컨테이너로 올리고싶다면 [https://hub.docker.com/r/confluentinc/cp-kafka-connect](https://hub.docker.com/r/confluentinc/cp-kafka-connect)을 확인하자.

```

     22 # A list of host/port pairs to use for establishing the initial connection to the Kafka cluster.
     23 bootstrap.servers=localhost:9092,localhost:9093,localhost:9094
     24
     25 # unique name for the cluster, used in forming the Connect cluster group. Note that this must not conflict with consumer group IDs
     26 group.id=connect-cluster-exam


```

23과 26번 라인 정도에 **bootstrap.servers** 와 **group.id**를 수정해준다. bootstrap.servers는 카프카 클러스터를 구성하는 브로커들의 ip를 적어주면되고 group.id는 커넥터 클러스터를 위한 고유한 이름을 지정해주면 된다. 이 아이디는 컨슈머 그룹의 id와 겹치면 안된다.

설정이 끝났다면 kafka home 디렉토리에서 아래 명령어로 카프카 커넥트 서버를 올려준다.

```
./bin/connect-distributed.sh ./etc/kafka/connect-distributed.properties
```

당연히 kafka broker 서버가 올라가있다고 가정한다. 만약 브로커들과 연결이 잘 되지 않으면 서버가 올라가지 않는다.

카프카 커넥트의 기본 포트는 8083을 사용한다. 아래 명령어로 서버가 잘 구동되었는지 확인해보자

```
curl http://localhost:8083

#결과
{"version":"2.5.0","commit":"66563e712b0b9f84","kafka_cluster_id":"r6OM5xD7Tt-55YvsMLGYvg"}
```



## RestAPI 기반의 Kafka Connect 사용하기

Kafka Connect는 친절하게도 RestAPI를 사용해서 커넥터를 생성하고 삭제할 수 있도록 도와준다.

- *GET /connectors* – 모든 커넥터를 조회한다.
- *GET /connectors/{name}* – {name}을 갖는 커넥터의 정보를 조회한다.
- *POST /connectors* – 커넥터를 생성, Body쪽에는 JSON Object 타입의 커넥터 config정보가 있어야한다.
- *GET /connectors/{name}/status* – 이 커넥터가 running인지, failed인지 paused 인지 현재 상태를 조회한다. 
- *DELETE /connectors/{name}* – {name}을 갖는 커넥터를 삭제시킨다.
- *GET /connector-plugins –* 카프카 커넥터 클러스터 내부에 설치된 플러그인들을 조회한다.



# Kafka Connect와 MariaDB 연결하기

앞에서 생성한 mariadb 컨테이너와 카프카 커넥터를 연결해보도록 하겠다. 

mariadb를 연결하기 위해서는 별도의 플러그인 jar파일이 필요한데 [https://downloads.mariadb.org/connector-java/](https://downloads.mariadb.org/connector-java/) 주소에서 **mariadb-java-client-x.x.x** 파일을 다운받아서 **/{confluent kafka home}/share/java/kafka/** 디렉토리에 위치시킨다. 나는 mariadb-java-client-2.6.0 파일을 사용했다.

**No suitable driver found for jdbc:mysql** 같은 에러가 나올 수 있고 이 에러는 jar파일의 경로문제이다.(~~개고생했음~~) 각자 환경에따라 알맞게 구성해야한다.

> apache kafka의 경우 kafka 폴더 내부에 /libs 경로에 넣으면 잘 동작할 것이다.

 **mariadb-java-client-2.6.0** 파일이 준비가 되었다면 JdbcSourceConnector가 plugin으로 설치가 되었는지를 다음의 명령어로 확인해보자

```
curl http://localhost:8083/connector-plugins | python -m json.tool
```

나같은경우는 초기에 아래와 같은 커넥터들이 존재하지 않았다. 

**※만약 존재한다면 다음(JdbcSourceConnector로 Connect 생성하기)으로 넘어가도 된다.**

```
..
    {
        "class": "io.confluent.connect.jdbc.JdbcSinkConnector",
        "type": "sink",
        "version": "5.5.0"
    },
    {
        "class": "io.confluent.connect.jdbc.JdbcSourceConnector",
        "type": "source",
        "version": "5.5.0"
    },
..

```

**※JdbcSourceConnector 가 없을때만 실행할것!**

1. Confluent Platform에는 이미 kafka-connect-jdbc.jar 파일이  **/{confluent kafka home}/share/java/kafka-connect-jdbc/** 내부에 존재한다. 만약 없다면 [https://www.confluent.io/hub/](https://www.confluent.io/hub/) 에 가서 **Kafka Connect JDBC**라는 이름으로 검색해서 관련 파일을 받는다.
2. kafka.connect.jdbc.jar파일을 위에서 사용했던 **/{confluent kafka home}/share/java/kafka/**에 넣고 재기동 한 뒤 다시 **curl localhost:8083/connector-plugins \| python -m json.tool**을 실행해본다.

> 마찬가지로 apache kafka의 경우 kafka 폴더 내부에 /libs 경로에 넣으면 잘 동작할 것이다.






## JdbcSourceConnector로 Connect 생성하기

커넥터의 연결은 위에서 설명한대로 RestAPI를 사용해서 연결한다.

```
echo '
{
  "name" : "my-first-connect",
  "config" : {
    "connector.class" : "io.confluent.connect.jdbc.JdbcSourceConnector",
    "connection.url" : "jdbc:mysql://127.0.0.1:3306/testdb",
    "connection.user" : "root",
    "connection.password" : "passwd",
    "mode": "incrementing",
    "incrementing.column.name" : "id",
    "table.whitelist" : "test",
    "topic.prefix" : "my_connect_",
    "tasks.max" : "3"
  }
}
' | curl -X POST -d @- http://localhost:8083/connectors --header "content-Type:application/json"
```

커넥터를 생성하기 위해 /connectors 경로로 위와 같은 정보들을 body에 담아서 커넥터를 생성한다. jdbc 커넥터의 설정옵션은 간단하게 다음과 같다.

- **connection.url, connection.user, connection.password**
  - DB에 접속하기 위한 설정 정보
- **mode, incrementing. colmn.name**
  - 실행하고 있는 동안 커넥터는 jdbc를 통해 rdb를 폴링한다. 변경이 있으면 카프카에 전달하고 변경 감지는 incrementing 방법으로 진행한다. mode는 incrementing외에도 bulk, timestamp 등이 있다.  incrementing.column.name 을 통해 변경을 감지한다.
- **table.whitelist**
  - 로드할 대상의 테이블을 지정한다. 반대로 blacklist 도 있다.
- **topic-prefix**
  - 카프카에 데이터를 넣을때 토픽 명을 결정할 접두어를 지정한다.
- **tasks.max**
  - 이 커넥터에서 만들어지는 최소의 테스크 수



아래 명령어로 curl 요청시 우리가 생성했던 커넥터의 이름이 나온다면 성공이다.

```
curl http://localhost:8083/connectors
```



## Kafka Console Consumer로 Source 커넥터 확인하기

우리가 이전에 생성한 Source 커넥터는 간편하게 kafka-console-consumer.sh 파일을 통해서 확인할 수 있다. 카프카 커넥터로 생성한 토픽의 이름은 **{topic.prefix}_{table.whitelist}** 이고 아래 명령어로도 확인할 수 있다.

```
./bin/zookeeper-shell localhost:2181 ls /brokers/topics
```

확인한 토픽명으로 컨슈머 콘솔을 띄워보자.

```
./bin/kafka-console-consumer --bootstrap-server localhost:9092,localhost:9093,localhost:9094 --topic my_connect_test --from-beginning
```

데이터들이 console에 잘 입력되어 있는 것을 확인할 수 있다.

![20200608_142120](https://user-images.githubusercontent.com/30790184/83996978-525a6380-a998-11ea-85db-ae72a0b4331e.png)

# Kafka Connect로 Flatfile Sink 만들기

Flatfile sink를 만드는건 그냥 파일로만 떨구면 되기 때문에 비교적 간단하다. 다음의 RestAPI를 카프카 커넥터에 요청해보자

```
echo '{
  "name" : "my-first-sink",
  "config" : {
    "connector.class" : "org.apache.kafka.connect.file.FileStreamSinkConnector",
    "file" : "/root/test.txt",
    "topics" : "my_connect_test"
  }
}
' | curl -X POST -d @- http://localhost:8083/connectors --header "content-Type:application/json"
```



데이터들이 test.txt로 잘 전달되는 것을 확인할 수 있다.

![20200608_142511](https://user-images.githubusercontent.com/30790184/83996972-5090a000-a998-11ea-8bf1-acf6d3a52456.png)



추가적으로 실시간으로 데이터들을 sink하는 모습을 보고싶다면 다음 명령어를 통해서 text.txt 파일을 추적할 수 있다.

```
tail -f test.txt
```

이제 mysql 콘솔에서 데이터를 직접 insert해보자. 왼쪽 쉘에 데이터가 정상적으로 들어오는 것을 확인할 수 있다.

![2020-06-08-14-27-28](https://user-images.githubusercontent.com/30790184/83996974-51c1cd00-a998-11ea-9c0e-3544776a0efe.gif)



# 마무리

만약 여러 분산된 노드들의 데이터 동기화에 있어서 batch 처리 등등의 방법이 있지만 Kafka Connect를 통해서 데이터들의 싱크를 맞춰줌으로써 데이터를 통합하는 작업을 보다 더 손쉽게 구성할 수 있고 AWS S3, Elasticsearch 다양한 곳과도 손쉽게 연결 가능하다.



<hr>
포스팅은 여기까지 하겠습니다. 퍼가실때는 출처를 반드시 남겨주세요!


<br>

**References**

-  https://www.baeldung.com/kafka-connectors-guide
-  https://docs.confluent.io/current/connect/references/restapi.html
-  https://docs.confluent.io/current/connect/kafka-connect-jdbc/index.html
-  https://gquintana.github.io/2019/12/10/Kafka-connect-plugin-install.html
-  실전 아파치 카프카 - 사사키 도루 등 6명 (한빛미디어)
