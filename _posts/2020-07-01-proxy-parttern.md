---
layout: post
title: "프록시(Proxy) 패턴 Feat.Java"
tags: [Java, Design Pattern, Proxy Pattern, Structural Pattern]
date: 2020-07-01
comments: true
---



<br>

# OverView

이번시간엔 구조패턴 중 하나인 프록시 패턴에 대해서 알아보도록 하겠다!

# 프록시(Proxy) 패턴 Feat.Java

만약 Spring을 사용해보신 개발자분이라면 Proxy 라는 용어가 크게 이질감이 느껴지지 않을 것이다. Spring JPA, Spring AOP 등등 많은곳에서 Proxy 패턴을 사용한다.

프록시 패턴은 **다른 객체에 대한 접근을 제어하거나 가상으로 객체를 만들어 실제 요청이 있을때만 높은 비용의 처리를 할 수 있게 실제 필요한 객체를 프록시 객체로 한번 감싸서 사용하는 패턴**이다. 이 글에서 원격지 프록시(Remote proxy)에 대해서는 다루지 않는다.

위에서 설명한 프록시 패턴을 각각 **보호용 프록시(Protect Proxy) 가상 프록시(Virtual Proxy)** 라고 한다. **보호용 프록시는 접근 권한이 있는지에 대해 검사할 수 있고 가상 프록시는 실제 요청이 있을때만 필요한 객체를 생성**하도록 할 수 있다.

이번 예제에서는 가상 프록시 패턴을 사용해서 JPA의 Lazy Loading을 흉내내보도록 하겠다.

# 가상 프록시 패턴으로 흉내내는 JPA Lazy Loading

jpa는 실제 엔티티 요소에 접근하기 이전까지 가상 프록시 객체를 두고 접근하는 시점에 실제 엔티티 요소에 대한 정보를 가져오도록 설계되었다. 따라서 객체 초기화 생성 비용을 절약할 수 있다.

예제 구성은 다음과 같다.

- **게시판 Board 클래스는 List 타입의 Reply 객체를 갖고 있다.**
- **Reply 타입의 BoardReply 객체는 생성하는데 약 1초가 걸린다고 가정한다.**
- **현재 당장 필요한 것은 Board이지 Reply이 아니다.**
- **따라서 Reply 타입의 ReplyProxy 객체를 통해 가상 프록시 패턴을 구현해서 lazy loading 전략을 구현하고 요청이 있을때만 실제 BoardReply 객체를 생성한다.**

![1200px-Proxy_pattern_diagram svg](https://user-images.githubusercontent.com/30790184/86193408-ceeef500-bb86-11ea-8ebb-876282dffa49.png)



- Proxy: 실제로 참조할 대상에 대한 참조자를 관리하고 Subject와 동일한 인터페이스를 제공하여 실제 대상을 대체할 수 있어야 한다.
- Subject: RealSubject와 Proxy에 공통적인 인터페이스를 정의한다.
- RealSubject: 프록시가 감싸고 있는 실제 객체이다.



## 인터페이스 구성하기

먼저 인터페이스를 다음과 같이 구성했다.

**Reply.java**

```java
package me.sup2is;

public abstract class Reply {

    int replyId;
    int parentId;
    String content;

    public Reply(int replyId, int parentId, String content) {
        this.replyId = replyId;
        this.parentId = parentId;
        this.content = content;
    }

    public abstract String getContent();

}

```

리플에 필요한 몇가지 멤버변수와 초기화 생성자, 그리고 `getContent()` 라는 추상메서드를 갖고 있는 추상 클래스로 인터페이스를 구성했다.

이제 이 Reply 클래스로 실제 구현체인 BoardReply과 프록시로 구성할 ProxyReply 클래스를 만들어보자.

**BoardReply.java**

```java
package me.sup2is;

public class BoardReply extends Reply{

    public BoardReply(int replyId, int parentId, String content) {
        super(replyId, parentId, content);
        delay();
    }

    @Override
    public String getContent() {
        return this.content;
    }

    private void delay() {
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}

```

**ReplyProxy.java**

```java
package me.sup2is;

public class ReplyProxy extends Reply {

    private Reply reply;

    public ReplyProxy(int replyId, int parentId, String content) {
        super(replyId, parentId, content);
    }

    @Override
    public String getContent() {
        if(reply == null) {
            reply = new BoardReply(this.replyId, this.parentId, this.content);
        }
        return reply.getContent();
    }
}

```

조금 극단적이지만 BoardReply 객체는 생성자 초기화 시점에서 약 1초가량 딜레이를 주도록 하였다. 그리고 ReplyProxy 객체는 생성자 초기화 시점에 아무런 딜레이가 없지만 실제 `getContent()` 메서드 호출시에 BoardReply을 생성후 BoardReply의 `getContent()` 메서드를 수행하도록 구현했다.

## Board 엔티티 만들고 프로그램 실행하기

간단하게 Board 엔티티를 구성해보고 프로그램을 실행해보자.

**Board.java**

```java
package me.sup2is;

import java.util.List;

public class Board {

    private int boardId;
    private String title;
    private String content;
    private List<Reply> replies;

    public Board(int boardId, String title, String content, List<Reply> replies) {
        this.boardId = boardId;
        this.title = title;
        this.content = content;
        this.replies = replies;
    }

    public int getBoardId() {
        return boardId;
    }

    public String getTitle() {
        return title;
    }

    public String getContent() {
        return content;
    }

    public List<Reply> getReplies() {
        return replies;
    }

    @Override
    public String toString() {
        return "Board{" +
                "boardId=" + boardId +
                ", title='" + title + '\'' +
                ", content='" + content + '\'' +
                ", replies=" + replies +
                '}';
    }
}

```

**Main.java**

```java
package me.sup2is;

import java.util.ArrayList;
import java.util.List;

public class Main {

    public static void main(String[] args) {
        
        long start = System.currentTimeMillis();
        //DB에서 board를 가져온다고 가정
        Board board = getBoardFromDatabase();
        long end = System.currentTimeMillis();

        System.out.println("Proxy 인스턴스 생성 시간 : " + (end - start) / 1000 + "초");
        System.out.println(board.toString());

        System.out.println("실제 BoardReply 인스턴스에 접근");
        start = System.currentTimeMillis();

        for (Reply reply : board.getReplies()) {
            System.out.println(reply.getContent());
        }

        end = System.currentTimeMillis();

        System.out.println("실제 BoardReply 생성 시간 : " + (end - start) / 1000 + "초");

    }

    //데이터 초기화
    private static Board getBoardFromDatabase() {

        int boardId = 1;
        List<Reply> replies = new ArrayList<>();
        //Proxy 객체로 replies 초기화
        for (int i = 0; i < 10; i++) {
            replies.add(new ReplyProxy(i + 1, boardId, "댓글 " + (i + 1)));
        }

        Board board = new Board(boardId, "게시글1", "게시글입니다.", replies);
        return board;
    }

}


```

프로그램 실행 결과는 다음과 같다.

```
Board 인스턴스 생성
ProxyReply 인스턴스 생성 시간 : 0초
Board{boardId=1, title='게시글1', content='게시글입니다.', replies=[me.sup2is.ReplyProxy@7d4991ad, me.sup2is.ReplyProxy@28d93b30, me.sup2is.ReplyProxy@1b6d3586, me.sup2is.ReplyProxy@4554617c, me.sup2is.ReplyProxy@74a14482, me.sup2is.ReplyProxy@1540e19d, me.sup2is.ReplyProxy@677327b6, me.sup2is.ReplyProxy@14ae5a5, me.sup2is.ReplyProxy@7f31245a, me.sup2is.ReplyProxy@6d6f6e28]}
실제 BoardReply 인스턴스에 접근
댓글 1
댓글 2
댓글 3
댓글 4
댓글 5
댓글 6
댓글 7
댓글 8
댓글 9
댓글 10
실제 BoardReply 생성 시간 : 10초
```

Board 인스턴스를 생성할때 Reply을 ProxyReply로 생성했기때문에 예상한대로 Board 인스턴스 생성은 매우 빠르게 진행된다. 이후에 실제 Reply의 값이 필요할때 실제 BoardReply을 생성하고 `getContent()` 메서드를 호출함으로써 가상 프록시 패턴을 구현했다.

# 마무리

실제로 JPA 소스를 하나하나씩 직접 까보지 않았고 구현 방법도 상당히 다를 것이지만 패턴 자체의 이해하기엔 크게 어렵다고 생각하지 않는다. Proxy 패턴은 정말 많이 사용하는 패턴이라 이번 기회에 알아두면 좋을 듯 하다.



<hr>
포스팅은 여기까지 하겠습니다. 퍼가실때는 출처를 반드시 남겨주세요!


예제: [https://github.com/sup2is/study/tree/master/design-pattern/virtual-proxy-pattern](https://github.com/sup2is/study/tree/master/design-pattern/virtual-proxy-pattern)

<br>

**References**

- GoF의 디자인 패턴 - 에릭 감마, 리처드 헬름, 랄프 존슨, 존 블리시디스
- [https://jdm.kr/blog/235](https://jdm.kr/blog/235)