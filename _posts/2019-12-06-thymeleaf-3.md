---

layout: post
title: "Thymeleaf Document 훑어보기 #3"
tags: [JAVA, Spring, Thymeleaf]
comments: true
parent: Thymeleaf
nav_order: 3
date: 2019-12-06
---



[setting-attribute-values](https://www.thymeleaf.org/doc/tutorials/3.0/usingthymeleaf.html#setting-attribute-values) 부터 이어서 진행하겠습니다!

<hr>
# Setting Attribute Values

## 1. Setting the value of any attribute

앞서 언급했듯이 Thymeleaf에서는 정적인 페이지 렌더링도 문제가 없도록 HTML5문법을 그대로 사용할 수 있다. 아래처럼 말이다.

```html
<form action="subscribe.html" th:attr="action=@{/subscribe}">
  <fieldset>
    <input type="text" name="email" />
    <input type="submit" value="Subscribe!" th:attr="value=#{subscribe.submit}"/>
  </fieldset>
</form>
```

보통은 action값에 #을 넣어주는 듯 하니 참고 바란다.

정적인 페이지에서는

```html
<form action="subscribe.html">
  <fieldset>
    <input type="text" name="email" />
    <input type="submit" value="Subscribe!" />
  </fieldset>
</form>
```

이렇게나오고

<br>

Controller를 태운 동적인 페이지에서는

```html
<form action="/gtvg/subscribe">
  <fieldset>
    <input type="text" name="email" />
    <input type="submit" value="¡Suscríbe!"/>
  </fieldset>
</form>
```

이렇게 렌더링 될 것이다.

<br>

만약 두개 이상의 attribute를 설정하고 싶다면 아래와 같이 하면 된다.

```html
<img src="../../images/gtvglogo.png" 
     th:attr="src=@{/images/gtvglogo.png},title=#{logo},alt=#{logo}" />
```

실제 렌더링은 이렇게된다.

```html
<img src="/gtgv/images/gtvglogo.png" title="Logo de Good Thymes" alt="Logo de Good Thymes" />
```



하지만 Thymeleaf를 조금이라도 본 사용자면 위와 같은 방법은 잘 사용하지 않는다는걸 아실꺼다. 아래의 섹션을 보자

<br>

## 2. Setting value to specific attributes



위에서 봤던 방법 말고 아래의 방법으로 markup을 하도록 하자 **th:attr=***는 여러 형태로 아래와 같이 변환 가능하다.

```html
<input type="submit" value="Subscribe!" th:value="#{subscribe.submit}"/>
```

```html
<form action="subscribe.html" th:action="@{/subscribe}">
```

```html
<li><a href="product/list.html" th:href="@{/product/list}">Product List</a></li>
```

위의 형태로 **th:value**, **th:href** 등으로 바인딩이 가능하다.

아래의 수많은 th:* tag들이 존재하니 필요할때 참고하자.

> | `th:abbr`               | `th:accept`           | `th:accept-charset` |
> | ----------------------- | --------------------- | ------------------- |
> | `th:accesskey`          | `th:action`           | `th:align`          |
> | `th:alt`                | `th:archive`          | `th:audio`          |
> | `th:autocomplete`       | `th:axis`             | `th:background`     |
> | `th:bgcolor`            | `th:border`           | `th:cellpadding`    |
> | `th:cellspacing`        | `th:challenge`        | `th:charset`        |
> | `th:cite`               | `th:class`            | `th:classid`        |
> | `th:codebase`           | `th:codetype`         | `th:cols`           |
> | `th:colspan`            | `th:compact`          | `th:content`        |
> | `th:contenteditable`    | `th:contextmenu`      | `th:data`           |
> | `th:datetime`           | `th:dir`              | `th:draggable`      |
> | `th:dropzone`           | `th:enctype`          | `th:for`            |
> | `th:form`               | `th:formaction`       | `th:formenctype`    |
> | `th:formmethod`         | `th:formtarget`       | `th:fragment`       |
> | `th:frame`              | `th:frameborder`      | `th:headers`        |
> | `th:height`             | `th:high`             | `th:href`           |
> | `th:hreflang`           | `th:hspace`           | `th:http-equiv`     |
> | `th:icon`               | `th:id`               | `th:inline`         |
> | `th:keytype`            | `th:kind`             | `th:label`          |
> | `th:lang`               | `th:list`             | `th:longdesc`       |
> | `th:low`                | `th:manifest`         | `th:marginheight`   |
> | `th:marginwidth`        | `th:max`              | `th:maxlength`      |
> | `th:media`              | `th:method`           | `th:min`            |
> | `th:name`               | `th:onabort`          | `th:onafterprint`   |
> | `th:onbeforeprint`      | `th:onbeforeunload`   | `th:onblur`         |
> | `th:oncanplay`          | `th:oncanplaythrough` | `th:onchange`       |
> | `th:onclick`            | `th:oncontextmenu`    | `th:ondblclick`     |
> | `th:ondrag`             | `th:ondragend`        | `th:ondragenter`    |
> | `th:ondragleave`        | `th:ondragover`       | `th:ondragstart`    |
> | `th:ondrop`             | `th:ondurationchange` | `th:onemptied`      |
> | `th:onended`            | `th:onerror`          | `th:onfocus`        |
> | `th:onformchange`       | `th:onforminput`      | `th:onhashchange`   |
> | `th:oninput`            | `th:oninvalid`        | `th:onkeydown`      |
> | `th:onkeypress`         | `th:onkeyup`          | `th:onload`         |
> | `th:onloadeddata`       | `th:onloadedmetadata` | `th:onloadstart`    |
> | `th:onmessage`          | `th:onmousedown`      | `th:onmousemove`    |
> | `th:onmouseout`         | `th:onmouseover`      | `th:onmouseup`      |
> | `th:onmousewheel`       | `th:onoffline`        | `th:ononline`       |
> | `th:onpause`            | `th:onplay`           | `th:onplaying`      |
> | `th:onpopstate`         | `th:onprogress`       | `th:onratechange`   |
> | `th:onreadystatechange` | `th:onredo`           | `th:onreset`        |
> | `th:onresize`           | `th:onscroll`         | `th:onseeked`       |
> | `th:onseeking`          | `th:onselect`         | `th:onshow`         |
> | `th:onstalled`          | `th:onstorage`        | `th:onsubmit`       |
> | `th:onsuspend`          | `th:ontimeupdate`     | `th:onundo`         |
> | `th:onunload`           | `th:onvolumechange`   | `th:onwaiting`      |
> | `th:optimum`            | `th:pattern`          | `th:placeholder`    |
> | `th:poster`             | `th:preload`          | `th:radiogroup`     |
> | `th:rel`                | `th:rev`              | `th:rows`           |
> | `th:rowspan`            | `th:rules`            | `th:sandbox`        |
> | `th:scheme`             | `th:scope`            | `th:scrolling`      |
> | `th:size`               | `th:sizes`            | `th:span`           |
> | `th:spellcheck`         | `th:src`              | `th:srclang`        |
> | `th:standby`            | `th:start`            | `th:step`           |
> | `th:style`              | `th:summary`          | `th:tabindex`       |
> | `th:target`             | `th:title`            | `th:type`           |
> | `th:usemap`             | `th:value`            | `th:valuetype`      |
> | `th:vspace`             | `th:width`            | `th:wrap`           |
> | `th:xmlbase`            | `th:xmllang`          | `th:xmlspace`       |

<br>

## 3. Setting more than one value at a time



위에서 봤던 여러개의 attribute 속성을 설정하는 방법은 아래와 같다.

```html
<img src="../../images/gtvglogo.png" 
     th:attr="src=@{/images/gtvglogo.png},title=#{logo},alt=#{logo}" />
```

위 코드는 

```html
<img src="../../images/gtvglogo.png" 
     th:src="@{/images/gtvglogo.png}" th:title="#{logo}" th:alt="#{logo}" />
```

로 표현될 수 있고

```html
<img src="../../images/gtvglogo.png" 
     th:src="@{/images/gtvglogo.png}" th:alt-title="#{logo}" />
```

`-` 을 통해서 묶어줄 수도 있다.

<br>



## 4. Appending and prepending



Thymeleaf는 attribute 에도 appending, prepending을 지원하는데 사용법은  **th:attrappend**,  **th:attrprepend**  으로 사용하면 된다.  만약 기본 class에서 추가하고 싶은 class가 있을때는 아래처럼 사용  가능하다.

```html
<input type="button" value="Do it!" class="btn" th:attrappend="class=${' ' + cssStyle}" />
```

위 코드는 다음과같이 렌더링된다.

```html
<input type="button" value="Do it!" class="btn warning" />
```



아마 다음편에서 자세하게 살펴보지 않을까 싶은데 **th:each** 태그에서 Thymeleaf의 expression으로 class를 동적으로 주는 방법도 가능하다.

```html
<tr th:each="prod : ${prods}" class="row" th:classappend="${prodStat.odd}? 'odd'">
```

자세한 설명은 생략한다.

<br>



## 5. Fixed-value boolean attributes



input tag의 checkbox나 radio 속성은 아래와 같이 checked라는 속성을 갖고 있다.

```html
<input type="checkbox" name="option2" checked /> <!-- HTML -->
<input type="checkbox" name="option1" checked="checked" /> <!-- XHTML -->
```

Thymeleaf는 다음과같이 true 또는 false값으로 해당 속성을 제어할 수 있다.

```html
<input type="checkbox" name="active" th:checked="${user.active}" />
```

이런 boolean속성을 갖는  th:* tag 역시 여러개가 있으니 필요할때 참고하면 될 듯하다.



> | `th:async`          | `th:autofocus` | `th:autoplay`   |
> | ------------------- | -------------- | --------------- |
> | `th:checked`        | `th:controls`  | `th:declare`    |
> | `th:default`        | `th:defer`     | `th:disabled`   |
> | `th:formnovalidate` | `th:hidden`    | `th:ismap`      |
> | `th:loop`           | `th:multiple`  | `th:novalidate` |
> | `th:nowrap`         | `th:open`      | `th:pubdate`    |
> | `th:readonly`       | `th:required`  | `th:reversed`   |
> | `th:scoped`         | `th:seamless`  | `th:selected`   |

<br>

## 6. Setting the value of any attribute (default attribute processor)



Thymeleaf는  default attribute processor 속성에 대해서도 **th:** 태그 사용이 가능하다. 

```html
<span th:whatever="${user.name}">...</span>
```

위 코드는 아래와 같이 렌더링된다.

```html
<span whatever="John Apricot">...</span>
```

<br>



## 7. Support for HTML5-friendly attribute and element names



HTML5에서 지원하는 data-* 속성 역시 Thymeleaf에서도 지원한다.

```html
<table>
    <tr data-th-each="user : ${users}">
        <td data-th-text="${user.login}">...</td>
        <td data-th-text="${user.name}">...</td>
    </tr>
</table>
```

위와 같이 사용가능한데 주의할점은 data-th:*가 아니라 data-th-*로 사용해야한다.

<br>

<hr>

<br>

포스팅은 여기까지 하겠습니다. 

<br>

퍼가실때는 링크와 출처를 반드시 명시해주세요. 감사합니다.

<br>

참고사이트

-  https://www.thymeleaf.org/doc/tutorials/3.0/usingthymeleaf.html 
-  https://github.com/thymeleaf/thymeleafexamples-gtvg 