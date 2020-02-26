---
layout: post
title: "JAVA 자료구조 뿌시기 -List- #1 ArrayList편"
comments: true
tags: [JAVA, DataStructure, ArrayList]
nav_order: 1
parent: Java
---





<br>



정말 바쁜 일상을 보내고 있다. 일과 연애 그리고 공부까지 세마리 토끼를 잡는건 쉽지 않고 어쩌면 모두 다 놓치는 길일 수도 있지만 나에게는 선택의 여지가 없다. .흐극ㄱ



최근에 인터넷 모 선배(안면식도 없지만)에게 조언을 구했는데  과연 내가 이 알고리즘을 하루에 2~3시간 투자하는게 의미가 있을까 .. 실무를 하고 있지만 실무와 연관성이 크게 없는거 같은데 ... 라는 생각이 문득 들어서 말이다.

그 선배님께서 해주신 말씀이 "5~10년차에게 기초질문을 해도 모르는사람이 허다하다." 였다.

다시 생각해보니 그게 어쩌면 나일수도 있을것 같다는 생각을 했다.

ArrayList가 어떤 클래스를 상속, 구현하고 왜 Cloneable 인터페이스를 상속하고 있는지 자세히 말할 수 없다. 내가 가장 많이 사용한 컬렉션이 ArrayList인데도 아직 나는 ArrayList의 자세한 동작 여부를 알지 못한다.



그래서 오늘부로 List, Map, Queue, Stack 등등 자료구조의 특성을 낱낱히 파헤치고 뿌시는시간을 갖겠다.

화이팅!



이글들은 java doc 1.8 기반으로 작성한다 영어실력이 많이 후달리기때문에 오역이나 의역이 있을 수 있다. feat. 구글 번역기



***



<br>

# Collection<E>



먼저 간단하게 List<E>, Set<E>, Queue<E> 등의 super interface가 되는 Collection interface를 살펴보자.  Collection interface의 super interface는 Iterable<E> 가 있는데 Iterable<E> 의 역할은 for-each 기능과 iterator() 의 기능을 가질 수 있다 따라서 ArrayList, Set 등 Collection을 구현한 모든 클래스는 for-earch 기능을 사용할 수 있다.

만약 자신만의 Collection 객체를 생성하고 싶을때는 Collection interface를 구현하는것보다 List나 Set interface를 구현하여 만드는 것을 권장하고 있다.

이 Collection interface를 구현하는 모든 Class는 두개의 표준 생성자를 제공해야하는데 파라미터가 없는 생성자와 Collection 타입의 파라미터를 받는 생성자이다. 

실제 ArrayList class를 살펴보면



> ```java
>     /**
>      * Constructs an empty list with an initial capacity of ten.
>      */
>     public ArrayList() {
>         this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
>     }
> 
>     /**
>      * Constructs a list containing the elements of the specified
>      * collection, in the order they are returned by the collection's
>      * iterator.
>      *
>      * @param c the collection whose elements are to be placed into this list
>      * @throws NullPointerException if the specified collection is null
>      */
>     public ArrayList(Collection<? extends E> c) {
>         elementData = c.toArray();
>         if ((size = elementData.length) != 0) {
>             // c.toArray might (incorrectly) not return Object[] (see 6260652)
>             if (elementData.getClass() != Object[].class)
>                 elementData = Arrays.copyOf(elementData, size, Object[].class);
>         } else {
>             // replace with empty array.
>             this.elementData = EMPTY_ELEMENTDATA;
>         }
>     }
> 
> ```



위와 같이 파라미터가 없는 생성자, Collection 타입을 받는 생성자가 있다. 물론 다른 생성자들도 있다.

위에서 언급한 생성자 두개 생성 원칙을 java에서 강제할 방법은 없지만 모든 java library들은 이 규칙을 준수하고 있다. 본인이 만약 새로운 Collection을 설계할 일이 있으면 참고해볼만 하다.

<br>

이것 이외에도 java doc에는 Collection interface를 구현하는 class에서 금지할 객체나 null object에 대한 처리 등 과 같은 exception 처리에 대한 가이드를 제공해주고 있다. 자신이 필요한 Collection이 만약 null 요소를 금지한다면 null check이후 NullPointerException 같은 에러를 던져주도록 설계하면 된다.

<br>

이것 이외에도 더욱 자세하게 Collection interface를 확인하고 싶다면 java doc을 확인하자!

<br>

# List<E>



이제 Collection을 super interface로 갖는 List interface이다. 

List interface의 super interface는 앞서 언급한 Collection과 Iterable interface가 있다. List interface는 추가된 요소에 대한 정밀한 제어가 가능하고 사용자는 int형 index를 활용해 요소에 접근할 수 있고 Set과는 달리 중복된 요소를 허용한다

List interface는 요소에 대한 기본적인 index 처리 메서드를 총 4가지 제공한다.

1. **add**(int index, [E](https://docs.oracle.com/javase/8/docs/api/java/util/List.html) element)
2. **set**(int index, [E](https://docs.oracle.com/javase/8/docs/api/java/util/List.html) element)
3. **get**(int index)
4. **remove**(int index)

이 index가 기반인 List는 보통 처음시작이 0일때 시작인데 LinkedList같은 경우 만약 1000번째 요소에 접근하기 위해서는 바로 1000번째를 가져오는게 아니라 0에서 1000번째 까지 요소를 지나쳐야하기때문에 많은 비용이 발생한다.

List interface는 ListIterator 라는 특수 반복자를 제공하는데 이것에 대해서는 나중에 Iterator 관련 포스팅에서 자세하게 다루도록 하겠다.

이것 이외에도 List에서 요소에대한 검색을 할때는 구현한 클래스의 검색메서드를 잘 확인해야하는데 예를 들면 contains 같은 ..? 메서드들이 일반적으로 선형검색으로 구현이 되어있기 때문에 사용시에 반드시 주의할것! 이라는 것 등등이 있다. 이것 역시 좀 더 자세하고 정확한 내용은 java doc을 확인하길 추천한다.



<br>

# ArrayList<E>



지금부터는 jdk에서 직접적으로 제공하는 List interface를 구현한 클래스에 대해서 살펴볼껀데 사실 AbstractList도 있지만 그냥 간단하게 List를 최소한으로 구현할 수 있게 만든 뼈대 클래스이기 때문에 넘어간다.



ArrayList class가 상속 및 구현하는 interface 먼저 살펴보도록 하겠다.

> ```java
> public class ArrayList<E> extends AbstractList<E>
> implements List<E>, RandomAccess, Cloneable, Serializable
> ```

<br>

사실 나같은경우는 조금 궁금했던 부분이 있었는데 AbstractList class는 List를 구현하는 추상클래스이다.

> ```java
> public abstract class AbstractList<E>
> extends AbstractCollection<E>
> implements List<E>
> ```

<br>

근데 ArrayList는 AbstractList class를 상속받았기때문에 AbstractList에서 구현하지 않은 List interface의 메서드를 반드시 정의해야하기때문에 ArrayList가 직접적으로 List interface를 구현할 필요가 없었다. 이게 조금 궁금해서 뒤져봤는데



```
Yes. It could've been omitted. But thus it is immediately visible that it is a List. Otherwise an extra click through the code / documentation would be required. I think that's the reason - clarity.

And to add what Joeri Hendrickx commented - it is for the purpose of showing that ArrayList implements List. AbstractList in the whole picture is just for convenience and to reduce code duplication between List implementations.
```



[링크](https://stackoverflow.com/questions/4387419/why-does-arraylist-have-implements-list)에서 위에 힌트를 얻을 수 있었다. ArrayList가 AbstractList를 상속받았지만 이 ArrayList가 List를 구현했다는 사실을 정확하게 알기 위해서는 AbstractList를 까봐야 하는 구조이다. 따라서 어떻게 보면 ArrayList는 List를 구현한 클래스라는 것을 정확하게 명시하기 위해서 추가해 놓은듯 하다.

<br>

앞서 List interface를 확인했기 때문에 AbstractList class나 List interface를 확인할 필요는 없다. 나머지 RandomAccess, Cloneable, Serializable 에 대해 알아보자

<br>

## Cloneable



이 Cloneable interface는 Object.clone과 아주 연관이 깊다. 이 interface를 구현하지 않았다면  Object.clone() 메서드 호출시 CloneNotSupported 을 던진다. 아래는 Object.cloen에서 발췌한 내용이다.



> The method `clone` for class `Object` performs a specific cloning operation. First, if the class of this object does not implement the interface `Cloneable`, then a `CloneNotSupportedException` is thrown. Note that all arrays are considered to implement the interface `Cloneable` and that the return type of the `clone` method of an array type `T[]` is `T[]` where T is any reference or primitive type. Otherwise, this method creates a new instance of the class of this object and initializes all its fields with exactly the contents of the corresponding fields of this object, as if by assignment; the contents of the fields are not themselves cloned. Thus, this method performs a "shallow copy" of this object, not a "deep copy" operation.



관례상 이 Cloenable interface를 구현한 경우 Object.clone() 메서드를 재정의해야하는데 ArrayList 역시 Object.clone()을 재정의했다.

> ```java
>     /**
>      * Returns a shallow copy of this <tt>ArrayList</tt> instance.  (The
>      * elements themselves are not copied.)
>      *
>      * @return a clone of this <tt>ArrayList</tt> instance
>      */
>     public Object clone() {
>         try {
>             ArrayList<?> v = (ArrayList<?>) super.clone();
>             v.elementData = Arrays.copyOf(elementData, size);
>             v.modCount = 0;
>             return v;
>         } catch (CloneNotSupportedException e) {
>             // this shouldn't happen, since we are Cloneable
>             throw new InternalError(e);
>         }
>     }
> ```



<br>

## RandomAccess

이 RandomAccess라는 interface는 Cloneable과 동일하게 marker interface의 역할을 한다. 이 interface를 어디서 어떻게 활용하는지는 정말 자세하게 다뤄야겠지만 일단 List의 요소자체에 random하게 접근 가능게 동작을 변경할 수 있도록 명시해주는 것 이다. 자세한 내용은 java doc을 참고하자 

<br>

## Serializable

이 interface 역시 marker 역할을 하는데 직렬화와 관계가 깊다. 이 interface를 사용하지 않고 serialize deserialize는 불가능하다. 이 직렬화 내용은 이정도만하고 자세한 내용은 [링크](https://nesoy.github.io/articles/2018-04/Java-Serialize) 에서 확인할 수 있다

<br>

***



뭐 이정도면 ArrayList가 상속 및 구현하는 interface에 대해서 전부 알아봤다고 할 수 있다. 사실 interface가 장황하게 많아서 그렇지 marker interface들을 빼면 List interface만 고려하면 된다.

그럼 지금부터 정말 자세하게 ArrayList 자체의 특성에 대해서 살펴보도록하겠다.

일단 ArrayList는 Resizable-array implementation of the List interface 이다. 기본적으로 확장이 가능하다는건데  이것이 바로 일반 array와 ArrayList에 가장 큰 차이점이다. array를 인스턴스화 할때는 배열초기화가 필수이지만 ArrayList같은경우는 그냥 메모리에 인스턴스화 시킬 수 있다는걸 알 수 있다.

ArrayList는 기본적으로 List interface에서 제공하는 모든 메서드들을 구현한 class 인데 null을 포함한 모든 요소들을 Generic type에 의해 저장할 수 있고 내부적으로 사용되는 array의 크기를 제어할 수 있는 방법을 제공한다.

나중에 알아볼 Vector와 매우매우 비슷하지만 가장 큰 차이점은 동기화되지 않는것이다. 

<br>

이어 다음 java doc 다음 구절이

> The size, isEmpty, get, set, iterator, and listIterator operations run in constant time. The add operation runs in amortized constant time, that is, adding n elements requires O(n) time. All of the other operations run in linear time (roughly speaking). The constant factor is low compared to that for the LinkedList implementation.

인데 이게 사실 RandomAccess interface와 관련이 깊다.

<br>



ArrayList는 capacity라는 것이 있는데 capacity는 array에 사용될 수 있는 요소의 개수를 의미한다. 항상 list가 갖는 array.length < capacity를 만족해야한다. ArrayList에는 default가 10개로 잡혀있다. ArrayList에 요소가 들어올때마다 array.length < capacity 를 만족해야하므로 ArrayList는 내부의 array에 대한 resize기능을 하는데 이것도 아래에 자세하게 언급하도록 한다.



ArrayList의 생성자는 총 세개인데 위에서 언급한 두개의 생성자 이외에 하나의 생성자가 더 있다 바로 

```java
    /**
     * Constructs an empty list with the specified initial capacity.
     *
     * @param  initialCapacity  the initial capacity of the list
     * @throws IllegalArgumentException if the specified initial capacity
     *         is negative
     */
    public ArrayList(int initialCapacity) {
        if (initialCapacity > 0) {
            this.elementData = new Object[initialCapacity];
        } else if (initialCapacity == 0) {
            this.elementData = EMPTY_ELEMENTDATA;
        } else {
            throw new IllegalArgumentException("Illegal Capacity: "+
                                               initialCapacity);
        }
    }

```



인데 미리 capacity를 제공해서 내부 array의 크기를 지정할 수 있다. 위와 같이하면 resize하는 비용을 상당히 줄일 수 있고 또는 아래 ensureCapacity() 메서드를 활용하면 된다.



```java
    /**
     * Increases the capacity of this <tt>ArrayList</tt> instance, if
     * necessary, to ensure that it can hold at least the number of elements
     * specified by the minimum capacity argument.
     *
     * @param   minCapacity   the desired minimum capacity
     */
    public void ensureCapacity(int minCapacity) {
        int minExpand = (elementData != DEFAULTCAPACITY_EMPTY_ELEMENTDATA)
            // any size if not default element table
            ? 0
            // larger than default for default empty table. It's already
            // supposed to be at default size.
            : DEFAULT_CAPACITY;

        if (minCapacity > minExpand) {
            ensureExplicitCapacity(minCapacity);
        }
    }
```

<br>



ArrayList는 기본적으로 비동기화 되어 있기 때문에 multithreading 환경에서의 주의가 필요하다. 나는 동시성에 관한 문제는 잘 모르지만 적어도 한개의 thread가 ArrayList에 대해 참조한다면 외부적으로 동기화 되어야 한다. 잘 모르기때문에 자세하게 언급도 불가하지만 동기화 할 수 있는 Object가 list 내부에 존재하지 않을 경우 Collections의 synchronizedList() 메서드를 활용하면 되는듯 하다.

<br>

> List list = Collections.synchronizedList(new ArrayList(...));

이런식으로 말이다. 위와 같이 생긴 ArrayList 객체는 함부로 iterator나 listIterator를 사용하여 수정 삭제하는 경우 ConcurrentModificationException를 발생시킬 수도 있다.

<br>

일반적으로 LinkedList와 ArrayList사이에 ArrayList의 사용비중이 더 높은데 그 이유는 바로 ArrayList는 메모리를 적게차지하고 더 빠르게 동작하기 때문이다. 이부분에 대해서는 나중에 직접 LinkedList를 분석할때 언급하도록 하겠지만 차이가 꽤 큰편이다. 될수있으면 LinkedList를 잊어버리고 List가 필요할때마다 ArrayList를 사용을 추천하고 있다. 



# Example



ArrayList를 분석하면서 궁금했던 예제 몇가지를 실행해서 분석해보도록하겠다. 



1. 실제 ArrayList 내부에 array가 resize 되는 부분
2. ArrayList를 생성할때 10000000 capacity를 부여하여 요소 천만개를 넣는것과 일반 생성자로 ArraList를 생성하여 요소 천만개를 넣는것과 속도차이가 어느정도인지
3. 그 외 잡다한것 ..



이정도가 궁금하기때문에 이제 실제 소스를 까보도록 하자 먼저 ArrayList를 기본생성자로 사용하고 요소 100개를 넣는 예제이다. 이때 살펴볼점은 array.lenght < capacity를 만족하도록 resize하는 부분이다.



\- Main.class

```java
package data.structure1;

import java.util.ArrayList;
import java.util.UUID;

public class Main{

	public static void main(String[] args) {
		ArrayList<String> fooList = new ArrayList<String>();
		
		for (int i = 0; i < 100; i++) {
			fooList.add("");
		}
		
		System.out.println(fooList);
	}
	
}


```



예제는 간단하게 구성하고 내부 동작을 확인하려한다. ArrayList의 Generic type을 String으로 선언하고 UUID 객체를 100번 넣어주는 예제다



ArrayList의 멤버변수가 많지 않으므로 함께 올렸다. 참고하면서 보면 더 좋을듯 하다

```java
    /**
     * Default initial capacity.
     */
    private static final int DEFAULT_CAPACITY = 10;

    /**
     * Shared empty array instance used for empty instances.
     */
    private static final Object[] EMPTY_ELEMENTDATA = {};

    /**
     * Shared empty array instance used for default sized empty instances. We
     * distinguish this from EMPTY_ELEMENTDATA to know how much to inflate when
     * first element is added.
     */
    private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};

    /**
     * The array buffer into which the elements of the ArrayList are stored.
     * The capacity of the ArrayList is the length of this array buffer. Any
     * empty ArrayList with elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA
     * will be expanded to DEFAULT_CAPACITY when the first element is added.
     */
    transient Object[] elementData; // non-private to simplify nested class access

    /**
     * The size of the ArrayList (the number of elements it contains).
     *
     * @serial
     */
    private int size;

    /**
     * Constructs an empty list with the specified initial capacity.
     *
     * @param  initialCapacity  the initial capacity of the list
     * @throws IllegalArgumentException if the specified initial capacity
     *         is negative
     */
```

<br>





ArrayList의 기본생성자로 생성하는 부분이다. 

```java
    /**
     * Constructs an empty list with an initial capacity of ten.
     */
    public ArrayList() {
        this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
    }

```

초기에 생성하면 this.element 는 {}로 메모리에 올라갈 것이다.

이후에 for문 안에 add 메서드의 내부동작을 확인하자



```java
    /**
     * Appends the specified element to the end of this list.
     *
     * @param e element to be appended to this list
     * @return <tt>true</tt> (as specified by {@link Collection#add})
     */
    public boolean add(E e) {
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        elementData[size++] = e;
        return true;
    }

```



for문이 첫번째 동작할때는 일때는 ensureCapacityInternal(size + 1) 의 매개변수는 1이 될것이다 (size 변수가 초기화 되지 않으면 0을 나타내기 때문에)



```java
    private static int calculateCapacity(Object[] elementData, int minCapacity) {
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
            return Math.max(DEFAULT_CAPACITY, minCapacity);
        }
        return minCapacity;
    }

    private void ensureCapacityInternal(int minCapacity) {
        ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
    }

    private void ensureExplicitCapacity(int minCapacity) {
        modCount++;

        // overflow-conscious code
        if (minCapacity - elementData.length > 0)
            grow(minCapacity);
    }
    
...
    
      
    private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
```



ensureCapacityInternal() 메서드를 확인하면 다시 calculateCapacity() 메서드로 현재 elementData와 매개변수로 넘어온 minCapacity값을 비교하여 반환될 capacity값을 결정한다. 초기에는 elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA 조건을 만족하기때문에 10으로 sizing된다. 



이제 ensureExplicitCapacity() 메서드에서 modCount를 제어하는데 사실 이 field는 AbstractList의 멤버변수이다.  이 field는 구조적으로 변경이 몇번 되었는가를 저장하는 변수인데  자세한 설명은 생략한다.



이후에는 grow() 메서드를 호출하는데 적당한 값으로 resize할 크기를 지정하고 Arrays.copyOf(elementData, newCapacity); 를 호출하여 ArrayList 내부에 elementData 의 사이즈를 늘려준다.



사실 이게 resiable-array의 전부이다. 우리가 ArrayList를 사용하면서 array의 크기를 신경쓰지 않고 마구마구 add를 때려주는것도 내부에 이 부분때문에 가능한 것이다.



그려면 capacity를 지정해서 객체를 넣는것과 아닌경우의 속도차이를 체감해보자



\- Main.class

```java
package data.structure1;

import java.util.ArrayList;

public class Main{

	public static void main(String[] args) {
		
		long start;
		long end;
		
		start = System.currentTimeMillis();
		ArrayList<String> fooList = new ArrayList<String>();
		end = System.currentTimeMillis();
		
		System.out.println( "capacity를 지정하지 않았을때 인스턴스 생성 시간 : " + ( end - start )/1000.0 +"초");		
		
		
		start = System.currentTimeMillis();
		for (int i = 0; i < 100000000; i++) {
			fooList.add("foo");
		}
		end = System.currentTimeMillis();
		System.out.println( "capacity를 지정하지 않았을때 실행 시간 : " + ( end - start )/1000.0 +"초");
		
		
		start = System.currentTimeMillis();
		ArrayList<String> barList = new ArrayList<String>(100000000);
		end = System.currentTimeMillis();
		
		System.out.println( "capacity를 지정했을때 인스턴스 생성 시간 : " + ( end - start )/1000.0 +"초");		
		
		start = System.currentTimeMillis();
		for (int i = 0; i < 100000000; i++) {
			barList.add("bar");
		}
		end = System.currentTimeMillis();
		
		System.out.println( "capacity를 지정했을때 실행 시간 : " + ( end - start )/1000.0 +"초");

	}
	
}



```





\- console

```
capacity를 지정하지 않았을때 인스턴스 생성 시간 : 0.0초
capacity를 지정하지 않았을때 실행 시간 : 0.784초
capacity를 지정했을때 인스턴스 생성 시간 : 0.101초
capacity를 지정했을때 실행 시간 : 0.348초
----------------------------------------------
capacity를 지정하지 않았을때 인스턴스 생성 시간 : 0.0초
capacity를 지정하지 않았을때 실행 시간 : 0.796초
capacity를 지정했을때 인스턴스 생성 시간 : 0.102초
capacity를 지정했을때 실행 시간 : 0.315초
----------------------------------------------
capacity를 지정하지 않았을때 인스턴스 생성 시간 : 0.0초
capacity를 지정하지 않았을때 실행 시간 : 0.779초
capacity를 지정했을때 인스턴스 생성 시간 : 0.101초
capacity를 지정했을때 실행 시간 : 0.309초

```



실행시간을 보면 인스턴스가 올라갈때는 어느정도 차이가 있지만 실제 add로 array를 resizing하는 부분이 없기때문에 약 실행시간이 반토막난것을 확인할 수 있다.



ArrayList는 이정도만 확인해보고 사실 안써봐서 몰랐던 List들인데 ArrayList를 상속받은 자료구조가 몇가지 있었다. 가볍게 살펴보도록 하자



# AttributeList

이 javax.management.AttributeList는 ArrayList를 상속받아서 기본적인 ArrayList의 동작방식과 비슷하다. 내부에 실제 Override한 메서드가 많지는 않고 데이터 요소를 지정하는 add(), set() 같은 메서드의 파라미터 타입만 Attribute 로 받는듯 하다. 이 자료구조는 MBean , MBeanServer 등등과 관련이 있고 이 기술은  JMX ( Java Management Extensions) 과 연관이 깊은듯 하니 자세한 설명은 생략한다 .. 

<br>

# RoleList

이름부터 RoleList인데 이것 역시 ArrayList를 상속받은 class이기때문에 동작방식이 비슷하다. 위에서 언급한 AttributeList 처럼 Role객체타입만 받고 null요소를 허용하지 않는다.



# RoleUnresolvedList

RoleList와 굉장히 유사하고 받는 파라미터 타입만 RoleUnresolved 객체만 받고있다.

***



ArrayList관련 포스팅은 이정도가 될 것 같다. 사실 언급하지 못한부분도 많지만 이정도면 ArrayList에 대해서 누구에게 설명할 수 있을것 같다.



<br>

포스팅은 여기까지 하겠습니다. 

예제 : https://github.com/sup2is/java-example/tree/master/java-data-structure1

<br>

퍼가실때는 링크와 출처를 반드시 명시해주세요. 감사합니다.

<br>



참고 : https://docs.oracle.com/javase/8/docs/api/
