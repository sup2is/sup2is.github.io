---

layout: post
title: "JAVA 자료구조 뿌시기 -Queue- #1 PriorityQueue편"
tags: [JAVA, DataStructure, Queue, PriorityQueue]
comments: true
nav_order: 4
parent: Java
---





앞서 3편동안 List에 대해 알아봤는데 이 글을 읽으시는 분들은 도움이 되실지 모르겠지만 ... 나에게는 아주 큰 도움이 되고 있다. 이어서 Queue도 한번 뒤적 뒤적 ..



------



# Queue<E>



Queue interface는 List와 동일하게 super interface를 Collection, Iterable로 두고 있다. 따라서 List와 동일하게 forEach 기능을 사용할 수 있고 Collection을 구현해야 한다.

Queue는 우선순위 기반으로 요소를 저장하도록 설계되었다. 기본 컬렉션 이외에도 추가 삽입, 추출 및 검사 메서드를 제공한다.



|             | *Throws exception*                                           | *Returns special value*                                      |
| ----------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **Insert**  | [`add(e)`](https://docs.oracle.com/javase/8/docs/api/java/util/Queue.html#add-E-) | [`offer(e)`](https://docs.oracle.com/javase/8/docs/api/java/util/Queue.html#offer-E-) |
| **Remove**  | [`remove()`](https://docs.oracle.com/javase/8/docs/api/java/util/Queue.html#remove--) | [`poll()`](https://docs.oracle.com/javase/8/docs/api/java/util/Queue.html#poll--) |
| **Examine** | [`element()`](https://docs.oracle.com/javase/8/docs/api/java/util/Queue.html#element--) | [`peek()`](https://docs.oracle.com/javase/8/docs/api/java/util/Queue.html#peek--) |

<br>

위에서 언급한 3가지 메서드가 표에 해당한다. Queue는 기본 Collection interface에서 제공하는 add(), remove(), element() 외에 offer(), poll(), peek() 메서드를 제공한다.

Queue는 통상적으로 FIFO (First-In-First-Out) 로 구현이 되는데 FIFO에 대해 간략하게 설명하자면 아래와 같다.

![FIFO](https://upload.wikimedia.org/wikipedia/commons/thumb/d/d3/Fifo_queue.png/525px-Fifo_queue.png)

출처 : https://upload.wikimedia.org/wikipedia/commons/thumb/d/d3/Fifo_queue.png/525px-Fifo_queue.png

<br>

간단하게 설명하면 요소를 들어오는 순서대로 꺼내오는 자료구조 형태이다. 그림을 보면 이해가 빠르다. 하지만 Queue 라는 interface가 항상 FIFO를 나타내는건 아니다. 곧 알아볼 Priority Queue 같은 경우는 제공하는 Comparator에 따라 최대힙, 최소힙을 구현할 수 있기 때문이다.

위에 언급한 Priority Queue라던지 아니면 일반 FIFO로 구현된 Queue던지 정렬방식에 상관하지않고 head 요소를 빼오는건 remove() 또는 poll() 메서드를 사용한다. 일반적인 Queue는 tail에 데이터가 들어가지만 아닌 Queue도 있다. 모든 Queue는 반드시 정렬 속성을 지정해줘야한다.

Queue에만 있는 메서드들이 다양하게 있지만 그중에 offer() 라는 메서드는 return 타입이 boolean이다. 맞다. 요소가 들어가면 true를 반환하고 아닐경우 false를 반환한다. javadoc에서는 Collection에 add()와 구현방식이 다르다고 적혀 있는데 실제로 보면 add()랑 offer()랑 별로 다를만한게 없다.. ArrayDeque, PriorityQueue 등 웬만한 Queue에서는 add() 내부에 offer()를 호출하거나 offer()가 add()를 호출, 또는 같은 메서드를 호출하기 때문이다. 

앞서 언급한 remove()와 poll()에 대해 알아보면 둘 다 head 에 있는 요소를 빼오는 동작 방식은 같으나 Queue가 비어있을때 remove()는 예외를던지고 poll()은 null을 반환하도록 설계되었다.

아래는 ArrayDeque의 실제 내부 동작 소스이다.

> ```java
> 
> 
> /**
>      * Retrieves and removes the head of the queue represented by this deque.
>      *
>      * This method differs from {@link #poll poll} only in that it throws an
>      * exception if this deque is empty.
>      *
>      * <p>This method is equivalent to {@link #removeFirst}.
>      *
>      * @return the head of the queue represented by this deque
>      * @throws NoSuchElementException {@inheritDoc}
>      */
>     public E remove() {
>         return removeFirst();
>     }
> 
> 
> ...
>     
>      /**
>      * @throws NoSuchElementException {@inheritDoc}
>      */
>     public E removeFirst() {
>         E x = pollFirst();
>         if (x == null)
>             throw new NoSuchElementException();
>         return x;
>     }
> 
> 
> ------------------------------------------------------------------------
>     
>     /**
>      * Retrieves and removes the head of the queue represented by this deque
>      * (in other words, the first element of this deque), or returns
>      * {@code null} if this deque is empty.
>      *
>      * <p>This method is equivalent to {@link #pollFirst}.
>      *
>      * @return the head of the queue represented by this deque, or
>      *         {@code null} if this deque is empty
>      */
>     public E poll() {
>         return pollFirst();
>     }
>     
> 
> ...
>     
>     public E pollFirst() {
>         int h = head;
>         @SuppressWarnings("unchecked")
>         E result = (E) elements[h];
>         // Element is null if deque empty
>         if (result == null)
>             return null;
>         elements[h] = null;     // Must null out slot
>         head = (h + 1) & (elements.length - 1);
>         return result;
>     }
> 
>     public E pollLast() {
>         int t = (tail - 1) & (elements.length - 1);
>         @SuppressWarnings("unchecked")
>         E result = (E) elements[t];
>         if (result == null)
>             return null;
>         elements[t] = null;
>         tail = t;
>         return result;
>     }
> 
> ```

<br>

element()와 peek() 도 있는데 내부동작방식을보면 element()는 대체로 요소가 없을때 예외를 던지도록 구현되었고 peek()은 그냥 null 요소를 반환하게 되어있는 것 같다. 둘의 공통점은 요소를 꺼내와도 remove()나 poll()처럼 요소를 삭제하지 않고 그냥 단순 return만 해준다.

<br>

Queue는 일반적으로 null요소를 허용하지 않는다.  예외적으로 LinkedList는 null 요소를 허용하지만 그래도 일반적으로 null 요소를 금지한다. 이유는 Queue가 비어있는 상태에서 poll()요소를 반환하면 null을 return하기 때문이다.

Queue를 구현할때는 equals(), hashCode() 를 정의할때도 주의해야한다. 이것에 대한 자세한 언급은 생략한다.

<br>

이정도가 Queue interface에 대한 javadoc의 설명이고 Queue는 사실 Queue로만 구현한 class는 PriorityQueue, ConcurrentLinkedQueue 정도밖에 없고 Queue의 sub interface인 BlockingDeque, BlockingQueue, Deque, TransferQueue interface 를 구현하는 경우가 더 많다.



이번시간에는 PriorityQueue, ConcurrentLinkedQueue에 대해 알아보도록할껀데 그전에 아~주 간단하게 AbstractQueue에 대해서 알아보도록 하겠다.



# AbstractQueue<E>

이 AbstractQueue는 말그대로 abstract class다. 이 AbstractQueue는 Queue를 구현하고있는데 add(), remove(), element() 모두 구현될 offer(), poll(), peek()을 호출하고 있다. AbstractQueue는 null 요소를 허용하지 않는다.

<br>

> ```java
>     /**
>      * Inserts the specified element into this queue if it is possible to do so
>      * immediately without violating capacity restrictions, returning
>      * <tt>true</tt> upon success and throwing an <tt>IllegalStateException</tt>
>      * if no space is currently available.
>      *
>      * <p>This implementation returns <tt>true</tt> if <tt>offer</tt> succeeds,
>      * else throws an <tt>IllegalStateException</tt>.
>      *
>      * @param e the element to add
>      * @return <tt>true</tt> (as specified by {@link Collection#add})
>      * @throws IllegalStateException if the element cannot be added at this
>      *         time due to capacity restrictions
>      * @throws ClassCastException if the class of the specified element
>      *         prevents it from being added to this queue
>      * @throws NullPointerException if the specified element is null and
>      *         this queue does not permit null elements
>      * @throws IllegalArgumentException if some property of this element
>      *         prevents it from being added to this queue
>      */
>     public boolean add(E e) {
>         if (offer(e))
>             return true;
>         else
>             throw new IllegalStateException("Queue full");
>     }
> 
>     /**
>      * Retrieves and removes the head of this queue.  This method differs
>      * from {@link #poll poll} only in that it throws an exception if this
>      * queue is empty.
>      *
>      * <p>This implementation returns the result of <tt>poll</tt>
>      * unless the queue is empty.
>      *
>      * @return the head of this queue
>      * @throws NoSuchElementException if this queue is empty
>      */
>     public E remove() {
>         E x = poll();
>         if (x != null)
>             return x;
>         else
>             throw new NoSuchElementException();
>     }
> 
>     /**
>      * Retrieves, but does not remove, the head of this queue.  This method
>      * differs from {@link #peek peek} only in that it throws an exception if
>      * this queue is empty.
>      *
>      * <p>This implementation returns the result of <tt>peek</tt>
>      * unless the queue is empty.
>      *
>      * @return the head of this queue
>      * @throws NoSuchElementException if this queue is empty
>      */
>     public E element() {
>         E x = peek();
>         if (x != null)
>             return x;
>         else
>             throw new NoSuchElementException();
>     }
> 
>     /**
>      * Removes all of the elements from this queue.
>      * The queue will be empty after this call returns.
>      *
>      * <p>This implementation repeatedly invokes {@link #poll poll} until it
>      * returns <tt>null</tt>.
>      */
>     public void clear() {
>         while (poll() != null)
>             ;
>     }
> ```





# PriorityQueue<E>

이어서 PriorityQueue는 다음과 같이 정의되어 있다.

> ```java
> public class PriorityQueue<E>
> extends AbstractQueue<E>
> implements Serializable
> ```

위에서 언급한 AbstractQueue를 super class로 두고있고 Serializable 되도록 설계되었다.

그럼 실제로 PriorityQueue가 위에서 언급한 null요소를 허용하지 않는지 부터 먼저 보자.

<br>

> ```java
>    
>     /**
>      * Inserts the specified element into this priority queue.
>      *
>      * @return {@code true} (as specified by {@link Collection#add})
>      * @throws ClassCastException if the specified element cannot be
>      *         compared with elements currently in this priority queue
>      *         according to the priority queue's ordering
>      * @throws NullPointerException if the specified element is null
>      */
>     public boolean add(E e) {
>         return offer(e);
>     } 
> ...
> 
> 	/**
>      * Inserts the specified element into this priority queue.
>      *
>      * @return {@code true} (as specified by {@link Queue#offer})
>      * @throws ClassCastException if the specified element cannot be
>      *         compared with elements currently in this priority queue
>      *         according to the priority queue's ordering
>      * @throws NullPointerException if the specified element is null
>      */
>     public boolean offer(E e) {
>         if (e == null)
>             throw new NullPointerException();
>         modCount++;
>         int i = size;
>         if (i >= queue.length)
>             grow(i + 1);
>         size = i + 1;
>         if (i == 0)
>             queue[0] = e;
>         else
>             siftUp(i, e);
>         return true;
>     }
> ```

<br>

add()는 offer()를 호출하도록 AbstractQueue에서 설계되었지만 PriorityQueue에서 명시적으로 한번 더 써준것 같다. offer()를 보면 넘어오는 요소가 null이면 NullPointerException을 떨구는걸 보니 마음이 편안해진다...

<br>

이제 PriorityQueue에 대해 자세히 살펴보자. PriorityQueue는 우선순위 heap을 기반으로 구현되었는데 heap이란 자료구조는 나중에 자세하게 언급하는걸로 하고 간단하게 설명하자면 최소힙, 최대힙으로 구분이되는데 최소힙은 PriorityQueue 내부에 있는 요소중 가장 작은 요소를 반환한다. 예를들어 최소힙으로 구성하고 int 요소를 순서대로 5 2 3 6 1 을 offer() 해줄 경우 일반적인 FIFO를 따른다면 poll()을 했을때 5가 나오겠지만 최소힙에서는 1이나온다. 반대로 최대힙은 요소중 가장 큰 수인 6을 반환한다.

PriorityQueue는 생성되는시점에 Comparator<E> interface 타입의 비교자를 갖는 생성자가 있는데 기본적으로 new PriortyQueue()로 인스턴스를 생성하면 PriorityQueue는 자연? 순서에따라 최소힙을 구성한다. 만약 Comparator를 제공하면 그 Comparator에 해당하는 heap을 구성하게 된다. 이 PriorityQueue는 비교불가능한 객체의 삽입을 허용하지 않는다.

<br>

PriorityQueue 역시 ArrayList처럼 capacity를 지정하여 인스턴스를 생성할 수 있고 grow()라는 메서드로 내부 배열을 증가시킨다.

> ```java
> /**
>  * Creates a {@code PriorityQueue} with the specified initial
>  * capacity that orders its elements according to their
>  * {@linkplain Comparable natural ordering}.
>  *
>  * @param initialCapacity the initial capacity for this priority queue
>  * @throws IllegalArgumentException if {@code initialCapacity} is less
>  *         than 1
>  */
> public PriorityQueue(int initialCapacity) {
>     this(initialCapacity, null);
> }
> ```

<br>

PriorityQueue도 Collection interface를 구현했기때문에 Iterator를 구현해야한다. 하지만 Iterator를 사용할 때 주의해야할 점은 최소힙, 최대힙으로 PriorityQueue를 구현했더라도 Iterator는 순서를 보장하지 않는다. 만약 필요할 경우엔 **Arrays.sort(pq.toArray())**를 고려해야한다.

<br>

이 PriorityQueue는 thread-safe하지 않기 때문에 multi thread환경에서는 역시 사용을 지양해야한다. 만약 multi thread환경일때는 PriorityBlockingQueue를 사용해야한다.

PriorityQueue의 enqueuing, dequeuing은 O(log(n))의 시간복잡도를 갖고 있다.



# Example

Priority Queue로 최대힙, 최소힙을 구현하는방법과 최대힙이나 최소힙을 구현했을떄 Iterator 호출시 순서를 보장하는지  확인해보도록 하겠다.



```java
    /**
     * Creates a {@code PriorityQueue} with the default initial capacity and
     * whose elements are ordered according to the specified comparator.
     *
     * @param  comparator the comparator that will be used to order this
     *         priority queue.  If {@code null}, the {@linkplain Comparable
     *         natural ordering} of the elements will be used.
     * @since 1.8
     */
    public PriorityQueue(Comparator<? super E> comparator) {
        this(DEFAULT_INITIAL_CAPACITY, comparator);
    }

```

정수값으로 최대힙을 구성한다고 가정했을때 방법은 생각보다 간단하다.

최소힙은 **new PriorityQueue();** 로 선언하면된다. 이럴 경우 정렬된 최상위 요소가 가장 작은 값을 나타낸다.

최대힙같은경우는 **new PriorityQueue(Collections.reverseOrder())** 로 생성해주면 최대힙으로 생성되고 정렬된 최상위 요소는 가장 큰 값을 나타낸다.



\- Main.class



```java
package data.structure4;

import java.util.Collections;
import java.util.PriorityQueue;

public class Main {

	public static void main(String[] args) {

		int[] arr = new int[] {1,5,3,7,8,4,2,9,6};
		
		PriorityQueue<Integer> minHeap = new PriorityQueue<>();
		
		for (int i = 0; i < arr.length; i++) {
			minHeap.add(arr[i]);
		}
		

		System.out.println("최소힙 실행 결과");
		
		for (int i = 0; i < arr.length; i++) {
			System.out.println(minHeap.poll());
		}
		
		System.out.println("============================");
		
		
		PriorityQueue<Integer> maxHeap = new PriorityQueue<>(Collections.reverseOrder());

		for (int i = 0; i < arr.length; i++) {
			maxHeap.add(arr[i]);
		}
		
		System.out.println("최대힙 실행 결과");
		
		for (int i = 0; i < arr.length; i++) {
			System.out.println(maxHeap.poll());
		}

	}

}
```

<br>

\- console

```
최소힙 실행 결과
1
2
3
4
5
6
7
8
9
============================
최대힙 실행 결과
9
8
7
6
5
4
3
2
1

```

<br>

PriorityQueue는 요소를 추가할때마다 siftUpUsingComparator() 라는 메서드를 호출 후 인스턴스 생성 당시 comparator 멤버변수를 통해서 비교작업을 실행한다. 따라서 요소를 한번에 넣던 중간에 넣던 최소값 또는 최대값을 유지할 수 있는것이다.

```java
    @SuppressWarnings("unchecked")
    private void siftUpUsingComparator(int k, E x) {
        while (k > 0) {
            int parent = (k - 1) >>> 1;
            Object e = queue[parent];
            if (comparator.compare(x, (E) e) >= 0)
                break;
            queue[k] = e;
            k = parent;
        }
        queue[k] = x;
    }
```

<br>

과연 iterator는 이 순서를 보장하는지 한번 확인해보자.

\- Main2.java

```java
package data.structure4;

import java.util.Iterator;
import java.util.PriorityQueue;

public class Main2 {

	public static void main(String[] args) {
	
		PriorityQueue<Integer> pq = new PriorityQueue<>();
		
		for (int i = 0; i < 50; i++) {
			pq.add((int) (Math.random() * 100));
		}
		
		System.out.println("PriorityQueue의 Iterator 사용");

		Iterator<Integer> iterator = pq.iterator();
		
		while (iterator.hasNext()) {
			System.out.println(iterator.next());
		}
		
		System.out.println("------------------------------");
		System.out.println("PriorityQueue의 poll() 사용");
		
		for (int i = 0; i < 50; i++) {
			System.out.println(pq.poll());
		}

	}

}
```

\- console

```
                                                                  
  PriorityQueue의 Iterator 사용     PriorityQueue의 poll()사용
                6                      6                           
                13                     9                          
                9                      12                          
                23                     13                         
                26                     13                         
                13                     19                         
                12                     23                         
                38                     26                         
                42                     26                         
                32                     28                         
                28                     32                         
                26                     36                         
                49                     37                         
                46                     38                         
                19                     39                         
                40                     40                         
                45                     40                         
                54                     41                         
                50                     42                         
                40                     43                         
                36                     45                         
                41                     46                         
                39                     46                         
                37                     47                         
                46                     49                         
                64                     50                         
                96                     51                         
                86                     51                         
                81                     52                         
                58                     52                         
                62                     54                         
                62                     55                         
                51                     57                         
                83                     58                         
                51                     62                         
                81                     62                         
                67                     64                         
                64                     64                         
                57                     67                         
                52                     74                         
                99                     81                         
                52                     81                         
                74                     81                         
                55                     83                         
                43                     86                         
                93                     90                         
                47                     90                         
                90                     93                         
                90                     96                         
                81                     99                         
                                                                  
```

결과가 꽤 재미있는편이다. Iterator를 사용할경우 순서를 보장하지 않기때문에 매우 주의해야한다.

<br>

마지막으로 비교불가능한객체에 대해서 PriorityQueue는 어떻게 동작할지 한번 확인해 보자

\- Main3.class

```java
package data.structure4;

import java.util.Comparator;
import java.util.PriorityQueue;

public class Main3 {

	public static void main(String[] args) {
	
 		User user1 = new User(27 , "클라우스");
 		User user2 = new User(31 , "유하");
 		User user3 = new User(26 , "스프링초보");
 		
 		//Comparator를 제공하지 않았을때
1. 		//PriorityQueue<User> pq = new PriorityQueue<User>();
 		
 		//Comparator를 제공했을때
2. 		PriorityQueue<User> pq = new PriorityQueue<User>(new MyComparator<User>());
 		
 		pq.add(user1);
 		pq.add(user2);
 		pq.add(user3);
 	
 		System.out.println("========");
 	
 		while (!pq.isEmpty()) {
			System.out.println(pq.poll());
		}
 	
	}

}

class MyComparator<T> implements Comparator<T> {

	@Override
	public int compare(T o1, T o2) {
		return Integer.compare(((User)o1).getAge() , ((User)o2).getAge()); 
	}

}

class User {
		
	private int age;
	private String name;
	
//getter, setter, tostring, constructure
	

}
```



User class 처럼 class가 Comparable을 구현하고 있지 않는 이상 1번같은경우는 2번째요소가 들어갈때 반드시 에러를 뱉는다. 이럴경우는 Priority Queue에 Comparator를 제공해줘야한다.

\- console

```
//Comparator 없이 실행

Exception in thread "main" java.lang.ClassCastException: data.structure4.User cannot be cast to java.lang.Comparable
   at java.util.PriorityQueue.siftUpComparable(PriorityQueue.java:653)
   at java.util.PriorityQueue.siftUp(PriorityQueue.java:648)
   at java.util.PriorityQueue.offer(PriorityQueue.java:345)
   at java.util.PriorityQueue.add(PriorityQueue.java:322)
   at data.structure4.Main3.main(Main3.java:19)

//Comparator 제공

User [age=26, name=스프링초보]
User [age=27, name=클라우스]
User [age=31, name=유하]


```

<br>

추가로 기본 원시타입 이외에 String 은 실패가 나지 않는다. 왜냐하면 String은 Comparable을 구현하고 있기 때문..



------



이정도로 Priority Queue의 분석을 마치겠다.



<br>

포스팅은 여기까지 하겠습니다. 

예제 : https://github.com/sup2is/java-example/tree/master/java-data-structure4

<br>

퍼가실때는 링크와 출처를 반드시 명시해주세요. 감사합니다.

<br>



참고 : https://docs.oracle.com/javase/8/docs/api/