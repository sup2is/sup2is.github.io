---

layout: post
title: "Thymeleaf Document 훑어보기 #2"
tags: [JAVA, Spring, Thymeleaf]
comments: true
parent: Thymeleaf
nav_order: 2
date: 2019-12-05
---



이전시간과 이어서 포스팅하도록 하겠습니다. [이전편](https://sup2is.github.io/thymeleaf-with-spring-1/)

<hr>


## 3. Expressions on selections (asterisk syntax)

앞서 봤던 variable expression ${...} 이외에도 asterisk로 접근하는 표현식을 사용할 수 있다 ex : *{...}

두개의 차이점은\*{...}로 표현식을 사용하면 page 내부에서 전체 Context를 나타내는게아니라 selected object가 선언된 Context 내부에서의 표현식이 가능하다. 만약 *{...}를 사용하지만 selected object가 선언된 Context내부가 아니라면 ${...} 와 같게 동작한다. 코드를 보자.

```html
  <div th:object="${session.user}">
    <p>Name: <span th:text="*{firstName}">Sebastian</span>.</p>
    <p>Surname: <span th:text="*{lastName}">Pepper</span>.</p>
    <p>Nationality: <span th:text="*{nationality}">Saturn</span>.</p>
  </div>
```

위에서 언급한 selected object는 바로 **th:object**로 선언된 ${session.user} 이다.

위 코드는 아래 코드와 동일하게 동작한다.

```html
<div>
  <p>Name: <span th:text="${session.user.firstName}">Sebastian</span>.</p>
  <p>Surname: <span th:text="${session.user.lastName}">Pepper</span>.</p>
  <p>Nationality: <span th:text="${session.user.nationality}">Saturn</span>.</p>
</div>
```

<br>

${...} 와 *{...}를 혼용해서 쓸 수도 있고

```html
<div th:object="${session.user}">
  <p>Name: <span th:text="*{firstName}">Sebastian</span>.</p>
  <p>Surname: <span th:text="${session.user.lastName}">Pepper</span>.</p>
  <p>Nationality: <span th:text="*{nationality}">Saturn</span>.</p>
</div>
```

th:object가 선언된 Context안엣서 #object 를통해서 객체에 접근할수도 있다.

```html
<div th:object="${session.user}">
  <p>Name: <span th:text="${#object.firstName}">Sebastian</span>.</p>
  <p>Surname: <span th:text="${session.user.lastName}">Pepper</span>.</p>
  <p>Nationality: <span th:text="*{nationality}">Saturn</span>.</p>
</div>
```

위에서 언급한것처럼 th:object가 선언되지 않은 Context면 $와 *는 별 차이가 없다.

```html
<div>
  <p>Name: <span th:text="*{session.user.name}">Sebastian</span>.</p>
  <p>Surname: <span th:text="*{session.user.surname}">Pepper</span>.</p>
  <p>Nationality: <span th:text="*{session.user.nationality}">Saturn</span>.</p>
</div>
```

사용하시는 분들의 입맛에 맞게 사용하면 될 듯 하다.

<br>



## 4. Link URLs

`Thymeleaf`에서 URL을 표현할때는 @{...} 를 사용한다. 

`Thymeleaf` 문서에서는 URL을 다음과 같이 표현했다

> There are different types of URLs:
>
> - Absolute URLs: `http://www.thymeleaf.org`
> - Relative URLs, which can be:
>   - Page-relative: `user/login.html`
>   - Context-relative: `/itemdetails?id=3` (context name in server will be added automatically)
>   - Server-relative: `~/billing/processInvoice` (allows calling URLs in another context (= application) in the same server.
>   - Protocol-relative URLs: `//code.jquery.com/jquery-2.0.3.min.js`

왜 이렇게 나눈지는 정확하게 잘 모르니까 패스 ..

아마  **org.thymeleaf.linkbuilder.ILinkBuilder**를 뒤져보면 답을 얻을 수도 있겠지만 일단 패스 ..

`Thymeleaf`는 일반적으로  **org.thymeleaf.linkbuilder.StandardLinkBuilder** 를 사용한다는것말 알아두자



실제 사용은 아래와 같다

```html
<!-- Will produce 'http://localhost:8080/gtvg/order/details?orderId=3' (plus rewriting) -->
<a href="details.html" 
   th:href="@{http://localhost:8080/gtvg/order/details(orderId=${o.id})}">view</a>

<!-- Will produce '/gtvg/order/details?orderId=3' (plus rewriting) -->
<a href="details.html" th:href="@{/order/details(orderId=${o.id})}">view</a>

<!-- Will produce '/gtvg/order/3/details' (plus rewriting) -->
<a href="details.html" th:href="@{/order/{orderId}/details(orderId=${o.id})}">view</a>
```

위 코드에서 **th:href** 속성을 잘 살펴보자

- th:href는 수식어 속성이다. `Thymeleaf`가 렌더링하면 실제로 <a> 태그의 href 속성으로 설정한다.
- 위에서 사용한 **orderId=${o.id}** 처럼 endpoint에 parameter를 전달할 수도 있다. encoding도 알아서 해준다고 한다.
- 파라미터가 여러개필요하다면 comma로 나눌 수도 있다. 

```html
@{/order/process(execId=${execId},execType='FAST')}
```

- URL path에도 직접 사용이 가능하다.

```
@{/order/{orderId}/details(orderId=${orderId})}
```

- 만약 /order/detail 처럼 /로시작하는 URL은 `Thymeleaf`가 기특하게 application 기본 context를 앞에 붙여준다.
-  쿠키가 활성화되지 않았거나 아직 알려지지 않은 경우, 세션이 보존되도록 상대 URL에 ";jsessionid=..." 접미사를 추가할 수 있다. 이것을 URL Rewrite라고 하며 `Thymeleaf`를 사용하면 모든 URL에 대해 responselet API의 response.encodeURL(...) 메커니즘을 사용하여 직접 다시 쓰는 필터를 연결할 수 있다. 
-  t:href 속성은 템플릿에 작동 중인 정적 href 속성을 가지고 있으므로, 프로토타이핑을 위해 직접 열 때 템플릿 링크가 브라우저에 의해 탐색 가능한 상태로 유지되도록 한다. 

~~밑에 두개는 번역기 복붙~~



#{...} 같은 message expression의 경우 href속성 내부에 사용할때는 아래와 같이 사용한다.

```html
<a th:href="@{${url}(orderId=${o.id})}">view</a>
<a th:href="@{'/details/'+${user.login}(orderId=${o.id})}">view</a>
```

<br>

- ### A menu for our home page

application 내부에서 페이지 이동은 아래와 같이 사용한다.

```html
<p>Please select an option</p>
<ol>
  <li><a href="product/list.html" th:href="@{/product/list}">Product List</a></li>
  <li><a href="order/list.html" th:href="@{/order/list}">Order List</a></li>
  <li><a href="subscribe.html" th:href="@{/subscribe}">Subscribe to our Newsletter</a></li>
  <li><a href="userprofile.html" th:href="@{/userprofile}">See User Profile</a></li>
</ol>
```



## 5. Fragments



fragments expression을 통해서 template화 할수 있다. 여기에서 사용되는 속성은 **th:insert** 그리고 **th:replace**이다. 

```html
<div th:insert="~{commons :: main}">...</div>
```

위 태그는 아래처럼 어디에서든지 사용 가능하다.

```html
<div th:with="frag=~{footer :: #main/text()}">
  <p th:insert="${frag}">
</div>
```



이 Fragments는 tutorials 뒤에 자세하게 Template Layout으로 자세하게 알아볼꺼니까. 그냥 넘어가도 될 듯 하다..ㅎㅎㅎㅎㅎ

<br>



## 6. Literals

- ### Text literals

Text는 앞서 알아봤던 th:text 내부에 single quotes를 사용하면 된다. 만약 문자를 escape하고싶다면 \를 붙여서 사용하자.

```html
<p>
  Now you are looking at a <span th:text="'working web application'">template file</span>.
</p>
```

<br>

- ### Number literals

Number는 그냥 사용하자

```html
<p>The year is <span th:text="2013">1492</span>.</p>
<p>In two years, it will be <span th:text="2013 + 2">1494</span>.</p>



```

<br>

- ### Boolean literals

Boolean 값은 **th:if**  속성을 사용한다.

```html
<div th:if="${user.isAdmin()} == false"> ...
```

이렇게 사용해도 `Thymeleaf`에서 아래처럼 동작시키는 것 같다.

```html
<div th:if="${user.isAdmin() == false}"> ...
```

<br>

- ### The null literal

null 또한 Boolean과 비슷하게 사용한다.

```html
<div th:if="${variable.something} == null"> ...
```

<br>

- ### Literal tokens

Number, Boolean, null 은 사실 특별한 경우고 공백이나 콤마가 없는 literals는 `Thymeleaf`가 간편하게 single quotes없이 사용할 수 있도록 해준다.

```html
<div th:class="'content'">...</div>
```

위처럼 'content' 를 ''로 감싸지않고

```html
<div th:class="content">...</div>
```

이렇게 사용해도 무방하다.

<br>

## 7. Appending texts

단순 text를 붙이는거는 + 연산자를 사용해서 붙이기만해도 쉽게 사용이 가능하다.

```html
<span th:text="'The name of the user is ' + ${user.name}">
```

<br>

## 8. Literal substitutions

하지만 문자 대체를 사용하면 +연산자 없이 expression을 포함하는 문자열을 쉽게 포함할 수 있으니 다음과 같이 사용하는게 더 좋을 듯 하다.

이런 식은 반드시 |연산자로 감싸서 사용한다.

```html
<span th:text="|Welcome to our application, ${user.name}!|">
```

지금 위에서 작성한 코드는 아래의 코드와 동일하게 동작한다.

```html
<span th:text="'Welcome to our application, ' + ${user.name} + '!'">
```

짱 좋군 ..



여러개의 expression연결 역시 가능하다.

```html
<span th:text="${onevar} + ' ' + |${twovar}, ${threevar}|">
```



| 안에 들어오는 값들은 ${...}, #{...}, *{...} 만 가능하고 boolean, numeric if같은 expression은 사용이 불가능하다.

<br>



## 9. Arithmetic operations



웬만한 +, -, *. /, % 는 다 **th:with**로 사용가능하다

```html
<div th:with="isEven=(${prodStat.count} % 2 == 0)">
```

아래의 경우 주의?를 주는듯 하다.

```html
<div th:with="isEven=${prodStat.count % 2 == 0}">
```

${...} 내부에서 / 또는 %를 사용해서 그런듯하다.

<br>

## 10. Comparators and Equality



compare 연산자인 >, <, >=, <=, ==, != 사용이 가능하다 xml에서만 <와 >을 \&gt; \&lt; 로 변환시켜주면 된다.

```html
<div th:if="${prodStat.count} &gt; 1">
<span th:text="'Execution mode is ' + ( (${execMode} == 'dev')? 'Development' : 'Production')">
```

-  `gt` (`>`), `lt` (`<`), `ge` (`>=`), `le` (`<=`), `not` (`!`). Also `eq` (`==`), `neq`/`ne` (`!=`). 

변환에 참고하자

<br>

## 11. Conditional expressions



Conditional expression은 두개의 expression 중 조건을 평가하는 결과에 따라 한개만 평가하는 식이다.

```html
<tr th:class="${row.even}? 'even' : 'odd'">
  ...
</tr>
```

중첩도 가능하다

```html
<tr th:class="${row.even}? (${row.first}? 'first' : 'even') : 'odd'">
  ...
</tr>
```



이 표현식은 ${...}, *{...}, #{...}, @{...} 외에 일반 '...' 에도 사용 가능하다.

단순 if만 표현하고싶다면

```html
<tr th:class="${row.even}? 'alt'">
  ...
</tr>
```

로도 가능하고 만약 false값이 반환될 경우 null값이 반환된다.



## 12. Default expressions (Elvis operator)

default expresson은 conditinal express에서 then 조건이 없는 특수한 표현식인데 Groovy같은 언어를 사용해보신 분들은 익숙하시겠지만 나는 안 익숙...하다 아래와 같이 사용한다

```html
<div th:object="${session.user}">
  ...
  <p>Age: <span th:text="*{age}?: '(no age specified)'">27</span>.</p>
</div>
```

그냥 간단하게 설명하면 user.age 값이 null일경우 (no age specified)를 표현해준다고 생각하면 편하다.

사용은 **?:**로 표현한다. 위 코드는 아래의 코드와 동일하게 동작한다.

<hr>
다음시간에는 Setting Attribute Values 쪽을 보도록 하겠습니다!



<br>

포스팅은 여기까지 하겠습니다. 

<br>

퍼가실때는 링크와 출처를 반드시 명시해주세요. 감사합니다.

<br>

참고사이트

-  https://www.thymeleaf.org/doc/tutorials/3.0/usingthymeleaf.html 
-  https://github.com/thymeleaf/thymeleafexamples-gtvg 