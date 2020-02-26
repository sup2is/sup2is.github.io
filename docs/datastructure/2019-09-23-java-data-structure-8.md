---

layout: post
title: "JAVA 자료구조 뿌시기 -Deque- #1 ArrayDeque편"
tags: [JAVA, DataStructure, Deque, ArrayDeque]
comments: true
nav_order: 8
parent: DataStructure
grand_parent: Java
---



사실 java에서의 Deque는 Queue의 sub interface이지만 자료구조에서 Queue와 Deque을 구분하기때문에 Deque 편을 만들었다. 열심히 확인해 보자.

***



# Deque<E>

`Deque` interface는 Queue자료구조와 매우 비슷한 성질을 갖고 있다.

<br>

![](https://media.geeksforgeeks.org/wp-content/uploads/anod.png)

<br>

이 `Deque`이라는 자료구조는 Queue와 매우매우 비슷한 성질을 갖고 있지만 약간 다르다. 간단하게 설명하면 Queue는 한곳으로만 들어오고 한곳으로만 나가는 반면에 `Deque`은 rear와 front 모두 삽입, 삭제가 가능하다.

위 그림에서 rear와 front라는 용어가 있는데 linear collection의 양 끝 부분을 의미한다. 그림으로 확인하면 조금 더 이해하기 수월 할 것 같다.

<br>

`Deque` interface는 양쪽 노드에 접근해서 삽입 또는 삭제,검사를 할 수 있는 메서드를 갖고 있다. 이 `Deque`의 add, remove, get 의 성질을 갖는 메서드는 총 2개의 동작방식을 나타내는데 한개는 실패시 예외를 던지는 것이고 또 다른 하나는 boolean값을 리턴한다.



|             | **First Element (Head)**                                     |                                                              | **Last Element (Tail)**                                      |                                                              |
| ----------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
|             | *Throws exception*                                           | *Special value*                                              | *Throws exception*                                           | *Special value*                                              |
| **Insert**  | [`addFirst(e)`](https://docs.oracle.com/javase/8/docs/api/java/util/Deque.html#addFirst-E-) | [`offerFirst(e)`](https://docs.oracle.com/javase/8/docs/api/java/util/Deque.html#offerFirst-E-) | [`addLast(e)`](https://docs.oracle.com/javase/8/docs/api/java/util/Deque.html#addLast-E-) | [`offerLast(e)`](https://docs.oracle.com/javase/8/docs/api/java/util/Deque.html#offerLast-E-) |
| **Remove**  | [`removeFirst()`](https://docs.oracle.com/javase/8/docs/api/java/util/Deque.html#removeFirst--) | [`pollFirst()`](https://docs.oracle.com/javase/8/docs/api/java/util/Deque.html#pollFirst--) | [`removeLast()`](https://docs.oracle.com/javase/8/docs/api/java/util/Deque.html#removeLast--) | [`pollLast()`](https://docs.oracle.com/javase/8/docs/api/java/util/Deque.html#pollLast--) |
| **Examine** | [`getFirst()`](https://docs.oracle.com/javase/8/docs/api/java/util/Deque.html#getFirst--) | [`peekFirst()`](https://docs.oracle.com/javase/8/docs/api/java/util/Deque.html#peekFirst--) | [`getLast()`](https://docs.oracle.com/javase/8/docs/api/java/util/Deque.html#getLast--) | [`peekLast()`](https://docs.oracle.com/javase/8/docs/api/java/util/Deque.html#peekLast--) |

<br>

이 `Deque` interface는 위에서 언급한것 처럼 Queue의 sub interface이기 때문에 큐처럼 쓰면 FIFO로도 사용이 가능하다.

| **Queue Method**                                             | **Equivalent Deque Method**                                  |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [`add(e)`](https://docs.oracle.com/javase/8/docs/api/java/util/Queue.html#add-E-) | [`addLast(e)`](https://docs.oracle.com/javase/8/docs/api/java/util/Deque.html#addLast-E-) |
| [`offer(e)`](https://docs.oracle.com/javase/8/docs/api/java/util/Queue.html#offer-E-) | [`offerLast(e)`](https://docs.oracle.com/javase/8/docs/api/java/util/Deque.html#offerLast-E-) |
| [`remove()`](https://docs.oracle.com/javase/8/docs/api/java/util/Queue.html#remove--) | [`removeFirst()`](https://docs.oracle.com/javase/8/docs/api/java/util/Deque.html#removeFirst--) |
| [`poll()`](https://docs.oracle.com/javase/8/docs/api/java/util/Queue.html#poll--) | [`pollFirst()`](https://docs.oracle.com/javase/8/docs/api/java/util/Deque.html#pollFirst--) |
| [`element()`](https://docs.oracle.com/javase/8/docs/api/java/util/Queue.html#element--) | [`getFirst()`](https://docs.oracle.com/javase/8/docs/api/java/util/Deque.html#getFirst--) |
| [`peek()`](https://docs.oracle.com/javase/8/docs/api/java/util/Queue.html#peek--) | [`peekFirst()`](https://docs.oracle.com/javase/8/docs/api/java/util/Deque.html#peek--) |

만약 `Deque`의 메서드중 표의 오른쪽 메서드만 쓰면 Queue를 사용하는것과 동일하다.

<br>

또 재미있는게 이 `Deque`는 Stack의 기능을 할 수도 있다.

| **Stack Method**                                             | **Equivalent Deque Method**                                  |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [`push(e)`](https://docs.oracle.com/javase/8/docs/api/java/util/Deque.html#push-E-) | [`addFirst(e)`](https://docs.oracle.com/javase/8/docs/api/java/util/Deque.html#addFirst-E-) |
| [`pop()`](https://docs.oracle.com/javase/8/docs/api/java/util/Deque.html#pop--) | [`removeFirst()`](https://docs.oracle.com/javase/8/docs/api/java/util/Deque.html#removeFirst--) |
| [`peek()`](https://docs.oracle.com/javase/8/docs/api/java/util/Deque.html#peek--) | [`peekFirst()`](https://docs.oracle.com/javase/8/docs/api/java/util/Deque.html#peekFirst--) |

<br>

`Deque`는 특별하게 removeFirstOccurrence() 와 removeLastOccurrence() 라는 메서드를 제공하는데 이건 구현체를 까보면서 설명하도록 하겠다.

이 `Deque`는 주의할점이 List처럼 index 관련 기능을 수행하지 않기때문에 index로 요소에 접근이 불가능하다.

`Deque`에서는 삽입 될 요소가 반드시 null이 아님을 반드시 금지시킬수는 없지만 이 또한 최대한 금지시키는것을 강력하게 권장하고 있다. 어떤 요소가 null값으로 삽입된 경우 이 요소를 꺼내올때도 null로 리턴, `Deque`가 비어있을때 요소를 꺼내올때도 null을 리턴하기 때문이다.



<br>



# ArrayDeque<E>

`ArrayDeque`는 `Deque`를 구현한 interface이다.

> ```java
> public class ArrayDeque<E>
> extends AbstractCollection<E>
> implements Deque<E>, Cloneable, Serializable
> ```

구조는 다음과 같이 되어 있다.

<br>

이 `ArrayDeque`는 내부 배열의 크기가 resizable할 수 있고 용량 제한이 없도록 구현되었다. 내부에서 필요에따라 배열을 확장시키는 구조이다. 이 `ArrayDeque`는 thread-safe하지 않기때문에 multi-thread 환경에서의 사용을 자제해야한다.

`ArrayDeque`는 역시 null 요소를 포함하지 않고 Stack class로 stack 자료구조를 구현하는것보다 `ArrayDeque`로 stack 자료구조를 구현하는게 더 빠르게 동작할 수 있고 queue 자료구조를 구현할때도 Queue interface보다 `ArrayDeque`가 더 빠를 수 있다. 신기하다.

<br>

대부분의 `ArrayDeque`의 메서드 실행시간은 동일한 시간을 갖는데 예외가 몇가지 있다. remove(), removeFirstOccurrence(), removeLastOccurrence(), contains(), iterator.remove() 그리고 대용량 연산의 경우 작업시간이 선형시간으로 실행되니 성능상에 주의가 필요하다.

이번엔 위에서 언급한 removeFirstOccurrence(), removeLastOccurrence() 메서드들은 파라미터로 있는 Object o 객체와 동일한객체(o.equalse()) 를 첫번째부터 탐색, 마지막부터 탐색해서 지우는 메서드이다. 만약 같은 객체를 발견하지 못한다면 return 값은 false이고 `Deque` 내부에 아무런 변화가 없다.

<br>

이 `ArrayDeque`역시 iterator를 호출 후 `ArrayDeque` 내부에 삽입 또는 삭제가 이뤄질경우 ConcurrentModificationException을 던지기때문에 주의해야한다.



# Example

과연 Stack 보다 `ArrayDeque`가 빠를지 한번 확인해보자. Queue자체도 비교해보고 싶긴한데 Queue만 구현한 class들이 주로 동시성관련 자료구조라서 Queue는 생략한다. 그나마 LinkedList인데 LinkedList역시 `Deque`를 구현하고 있다.



\- MyArrayDeque.class

```java
package data.structure8;

import java.util.ArrayDeque;
import java.util.LinkedList;
import java.util.Stack;

public class MyArrayDeque {

	public static void main(String[] args) throws InterruptedException {

		ArrayDeque<Integer> arrayDeque = new ArrayDeque<>();
		Stack<Integer> stack = new Stack<>();
		
		long start;
		long end;
		
		start = System.currentTimeMillis();
		
		for (int i = 0; i < 5000000; i++) {
			arrayDeque.addFirst(i);
		}
		
		while (!arrayDeque.isEmpty()) {
			arrayDeque.pollLast();
		}
		
		end = System.currentTimeMillis();
		
		System.out.println( "arrayDeque에 요소 오백만개 넣고 빼는데 걸리는 시간 : " + ( end - start )/1000.0 +"초");		
		
		
		start = System.currentTimeMillis();
		
		for (int i = 0; i < 5000000; i++) {
			stack.push(i);
		}
		
		while (!stack.isEmpty()) {
			stack.pop();
		}
		
		end = System.currentTimeMillis();
		
		System.out.println( "stack에 요소 오백만개 넣고 빼는데 걸리는 시간 : " + ( end - start )/1000.0 +"초");		
		
		
	}

}
```

<br>

\- console

```
arrayDeque에 요소 오백만개 넣고 빼는데 걸리는 시간 : 0.144초
stack에 요소 오백만개 넣고 빼는데 걸리는 시간 : 0.464초
============================================================
arrayDeque에 요소 오백만개 넣고 빼는데 걸리는 시간 : 0.177초
stack에 요소 오백만개 넣고 빼는데 걸리는 시간 : 0.465초
============================================================
arrayDeque에 요소 오백만개 넣고 빼는데 걸리는 시간 : 0.17초
stack에 요소 오백만개 넣고 빼는데 걸리는 시간 : 0.459초
```

약 세배정도 빠르다.

<br>



# LinkedList<E>

다음은 `LinkedList` 의 구조이다.

> ```java
> public class LinkedList<E>
> extends AbstractSequentialList<E>
> implements List<E>, Deque<E>, Cloneable, Serializable
> ```

이` LinkedList`는 위에서 확인할 수 있듯이 List interface와 `Deque` interface를 동시에 구현 했다. 따라서 Queue의 기능과 List의 index기반 기능을 동시에 사용할 수 있다.



이 `LinkedList`에 대한 설명은 [여기](https://sup2is.github.io/java-data-structure-2/)에서 먼저 살펴봤으니 자세한 설명은 생략한다

<br>

***



이제 Deque에는[ConcurrentLinkedDeque](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/ConcurrentLinkedDeque.html), [LinkedBlockingDeque](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/LinkedBlockingDeque.html) 가 남아있긴 한데 어차피 Queue에서 살펴본 CConcurrentLinkedQueue, LinkedBlockingQueue랑 별반 다를점이 크게 없어보이기때문에 생략하도록하고 다음시간에는 Set 에대해 알아보도록 하겠습니다



<br>

포스팅은 여기까지 하겠습니다. 

예제 : https://github.com/sup2is/java-example/tree/master/java-data-structure8

<br>

퍼가실때는 링크와 출처를 반드시 명시해주세요. 감사합니다.

<br>



참고 : https://docs.oracle.com/javase/8/docs/api/