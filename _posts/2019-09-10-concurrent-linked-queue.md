---

layout: post
title: "Java 자료구조 파헤치기 #6 ConcurrentLinkedQueue"
tags: [JAVA, DataStructure, Queue, ConcurrentLinkedQueue]
date: 2019-09-10
comments: true
nav_order: 5
parent: DataStructure
grand_parent: Java
---



이번에는 ConcurrentLinkedQueue에 대해 알아보자

------

# ConcurrentLinkedQueue<E>

ConcurrentLinkedQueue는 다음과 같이 상속 및 구현하고 있다.

> ```java
> public class ConcurrentLinkedQueue<E>
> extends AbstractQueue<E>
> implements Queue<E>, Serializable
> ```

저번시간에 확인했던 PriorityQueue와 구조자체는 같다.

<br>

ConcurrentLinkedQueue는 thread-safe한 FIFO 자료구조이다. 이 ConcurrentLinkedQueue는 LinkedList와 같이 Node 라는 내부 class를 기반으로 동작한다.

> ```java
>     private static class Node<E> {
>         volatile E item;
>         volatile Node<E> next;
> 
>         /**
>          * Constructs a new node.  Uses relaxed write because item can
>          * only be seen after publication via casNext.
>          */
>         Node(E item) {
>             UNSAFE.putObject(this, itemOffset, item);
>         }
> 
>         boolean casItem(E cmp, E val) {
>             return UNSAFE.compareAndSwapObject(this, itemOffset, cmp, val);
>         }
> 
>         void lazySetNext(Node<E> val) {
>             UNSAFE.putOrderedObject(this, nextOffset, val);
>         }
> 
>         boolean casNext(Node<E> cmp, Node<E> val) {
>             return UNSAFE.compareAndSwapObject(this, nextOffset, cmp, val);
>         }
> 
>         // Unsafe mechanics
> 
>         private static final sun.misc.Unsafe UNSAFE;
>         private static final long itemOffset;
>         private static final long nextOffset;
> 
>         static {
>             try {
>                 UNSAFE = sun.misc.Unsafe.getUnsafe();
>                 Class<?> k = Node.class;
>                 itemOffset = UNSAFE.objectFieldOffset
>                     (k.getDeclaredField("item"));
>                 nextOffset = UNSAFE.objectFieldOffset
>                     (k.getDeclaredField("next"));
>             } catch (Exception e) {
>                 throw new Error(e);
>             }
>         }
>     }
> 
>     /**
>      * A node from which the first live (non-deleted) node (if any)
>      * can be reached in O(1) time.
>      * Invariants:
>      * - all live nodes are reachable from head via succ()
>      * - head != null
>      * - (tmp = head).next != tmp || tmp != head
>      * Non-invariants:
>      * - head.item may or may not be null.
>      * - it is permitted for tail to lag behind head, that is, for tail
>      *   to not be reachable from head!
>      */
>     private transient volatile Node<E> head;
> 
>     /**
>      * A node from which the last node on list (that is, the unique
>      * node with node.next == null) can be reached in O(1) time.
>      * Invariants:
>      * - the last node is always reachable from tail via succ()
>      * - tail != null
>      * Non-invariants:
>      * - tail.item may or may not be null.
>      * - it is permitted for tail to lag behind head, that is, for tail
>      *   to not be reachable from head!
>      * - tail.next may or may not be self-pointing to tail.
>      */
>     private transient volatile Node<E> tail;
> ```



ConcurrentLinkedQueue 의 멤버변수인 Node 타입의 head는 가장 나중에 들어온요소, tail은 가장 최근에 들어온 요소를 나타낸다. ConcurrentLinkedQueue는 thread-safe하기때문에 multi-thread환경에서 여러개의 thread들이 접근할때 사용한다. PriorityQueue와 같이 null 요소를 허용하지 않는다.

<br>

이 ConcurrentLinkedQueue는 Maged M. Michael and Michael L. Scott. 이라는 분이 만든 non-blocking 알고리즘을 사용했다. 내부가 꽤나 복잡한듯 하니 자세한 설명은 생략하고 non-blocking 에 대해 간단하게 알아보자.

<br>

non-blocking은 blocking과 상반되는 개념인데 multi-thread환경일때 주로 사용된다.  [링크](https://homoefficio.github.io/2017/02/19/Blocking-NonBlocking-Synchronous-Asynchronous/)에 정말 좋은 글이 있으니 참고 바라고 글의 내용을 참고하여 간단하게 설명하자면 blocking은 호출되는 함수의 결과값이 수행에 종속되어있으면 blocking이고 non-blocking은 호출되는 함수의 결과값이 수행에 종속적이지 않을때이다.

[여기](https://parkcheolu.tistory.com/33)에도 좋은 문구가 있는듯 하여 첨부한다.

>  블로킹 알고리즘과 논 블로킹 알고리즘의 차이는 요청된 동작이 수행될 수 없을 때 어떻게 대응하느냐에 있다.

나중에 자세하게 non-blocking과 blocking에대해 포스팅하는걸로하고 다시 ConcurrentLinkedQueue로 돌아가자.

<br>

ConcurrentLinkedQueue의 Iterator interface의 사용은 꽤 흥미로운편이다. 보통 우리가 지금까지 봤었던 Collection의 Iterator는 Iterator 객체를 얻어낸 후 자료구조의 변화가 있으면 ConcurrentModificationException을 던지기 마련인데 ConcurrentLinkedQueue는 그렇지 않다.

<br>

또 나름 주의깊게 봐야할 곳이 다른 Collection들과는 달리 size()메서드는 일정시간 연산으로 수행되지 않기때문에 주의해야한다. ConcurrentLinkedQueue의 비동기적 특성때문인데 현재 ConcurrentLinkedQueue에 들어있는 요소의 수를 파악하려면 요소 전체를 탐색해야한다. 이 size()를 수행할때와 동시에 ConcurrentLinkedQueue에 대한 수정이 있으면 size()는 부정확한 결과를 호출할 수 있으니 매우매우 주의해야한다. 또 대량 작업 addAll() , removeAll(), receveAll(), containAll(), equalse(), toArray()는 원자성을 보장하지 않는다.

<br>

# Example

multi-thread 환경을 테스트 및 디버깅하는건 생각보다 골치아픈 일이니 어느정도 감안하고 예제를 확인해보도록 하겠다.

Iterator가 생성된 이후에 요소를 추가하는것에 대하여 LinkedList와 ConcurrentLinkedQueue를 비교해보자.



\- Main.class

```java
package data.structure5;

import java.util.Iterator;
import java.util.LinkedList;
import java.util.concurrent.ConcurrentLinkedQueue;

public class Main {

	public static void main(String[] args) {

		ConcurrentLinkedQueue<Integer> clq = new ConcurrentLinkedQueue<>();

		LinkedList<Integer> list = new LinkedList<>();
		
		for (int i = 0; i < 10; i++) {
			list.add(i);
			clq.add(i);
		}
		
		Iterator<Integer> listIterator = list.iterator();
		Iterator<Integer> queueIterator = clq.iterator();
		
		while (queueIterator.hasNext()) {
			System.out.println(queueIterator.next());
			clq.poll();
			clq.add((int) (Math.random() * 10));
		}
		
		while (listIterator.hasNext()) {
			System.out.println(listIterator.next());
			//ConcurrentModificationException 발생
			list.add((int) Math.random() * 10);
		}
		
	}
}

```

javadoc 내용 그대로 ConcurrentLinkedQueue는 Iterator가 생성된 이후에 내부 요소를 추가해도 ConcurrentModificationException가 발생하지 않지만 LinkedList는 내부 요소 추가 즉시 에러가 발생한다.

<br>

추가적으로 size()메서드의 호출도 비교해보자.

<br>

\- Main2.class



```java
package data.structure5;

import java.util.LinkedList;
import java.util.concurrent.ConcurrentLinkedQueue;

public class Main2 {

	public static void main(String[] args) {
		
		long start, end;

		ConcurrentLinkedQueue<Integer> clq = new ConcurrentLinkedQueue<>();
		LinkedList<Integer> list = new LinkedList<>();
		
		for (int i = 0; i < 5000000; i++) {
			clq.add(i);
			list.add(i);
		}
		
		start = System.currentTimeMillis();
		System.out.println(clq.size());
		end = System.currentTimeMillis();
		System.out.println( "ConcurrentLinkedQueue의 오백만개 size() 수행시간 : " + ( end - start )/1000.0 +"초");
		
		start = System.currentTimeMillis();
		System.out.println(list.size());
		end = System.currentTimeMillis();
		System.out.println( "LinkedList의 오백만개 size() 수행시간 : " + ( end - start )/1000.0 +"초");		
		
	}
}

```

다음과 같은 결과값을 확인할 수 있다.

\- console

```
ConcurrentLinkedQueue의 오백만개 size() 수행시간 : 0.048초
LinkedList의 오백만개 size() 수행시간 : 0.0초

```

<br>

추가적으로 ConcurrentLinkedQueue의 내부 동작방식은 Singly Linked List로 동작한다. Doubly Linked List인 LinkedList는 Node라는 내부 클래스에 이전요소, 다음요소에 해당하는 Node 인스턴스를 갖고있지만 ConcurrentLinkedQueue는 다음요소에 해당하는 Node 인스턴스만 갖고 있다. 어쩌면 이게 당연할수도 있다고 생각되는게 Queue라는게 FIFO로 구현되어있고 내부 index기반이 아니라 head, tail만 요소에 접근할 수 있기 때문에 굳이 양방향 연결 리스트일 필요가 없다고 생각되기 때문이다.



***



다음시간은 BlockingQueue에 대해서 까보도록 하자!



<br>

포스팅은 여기까지 하겠습니다. 

예제 : https://github.com/sup2is/java-example/tree/master/java-data-structure5

<br>

퍼가실때는 링크와 출처를 반드시 명시해주세요. 감사합니다.

<br>



참고 : https://docs.oracle.com/javase/8/docs/api/