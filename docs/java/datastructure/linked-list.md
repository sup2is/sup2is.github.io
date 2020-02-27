---
layout: post
title: "LinkedList"
tags: [JAVA, DataStructure, LinkedList]
comments: true
date: 2019-08-23
nav_order: 2
parent: DataStructure
grand_parent: Java
---







오늘은 LinkedList에 대해 알아보는 시간을 갖겠습니당



***



# LinkedList<E>

LinkedList에 대해 알아보자.  먼저 LinkedList는 다음 class를 상속 및 구현하고 있다.



> ```java
> public class LinkedList<E>
> extends AbstractSequentialList<E>
> implements List<E>, Deque<E>, Cloneable, Serializable
> ```



LinkedList는 List interface의 거의 웬만한 메서드는 전부 구현한 클래스다. 이 LinkedList는 말 그대로 연결리스트를 뜻하고 있는데 Doubly Linked List 로 구현되어 있다. 간단하게 Doubly Linked List와 Singly Linked List에 대해 설명한다.



![1_iMYmkYDCSrXXdwpbqm-ekA](https://user-images.githubusercontent.com/30790184/63559933-dcaf1380-c58e-11e9-819a-c82236d44e0d.jpeg)

이미지 출처 : https://medium.com/journey-of-one-thousand-apps/data-structures-in-the-real-world-508f5968545a



## Singly Linked List

Singly Linked List는 한개의 노드안에 데이터와 다음 노드의 주소값을 포함하는 자료구조이다. 그림에서 확인할 수 있듯이 바로 다음 노드의 주소만 가지고 있기 때문에 양방향 탐색이 불가능하다.



## Doubly Linked List

Doubly  Linked List는 한개의 노드안에 데이터, 이전 노드의 주소값, 다음 노드의 주소값을 포함하는 자료구조이다. 그림을 보면 더 쉽게 이해가 가능한데 이전 또는 다음 노드의 주소값을 알기때문에 양방향탐색이 가능하다. 



Doubly Linked List와 Singly Linked List에 대한 설명은 [링크](https://opentutorials.org/module/1335/8940)에서 확인할 수 있다.

<br>



***



이 LinkedList는 List interface와 Deque interface을 동시에 구현하고 있고 null 요소를 포함한다. 이 LinkedList는 Doubly Linked List로 구현되었기 때문에 역시 양방향 탐색으로 인덱싱을 한다.



이것 이외에는 사실 javadoc에 써진 내용은 ArrayList와 비슷하다 thread-safe하지 않기때문에 multi thread환경에서 사용시 주의해야하고 iterator() 또는 listIterator()로 노드에대한 수정??도 각별한 주의가 필요하다.



LinkedList는 List뿐만아니라 Deque interface도 구현했기때문에 자료구조의 queue역할도 동시에 가능하다. queue라는 자료구조는 FIFO (First-In-First-Out) 의 기능을 하는 자료구조인데 이건 추후에 Queue interface를 분석할때 자세하게 언급하겠다.



안에를 조금 들여다보면

> ```java
>  	transient int size = 0;
> 
>     /**
>      * Pointer to first node.
>      * Invariant: (first == null && last == null) ||
>      *            (first.prev == null && first.item != null)
>      */
>     transient Node<E> first;
> 
>     /**
>      * Pointer to last node.
>      * Invariant: (first == null && last == null) ||
>      *            (last.next == null && last.item != null)
>      */
>     transient Node<E> last;
> 
> //...여러 메서드들 .. add() ,set() 등
> 
> 	private static class Node<E> {
>         E item;
>         Node<E> next;
>         Node<E> prev;
> 
>         Node(Node<E> prev, E element, Node<E> next) {
>             this.item = element;
>             this.next = next;
>             this.prev = prev;
>         }
>     }
> 
> ```

LinkedList의 내부 객체관리는 ArrayList와는 다르게 Node라는 private static class 를 사용한다.



다음 add 메서드를 한번 확인해보자



> ```java
>     /**
>      * Appends the specified element to the end of this list.
>      *
>      * <p>This method is equivalent to {@link #addLast}.
>      *
>      * @param e element to be appended to this list
>      * @return {@code true} (as specified by {@link Collection#add})
>      */
>     public boolean add(E e) {
>         linkLast(e);
>         return true;
>     }
>     
>     ...
>         
>         /**
>      * Links e as first element.
>      */
>     private void linkFirst(E e) {
>         final Node<E> f = first;
>         final Node<E> newNode = new Node<>(null, e, f);
>         first = newNode;
>         if (f == null)
>             last = newNode;
>         else
>             f.prev = newNode;
>         size++;
>         modCount++;
>     }
> 
>     
>         /**
>      * Links e as last element.
>      */
>     void linkLast(E e) {
>         final Node<E> l = last;
>         final Node<E> newNode = new Node<>(l, e, null);
>         last = newNode;
>         if (l == null)
>             first = newNode;
>         else
>             l.next = newNode;
>         size++;
>         modCount++;
>     }
> ```



LinkedList 의 add() 메서드를 실행하면 LinkedList의 마지막요소에 데이터가 추가되는데 동작 방식을 한번 자세하게 살펴보도록 하겠다.



\- Main.class

```java
public class Main{

	public static void main(String[] args) {
		
		
		1. LinkedList<String> fooList = new LinkedList<>();
		
		2. fooList.add("foo");
		3. fooList.add("bar");
		
	}
	
}
```



1번에서 fooList가 기본생성자로 생성이 됐다면 초기에 fooList가 갖고있는 last 라는 멤버변수는 null 상태일 것이다. 



2번에서 add() 메서드를 실행하면 linkLast() 메서드로 내가 추가하려는 "foo" 라는 요소가 파라미터로 들어가게된다. 초기에 실제 생성되는 노드는 다음과 같다.

```
final Node<E> newNode = new Node<>(null, "foo", null);
```



위에 Node라는 private class를 확인해보면 첫번째 인자는 이전요소, 두번째 인자는 현재요소, 세번째인자는 다음요소를 나타내는데 초기에 생성되는 Node는 "foo" 라는 현재요소만으로 인스턴스가 생성이 된다.



이후에 초기 l이라는 변수는 null 값이므로 지금 생성된 newNode는 LinkedList의 멤버변수인 last와 first에 바인딩 된다.

```java
	last = newNode;    
	if (l == null)
            first = newNode;
        else
            l.next = newNode;
```



3번이 실행되면 마찬가지로 linkLast() 메서드의 인자로 "bar"가 들어가는데 이때 newNode는 다음과 같이 생성이 된다.

```
final Node<E> newNode = new Node<>( ("foo" 요소를 갖는 Node객체) , "bar", null);
```

 

약간 느낌이 온다 .. 



이런식으로 LinkedList는 내부적으로 Doubly Linked List의 기능을 수행하고 있다. 그럼 양방향 탐색에 대한 처리는 어떻게 할지도 약간 궁금하다. get() 메서드에 내부동작을 살펴보자.



> ```java
> 
>     // Positional Access Operations
> 
>     /**
>      * Returns the element at the specified position in this list.
>      *
>      * @param index index of the element to return
>      * @return the element at the specified position in this list
>      * @throws IndexOutOfBoundsException {@inheritDoc}
>      */
>     public E get(int index) {
>         checkElementIndex(index);
>         return node(index).item;
>     }
> 
> 
> ...
> 
>     
>     
>     private void checkElementIndex(int index) {
>         if (!isElementIndex(index))
>             throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
>     }
> 
> ...
>     
>     /**
>      * Tells if the argument is the index of a valid position for an
>      * iterator or an add operation.
>      */
>     private boolean isPositionIndex(int index) {
>         return index >= 0 && index <= size;
>     }
> 
> ...
>     
>     /**
>      * Returns the (non-null) Node at the specified element index.
>      */
>     Node<E> node(int index) {
>         // assert isElementIndex(index);
> 
>         if (index < (size >> 1)) {
>             Node<E> x = first;
>             for (int i = 0; i < index; i++)
>                 x = x.next;
>             return x;
>         } else {
>             Node<E> x = last;
>             for (int i = size - 1; i > index; i--)
>                 x = x.prev;
>             return x;
>         }
>     }
> 
> ```



우리가 가지고있는 fooList의 size는 2이기때문에 get(3)으로 호출하면 당연히 IndexOutOfBoundsException이 떨어질 것이다. 사실 자세하게 봐야하는부분이 node() 라는 메서드이다. 앞에서 언급은 안했지만 LinkedList는 ArrayList 에서 mark interface 역할을하는 RandomAccess interface를 구현하지 않았다. 이말은 바로 LinkedList 내부에서 직접 어떤작업을 한다는건데 node() 메서드가 바로 이부분이다.



조금 더 쉽게 알아보기 위하여 앞서 fooList에 요소를 다음과 같이 추가한다

```java
public class Main {

	public static void main(String[] args) {
		
		
		LinkedList<String> fooList = new LinkedList<>();
		
		for (int i = 0; i < 10; i++) {
			fooList.add("foo" + i);
		}
		
		fooList.get(3);
		
	}
}

```

<br>

총 10개의 "foo{i}" 값이 fooList에 들어가고 fooList에 3번째 인덱스 값을 꺼내오도록 하겠다. 

<br>

> ```
>   1     /**
>   2      * Returns the (non-null) Node at the specified element index.
>   3      */
>   4     Node<E> node(int index) {
>   5         // assert isElementIndex(index);
>   6
>   7         if (index < (size >> 1)) {
>   8             Node<E> x = first;
>   9             for (int i = 0; i < index; i++)
>  10                 x = x.next;
>  11             return x;
>  12         } else {
>  13             Node<E> x = last;
>  14             for (int i = size - 1; i > index; i--)
>  15                 x = x.prev;
>  16             return x;
>  17         }
>  18     }
> 
> ```

10개의 요소가 들어간 이후에 fooList에 size 멤버변수는 10인데 이는 2진수로 1010이다. 7번째 라인에 시프트연산을하는데  1010 을 >> 연산을하면 비트를 한개씩 밀어내기때문에 0101이되고 그 값은 10진수로 5로 표현이된다. 7번째 라인의 if문(3 < 5)을 만족하므로 첫번째 노드부터 순차적으로 검색한다.

<br>

LinkedList는 위에서도 언급했듯이 양방향 탐색으로 구현이 되었기 때문에 탐색도 현재 사이즈와 index 값을 비교하여 뒤에서부터 탐색하거나 처음부터 탐색하는 방법으로 구현되어있다. ArrayList는 RandomAccess로 Random하게 요소에 접근하는게 아마 LinkedList와 ArrayList와의 가장 큰 차이점이 아닐까 싶다.

<br>





# Example

그럼 ArrayList와 LinkedList에 요소를 넣을때, 탐색할때의 속도차이를 한번 비교해보도록 하겠다.



```java
public class Main {

	public static void main(String[] args) {
		
		
		
		ArrayList<String> arrayList = new ArrayList<>();
		LinkedList<String> linkedList = new LinkedList<>();
		long start , end;

		start = System.currentTimeMillis();
		for (int i = 0; i < 10000000; i++) {
			arrayList.add("foo" + i);
		}
		end = System.currentTimeMillis();
		System.out.println("arrayList에 요소 천만개 넣을때 걸리는 시간 : " + (end - start)/1000.0 + "초");

		
		start = System.currentTimeMillis();
		for (int i = 0; i < 10000000; i++) {
			linkedList.add("bar" + i);
		}
		end = System.currentTimeMillis();
		System.out.println("linkedList에 요소 천만개 넣을때 걸리는 시간 : " + (end - start)/1000.0 + "초");
		

		start = System.currentTimeMillis();
		arrayList.get(5000000);
		end = System.currentTimeMillis();
		System.out.println("arrayList에 오백만번째 요소 가져올때 걸리는 시간 : " + (end - start)/1000.0 + "초");
		
		
		start = System.currentTimeMillis();
		linkedList.get(5000000);
		end = System.currentTimeMillis();
		System.out.println("linkedList에 오백만번째 요소 가져올때 걸리는 시간 : " + (end - start)/1000.0 + "초");
		
		
	}
}
```



실행결과는 상당히 놀랍다.



```
arrayList에 요소 천만개 넣을때 걸리는 시간 : 0.963초
linkedList에 요소 천만개 넣을때 걸리는 시간 : 2.706초
arrayList에 오백만번째 요소 가져올때 걸리는 시간 : 0.0초
linkedList에 오백만번째 요소 가져올때 걸리는 시간 : 2.962초
===================================================
arrayList에 요소 천만개 넣을때 걸리는 시간 : 1.031초
linkedList에 요소 천만개 넣을때 걸리는 시간 : 2.888초
arrayList에 오백만번째 요소 가져올때 걸리는 시간 : 0.0초
linkedList에 오백만번째 요소 가져올때 걸리는 시간 : 4.819초
===================================================
arrayList에 요소 천만개 넣을때 걸리는 시간 : 0.904초
linkedList에 요소 천만개 넣을때 걸리는 시간 : 2.741초
arrayList에 오백만번째 요소 가져올때 걸리는 시간 : 0.0초
linkedList에 오백만번째 요소 가져올때 걸리는 시간 : 0.101초
===================================================
arrayList에 요소 천만개 넣을때 걸리는 시간 : 0.929초
linkedList에 요소 천만개 넣을때 걸리는 시간 : 2.783초
arrayList에 오백만번째 요소 가져올때 걸리는 시간 : 0.0초
linkedList에 오백만번째 요소 가져올때 걸리는 시간 : 0.113초
```

총 네번실행했을때 ArrayList에 천만개의 string요소를 추가할때는 1초 LinkedList는 거의 3초가량걸렸다. LinkedList는 내부에서 Node 라는 인스턴스를 하나 더 만들어서 그러지 않을까 싶다. get으로 중간번째 쯤 있는 요소를 가져올때도 속도차이는 매우매우 차이가 난다. ArrayList는 0초, LinkedList는 첫번째, 두번째는 3~5초가량걸렸고 이후 실행할떄는 0.1초정도 걸렸다.



정말정말 놀라운거는 LinkedList를 실행하면 실행할수록 속도가 줄어들었고 8회차?정도 됐을때는 중간요소 탐색에 0.01초가 걸렸다 .. 



뭐지 ..? jvm내부에서 캐싱을하나 ..? 내 pc 메모리문제였나 ...? 뭔가 다시 재현이 안된다 ...



아무튼간에 그래도 ArrayList보다 LinkedList가 확실히 작업시간이 오래걸리긴하니 LinkedList보다 ArrayList의 사용을 권장하는 이유가 바로 이거지 않을까 싶다.



<br>

***



다음시간에는 Vector를 다루도록 하겠습니다.



<br>

포스팅은 여기까지 하겠습니다. 

예제: https://github.com/sup2is/java-example/tree/master/java-data-structure2

<br>

퍼가실때는 링크와 출처를 반드시 명시해주세요. 감사합니다.

<br>



참고 : https://docs.oracle.com/javase/8/docs/api/
