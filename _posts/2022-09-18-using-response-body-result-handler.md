---
layout: post
title: "ResponseBodyResultHandler 사용해서 공통 Response 처리하기"
tags: [Reactive, Mono, Flux, Kotlin, ResponseBodyResultHandler]
date: 2022-09-18
comments: true
---

<br>



# Overview

이번시간에는 ResponseBodyResultHandler를 사용해서 response 타입에 대한 공동처리를 하는 방법에 대해서 알아보도록 하겠다.



# ResponseBodyResultHandler가 필요한 상황

보통 일반적으로 프로젝트마다 사용하는 공통 response wrapper가 있다 예를 들면 아래 ApiResponse 같은 클래스다.

```kotlin
import com.fasterxml.jackson.annotation.JsonInclude

@JsonInclude(JsonInclude.Include.NON_NULL)
class ApiResponse<T> (
    val data: T?,
    var result: Boolean = true,
    var message: String? = null
) {

    companion object {
        fun success(data: Any): ApiResponse<Any> {
            return ApiResponse(data)
        }

        fun failed(message: String? = null): ApiResponse<Any> {
            return ApiResponse(null, false, message)
        }
    }
}

```

이제 이 wrapper를 사용하는 Controller나 Service 레이어는 반환하고싶은 data타입을 wrapping해서 클라이언트에게 일관된 형태의 response를 제공해준다 

```kotlin
@RestController
@RequestMapping("/boards")
class BoardController(
    val boardService: BoardService
) {

    @PostMapping
    fun create(@RequestBody boardRequestDto: BoardRequestDto) =
        boardService.create(boardRequestDto).map { ApiResponse(it) }

    @PutMapping("/{boardId}")
    fun update(
        @RequestBody boardUpdateDto: BoardUpdateDto,
        @PathVariable("boardId") boardId: Long
    ) =
        boardService.update(boardId, boardUpdateDto).map { ApiResponse(it) }

    @GetMapping("/{boardId}")
    fun getOne(@PathVariable("boardId") boardId: Long) = boardService.get(boardId).map { ApiResponse(it) }

    @GetMapping
    fun getAll() = boardService.getAll().map { ApiResponse(it) }
}
```

```json
{
  "data": {
    "id": 1,
    "title": "title1",
    "contents": "contents",
    "author": "author",
    "createAt": "2022-06-22T09:48:39.361",
    "updateAt": "2022-06-22T09:48:39.361"
  },
  "result": true
}
```

위와 같은 코드에 동작은 전혀 문제가 없다. 하지만 `map { ApiResponse(it) }` 와 같은 형태의 보일러플레이트 코드는 없앨 수 있으면 없애는게 좋다.

# ResponseBodyResultHandler 알아보기

위와 같은 보일러플레이트 코드는 `ResponseBodyResultHandler` 를 사용해서 중앙 집중식으로 한번에 처리할 수 있다. Spring MVC에서는 `ResponseAdvice`가 같은 역할을 해줬다.

`ResponseBodyResultHandler`는 `HandlerResultHandler` 의 구현체다. `WebFluxConfigurationSupport`에서 기본적으로 등록시켜주는것을 확인할 수 있다. 

```java
// WebFluxConfigurationSupport.java

	@Bean
	public ResponseBodyResultHandler responseBodyResultHandler(
			@Qualifier("webFluxAdapterRegistry") ReactiveAdapterRegistry reactiveAdapterRegistry,
			ServerCodecConfigurer serverCodecConfigurer,
			@Qualifier("webFluxContentTypeResolver") RequestedContentTypeResolver contentTypeResolver) {

		return new ResponseBodyResultHandler(serverCodecConfigurer.getWriters(),
				contentTypeResolver, reactiveAdapterRegistry);
	}
```

기본적으로 `ResponseBodyResultHandler`는 `@ResponseBody` 애너테이션이 붙은 메서드에만 동작하도록 되어있다. 우리가 `@ResponseBody` 애너테이션으로 json body를 리턴해줄 수 있게 도와주는 역할을 하는 것 같다.

```java
// ResponseBodyResultHandler.java

	@Override
	public boolean supports(HandlerResult result) {
		MethodParameter returnType = result.getReturnTypeSource();
		Class<?> containingClass = returnType.getContainingClass();
		return (AnnotatedElementUtils.hasAnnotation(containingClass, ResponseBody.class) ||
				returnType.hasMethodAnnotation(ResponseBody.class));
	}

	@Override
	public Mono<Void> handleResult(ServerWebExchange exchange, HandlerResult result) {
		Object body = result.getReturnValue();
		MethodParameter bodyTypeParameter = result.getReturnTypeSource();
		return writeBody(body, bodyTypeParameter, exchange);
	}
```

# ResponseBodyResultHandler 적용하기

새로운 `ResponseBodyResultHandler`의 타입을 만들어서 wrapping을 하거나 결과값을 제어하도록 만들 수 있다. `supports()` 메서드와  `handleResult()` 를 재정의한 `ResponseWrapper` 클래스다. 

```kotlin
import me.sup2is.kotlinreactiveboard.api.controller.model.ApiResponse
import org.springframework.http.codec.HttpMessageWriter
import org.springframework.web.reactive.HandlerResult
import org.springframework.web.reactive.accept.RequestedContentTypeResolver
import org.springframework.web.reactive.result.method.annotation.ResponseBodyResultHandler
import org.springframework.web.server.ServerWebExchange
import reactor.core.publisher.Flux
import reactor.core.publisher.Mono
import reactor.kotlin.core.publisher.toMono

class ResponseWrapper(
    writers: MutableList<HttpMessageWriter<*>>,
    resolver: RequestedContentTypeResolver
) : ResponseBodyResultHandler(writers, resolver) {

    override fun supports(result: HandlerResult): Boolean {
        val isSupportTypes = result.returnType.resolve() == Mono::class.java ||
            result.returnType.resolve() == Flux::class.java
        val isAlreadyResponse = result.returnType.generics[0] == ApiResponse::class
        return isSupportTypes && !isAlreadyResponse
    }

    override fun handleResult(exchange: ServerWebExchange, result: HandlerResult): Mono<Void> {

        val body = when (val value = result.returnValue) {
            is Mono<*> -> value
            is Flux<*> -> value.collectList()
            else -> throw ClassCastException("The \"body\" should be Mono<*> or Flux<*>!")
        }.map {
            ApiResponse.success(it)
        }.onErrorResume {
            ApiResponse.failed(it.message).toMono()
        }

        val returnTypeSource = result.returnTypeSource

        return writeBody(body, returnTypeSource, exchange)
    }
}

```

`supports()` 에서는 해당 핸들러가 동작할 것인지를 Boolean 타입으로 리턴해주면 된다. 이 코드를 확인하지 못했을 수 있기 때문에 한번 더 wrapping되는 불상사를 막기 위해 `isAlreadyResponse` 같은 변수를 사용할 수 있다.

`handleResult()`에서는 실제로 원하는 wrapper 타입에 wrapping하는 코드를 볼 수 있다. 

마지막으로 `ResponseWrapper` 타입의 빈을 등록해주면  `map { ApiResponse(it) }` 와 같은 형태의 보일러플레이트 코드를 제거할 수 있다.

```kotlin
@Configuration
class WebConfig {

    @Bean
    fun requestWrapper(
        serverCodecConfigurer: ServerCodecConfigurer,
        requestedContentTypeResolver: RequestedContentTypeResolver
    ): ResponseWrapper {
        return ResponseWrapper(serverCodecConfigurer.writers, requestedContentTypeResolver)
    }
}
```

# ResponseBodyResultHandler 동작 확인하기

이제 실제로 잘 동작하는지 확인해보면 된다. 아래와 같이 테스트도 한번 짜봤다.

```kotlin

@WebFluxTest(BoardController::class)
@Import(WebConfig::class)
class ResponseWrapperTest {

    @Autowired
    lateinit var webTestClient: WebTestClient

    @MockBean
    lateinit var boardService: BoardService

    @Test
    fun `API가 성공하면 ApiResponse에 wrapping한다`() {

        // given
        given(boardService.get(anyLong()))
            .willReturn(Board().toMono())

        // when & then
        webTestClient.get()
            .uri("/boards/{boardId}", 1)
            .exchange()
            .expectStatus().isOk
            .expectBody(ApiResponse::class.java)
            .consumeWith {
                assertThat(it.responseBody!!.result).isTrue
                assertThat(it.responseBody!!.data).isNotNull
            }
    }

    @Test
    fun `API가 성공하면 ApiResponse에 wrapping한다2`() {

        // given
        given(boardService.getAll())
            .willReturn(Flux.just(Board(), Board()))

        // when & then
        webTestClient.get()
            .uri("/boards")
            .exchange()
            .expectStatus().isOk
            .expectBody(ApiResponse::class.java)
            .consumeWith {
                assertThat(it.responseBody!!.result).isTrue
                assertThat(it.responseBody!!.data).isNotNull
            }
    }
}
```

실제로 동작하는부분을 간단하게 보면 Webflux에서 모든 요청은 결국 `DispatcherHandler`의 `handle()` 메서드를 통과하는데 `DispatcherHandler.handleResult()` 라는 메서드에서 핸들러를 찾고 핸들링하는 코드를 동작시킨다.

```kotlin
// DispatcherHandler.java

	@Override
	public Mono<Void> handle(ServerWebExchange exchange) {
		if (this.handlerMappings == null) {
			return createNotFoundError();
		}
		if (CorsUtils.isPreFlightRequest(exchange.getRequest())) {
			return handlePreFlight(exchange);
		}
		return Flux.fromIterable(this.handlerMappings)
				.concatMap(mapping -> mapping.getHandler(exchange))
				.next()
				.switchIfEmpty(createNotFoundError())
				.flatMap(handler -> invokeHandler(exchange, handler))
				.flatMap(result -> handleResult(exchange, result));
	}

	private Mono<Void> handleResult(ServerWebExchange exchange, HandlerResult result) {
		return getResultHandler(result).handleResult(exchange, result)
				.checkpoint("Handler " + result.getHandler() + " [DispatcherHandler]")
				.onErrorResume(ex ->
						result.applyExceptionHandler(ex).flatMap(exResult -> {
							String text = "Exception handler " + exResult.getHandler() +
									", error=\"" + ex.getMessage() + "\" [DispatcherHandler]";
							return getResultHandler(exResult).handleResult(exchange, exResult).checkpoint(text);
						}));
	}

	private HandlerResultHandler getResultHandler(HandlerResult handlerResult) {
		if (this.resultHandlers != null) {
			for (HandlerResultHandler resultHandler : this.resultHandlers) {
				if (resultHandler.supports(handlerResult)) {
					return resultHandler;
				}
			}
		}
		throw new IllegalStateException("No HandlerResultHandler for " + handlerResult.getReturnValue());
	}
```

이때 눈여겨볼 점은 `getResultHandler()` 에서 `this.resultHandlers` 을 반복하는데 지원하는 핸들러타입을 마주치면 바로 해당 핸들러를 반환한다는 점이다. 따라서 `HandlerResultHandler` 의 order를 잘 확인해야 한다. 아래는 Spring이 기본적으로 등록해주는 `HandlerResultHandler` 타입들의 설명과 Default order다.

| Result Handler Type         | Return Values                                                | Default Order     |
| --------------------------- | ------------------------------------------------------------ | ----------------- |
| ResponseEntityResultHandler | ResponseEntity, 전형적으로 @Controller 인스턴스들로부터의 result가 이에 해당한다. | 0                 |
| ServerResponseResultHandler | ServerResponse, 전형적으로 functional endpoints로부터의 result가 이에 해당한다. | 0                 |
| ResponseBodyResultHandler   | @ResponseBody 메소드들이나 @RestController 클래스(@Controller + @ResponseBody) 로부터의 리턴 값들을 처리한다. | 100               |
| ViewResolutionResultHandler | CharSequence, View, Model, Map, [Rendering](https://docs.spring.io/spring-framework/docs/5.1.8.RELEASE/javadoc-api/org/springframework/web/reactive/result/view/Rendering.html)(Srping MVC의 ModelAndView)를 처리하며, 기타 Object들은 model attribute로 간주된다. | Integer.MAX_VALUE |



# 정리

프로젝트를 진행하면서 공통으로 Response를 처리해야할 일이 있다면 `ResponseBodyResultHandler`를 적용해보자.



<br>

***

포스팅은 여기까지 하겠습니다. 감사합니다!

예제: [https://github.com/sup2is/kotlin-reactive-board/blob/main/api/src/main/kotlin/me/sup2is/kotlinreactiveboard/api/config/ResponseWrapper.kt](https://github.com/sup2is/kotlin-reactive-board/blob/main/api/src/main/kotlin/me/sup2is/kotlinreactiveboard/api/config/ResponseWrapper.kt)

<br>

- **[References]**
  - [https://stackoverflow.com/questions/71084011/spring-webflux-add-a-wrapping-class-before-serialization](https://stackoverflow.com/questions/71084011/spring-webflux-add-a-wrapping-class-before-serialization)
  - [https://dreamchaser3.tistory.com/12](https://dreamchaser3.tistory.com/12)
  
