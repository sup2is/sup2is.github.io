---

layout: post
title: "Thymeleaf Document 훑어보기 #1"
tags: [JAVA, Spring, Thymeleaf]
comments: true
parent: Thymeleaf
nav_order: 1
date: 2019-12-05
---



앞으로 만들 토이프로젝트에 `Thymeleaf`를 사용할것이기 때문에 당분간 포스팅은 SpringBoot + `Thymeleaf` 으로 갈 예정입니다. `Thymeleaf` 공홈내용 기준으로 practice를 작성하는것이기 때문에 그냥 공홈을 보시는걸 추천드립니다.. ㅎㅎㅎㅎㅎ 

<hr>



# Thymeleaf

먼저 `Thymeleaf`에 대해 간단하게 설명 후 지나가자. 발음은 그냥 타임리프로 하면된다. 

`Thymeleaf`는 HTML, XML, JavaScript, CSS 및 일반 텍스트를 Java 진영에서 사용할 수 있게 도와주는 Java Template engine이다. 비슷한 Template engine으로 Mustache, Handlebars, Velocity 등등 많이 존재하고 JSP도 일종의 템플릿엔진이다. 

요즘은 vue나 react로 FE와 BE를 완전히 분리한 형태로 많이 사용하는 추세이지만 `Thymeleaf` 고려하는것도 나쁘지 않은 선택이라고 생각한다. 왜냐면 Spring Boot에서 이제 JSP 사용은 지양하고 `Thymeleaf`를 적극 사용하라고 가이드해주기 때문이다. (~~뇌피셜~~)

`Thymeleaf`가 여러 Template engine 가운데에서도 각광받는 이유는 바로 순수 HTML5 tag 자체로 동작이 가능하도록 설계되었기 때문이다. JSP같은경우 foreach문을 사용하려면 jstl을 사용해야하고 mustache 계열의 template engine은 {{}}와 같은 새로운 문법을 사용해야하지만 `Thymeleaf`는 HTML5 문법 그대로 사용이 가능하다.

<br>

`Thymeleaf`는 HTML, XML, TEXT, JAVASCRIPT, CSS, RAW 데이터까지 6가지 종류의 템플릿을 처리할 수 있는데 나는 HTML만 주의깊게 볼것이다.

<br>

>  So if you are a Spring MVC user you are not wasting your time, as almost everything you learn here will be of use in your Spring applications.

위에는 공홈 내용을 직접 가져다 붙였다.



`Thymeleaf`는 Standard Dialect를 제공하는데 혹시 만약 JSP의 tag library를 사용해본적이 있다면 아래의 내용을 다음과 같이 바꿀 수 있다.

```html
<form:inputText name="userName" value="${user.name}" />
```

using `Thymeleaf`

```html
<input type="text" name="userName" value="James Carrot" th:value="${user.name}" />
```

<br>

위와 같이 작성했을때 JSP tag library는 정적인 페이지에서는 동작하지 않겠지만 `Thymeleaf`를 사용하면 정적으로 렌더링 역시 가능하다. ${user} 를 모르면 value에 적힌 "James Carrot"으로 렌더링 될 것이다.

<br>

`Thymeleaf`에서 정~말 친절하게 공식 tutorial repository를 제공해주고 있다. 정말 꿀이군 ..

해당 repository는  https://github.com/thymeleaf/thymeleafexamples-gtvg 이고 실행방법은 아래에 있으니 참고하도록 하자!



> ```
> git clone https://github.com/thymeleaf/thymeleafexamples-gtvg.git
> ```
>
> To build this project you will need Maven 2. You can get it at:
>
> ```
>  http://maven.apache.org
> ```
>
> Clean compilation products:
>
> ```
>  mvn clean
> ```
>
> Compile:
>
> ```
>  mvn compile
> ```
>
> Run in a tomcat server:
>
> ```
>  mvn tomcat7:run
> ```
>
> Once started, the application should be available at:
>
> ```
>  http://localhost:8080/gtvg
> ```



<br>

![grocery example](https://www.thymeleaf.org/doc/tutorials/3.0/images/usingthymeleaf/gtvg-view.png)

<br>

공식 예제에선 `Thymeleaf`의 내부 동작? 에 대해서 간단하게 설명하는듯 하니 필요하신분은 [링크]( https://www.thymeleaf.org/doc/tutorials/3.0/usingthymeleaf.html#creating-and-configuring-the-template-engine )를 확인해도 좋을 듯 하다.



# Using Texts



## 1. A multi-language welcome

- ### Using th:text and externalizing text



공식문서와 함께 예제를 작성할것이기때문에 예제의 Grocery site를 참고하여 작성하도록 하겠다. 그냥 편안하게술술 읽어도 문제는 없을것같다..



\- /WEB-INF/templates/home.html

```html
<!DOCTYPE html>

<html xmlns:th="http://www.thymeleaf.org">

  <head>
    <title>Good Thymes Virtual Grocery</title>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
    <link rel="stylesheet" type="text/css" media="all" 
          href="../../css/gtvg.css" th:href="@{/css/gtvg.css}" />
  </head>

  <body>
  
    <p th:text="#{home.welcome}">Welcome to our grocery store!</p>
  
  </body>

</html>
```



일단 위에서 언급한것처럼 `Thymeleaf` engine은 HTML5로 설계되었기때문에 어느 브라우저든지 렌더링이 가능하다.

조금 특별한 부분이 있다면 p tag 쪽인데 **th:text** 라는 이상한 문법이 하나 있다. 이제 앞으로 만나게 될 `Thymeleaf` 문법은 **th:*** 를 사용한다. prefix를 따로 정해줄 수 있는데 th가 일반적인 관례인듯하다.

th:를 용하려면 html tag에 다음과 같이 작성하면 끝이다.

```html
<html xmlns:th="http://www.thymeleaf.org">
```

아.. 물론 `Thymeleaf` library가 있어야한다. grocery 프로젝트 pom.xml에 보면 아래와 같이 dependency가 존재한다.

```xml
    <dependency>
      <groupId>org.thymeleaf</groupId>
      <artifactId>thymeleaf</artifactId>
      <version>${thymeleaf.version}</version>
      <scope>compile</scope>
    </dependency>
```

<br>

내부에서 home.html과 home_{lang}.properties 무슨 연관이 있는지 Controller에서 다음과 같이 보내면 message externalized 도 가능하다.



```java
public class HomeController implements IGTVGController {

    public void process(
            final HttpServletRequest request, final HttpServletResponse response,
            final ServletContext servletContext, final ITemplateEngine templateEngine)
            throws Exception {
        
        WebContext ctx = 
                new WebContext(request, response, servletContext, request.getLocale());
        
        templateEngine.process("home", ctx, response.getWriter());
        
    }

}
```



위에서 WebContext는  **org.thymeleaf.context.IContext** 를 구현하고 있는데

```java
public interface IContext {

    public Locale getLocale();
    public boolean containsVariable(final String name);
    public Set<String> getVariableNames();
    public Object getVariable(final String name);
    
}
```

locale을 얻어오는 부분이 message externalized와 연관이 있다. 자세한 설명은 공홈을 참조하자.



그냥 재미로 home_ko.properties의 파일을 생성하고

```
home.welcome=안녕 Thymeleaf, {0} (from default messages)!
```

로 home.welcome을 수정해봤다. 잘 나온다

![thymeleaf-1](https://user-images.githubusercontent.com/30790184/70196852-55586b00-174d-11ea-810c-02b1729c8068.png)





<br>

## 2. More on texts and variables

- ### Unescaped Text

다음과같은 message는 `Thymeleaf`가 렌더링되면 <b> tag를 escape한다.

```html
home.welcome=Welcome to our <b>fantastic</b> grocery store!
```

즉 다음과 같이 작동한다

```html
<p>Welcome to our &lt;b&gt;fantastic&lt;/b&gt; grocery store!</p>
```

<br>

이 경우엔

th:text 속성이아니라 **th:utext** 속성을 사용하면 된다

```html
<p th:utext="#{home.welcome}">Welcome to our grocery store!</p>
```

위 태그는 다음과같이 렌더링된다

```html
<p>Welcome to our <b>fantastic</b> grocery store!</p>
```

참고로 th:utext의 뜻은 unescaped text 이다.

<br>

- ### Using and displaying variables

message같은 경우는 #{...}을 사용했다면 변수값을 꺼낼때는 ${...}를 사용한다.

grocery 사이트의 홈페이지에서는 날짜를 찍어주는 부분이 다음과 같이 있다.



```html
  <p th:utext="#{home.welcome}">Welcome to our grocery store!</p>
  <p>Today is: <span th:text="${today}">13 February 2011</span></p>
```

위에서 ${today}는 Controller에서 넘어온 변수값이다 Spring EL과 매우 유사한 형태여서 전혀 어색하지 않다.



\- HomeController.java

```java
public void process(
            final HttpServletRequest request, final HttpServletResponse response,
            final ServletContext servletContext, final ITemplateEngine templateEngine)
            throws Exception {
        
    SimpleDateFormat dateFormat = new SimpleDateFormat("dd MMMM yyyy");
    Calendar cal = Calendar.getInstance();
        
    WebContext ctx = 
            new WebContext(request, response, servletContext, request.getLocale());
    ctx.setVariable("today", dateFormat.format(cal.getTime()));
        
    templateEngine.process("home", ctx, response.getWriter());
        
}
```



뿐만아니라 Java에서 Member member라는 Entity객체가 있을 경우 ${member.id}로 접근하여 Member 인스턴스의 getId()를 호출할 수도 있다.



# Standard Expression Syntax



`Thymeleaf`에는 Standard Expression Syntax이 존재한다. 위에서 확인했던 #{...}, ${...} 이외에도 몇가지가 더 존재한다.

- Simple expressions:
  - Variable Expressions: `${...}`
  - Selection Variable Expressions: `*{...}`
  - Message Expressions: `#{...}`
  - Link URL Expressions: `@{...}`
  - Fragment Expressions: `~{...}`
- Literals
  - Text literals: `'one text'`, `'Another one!'`,…
  - Number literals: `0`, `34`, `3.0`, `12.3`,…
  - Boolean literals: `true`, `false`
  - Null literal: `null`
  - Literal tokens: `one`, `sometext`, `main`,…
- Text operations:
  - String concatenation: `+`
  - Literal substitutions: `|The name is ${name}|`
- Arithmetic operations:
  - Binary operators: `+`, `-`, `*`, `/`, `%`
  - Minus sign (unary operator): `-`
- Boolean operations:
  - Binary operators: `and`, `or`
  - Boolean negation (unary operator): `!`, `not`
- Comparisons and equality:
  - Comparators: `>`, `<`, `>=`, `<=` (`gt`, `lt`, `ge`, `le`)
  - Equality operators: `==`, `!=` (`eq`, `ne`)
- Conditional operators:
  - If-then: `(if) ? (then)`
  - If-then-else: `(if) ? (then) : (else)`
  - Default: `(value) ?: (defaultvalue)`
- Special tokens:
  - No-Operation: `_`

<br>

JSP의 jstl을 사용해본사람이면 if 문이 얼마나 불편한지 알 것이다.. 하지만 `Thymeleaf`에서는 다음과 같이 표기가 가능하다.

```
'User is of type ' + (${user.isAdmin()} ? 'Administrator' : (${user.type} ?: 'Unknown'))
```



## 1. Messages



`Thymeleaf`에서 message externalize를 동적으로 사용가능하다 Spring에서 MessageSource와 사용이 비슷하다.

```
home.welcome=안녕 Thymeleaf, {0} (from default messages)!
```

이렇게 괄호로 넣어주고 아래와같이 파라미터를 넘겨주면

```html
<p th:utext="#{home.welcome(${session.user.name})}">
  Welcome to our grocery store, Sebastian Pepper!
</p>
```



![thymeleaf-1](https://user-images.githubusercontent.com/30790184/70196852-55586b00-174d-11ea-810c-02b1729c8068.png)

위와 같이 John Apricot으로 매핑이 잘된다 저 이름은 예제 프로젝트에서 GTVGFilter안에 존재한다.

\- GTVGFilter.java

```java
//...
    private static void addUserToSession(final HttpServletRequest request) {
        // Simulate a real user session by adding a user object
        request.getSession(true).setAttribute("user", new User("John", "Apricot", "Antarctica", null));
    }
//...
```

이 message expression는 value expression와 함께 사용 역시 가능하다

```html
<p th:utext="#{${welcomeMsgKey}(${session.user.name})}">
  Welcome to our grocery store, Sebastian Pepper!
</p>
```

이때 주의할점은 ${welcomeMsgKey}는 th:utext 내부에 있기때문에 unescape될것 같지만 미이 escape된 상태로 나온다고 가정한다.

<br>

## 2. Variables



${...}는 Variable expression이다. 위에서 언급한 태그

```html
<p th:utext="#{home.welcome(${session.user.name})}">
  Welcome to our grocery store, Sebastian Pepper!
</p>
```

의 실제 내부 동작은 다음과 같이 이루어진다.

```java
((User) ctx.getVariable("session").get("user")).getName();
```

${...} 내부에서 .뿐만아니라 []로도 접근이 가능하다 아래는 참고용이다.



> ```
> /*
>  * Access to properties using the point (.). Equivalent to calling property getters.
>  */
> ${person.father.name}
> 
> /*
>  * Access to properties can also be made by using brackets ([]) and writing 
>  * the name of the property as a variable or between single quotes.
>  */
> ${person['father']['name']}
> 
> /*
>  * If the object is a map, both dot and bracket syntax will be equivalent to 
>  * executing a call on its get(...) method.
>  */
> ${countriesByCode.ES}
> ${personsByName['Stephen Zucchini'].age}
> 
> /*
>  * Indexed access to arrays or collections is also performed with brackets, 
>  * writing the index without quotes.
>  */
> ${personsArray[0].name}
> 
> /*
>  * Methods can be called, even with arguments.
>  */
> ${person.createCompleteName()}
> ${person.createCompleteNameWithSeparator('-')}
> ```



- ### Expression Basic Objects

`Thymeleaf`에서는 기본적으로 제공하는 Basic Objects가 있다.

- `#ctx`: the context object.
- `#vars:` the context variables.
- `#locale`: the context locale.
- `#request`: (only in Web Contexts) the `HttpServletRequest` object.
- `#response`: (only in Web Contexts) the `HttpServletResponse` object.
- `#session`: (only in Web Contexts) the `HttpSession` object.
- `#servletContext`: (only in Web Contexts) the `ServletContext` object.

참고하도록 하자.

<br>

- ### Expression Utility Objects

`Thymeleaf`는 Basic Object 외에도 기본적인 utility set을 지원해주니 필요하다면 참고하자.



- `#execInfo`: information about the template being processed.
- `#messages`: methods for obtaining externalized messages inside variables expressions, in the same way as they would be obtained using #{…} syntax.
- `#uris`: methods for escaping parts of URLs/URIs
- `#conversions`: methods for executing the configured *conversion service* (if any).
- `#dates`: methods for `java.util.Date` objects: formatting, component extraction, etc.
- `#calendars`: analogous to `#dates`, but for `java.util.Calendar` objects.
- `#numbers`: methods for formatting numeric objects.
- `#strings`: methods for `String` objects: contains, startsWith, prepending/appending, etc.
- `#objects`: methods for objects in general.
- `#bools`: methods for boolean evaluation.
- `#arrays`: methods for arrays.
- `#lists`: methods for lists.
- `#sets`: methods for sets.
- `#maps`: methods for maps.
- `#aggregates`: methods for creating aggregates on arrays or collections.
- `#ids`: methods for dealing with id attributes that might be repeated (for example, as a result of an iteration).

<br>

- ### Reformatting dates in our home page

위에 #calendars를 사용하여 date객체를 다음과같이 formatting할 수 있다.

```html
<p>
  Today is: <span th:text="${#calendars.format(today,'dd MMMM yyyy')}">13 May 2011</span>
</p>
```



<hr>

너무 길어질꺼같아서 다음편으로 넘어가도록 하겠다! 

<br>

포스팅은 여기까지 하겠습니다. 

<br>

퍼가실때는 링크와 출처를 반드시 명시해주세요. 감사합니다.

<br>

참고사이트

-  https://www.thymeleaf.org/doc/tutorials/3.0/usingthymeleaf.html 
-  https://github.com/thymeleaf/thymeleafexamples-gtvg 