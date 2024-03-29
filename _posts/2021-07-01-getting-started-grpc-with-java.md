---
layout: post
title: "gRPC 시작하기 Feat.Java"
tags: [gRPC, Protobuf]
date: 2021-07-01
comments: true
---

<br>



# Overview

이번시간에는 gRPC에 대해 알아보는 시간을 갖도록 하겠다.



# gRPC란?

사용된 아키텍처, 언어, 스타일과 상관 없이 프로세스간 통신 기술은 최신 분산 소프트웨어의 가장 중요한 부분이다. 분산 소프트웨어에서 얼마나 많이 네트워크 비용을 줄이느냐에 따라 애플리케이션의 속도를 좌우할 수 있기 때문이다.

많은곳에서 REST를 사용해서 분산 소프트웨어간 통신을 하고 있지만 대체제로 등장한 gRPC도 많이 사용하는 추세이다.

gRPC를 사용하면 마치 로컬 객체의 함수를 호출하는 것처럼 쉽게 분산된 이기종 애플리케이션을 연결하고 호출할 수 있다. gRPC는 서비스 인터페이스를 기반으로 동작하는데 이 서비스 인터페이스는 google의 protobuf를 사용해서 정의한다.

protobuf가 생성한 코드를 사용해 서버쪽에서는 인터페이스를 구현하고 클라이언트쪽에서는 스텁이라는 코드를 사용해서 이기종간 통신이 가능할 수 있게한다.

아래는 gRPC를 사용하면서 얻을 수 있는 장점들이다.

- **프로세스간 통신 효율성**
  - JSON, XML 등의 텍스트 기반 프로토콜을 사용하지 않고 protobuf 기반 바이너리 프로토콜을 사용해 서비스간 통신을 함
  - 바이너리라 매우 빠르고 HTTP/2를 사용하기 때문에 통신속도가 매우 빠른편
- **간단 명확한 서비스 인터페이스와 스키마**
  - 먼저 서비스 인터페이스를 정의하고 나중에 구현 세부사항을 작업
  - 정적 타이핑 장점을 그대로 가져옴
- **폴리글랏**
  - gRPC는 protobuf기반으로 동작하기 때문에 protobuf가 해당 언어를 지원하지 않는 이상 특정 언어에 구애받지 않음
- **스트리밍**
  - 클라이언트 스트리밍, 서버 스트리밍 아울러 서버-클라이언트 스트리밍을 기본적으로 지원함
- **유용한 내장 기능**
  - 인증, 암호화, 인터셉터 등등 여러 유용한 기능들을 제공함
- **많은 기업들을 통해 검증되고 사용**

아래는 gRPC를 사용하면서 얻을 수 있는 단점들이다.

- **외부 서비스 부적합**
  - 외부에 노출해야 하는 서비스라면 클라이언트 역시 gRPC에 대한 이해가 있어야함
  - 정적 타이핑의 단점에 따라 서비스 유연성이 저하됨
- **서비스 정의의 급격한 변경에 따른 개발 프로세스 복잡성**
  - 실제 애플리케이션의 빈번한 스키마 수정에 따른 인터페이스 변경으로 서버-클라이언트의 코드수정이 불가피함
- **상대적으로 작은 생태계**
  - 기존 REST, HTTP에 비해 생태계가 상대적으로 작음

위 장단점에서 얻을 수 있는 결론은 **gRPC는 외부에 API를 노출하지않고 폴리글랏 형태의 서비스를 구축하기에 적합하며 protobuf + HTTP/2 스펙을 활용하여 빠른 분산소프트웨어 통신에 적합하다.** 라고 이해할 수 있다.

# gRPC vs REST

가장 대표적으로 많이 사용하는 REST와 gRPC를 비교해보면 다음과 같다.

- REST는 JSON, XML 등 사람이 이해할 수 있는 텍스트 기반 프로토콜을 사용한다. 하지만 gRPC는 미리 정의된 protobuf 인터페이스를 통해 서비스끼리만 이해할 수 있는 바이너리 프로토콜을 사용한다. 따라서 속도만 놓고 본다면 gRPC가 더 빠르다.
- REST는 비교적 접근하기 쉬운 HTTP + 자원 지향 아키텍처 기반이기 때문에 클라이언트는 서버측에서 제공하는 리소스만 바라보면 된다. 하지만 gRPC는 미리 정의된 protobuf 인터페이스를 사용하기 때문에 인터페이스가 변경되면 서버, 클라이언트 모두 변경해줘야한다. 따라서 REST는 외부에 API를 제공하기에 적합하고 gRPC의 경우 외부에 API를 제공하기에는 제한적이다.

# Protobuf

[protocol buffer](https://developers.google.com/protocol-buffers) 이하 protobuf는 google에서 제공하는 확장가능한 데이터 직렬화 프로토콜이다. 지원하는 언어목록은 Java, Python, Objective-C, C++, Go 등이 있다. protobuf의 자세한 내용은 공홈에서 확인할 수 있다.

gRPC에 사용할 protobuf의 대략적인 모습은 아래와 같다.

```protobuf
syntax = "proto3";

package ecommerce;

service ProductInfo {
    rpc addProduct(Product) returns (ProductID);
    rpc getProduct(ProductID) returns (Product);
}

message Product {
    string id = 1;
    string name = 2;
    string description = 3;
    float price = 4;
}

message ProductID {
    string value = 1;
}


```

먼저 `message`를 작성해서 서비스간 통신에 사용될 메시지를 정의해야한다. 그리고 `service` 를 정의할때 기존에 작성해놓은 `message`들을 파라미터로받거나 리턴타입으로 사용하면된다.

자바 기준으로 `service`는 `interface`에 있는 추상메서드 라고 생각하면 편하고 `message`는 Dto, 값 객체라고 생각하면 편하다.

다양한 자료형, import, default value 등등 자세한 내용은 공홈에서 확인할 수 있다.

이제 작성한 이 .proto을 각각 언어에 맞는 protoc를 사용해서 컴파일한뒤에 서버측에서는 미리 정의한 `service` 의 구현체를 작성하고 클라이언트에서는 제공되는 `service` 의 stub 객체를 활용해서 통신하면 된다.

# Example

보통 gRPC 서버를 자바로 작성하는것 같진 않지만 쉬운 이해를 위해 서버와 클라이언트 모두 자바를 사용하도록 하겠다. 위에서 언급했듯이 gRPC는 서비스 인터페이스를 정의하는것 부터 시작한다.

> **protoc가 필요하기 때문에 반드시 protoc를 설치해야한다**
>
> 맥 기준으로 `brew install protobuf` 하면 설치할 수 있다. 자세한 내용은 공홈 참고 
>
> [https://grpc.io/docs/protoc-installation/](https://grpc.io/docs/protoc-installation/)

먼저 패키지 구조는 아래와 같다.



![스크린샷 2021-07-01 오전 7 10 01](https://user-images.githubusercontent.com/30790184/124038880-d61c3d80-da3c-11eb-8435-fc5f13b7b1eb.png)




## Protocol Idl

protocol-Idl엔 서비스 인터페이스 정의와 빌드옵션을 정의한다.

```protobuf
syntax = "proto3";

package me.sup2is;

service Greeter {
  rpc hello(Hello.Request) returns (Hello.Response);
}

message Hello {
  message Request {
    int32 age = 1;
    string name = 2;
  }

  message Response {
    string str = 1;
  }
}

```

위와 같이 greeter.proto 파일을 생성했다면 protoc를 활용해서 컴파일 할 수 있지만 자바진영이기 때문에 maven 또는 gradle build 옵션에 추가해서 빌드도구에 컴파일 과정을 통합할 수 있다.

나는 maven을 활용했다.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>me.sup2is</groupId>
    <artifactId>protocol-idl</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <maven.compiler.source>11</maven.compiler.source>
        <maven.compiler.target>11</maven.compiler.target>
    </properties>

    <dependencies>
        <dependency>
            <groupId>io.grpc</groupId>
            <artifactId>grpc-protobuf</artifactId>
            <version>1.16.1</version>
        </dependency>
        <dependency>
            <groupId>io.grpc</groupId>
            <artifactId>grpc-stub</artifactId>
            <version>1.16.1</version>
        </dependency>
        <dependency>
            <groupId>com.google.protobuf</groupId>
            <artifactId>protobuf-java</artifactId>
            <version>3.17.3</version>
        </dependency>
        <dependency>
            <groupId>javax.annotation</groupId>
            <artifactId>javax.annotation-api</artifactId>
            <version>1.3.1</version>
        </dependency>
    </dependencies>
    
    <build>
        <extensions>
            <extension>
                <groupId>kr.motd.maven</groupId>
                <artifactId>os-maven-plugin</artifactId>
                <version>1.4.1.Final</version>
            </extension>
        </extensions>
        <plugins>
            <plugin>
                <groupId>org.xolstice.maven.plugins</groupId>
                <artifactId>protobuf-maven-plugin</artifactId>
                <version>0.6.1</version>
                <configuration>
                    <protocArtifact>com.google.protobuf:protoc:3.2.0:exe:${os.detected.classifier}</protocArtifact>
                </configuration>
                <executions>
                    <execution>
                        <id>protoc-java</id>
                        <goals>
                            <goal>compile</goal>
                        </goals>
                    </execution>
                    <execution>
                        <id>protoc-grpc-java</id>
                        <goals>
                            <goal>compile-custom</goal>
                        </goals>
                        <configuration>
                            <pluginId>grpc-java</pluginId>
                            <pluginArtifact>io.grpc:protoc-gen-grpc-java:1.3.0:exe:${os.detected.classifier}
                            </pluginArtifact>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
</project>
```

> maven은 3.8.1, protoc는 3.17.3 이다.

이제 `mvn clean compile`  명령어를 입력하면 `target` 폴더에 아래와 같은 컴파일 결과물을 확인할 수 있다.

![스크린샷 2021-07-01 오전 7 20 56](https://user-images.githubusercontent.com/30790184/124038909-e6341d00-da3c-11eb-9f4a-789aeae3327b.png)

인터페이스가 정의되었으면 이제 gRPC 서버와 클라이언트를 작성하면 된다. 서버와 클라이언트 모두 protoc로 컴파일한 클래스들을 사용해야 하기 때문에 각각 pom.xml에 아래와 같이 디펜던시를 등록해줘야한다.

```xml
...

<dependencies>
    <dependency>
        <groupId>me.sup2is</groupId>
        <artifactId>protocol-idl</artifactId>
        <version>1.0-SNAPSHOT</version>
        <scope>compile</scope>
    </dependency>
</dependencies>

...
```





## grpc-server

**GreeterImpl.java**

```java
package me.sup2is;

import io.grpc.stub.StreamObserver;

public class GreeterImpl extends GreeterGrpc.GreeterImplBase {

    @Override
    public void hello(final Hello.Request request, final StreamObserver<Hello.Response> responseObserver) {

        final String str = "Hello " + request.getName() + "(" + request.getAge() + ")";
        System.out.println(str);

        final Hello.Response response = Hello.Response.newBuilder()
                .setStr(str)
                .build();

        responseObserver.onNext(response);
        responseObserver.onCompleted();

    }
}
```

protoc가 생성한 `GreeterGrpc` 이너클래스인 `GreeterImplBase` 을 상속받은 뒤 greeter.proto 파일에서 정의한 `hello` rpc 메서드를 재정의해주는 형태로 사용하면 된다. `message`로 정의한 `Request` 또는 `Response`는 빌더형태로 제공되어 쉽게 사용 가능하다.



**GreeterServer.java**

```java
package me.sup2is;

import java.io.IOException;
import java.util.concurrent.TimeUnit;

import io.grpc.Server;
import io.grpc.ServerBuilder;

public class GreeterServer {

    private Server server;

    private void start() throws IOException {
        /* The port on which the server should run */
        int port = 50051;
        server = ServerBuilder.forPort(port)
                .addService(new GreeterImpl())
                .build()
                .start();
        System.out.println("Server started, listening on " + port);
        Runtime.getRuntime().addShutdownHook(new Thread() {
            @Override
            public void run() {
                // Use stderr here since the logger may have been reset by its JVM shutdown hook.
                System.err.println("*** shutting down gRPC server since JVM is shutting down");
                try {
                    GreeterServer.this.stop();
                } catch (InterruptedException e) {
                    e.printStackTrace(System.err);
                }
                System.err.println("*** server shut down");
            }
        });
    }

    private void stop() throws InterruptedException {
        if (server != null) {
            server.shutdown().awaitTermination(30, TimeUnit.SECONDS);
        }
    }

    /**
     * Await termination on the main thread since the grpc library uses daemon threads.
     */
    private void blockUntilShutdown() throws InterruptedException {
        if (server != null) {
            server.awaitTermination();
        }
    }

    /**
     * Main launches the server from the command line.
     */
    public static void main(String[] args) throws IOException, InterruptedException {
        final GreeterServer server = new GreeterServer();
        server.start();
        server.blockUntilShutdown();
    }


}
```

grpc 라이브러리에서 제공해주는 여러 클래스들을 활용해서 서버를 올릴 수 있는데 `ServerBuilder` 에 작성한 `GreeterImpl` 클래스를 등록함으로써 재정의한 메서드를 실행시키게 할 수 있다.



## grpc-client

**GreeterClient.java**

```java
package me.sup2is;

import io.grpc.Channel;
import io.grpc.ManagedChannel;
import io.grpc.ManagedChannelBuilder;
import io.grpc.StatusRuntimeException;

import java.util.concurrent.TimeUnit;

public class GreeterClient {

    private final GreeterGrpc.GreeterBlockingStub greeterBlockingStub;

    public GreeterClient(Channel channel) {
//         'channel' here is a Channel, not a ManagedChannel, so it is not this code's responsibility to
//         shut it down.

//         Passing Channels to code makes code easier to test and makes it easier to reuse Channels.
        greeterBlockingStub = GreeterGrpc.newBlockingStub(channel);
    }

    public static void main(String[] args) throws Exception {
        String user = "hyeon seop";
        int age = 28;

        // Access a service running on the local machine on port 50051
        String target = "localhost:50051";

        ManagedChannel channel = ManagedChannelBuilder.forTarget(target)
                .usePlaintext()
                .build();

        try {
            GreeterClient client = new GreeterClient(channel);
            client.greet(user, age);
        } finally {
            // ManagedChannels use resources like threads and TCP connections. To prevent leaking these
            // resources the channel should be shut down when it will no longer be used. If it may be used
            // again leave it running.
            channel.shutdownNow().awaitTermination(5, TimeUnit.SECONDS);
        }
    }

    public void greet(String name, int age) {
        System.out.println("Will try to greet " + name  + "(" + age + ")"+ " ...");

        final Hello.Request request = Hello.Request.newBuilder()
                .setAge(age)
                .setName(name)
                .build();

        Hello.Response response;

        try {
            response = greeterBlockingStub.hello(request);
        } catch (StatusRuntimeException e) {
            System.out.println("RPC failed:" +  e.getStatus());
            return;
        }
        System.out.println("Greeter Server: " + response.getStr());
    }

}
```

클라이언트는 protoc가 생성한 stub이라는 객체를 사용해서 서버와 통신하도록 구성한다.



</br>

이제 터미널을 실행시켜서 서버와 클라이언트를 실행시켜보도록 하겠다.


![화면 기록 2021-07-01 오전 7 18 45](https://user-images.githubusercontent.com/30790184/124038936-efbd8500-da3c-11eb-87d0-aaabfd73a6c0.gif)

예상한대로 서버와 클라이언트간 통신이 잘 이뤄지는것을 확인할 수 있다.


# 마무리

이 글은 대부분 [gRPC 시작에서 운영까지](http://www.yes24.com/Product/Goods/94489227?OzSrank=1)을 참조했다. 간단한 예제에서는 blockingstub으로 단일 요청, 단일 응답의 구조를 사용했지만 스트림형태로 복수 요청, 복수 응답의 구조를 사용할 수 있고 동기식 요청방식 이외에 비동기식 요청방식도 사용할 수 있다.



<br>

***

포스팅은 여기까지 하겠습니다. 퍼가실때는 출처를 반드시 남겨주세요!

전체 예제 : [https://github.com/sup2is/study/tree/master/grpc/getting-started-grpc-java](https://github.com/sup2is/study/tree/master/grpc/getting-started-grpc-java)

<br>

**References**

- gRPC 시작에서 운영까지 - [카순 인드라시리](http://www.yes24.com/SearchCorner/Result?domain=ALL&author_yn=Y&query=%c4%ab%bc%f8+%c0%ce%b5%e5%b6%f3%bd%c3%b8%ae), [다네쉬 쿠루푸](http://www.yes24.com/SearchCorner/Result?domain=ALL&author_yn=Y&query=%b4%d9%b3%d7%bd%ac+%c4%ed%b7%e7%c7%aa) 
- [https://github.com/grpc-up-and-running/samples](https://github.com/grpc-up-and-running/samples)
