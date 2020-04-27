---
layout: post
title: "Spring Boot에서 WebSocket 사용하기 #1"
tags: [Spring Framework, websocket, STOMP]
comments: true
grand_parent: Spring
parent: Websocket
nav_order: 1
date: 2019-06-05
---













사실 내가 쓴 글 말고 정말정말 정리가 잘 된 글이 있다. spring doc을 기준으로 글을 작성하려고 했으나 뭔말인지 몰라서 찾아봤는데 이분이 쓴 글이 정말 정리가 잘 되어 있었다.  [이곳](https://supawer0728.github.io/2018/03/30/spring-websocket/) 을 확인하면 좋을 듯 하다. 이 글에도 해당 사이트의 글을 매우 많이 참고했다. ~~정리가 너무 잘되어있어서 깃헙을 방문했는데 nhn이신거같다 .. 이정도로 글 쓰면 nhn 갈 수 있을꺼 같다 ..~~ 

<br>





# WebSocket

***



RFC6455 에 정의된 웹소켓은 웹어플리케이션에서 클라이언트와 서버간의 양방향 통신을 하는데 아주 중요한 역할을 한다. full-duflex 통신, 즉 양방향 통신을 웹상에서도 사용 할 수 있다는 것이다.  websocket이 사용되기에 가장 적합한 환경은 클라이언트와 서버 사이의 높은 빈도의 메세지교환, 낮은 대기시간으로 이벤트를 교환해야하는 웹 어플리케이션이다. 예를 들면 웹기반 채팅프로그램이다. 참고로 IE는 10부터 지원한다.

<br>

websocket은 HTTP status값 101(switching protocol)을 사용한다. 초기에 핸드쉐이크가 성공한다고 가정하면 HTTP 요청은 열린 상태로 유지되며 클라이언트와 서버 모두 이를 사용하여 메세지를 서로 보낼 수 있다

<br>



> As explained in the [introduction](https://docs.spring.io/spring-framework/docs/5.0.0.M1/spring-framework-reference/html/websocket.html#websocket-intro-sub-protocol), direct use of a WebSocket API is too low level for applications — until assumptions are made about the format of a message there is little a framework can do to interpret messages or route them via annotations. This is why applications should consider using a sub-protocol and Spring’s [STOMP over WebSocket](https://docs.spring.io/spring-framework/docs/5.0.0.M1/spring-framework-reference/html/websocket.html#websocket-stomp) support.When using a higher level protocol, the details of the WebSocket API become less relevant, much like the details of TCP communication are not exposed to applications when using HTTP. Nevertheless this section covers the details of using WebSocket directly.

<br>

spring doc에서 확인해보면 다음과같이 WebSocket API의 직접적인 사용은 지양하는 편이고 Spring의 STOMP 및 서브 프로토콜 사용을 고려해야한다고 말해주고 있다. 하지만 spring doc과 마찬가지로 WebSocket API를 직접 사용하는 예제를 살펴보겠다.



<br>

# WebSocket API

***



WebSocket 서버를 생성하는것은 WebSocketHandler를 구현하면 된다. spring에서 제공하는 TextWebSocketHandler  또는 BinaryWebSocketHandler 상속받아서 확장하는게 좋을 듯 하다.



> ```java
> import org.springframework.web.socket.WebSocketHandler;
> import org.springframework.web.socket.WebSocketSession;
> import org.springframework.web.socket.TextMessage;
> 
> public class MyHandler extends TextWebSocketHandler {
> 
>     @Override
>     public void handleTextMessage(WebSocketSession session, TextMessage message) {
>         // ...
>     }
> 
> }
> ```



<br>

> ```java
> import org.springframework.web.socket.config.annotation.EnableWebSocket;
> import org.springframework.web.socket.config.annotation.WebSocketConfigurer;
> import org.springframework.web.socket.config.annotation.WebSocketHandlerRegistry;
> 
> @Configuration
> @EnableWebSocket
> public class WebSocketConfig implements WebSocketConfigurer {
> 
>     @Override
>     public void registerWebSocketHandlers(WebSocketHandlerRegistry registry) {
>         registry.addHandler(myHandler(), "/myHandler");
>     }
> 
>     @Bean
>     public WebSocketHandler myHandler() {
>         return new MyHandler();
>     }
> 
> }
> ```





정말 spring doc확인해보면 예제로 위에 딱 두개있다. 웹소켓을 처음 써보는 나는 뭐가 뭔지 모르니까 진짜 그대로 한번 실행해 봤다.

<br>

WebSocket 테스트는 Postman으로 한계가 있다. 크롬 웹 스토어에 'Smart Websocket Client' 로 테스트를 진행해봤다.



\- MyHandler.class

```java
package com.sup2is.websocket;

import org.springframework.web.socket.TextMessage;
import org.springframework.web.socket.WebSocketSession;
import org.springframework.web.socket.handler.TextWebSocketHandler;

public class MyHandler extends TextWebSocketHandler {

	@Override
	public void handleTextMessage(WebSocketSession session, TextMessage message) {

		System.out.println(session.toString());
		System.out.println(message.toString());
		
	}

}

```



<br>

나같은 경우는 예제와 클래스이름도 똑같이했고 단순히 넘어오는 session값과 message값을 콘솔에 찍는거였다. 

<br>

Smart Websocket Client로 실행해봤다. 커넥션을 연결할 때 프로토콜은 http가 아니라 ws를 사용한다.

![1](https://user-images.githubusercontent.com/30790184/58943374-8b2a9000-87ba-11e9-8fc8-261d0804ea90.png)

단순히 hello world로 찍어봤는데 값이 잘 들어온다

\- console

```
StandardWebSocketSession[id=2168a326-65ec-f332-a8e3-d8fa6d485225, uri=ws://127.0.0.1:8080/myHandler]
TextMessage payload=[hello worl..], byteCount=11, last=true]
```

정말 간단하다 ..

<br>

추가적으로 handshake의 after, before 정보를 HandshakeInterceptor를 통해 가로 챌 수 있다. spring에서 제공하는 HttpSessionHandshakeInterceptor 을 다음과 같이 넣어준다.

<br>

\- WebSocketConfig.class

```java
    @Override
    public void registerWebSocketHandlers(WebSocketHandlerRegistry registry) {
        registry.addHandler(myHandler(), "/myHandler")
        	.addInterceptors(new HttpSessionHandshakeInterceptor())
        	.setAllowedOrigins("*");
    }

```

마찬가지로 cors 설정도 할 수 있다. 

<br>



# SockJS

***

WebSocket에서는 위에서 언급했듯이 모든브라우저가 지원하지 않기때문에 Fallback Options으로 SockJS를 활용 할 수 있다. (ie8 or ie9) SockJS는 javascript library다. 

<br>

> ```javascript
> @Configuration
> @EnableWebSocket
> public class WebSocketConfig implements WebSocketConfigurer {
> 
>     @Override
>     public void registerWebSocketHandlers(WebSocketHandlerRegistry registry) {
>         registry.addHandler(myHandler(), "/myHandler").withSockJS();
>     }
> 
>     @Bean
>     public WebSocketHandler myHandler() {
>         return new MyHandler();
>     }
> 
> }
> ```

<br>

사실 서버에서 작성하는건 .withSockJS()를 추가해주는것 뿐이다. client에서 SockJS를 사용할지의 유무만 선택하면 된다.



# STOMP

***



STOMP는 Simple Text-Oriented Messaging Protocol의 약자이다. WebSocket은 Text또는 binary이라는 두가지 유형의 메세지를 정의하지만 그 내용은 직접적으로 정의하지 않는다 따라서 STOMP라는 프로토콜을 적용한다.

> ```
> COMMAND
> header1:value1
> header2:value2
> 
> Body^@
> ```

<br>

> ```
> SUBSCRIBE
> id:sub-1
> destination:/topic/price.stock.*
> 
> ^@
> ```

<br>

> ```
> SEND
> destination:/queue/trade
> content-type:application/json
> content-length:44
> 
> {"action":"BUY","ticker":"MMM","shares",44}^@
> ```



STOMP의 골격, 구조는 위와 같은 형태로 생성된다.

<br>

클라이언트는 SEND또는 SUBSCRIPT 명령을 사용하여 메세지의 내용과 수신 대상을 설명하는 HEADER와 함께 메세지를 보내거나 구독할 수 있다. 이렇게하면 다른 연결된 클라이언트로 메세지를 보내거나 서버로 메세지를 보내 일부 작업을 수행하도록 요청할 수 있는 간단한 게시, 구독 기능을 사용할 수 있다. 정말 근사하다...

<br>



spring에서는 spring-messaging, spring-websocket모듈을 통해서 WebSocket의 STOMP를 제공해주고 있다. STOMP를 사용하기위해서는 앞에서 언급한 WebSocketConfig.class가 약간은 변형이된다.

<br>

> ```java
> import org.springframework.web.socket.config.annotation.EnableWebSocketMessageBroker;
> import org.springframework.web.socket.config.annotation.StompEndpointRegistry;
> 
> @Configuration
> @EnableWebSocketMessageBroker
> public class WebSocketConfig implements WebSocketMessageBrokerConfigurer {
> 
>     @Override
>     public void registerStompEndpoints(StompEndpointRegistry registry) {
>         registry.addEndpoint("/portfolio").withSockJS();
>     }
> 
>     @Override
>     public void configureMessageBroker(MessageBrokerRegistry config) {
>         config.setApplicationDestinationPrefixes("/app");
>         config.enableSimpleBroker("/topic", "/queue");
>     }
> 
> }
> ```

<br>

클라이언트에서는 다음과 같이 연결하면 된다.

> ```javascript
> var socket = new SockJS("/spring-websocket-portfolio/portfolio");
> var stompClient = Stomp.over(socket);
> 
> stompClient.connect({}, function(frame) {
> }
> ```

<br>

또는

> ```javascript
> var socket = new WebSocket("/spring-websocket-portfolio/portfolio");
> var stompClient = Stomp.over(socket);
> 
> stompClient.connect({}, function(frame) {
> }
> ```

위 아래의 차이점은 SockJS의 사용 유무이다. 이왕하는거 접근성이 더 좋은 SockJS를 사용하는것을 고려해볼 만 하다.

<br>

# Annotaion

-----

WebSocket에서 사용하는 몇가지 어노테이션을 정리해보고 실제 예제를 통해 server 와 client 모두 구현해보는 시간을 가져보겠다.

<br>

## @EnableWebSocketMessageBroker

***

@Configuration이 선언된 클래스에 추가하면 broker기반의 high-level(STOMP인듯?) WebSocket Messaging을 구현할 수 있다.

<br>

## @MessageMapping

***

@RequestMapping과 비슷하게 동작하는데 입력된 value 값을 포함하는 메세지의 endpoint가 된다. 또는 @SendTo 어노테이션을 사용해서 임의로 전달 할 수도 있다.

<br>

## @SendTo

***

client에서 구독하는 endpoint? 가된다. 자세한 사용법은 아래 예제에서 언급한다.

<br>

# Example

다른 블로그들과 비슷하게 web으로 구현한 소켓 채팅 프로그램을 사용할 것이다.  [여기](https://spring.io/guides/gs/messaging-stomp-websocket/)의 내용을 토대로 작성했으니 참고해도 좋다.  



\- WebSocketConfig.class

```java
package hello;

import org.springframework.context.annotation.Configuration;
import org.springframework.messaging.simp.config.MessageBrokerRegistry;
import org.springframework.web.socket.config.annotation.EnableWebSocketMessageBroker;
import org.springframework.web.socket.config.annotation.StompEndpointRegistry;
import org.springframework.web.socket.config.annotation.WebSocketMessageBrokerConfigurer;

@Configuration
@EnableWebSocketMessageBroker
public class WebSocketConfig implements WebSocketMessageBrokerConfigurer {

    @Override
    public void configureMessageBroker(MessageBrokerRegistry config) {
        config.enableSimpleBroker("/topic");
        config.setApplicationDestinationPrefixes("/app");
    }

    @Override
    public void registerStompEndpoints(StompEndpointRegistry registry) {
        registry.addEndpoint("/gs-guide-websocket").withSockJS();
    }

}
```

위에서 언급했듯이@Configuration @EnableWebSocketMessageBroker를 세트로 사용해서 MessageBroker 기반의 Config 선언을 해준다. WebSocketMessageBrokerConfigurer.interface를 구현받아서 MessageBroker를 configureMessageBroker() 메서드로 정의할 수 있다. 위에서 확인할 수 있는 setApplicationDestinationPrefixes()는 말그대로 이 어플리케이션의 접두어 정도의 역할을 한다. enableSimpleBroker()는 메모리기반의 Message Broker 를 선언해주는데 파라미터로 넘어간 "/topic" 이 접두어로 붙어있는 클라이언트들에게 메세지를 전달해주는 역할을 한다.

<br>

registerStompEndpoints() 는 최초의 websocket을 생성하는 endpoint를 지정해준다 여기에서 sockJS의 사용유무를 결정할 수 있다.

<br>

\- GreetingController.class

```java
package hello;

import org.springframework.messaging.handler.annotation.MessageMapping;
import org.springframework.messaging.handler.annotation.SendTo;
import org.springframework.stereotype.Controller;
import org.springframework.web.util.HtmlUtils;

@Controller
public class GreetingController {


    @MessageMapping("/hello")
    @SendTo("/topic/greetings")
    public Greeting greeting(HelloMessage message) throws Exception {
        Thread.sleep(1000); // simulated delay
        return new Greeting("Hello, " + HtmlUtils.htmlEscape(message.getName()) + "!");
    }

}
```

Controller 역할을 하는 GreetingController다. 위에서 언급한 @MessageMapping을 통해 실제 유저가 어떤 메세지를 전달할 endpoint가 된다. 메서드 내부는 그냥 받아온 메세지를 Greeting 객체로 생성해서 넘겨주는 역할을 한다. @SendTo도 마찬가지로 client들의 구독 endpoint가 된다.



<br>

... 놀랍지만 java소스는 저게 끝이다

<br>



클라이언트에서 주의깊게 봐야 할 부분은 바로 javascript부분이다.

\- app.js

```javascript
var stompClient = null;

function setConnected(connected) {
    $("#connect").prop("disabled", connected);
    $("#disconnect").prop("disabled", !connected);
    if (connected) {
        $("#conversation").show();
    }
    else {
        $("#conversation").hide();
    }
    $("#greetings").html("");
}

function connect() {
    var socket = new SockJS('/gs-guide-websocket');
    stompClient = Stomp.over(socket);
    stompClient.connect({}, function (frame) {
        setConnected(true);
        console.log('Connected: ' + frame);
        stompClient.subscribe('/topic/greetings', function (greeting) {
            showGreeting(JSON.parse(greeting.body).content);
        });
    });
}

function disconnect() {
    if (stompClient !== null) {
        stompClient.disconnect();
    }
    setConnected(false);
    console.log("Disconnected");
}

function sendName() {
    stompClient.send("/app/hello", {}, JSON.stringify({'name': $("#name").val()}));
}

function showGreeting(message) {
    $("#greetings").append("<tr><td>" + message + "</td></tr>");
}

$(function () {
    $("form").on('submit', function (e) {
        e.preventDefault();
    });
    $( "#connect" ).click(function() { connect(); });
    $( "#disconnect" ).click(function() { disconnect(); });
    $( "#send" ).click(function() { sendName(); });
});
```

spring doc의 예제에서는 SockJS를 사용하고 있다.

<br>

```javascript
function connect() {
    var socket = new SockJS('/gs-guide-websocket');
    stompClient = Stomp.over(socket);
    stompClient.connect({}, function (frame) {
        setConnected(true);
        console.log('Connected: ' + frame);
        stompClient.subscribe('/topic/greetings', function (greeting) {
            showGreeting(JSON.parse(greeting.body).content);
        });
    });
}

```

커넥션이 일어나는 부분이다. new SockJS('/gs-guide-websocket'); 를 이용해 위의 config에서 정의한 '/gs-guide-websocket' 과 커넥션을 여는 부분이다. 바로 아래에 Stomp.over() 로 client를 가져온다. 이어서 connect() 메서드를 호출한다.



> ```javascript
>     var headers = {
>       login: 'mylogin',
>       passcode: 'mypasscode',
>       // additional header
>       'client-id': 'my-client-id'
>     };
>     client.connect(headers, connectCallback);
> ```

connect() 메서드는 두개의 파라미터를 받는데 하나는 인증? 관련한 부분같다 예제에서는 넘어가고 위는 참고만 하길 바란다.



커넥션이 성공적으로 이루어지면 callback으로 위의 함수블록을 실행한다. 이제 위에서 client는 구독할 endpoint를 지정할 수 있다. 우리는 위에서 @SendTo("/topic/greetings") 로 지정해줬으니 client도 마찬가지로 "/topic/greetings"를 구독하여 이쪽으로 넘어오는 모든 메세지를 구독받을 수 있다.

<br>

 마찬가지로 뒤에 넘어오는 함수블록은 "/topic/greetings" 으로 넘어온 메시지가 있을 경우 호출되는 메서드이다.

<br>

아래는 시뮬레이션이다. (아무것도 안보태고 진짜 spring doc 예제로만 실행했음)



![녹화_2019_06_14_01_18_31_5](https://user-images.githubusercontent.com/30790184/59698551-de74f780-922a-11e9-95f1-802792d6354e.gif)





<br>

spring 공식 홈페이지의 해설은 이정도가 될 것 같다. 다음시간에는 조금 더 응용한 websocket을 사용해 보도록 하겠다.

<br>

포스팅은 여기까지 하겠습니다.  ~~모든예제는 제 github에서 확인하실 수 있습니다.~~ 이건 <https://github.com/spring-guides/gs-messaging-stomp-websocket.git> 여기에서 확인하자



<br>

퍼가실때는 링크와 출처를 반드시 명시해주세요. 감사합니다.

<br>





출처 : <https://docs.spring.io/spring-framework/docs/5.0.0.M1/spring-framework-reference/html/websocket.html#websocket-server-handshake>

출처 : <https://supawer0728.github.io/2018/03/30/spring-websocket/>

출처 : <https://hwiveloper.github.io/2019/01/10/spring-boot-stomp-websocket/>

출처 : <https://spring.io/guides/gs/messaging-stomp-websocket/>

출처 : <http://jmesnil.net/stomp-websocket/doc/>