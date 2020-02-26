---
layout: post
title: "Java의 Vector, Stack"
tags: [JAVA, DataStructure, Vector, Stack]
comments: true
nav_order: 3
parent: DataStructure
grand_parent: Java
---





<br>

이번시간에는 Vector와 Stack에 대해 알아보도록 하자.



***



# Vector<E>

Vector는 사실 ArrayList와 구현방법이 매우매우 비슷하다. List, RandomAccess, Cloneable, Serializable 을 구현하고 AbstractList을 상속받은것도 ArrayList와 동일하다. 그럼 어디에 차이점이 있을지 한번 알아보도록 하자



Vector 역시 내부에 elementData라는 Object 배열타입을 멤버변수로 두고 객체 요소를 관리한다. 동작여부는 정말 ArrayList와 비슷하게 grow() 메서드로 현재 elementData의 크기를 비교하여 확장을한다.



> ```java
>     /**
>      * Appends the specified element to the end of this Vector.
>      *
>      * @param e element to be appended to this Vector
>      * @return {@code true} (as specified by {@link Collection#add})
>      * @since 1.2
>      */
> 
> 
> 	public synchronized boolean add(E e) {
>         modCount++;
>         ensureCapacityHelper(elementCount + 1);
>         elementData[elementCount++] = e;
>         return true;
>     }   
> 
> 
> ...
> 
> 
> 	/**
>      * This implements the unsynchronized semantics of ensureCapacity.
>      * Synchronized methods in this class can internally call this
>      * method for ensuring capacity without incurring the cost of an
>      * extra synchronization.
>      *
>      * @see #ensureCapacity(int)
>      */
>     private void ensureCapacityHelper(int minCapacity) {
>         // overflow-conscious code
>         if (minCapacity - elementData.length > 0)
>             grow(minCapacity);
>     }
> 
> 
> ...
> 
> 
> 
> 	/**
>      * The maximum size of array to allocate.
>      * Some VMs reserve some header words in an array.
>      * Attempts to allocate larger arrays may result in
>      * OutOfMemoryError: Requested array size exceeds VM limit
>      */
>     private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;
> 
>     private void grow(int minCapacity) {
>         // overflow-conscious code
>         int oldCapacity = elementData.length;
>         int newCapacity = oldCapacity + ((capacityIncrement > 0) ?
>                                          capacityIncrement : oldCapacity);
>         if (newCapacity - minCapacity < 0)
>             newCapacity = minCapacity;
>         if (newCapacity - MAX_ARRAY_SIZE > 0)
>             newCapacity = hugeCapacity(minCapacity);
>         elementData = Arrays.copyOf(elementData, newCapacity);
>     }
> 
>     private static int hugeCapacity(int minCapacity) {
>         if (minCapacity < 0) // overflow
>             throw new OutOfMemoryError();
>         return (minCapacity > MAX_ARRAY_SIZE) ?
>             Integer.MAX_VALUE :
>             MAX_ARRAY_SIZE;
>     }
> 
> ```

<br>



사실 우리가 Vector에서 중요하게 봐야할 점은 따로 있다. Vector는 get(), add(), set() 등등 elementData에 접근하는 웬만한 메서드가 거의 synchronized 처리가 되어있다. 



synchronized는 java의 예약어인데 동기화와 연관이 깊다. 간단하게 설명하면 n개의 thread가 있는 multi thread 환경이라고 가정하고 Vector에 접근하는 thread를 A와 B가 있다고 가정하자.

이때 A가 Vector의 synchronized 처리된 메서드를 호출하는데 이 메서드의 호출시간이 10초가 걸린다고 가정했을때 B는 A의 작업이 끝난 이후에 vector에 접근할 수 있다. A와 B가 동시에 add() 메서드를 호출한다고 해도 아주 미세한 차이로 우선순위를 잡기때문에 먼저 점유한 thread의 작업이 끝나야만 다음 thread가 작업할 수 있다.



synchroinized에 대한 자세한 내용은 [링크](https://tourspace.tistory.com/54)에서 확인할 수 있다.

Vector는 java1.2에서 List interface를 구현하도록 새롭게 추가되고 위에서 언급한대로 ArrayList와 Vector의 가장 큰 차이점은 동기화 처리이므로 multi thread환경에서는 Vector의 사용을 고려해볼만 하다.

<br>

# Example

사실 동시성에 관한 테스트를 ArrayList와 Vector를 비교하면서 테스트를 하는 자료를 찾아보았는데 별로 없었다. 그리고 사실이게 적절한 테스트인지도 잘 모르겠지만 마땅한 자료가 없어서 직접 작성했다. 작성하면서도 이게 제대로 하고있는건가 ..? 라는 생각을 했으니 언제까지나 참고만 해주길 바란다.



\- FooRunner.class

```java
package data.structure3;

import java.util.List;

public class FooRunner extends Thread{

	List<String> myList;
	
	public FooRunner(List<String> list) {
		this.myList = list;
	}
	
	@Override
	public void run() {
		for (int i = 0; i < 10; i++) {
			
			try {
				Thread.sleep((long) (Math.random()*10));
			} catch (InterruptedException ignore) {}
			
			myList.add("foo");
		}
	}

}

```

FooRunner는 1~9 millisecond 만큼 랜덤 Thread.sleep()으로 멈춘 뒤 list에 add("foo") 시키는 thread이다.

<br>

\- MyArrayListTest.class

```java
package data.structure3;

import java.util.ArrayList;
import java.util.Vector;

public class MyArrayListTest {

	public static void main(String[] args) {
		
		
		MyArrayList<String> myArrayList = new MyArrayList<>();
		new FooRunner(myArrayList).start();
		new FooRunner(myArrayList).start();
		new FooRunner(myArrayList).start();
		new FooRunner(myArrayList).start();
		new FooRunner(myArrayList).start();
		new FooRunner(myArrayList).start();
		new FooRunner(myArrayList).start();
		new FooRunner(myArrayList).start();
		new FooRunner(myArrayList).start();
		new FooRunner(myArrayList).start();
		
	}
}

class MyArrayList<E> extends ArrayList<E> {
	
	@Override
	public boolean add(E e) {
		
		try {
			Thread.sleep((long) (Math.random()*10));
		} catch (InterruptedException ignore) {}
		
		System.out.println(this.size());
		return super.add(e);
	}
}
```

<br>



\- MyVectorTest.class

```java
package data.structure3;

import java.util.Vector;

public class MyVectorTest {

	public static void main(String[] args) {
		
		MyVector<String> myVector = new MyVector<>();

		new FooRunner(myVector).start();
		new FooRunner(myVector).start();
		new FooRunner(myVector).start();
		new FooRunner(myVector).start();
		new FooRunner(myVector).start();
		new FooRunner(myVector).start();
		new FooRunner(myVector).start();
		new FooRunner(myVector).start();
		new FooRunner(myVector).start();
		new FooRunner(myVector).start();
		
	}
}

class MyVector<E> extends Vector<E> {
	
	@Override
	public synchronized boolean add(E e) {
		
		try {
			Thread.sleep((long) (Math.random()*10));
		} catch (InterruptedException ignore) {}
		
		System.out.println(this.size());
		return super.add(e);
	}
}

```



Vector와 ArrayList를 상속받은 MyVector, MyArrayList라는 class를 생성해두고 이들의 역할은 그냥 super객체의 add메서드를 호출하기 전에 어떤작업을 수행하고 현재 list의 size() 메서드를 호출 후 data를 넣어주는 역할을 한다.



실행결과를 확인해보자

```

//	  Vector		  ArrayList
        0                 0
        1                 1
        2                 2
        3                 3
        4                 4
        5                 4
        6                 6
        7                 7
        8                 8
        9                 8
        10                8
        11                8
        12                12
        13                13
        14                14
        15                14
        16                14
        17                17
        18                18
        19                19
        20                20
        21                20
        22                22
        23                22
        24                22
        25                25
        26                26
        27                27
        28                27
        29                29
        30                30
        31                30
        32                30
        33                30
        34                30
        35                35
        36                36
        37                37
        38                38
        39                38
        40                38
        41                38
        42                42
        43                43
        44                44
        45                44
        46                44
        47                44
        48                48
        49                49
        50                49
        51                51
        52                52
        53                53
        54                54
        55                54
        56                56
        57                56
        58                58
        59                59
        60                60
        61                60
        62                62
        63                63
        64                63
        65                65
        66                65
        67                67
        68                67
        69                69
        70                70
        71                71
        72                71
        73                73
        74                73
        75                75
        76                76
        77                77
        78                77
        79                79
        80                79
        81                81
        82                79
        83                83
        84                84
        85                85
        86                86
        87                86
        88                88
        89                89
        90                89
        91                91
        92                92
        93                93
        94                94
        95                95
        96                96
        97                97
        98                98
        99                99
```

<br>

Thread를 각 class마다 10개를 실행했으니 size가 100번 호출되는건 MyArrayListTest class나 MyVectorTest class나 같아야한다 하지만 자세히보면 실행결과가 몹시 다르다. 

MyVectorTest는 0~99까지 현재 list의 size()의 호출값이 매우 일정하지만 MyArrayListTest에서는 0-99까지의 size()호출값이 매우 불안정하다. 이는 위에서 언급한것처럼 synchronized 와 매우 밀접한데 Thread A~J 까지 있다고 가정할때 A thread가 먼저 add를 호출하고 있는 과정에서 또다른 Thread가 add 메서드에 접근하기 때문이다. 

ArrayList의 경우 동기화 처리가 안되어있기 때문에 A Thread의 값을 추가하기 이전에 또다른 Thread가 접근해서 size() 메서드를 호출하면 지금과같은 상황이 이루어진다. Vector는 Thread의 동기화를보장하기때문에 A Thread의 add() 가 끝나기 이전에 다른 Thread는 add()에 접근할 수 없다.



따라서 multi thread환경에서는 자료구조 선택을 잘해야한다. 지금이야 단순 size()를 호출하는것 뿐이지만 어플리케이션에서 이런 결함은 아주아주 찾기 힘든 버그가 될 수 있다.



이제 Vector와 ArrayList의 작업수행시간이 어떤게 더 빠른지 확인해 보자.



\- Main.class

```java
package data.structure3;

import java.util.ArrayList;
import java.util.Vector;

public class Main {

	public static void main(String[] args) {
		
		long start;
		long end;
		
		ArrayList<String> fooList = new ArrayList<String>();
		
		start = System.currentTimeMillis();
		for (int i = 0; i < 100000000; i++) {
			fooList.add("foo");
		}
		end = System.currentTimeMillis();
		System.out.println( "ArrayList의 String 요소 천만개 추가시간 : " + ( end - start )/1000.0 +"초");
		
		
		Vector<String> barVector = new Vector<String>();
		
		start = System.currentTimeMillis();
		for (int i = 0; i < 100000000; i++) {
			barVector.add("bar");
		}
		end = System.currentTimeMillis();
		
		System.out.println( "Vector의 String 요소 천만개 추가시간 : " + ( end - start )/1000.0 +"초");
		
		
		
		start = System.currentTimeMillis();
		fooList.get(5000000);
		end = System.currentTimeMillis();
		System.out.println( "ArrayList의 오백만번째 데이터 가져오는 시간 : " + ( end - start )/1000.0 +"초");
		
		start = System.currentTimeMillis();
		fooList.get(5000000);
		end = System.currentTimeMillis();
		System.out.println( "Vector의 오백만번째 데이터 가져오는 시간 : " + ( end - start )/1000.0 +"초");

		
	}
	
	
}
```



<br>

\- console

```
ArrayList의 String 요소 천만개 추가시간 : 0.681초
Vector의 String 요소 천만개 추가시간 : 2.411초
ArrayList의 오백만번째 데이터 가져오는 시간 : 0.0초
Vector의 오백만번째 데이터 가져오는 시간 : 0.0초
===============================================
ArrayList의 String 요소 천만개 추가시간 : 0.693초
Vector의 String 요소 천만개 추가시간 : 2.408초
ArrayList의 오백만번째 데이터 가져오는 시간 : 0.0초
Vector의 오백만번째 데이터 가져오는 시간 : 0.0초
===============================================
ArrayList의 String 요소 천만개 추가시간 : 0.682초
Vector의 String 요소 천만개 추가시간 : 2.444초
ArrayList의 오백만번째 데이터 가져오는 시간 : 0.0초
Vector의 오백만번째 데이터 가져오는 시간 : 0.0초

```



add() 같은 경우는 차이가 꽤 큰편이고 요소를 탐색하는건 둘다 RandomAccess interface를 상속해서 그런지 별로 큰 차이가 없다. 







Vector에 대해서는 이정도만 다루하겠다.

이제 이어서 Stack에 대해서도 간단하게 다뤄볼껀데 Vector의 기본골격안에서 이뤄지기때문에 차이점이 없다고 보면 된다.



# Stack<E>



Stack은 자료구조에서 Last-In-First-Out (LIFO) 을 나타내는 자료구조이다. 후입선출을 뜻하는데 아래 그림을 보면 이해가 빠르게 될 것 같다.



![img](https://upload.wikimedia.org/wikipedia/commons/thumb/b/b4/Lifo_stack.png/350px-Lifo_stack.png)

이미지 출처 : https://en.wikipedia.org/wiki/Stack_(abstract_data_type)

<br>

이 Stack은 Vector를 상속받았는데 재정의한 메서드는 고작 5개밖에 없다.



> | `boolean` | `empty()`Tests if this stack is empty.                       |
> | --------- | ------------------------------------------------------------ |
> | `E`       | `peek()`Looks at the object at the top of this stack without removing it from the stack. |
> | `E`       | `pop()`Removes the object at the top of this stack and returns that object as the value of this function. |
> | `E`       | `push(E item)`Pushes an item onto the top of this stack.     |
> | `int`     | `search(Object o)`Returns the 1-based position where an object is on this stack. |



이제 Stack에서 요소를 어떻게 다루는지 파악해보자.



> ```java
>     /**
>      * Pushes an item onto the top of this stack. This has exactly
>      * the same effect as:
>      * <blockquote><pre>
>      * addElement(item)</pre></blockquote>
>      *
>      * @param   item   the item to be pushed onto this stack.
>      * @return  the <code>item</code> argument.
>      * @see     java.util.Vector#addElement
>      */
>     public E push(E item) {
>         addElement(item);
> 
>         return item;
>     }
> ```

push()같은 경우는그냥 Vector에 addElement를 호출하는것 뿐이다.

<br>

> ```java
>     /**
>      * Looks at the object at the top of this stack without removing it
>      * from the stack.
>      *
>      * @return  the object at the top of this stack (the last item
>      *          of the <tt>Vector</tt> object).
>      * @throws  EmptyStackException  if this stack is empty.
>      */
>     public synchronized E peek() {
>         int     len = size();
> 
>         if (len == 0)
>             throw new EmptyStackException();
>         return elementAt(len - 1);
>     }
> ```

top 요소를 꺼내오는 peek()도 Vector에 있는 size() 호출 후 size() - 1번째요소를 return해주기만 한다.

<br>

> ```java
>     /**
>      * Removes the object at the top of this stack and returns that
>      * object as the value of this function.
>      *
>      * @return  The object at the top of this stack (the last item
>      *          of the <tt>Vector</tt> object).
>      * @throws  EmptyStackException  if this stack is empty.
>      */
>     public synchronized E pop() {
>         E       obj;
>         int     len = size();
> 
>         obj = peek();
>         removeElementAt(len - 1);
> 
>         return obj;
>     }
> ```

top요소를 꺼내고 지우는 pop()도 위에 언급한 peek()으로 데이터를 가져오고 Vector의 removeElementAt()을 호출해서 지운다.

<br>

> ```javascript
>     /**
>      * Tests if this stack is empty.
>      *
>      * @return  <code>true</code> if and only if this stack contains
>      *          no items; <code>false</code> otherwise.
>      */
>     public boolean empty() {
>         return size() == 0;
>     }
> 
>     /**
>      * Returns the 1-based position where an object is on this stack.
>      * If the object <tt>o</tt> occurs as an item in this stack, this
>      * method returns the distance from the top of the stack of the
>      * occurrence nearest the top of the stack; the topmost item on the
>      * stack is considered to be at distance <tt>1</tt>. The <tt>equals</tt>
>      * method is used to compare <tt>o</tt> to the
>      * items in this stack.
>      *
>      * @param   o   the desired object.
>      * @return  the 1-based position from the top of the stack where
>      *          the object is located; the return value <code>-1</code>
>      *          indicates that the object is not on the stack.
>      */
>     public synchronized int search(Object o) {
>         int i = lastIndexOf(o);
> 
>         if (i >= 0) {
>             return size() - i;
>         }
>         return -1;
>     }
> ```

size()는 볼것도 없고 search는()는 Vector의 lastIndexOf()를 호출해서 위에서 부터 탐색한 o 객체의 index를 반환한다. 

<br>



Stack은 이게 전부다. Stack은 Vector를 상속받았기때문에 이또한 역시 thread safe하다.



***



다음시간부터는 Collection inteface Queue를 다루도록 하겠습니다.



<br>

포스팅은 여기까지 하겠습니다. 

예제 : https://github.com/sup2is/java-example/tree/master/java-data-structure3

<br>

퍼가실때는 링크와 출처를 반드시 명시해주세요. 감사합니다.

<br>



참고 : https://docs.oracle.com/javase/8/docs/api/
