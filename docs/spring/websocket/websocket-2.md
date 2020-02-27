---
layout: post
title: "Spring Boot에서 WebSocket 사용하기 #2"
tags: [Spring Framework, websocket, STOMP]
comments: true
grand_parent: Spring
parent: Websocket
nav_order: 2
date: 2019-06-21
---









# Example (Spring boot 2.3.1)

------

저번시간에 websocket에 대하여 간단하게 알아봤으니 이제 시나리오를 정해서 실제 websocket을 구현해보는 시간을 갖겠다. 

<br>

간단하게 시나리오를 정하자면 화면에는 1-5까지의 채팅방이 있다. 예를들면 1번방은 spring방 2번방은 python방 ... 이렇게 화면에 두고 사용자는 spring방을 클릭하면 spring방 사람들과 대화할 수 있고 python방을 클릭하면 python 방 사람들과 대화 할 수 있다. 

그렇다. 간단하다. 화면은 초 간단하게 구성한다. 







\- pom.xml

```xml
	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-websocket</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
        <dependency>
            <groupId>org.webjars</groupId>
            <artifactId>webjars-locator-core</artifactId>
        </dependency>
        <dependency>
            <groupId>org.webjars</groupId>
            <artifactId>sockjs-client</artifactId>
            <version>1.0.2</version>
        </dependency>
        <dependency>
            <groupId>org.webjars</groupId>
            <artifactId>stomp-websocket</artifactId>
            <version>2.3.3</version>
        </dependency>
        <dependency>
            <groupId>org.webjars</groupId>
            <artifactId>bootstrap</artifactId>
            <version>3.3.7</version>
        </dependency>
        <dependency>
            <groupId>org.webjars</groupId>
            <artifactId>jquery</artifactId>
            <version>3.1.0</version>
        </dependency>
	</dependencies>

```

spring 공식 예제와 동일하게 나 또한 역시 webjar를 사용할 것이다. webjar 말고 cdn을 사용해도 무방하다.

<br>

사실.. 화면도 별반 다를게 없다 ..

<br>

\- index.html

```html
<!DOCTYPE html>
<html>
<head>
	<meta charset="utf-8">
    <title>Hello WebSocket</title>
    <link href="/webjars/bootstrap/css/bootstrap.min.css" rel="stylesheet">
    <link href="/main.css" rel="stylesheet">
    <script src="/webjars/jquery/jquery.min.js"></script>
    <script src="/webjars/sockjs-client/sockjs.min.js"></script>
    <script src="/webjars/stomp-websocket/stomp.min.js"></script>
    <script charset="utf-8" src="/app.js"></script>
</head>
<body>
<noscript><h2 style="color: #ff0000">Seems your browser doesn't support Javascript! Websocket relies on Javascript being
    enabled. Please enable
    Javascript and reload this page!</h2></noscript>
<div id="main-content" class="container">
    <div class="row">
        <div class="col-md-6">
            <form class="form-inline">
                <div class="form-group">
                    <label for="connect">WebSocket connection:</label>
                    <button id="connect" class="btn btn-default" type="submit">Connect</button>
                    <button id="disconnect" class="btn btn-default" type="submit" disabled="disabled">Disconnect
                    </button>
                </div>
            </form>
        </div>
        
        <div class="col-md-6">
            <form class="form-inline">
                <div class="form-group">
                    <label for="name">What is your name?</label>
                    <input type="text" id="name" class="form-control" placeholder="Your name here...">
                </div>
            </form>
        </div>
    </div>
    
    <br>
    <br>
    
    <div class="row">
    	<div class="col-md-2">
     		<a href="#">spring</a>
    	</div>
    	<div class="col-md-2">
    		<a href="#">python</a>
    	</div>
    	<div class="col-md-2">
    		<a href="#">C</a>
    	</div>
    	<div class="col-md-2">
    		<a href="#">vue</a>
   		</div>
    	<div class="col-md-2">
    		<a href="#">ruby</a>
    	</div>
    </div>
    <br>
    <br>
    
    <div class="row">
        <div class="col-md-12">
            <table id="conversation" class="table table-striped">
                <thead>
                <tr>
                    <th colspan="2">Chat (room is : <span id="currentRoom"> </span>)</th>
                </tr>
				<tr>
					<th>
						<input type="text" id="content" class="form-control">
					</th>
					<th>
						<button id="send" type="button" class="form-control" >Send</button>
					</th>
				</tr>
                </thead>
                <tbody id="chats">
                </tbody>
            </table>
        </div>
    </div>
</div>
</body>
</html>
```

<br>

![1](https://user-images.githubusercontent.com/30790184/59863383-9509e100-93bf-11e9-80ca-a49326f70668.png)

<br>

오른쪽상단에 이름을 입력하고 하단에있는 방을 선택한 뒤 왼쪽상단에 Connect 버튼을 클릭하면 해당이름으로 선택한 채팅방에 접속할 수 있다.  방에 입장했을때는 현재 입장한 방의 사용자들끼리만 대화할 수 있다. 다른 방을 입장할 경우 이전 대화내역은 삭제된다. 

<br>

구현이 완료되었을때의 모습이다.

<br>

![녹화_2019_06_24_01_15_19_344](https://user-images.githubusercontent.com/30790184/59979104-37bf9b00-961e-11e9-963f-33561b2eeab2.gif)

~~멘트 정말 구림~~

<br>

먼저 WebSocketConfig를 살펴보자.

<br>

\- WebSocketConfig.class

```java
package com.sup2is.websocket;

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
        registry.addEndpoint("/chat").withSockJS();
    }
}
```

<br>

이것도 공식예제와 별로 다를게 없다.



\- Chat.class

```java
package com.sup2is.websocket.model;

public class Chat {

	private String name;
	private String content;
	
	public Chat(String name, String content) {
		this.name = name;
		this.content = content;
	}
	
	public String getName() {
		return name;
	}
	public void setName(String name) {
		this.name = name;
	}
	public String getContent() {
		return content;
	}
	public void setContent(String content) {
		this.content = content;
	}
	
}
```

<br>

채팅의 내용을 담아줄 entity, vo 객체이다. 그냥 딱 vo 용도이다.



\- ChatController.class

```java
package com.sup2is.websocket;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.messaging.handler.annotation.DestinationVariable;
import org.springframework.messaging.handler.annotation.MessageMapping;
import org.springframework.messaging.handler.annotation.SendTo;
import org.springframework.messaging.simp.SimpMessagingTemplate;
import org.springframework.stereotype.Controller;

import com.sup2is.websocket.model.Chat;

@Controller
public class ChatController {
	
	@Autowired
	private SimpMessagingTemplate simpMessagingTemplate;
	
    @MessageMapping("/chat/{room}/{name}")
    public void chat(@DestinationVariable("room") String room,
    		@DestinationVariable("name") String name , String content) throws Exception {
    	simpMessagingTemplate.convertAndSend("/topic/" + room , new Chat(name, content));
    }

}
```

<br>

사실 주의깊게 살펴볼 부분이 여기인것같다. 공홈의 예제랑 조금 다른부분이 있다. 일단 @RequestMapping에서나 사용할법한 {}가 @MessageMapping("/chat/{room}/{name}") 에서도 들어가 있다. Web에서 사용할때는 @PathVariable을 사용하지만 @MessageMapping에서 추출할 때는 @DestinationVariable을 사용한다. 

<br>

또 Spring에서 제공해주는 SimpMessagingTemplate.class를 사용한다. SimpMessagingTemplate을 기존에 사용하던 @SendTo 어노테이션을 대체해주는데 여러가지 메서드를 지원해준다 위 예제에서 사용한 convertAndSend()같은 경우는 파라미터를 두개받는데 첫번째가 destination 두번째는 payload를 뜻한다. 자세한 내용은 [spring doc](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/messaging/simp/SimpMessagingTemplate.html)을 확인하자.

<br>

놀랍지만 java쪽은 이게 끝이다 .. 다음은 client쪽이다.

\- app.js

```javascript
var stompClient = null;
var currentRoom = null;
var subscription = null;


function setRoom(roomName) {
	currentRoom = roomName.text();
	$("#currentRoom").text(currentRoom);
	
	if(subscription !== null) {
		subscription.unsubscribe();
		subscription = stompClient.subscribe('/topic/' + currentRoom, function (chat) {
            showChat(JSON.parse(chat.body).name + " : " + JSON.parse(chat.body).content);
        });
		$("#chats").html("");
	}
	
}

function setConnected(connected) {
	
    $("#connect").prop("disabled", connected);
    $("#disconnect").prop("disabled", !connected);
    if (connected) {
        $("#name").attr("disabled" , true);
    }
    else {
        $("#name").attr("disabled" , false);
    }
}

function connect() {
	
	if(!validation()){
		return
	}

    var socket = new SockJS('/chat');
    stompClient = Stomp.over(socket);
    stompClient.connect({}, function (frame) {
        setConnected(true);
        subscription = stompClient.subscribe('/topic/' + currentRoom, function (chat) {
            showChat(JSON.parse(chat.body).name + " : " + JSON.parse(chat.body).content);
        });
    });
    
}

function validation() {

	if($("#name").val() === "") {
		alert("이름을 입력해주세요");
		return false;
	}
	
	if(currentRoom === null) {
		alert("방을 선택해주세요");
		return false;
	}
	
	return true;
}


function disconnect() {
    if (stompClient !== null) {
        stompClient.disconnect();
    }
    setConnected(false);
}

function send() {
	var name = $("#name").val();
	var content = $("#content").val();
    stompClient.send("/app/chat/"+ currentRoom +"/" + name , {}, content);
    $("#content").val("");
}

function showChat(message) {
    $("#chats").append("<tr><td colspan='2'>" + message + "</td></tr>");
}

$(function () {
    $("form").on('submit', function (e) {e.preventDefault();});
    $("#connect").click(function() { connect(); });
    $("#disconnect").click(function() { disconnect(); });
    $("#send").click(function() { send(); });
    $("a[href=\\#]").click(function() {setRoom($(this))})
});


```

<br>

여기도 마찬가지로 공홈소스에서 약간만 변형된부분을 갖는다. 몇가지만 살펴보면

```javascript
function connect() {
	
	if(!validation()){
		return
	}

    var socket = new SockJS('/chat');
    stompClient = Stomp.over(socket);
    stompClient.connect({}, function (frame) {
        setConnected(true);
        subscription = stompClient.subscribe('/topic/' + currentRoom, function (chat) {
            showChat(JSON.parse(chat.body).name + " : " + JSON.parse(chat.body).content);
        });
    });
    
}
```



connect() 메서드는 초기에 stomp client 객체를 얻어오고 callback으로 subscribe를 지정한다 이때 destination은 '/topic/ {currentRoom}'이 되는데 spring방을 클릭하면  '/topic/spring' 을 구독하는 상태가 된다. 이런식으로 지정하면 spring방을 클릭한 모든사람들은 '/topic/spring' 을 구독하므로 같은 방에 있는 효과를 얻을 수 있다.

<br>

공홈이랑 다른부분은 stompClient.subscribe()가 반환하는 객체를 subscription 변수에 저장하는부분인데 이게 필요한부분은 바로 이부분이다.

```javascript
function setRoom(roomName) {
	currentRoom = roomName.text();
	$("#currentRoom").text(currentRoom);
	
	if(subscription !== null) {
		subscription.unsubscribe();
		subscription = stompClient.subscribe('/topic/' + currentRoom, function (chat) {
            showChat(JSON.parse(chat.body).name + " : " + JSON.parse(chat.body).content);
        });
		$("#chats").html("");
	}
	
}
```

setRoom()은 방을 클릭할때 동작하는 메서드인데 subscription객체가 null이 아니면 기존에 '/topic/spring' 을 구독한 상태를 unsubscribe()로 해제시키고 바뀐 currentRoom을 구독하는 형태가 된다. 이런식이면 spring방에 있던 유저가 python방을 클릭할때 python방을 구독하므로 우리가 원하는 형태로 구현이 가능해진다.



<br>

이번에 websocket으로 작은 toyproject를 진행하기전에 맛보기로 포스팅을 먼저 해본건데 사실 spring에서 다해줘서 할게 없다 ... 너무너무 편리한기능이고 정말 심도있게 다뤄볼만 할 정도로 강력하고 좋은 기능인것같다.



<br>

포스팅은 여기까지 하겠습니다.  모든예제는 제 github에서 확인하실 수 있습니다. 

<https://github.com/sup2is/spring-example/tree/master/websocket-1>



<br>

퍼가실때는 링크와 출처를 반드시 명시해주세요. 감사합니다.

<br>



출처 : <https://stackoverflow.com/questions/27047310/path-variables-in-spring-websockets-sendto-mapping>