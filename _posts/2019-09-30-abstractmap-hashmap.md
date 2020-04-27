---

layout: post
title: "Java 자료구조 파헤치기 #9 AbstractMap, HashMap"
tags: [JAVA, DataStructure, Map, AbstractMap, HashMap]
comments: true
date: 2019-09-30
nav_order: 9
parent: DataStructure
grand_parent: Java
---



사실 java Collection의 마지막인 Set interface를 다루려했으나 내부에 Map 인스턴스를 두고 활용하는 Set 구현체가 몇개 있어서 포스팅 도중 다음편을 Map으로 해야겠다고 생각하여 Map을 먼저 다루고 Set을 다루도록 하겠다.

***



# Map<K, V>

사실 이 `Map`은 java Collection Framework에 해당하는 Collection이 아니다 Collection을 super interface를 두지 않기 때문이다. 일반적으로 생각해봐도 Collection<E>의 제네릭타입은 한개, `Map`의 제네릭타입은<K, V> 로 두개이기 때문이다. 이 key:value 관계가 Collection 하위에 속하지 못하는 가장 큰 이유가 된다. 자세한 사항은 [여기](https://stackoverflow.com/questions/2651819/why-doesnt-java-map-extend-collection)를 참고해도 좋을 듯 하다.

따라서 `Map`은 `Map` 자체의 고유한 interface를 두고 있다

<br>

`Map`은 위에서 언급한것처럼 key:value의 구조를 갖고 있는데 이 key값은 value값과 1:1매칭이 원칙이기때문에 key가 중복이 될 수는 없다. 이 `Map`은 세가지의 Collection view를 갖고 있다.  keySet(), entrySet(), values() 메서드이다. 각각 key값, key:value값, value값을 얻어올 수 있는데 keySet()과 entrySet()은 중복이 불가능하기때문에 Set interface로 반환되고 value값은 중복이 가능하기때문에 Collection interface 타입으로 반환된다.

이 Collection view의 정렬은 구현체 내부 Comparator를 갖고 있는 TreeMap의 경우 Comparator를 갖기때문에 인스턴스 생성시 만들어놓은 Comparator가 있다면 순서를 보장한다. 하지만 `HashMap` 같은 Comparator가 없는경우 반복 순서를 보장하지 않는다.



> ```java
> //TreeMap의 Comparator를 갖는 생성자
> 
>     /**
>      * Constructs a new, empty tree map, ordered according to the given
>      * comparator.  All keys inserted into the map must be <em>mutually
>      * comparable</em> by the given comparator: {@code comparator.compare(k1,
>      * k2)} must not throw a {@code ClassCastException} for any keys
>      * {@code k1} and {@code k2} in the map.  If the user attempts to put
>      * a key into the map that violates this constraint, the {@code put(Object
>      * key, Object value)} call will throw a
>      * {@code ClassCastException}.
>      *
>      * @param comparator the comparator that will be used to order this map.
>      *        If {@code null}, the {@linkplain Comparable natural
>      *        ordering} of the keys will be used.
>      */
>     public TreeMap(Comparator<? super K> comparator) {
>         this.comparator = comparator;
>     }
> 
> 
> ```



변경 가능한객체, 즉 mutable한 객체가 이 `Map`의 key값으로 들어온다면 매우매우 주의해야한다.  개체가 지도의 키인동안 개체의 값이 비교에 영향을 미치는 방식으로 변경이 된다면 이경우 동작하지 않는다.

<br>

모든 `Map`구현 클래스는 다음과 같은 두개의 생성자를 포함해야한다.

- 빈 맵을 생성하는 파라미터가 없는 생성자
- 동일한 `Map` 인스턴스를 갖는 생성자 (복사 기능)



> ```java
> //TreeMap
> 
>     /**
>      * Constructs a new tree map containing the same mappings as the given
>      * map, ordered according to the <em>natural ordering</em> of its keys.
>      * All keys inserted into the new map must implement the {@link
>      * Comparable} interface.  Furthermore, all such keys must be
>      * <em>mutually comparable</em>: {@code k1.compareTo(k2)} must not throw
>      * a {@code ClassCastException} for any keys {@code k1} and
>      * {@code k2} in the map.  This method runs in n*log(n) time.
>      *
>      * @param  m the map whose mappings are to be placed in this map
>      * @throws ClassCastException if the keys in m are not {@link Comparable},
>      *         or are not mutually comparable
>      * @throws NullPointerException if the specified map is null
>      */
>     public TreeMap(Map<? extends K, ? extends V> m) {
>         comparator = null;
>         putAll(m);
>     }
> 
> 
> //HashMap
> 
>     /**
>      * Constructs a new <tt>HashMap</tt> with the same mappings as the
>      * specified <tt>Map</tt>.  The <tt>HashMap</tt> is created with
>      * default load factor (0.75) and an initial capacity sufficient to
>      * hold the mappings in the specified <tt>Map</tt>.
>      *
>      * @param   m the map whose mappings are to be placed in this map
>      * @throws  NullPointerException if the specified map is null
>      */
>     public HashMap(Map<? extends K, ? extends V> m) {
>         this.loadFactor = DEFAULT_LOAD_FACTOR;
>         putMapEntries(m, false);
>     }
> 
> ```



이 생성자들을 반드시 제공해야한다는 규칙은 없지만 반드시 지켜야하는 권고사항이다. JDK의 모든 `Map`구현체는 이 두가지의 표준생성자를 제공하고있다.

<br>

수정 불가능한 `Map` 구현체를 수정하려는 작업이 있을경우 UnsupportedOperationException 을 호출 할 수 있지만 반드시 이런식으로 구현할 필요는 없다.

<br>

`Map` 구현체중 key:value에 null의 요소를 금지하는 경우도 있고 key값의 어떤 일정한 타입을 금지하는 경우도 있다. 이 경우 삽입연산에서 NullPointer Exception이나 ClassCastException을 만날 수 있다. 부적격한 key 또는 value 에 대한 질의연산중에서도 어떤 exception을 만나거나 또는 boolean false값을 얻을 수도 있기때문에 반드시 본인이 사용하는 `Map` 구현체의 특징을 잘 알아야 할 것이다.

<br>

이외에 두 단락은 참고만 하는걸로 하자 ..

> ```
> Many methods in Collections Framework interfaces are defined in terms of the equals method. For example, the specification for the containsKey(Object key) method says: "returns true if and only if this map contains a mapping for a key k such that (key==null ? k==null : key.equals(k))." This specification should not be construed to imply that invoking Map.containsKey with a non-null argument key will cause key.equals(k) to be invoked for any key k. Implementations are free to implement optimizations whereby the equals invocation is avoided, for example, by first comparing the hash codes of the two keys. (The Object.hashCode() specification guarantees that two objects with unequal hash codes cannot be equal.) More generally, implementations of the various Collections Framework interfaces are free to take advantage of the specified behavior of underlying Object methods wherever the implementor deems it appropriate.
> 
> Some map operations which perform recursive traversal of the map may fail with an exception for self-referential instances where the map directly or indirectly contains itself. This includes the clone(), equals(), hashCode() and toString() methods. Implementations may optionally handle the self-referential scenario, however most current implementations do not do so.
> ```



<br>



# AbstractMap<K, V>

이제 `Map`의 구현체를 살펴보기 전에 `AbstractMap` class를 잠깐 보고 가도록 하겠다. 아래는 `AbstractMap`의 구조이다.

```java
public abstract class AbstractMap<K,V>
extends Object
implements Map<K,V>
```

이 `AbstractMap` class는 `Map`을 아주 최소한만 구현하기 위해 만들어놓은 skeleton class 이다. 수정 불가능한 `Map`을 구현하려면 이 class를 상속받아서 entrySet() 메서드만 구현하면 된다. 

만약 수정 가능한 `Map`을 구현하려면 해당 class를 상속받고 put 메서드를 재정의하면 된다. 만약 그렇게 하지 않으면 UnsupportedOperationException 발생할 것이다. 그리고 entrySet().iterator()의 remove() 역시 재정의 해야한다.

위에서 언급한것과 마찬가지로 이 `AbstractMap`의 구현체는 두개의 표준 생성자를 제공하도록 구현해야한다.



# HashMap<K, V>

다음은 `HashMap`의 구조이다

```java
public class HashMap<K,V>
extends AbstractMap<K,V>
implements Map<K,V>, Cloneable, Serializable
```

`HashMap`은 위에서 언급한 `AbstractMap` 이라는 abstract class를 상속받아서 구현한 인스턴스 가능 객체이다. 이 `HashMap`을 hash table 기반으로 동작하는데 hash table에 대한 자세한 설명은 [이곳](https://bcho.tistory.com/1072)을 확인해주길 바란다. 내 나름대로 해석해서 간략하게 표현해보자면 

![](https://upload.wikimedia.org/wikipedia/commons/thumb/7/7d/Hash_table_3_1_1_0_1_0_0_SP.svg/1200px-Hash_table_3_1_1_0_1_0_0_SP.svg.png)

출처 : https://en.wikipedia.org/wiki/Hash_table

hash table은 key를 통해 value를 매칭시켜주는 자료구조이다. 데이터를 저장할때는 hash 함수를 통해 key에 해당하는 index값을 가져오고 그곳에 value를 저장시키고 데이터를 꺼내올때도 역시 hash 함수를 통해 key에 해당하는 index값을 가져온 뒤 그곳에 value를 꺼내오는 구조이다.

<br>

위 그림처럼 John Smith라는 key가 어떠한 hash 함수에의해 02번째라는 index번호가 지어졌기때문에 위와 같은 그림처럼 동작된다.

이 hash table의 단점이 있다면 hash 함수가 만들어낸 index가 충돌할 수 있다는 점이 있다는것이다. 이걸 해결하는 방법은 위에서 언급한 [이곳](https://bcho.tistory.com/1072)을 확인하면 될 것 같다.

사실 굉장히 큰 단점이긴 하지만 이 hash table은 O(1)의 시간복잡도를 갖고 있기 때문에 굉장히 빠른연산을 도와준다. List처럼 어떠한 index의 값을 얻기 위해서는 선형검색을 피할 수 없는 O(n)의 구조인데 hash table은 그냥 hash 함수를 통한 key의 index 값을 얻어오면 연산이 끝나기 때문이다. 이 [블로그](https://ratsgo.github.io/dat a%20structure&algorithm/2017/10/25/hash/)역시 굉장히 좋은글이라 함께 첨부한다.

<br>

이 `HashMap`의 모든 `Map` 동작은 null을 포함하도록 설계됐다. key 또는 value값에 null이 올 수 있다. 동기화처리가 되어있지 않고 null을 포함하는것만 제외하면 java.util.Hashtable과 동일한 기능을 한다. 

`HashMap`의 기본동작중 get(), put()은 hash 함수가 buckets 사이에 요소를 적절히 분산시킨다고 가정할 때 일정한 시간 성능을 제공한다. 위에서 언급했던 keySet(), entrySet(), values()의 시간은 `HashMap`의 capacity(number of buckets) + `HashMap`의 size(the number of key-value mappings)에 비례하는 시간이 필요하다. 따라서 반복 성능이 중요한 경우 초기 용량을 너무 높게 설정하지 않는것이 매우 중요하다. load factor 역시 너무 낮아도 반복성능이 좋지 않다. load factor 는 아래에서 언급한다.

<br>

`HashMap`은 capacity와 load factor를 파라미터로 받는 생성자가 존재한다.

> ```java
>     /**
>      * Constructs an empty <tt>HashMap</tt> with the specified initial
>      * capacity and load factor.
>      *
>      * @param  initialCapacity the initial capacity
>      * @param  loadFactor      the load factor
>      * @throws IllegalArgumentException if the initial capacity is negative
>      *         or the load factor is nonpositive
>      */
>     public HashMap(int initialCapacity, float loadFactor) {
>         if (initialCapacity < 0)
>             throw new IllegalArgumentException("Illegal initial capacity: " +
>                                                initialCapacity);
>         if (initialCapacity > MAXIMUM_CAPACITY)
>             initialCapacity = MAXIMUM_CAPACITY;
>         if (loadFactor <= 0 || Float.isNaN(loadFactor))
>             throw new IllegalArgumentException("Illegal load factor: " +
>                                                loadFactor);
>         this.loadFactor = loadFactor;
>         this.threshold = tableSizeFor(initialCapacity);
>     }
> 
> 
> 
>     /**
>      * Constructs an empty <tt>HashMap</tt> with the specified initial
>      * capacity and the default load factor (0.75).
>      *
>      * @param  initialCapacity the initial capacity.
>      * @throws IllegalArgumentException if the initial capacity is negative.
>      */
>     public HashMap(int initialCapacity) {
>         this(initialCapacity, DEFAULT_LOAD_FACTOR);
>     }
> ```

capacity같은 경우는 초기 hash table의 buckets 용량을 지정하는것이고 load factor는 hash table의 buckets 크기가 자동으로 증가하기 전에 얼마나 가득차도록 허용 할 수있는지에 대한 수치이다. 만약 이 load factor에 해당하는 값 이상의 buckets크기가 된다면 이때 hash table의 크기를 약 2배 증설한다. 이 증설과정에서 데이터 구조는 rebuilt된다.

<br>

만약 따로 load factor를 지정하지 않을 경우 default 값으로 .75 값이 설정이 된다. 가장 기본적인 설정만큼 가장 적절한 설정인데 이 load factor가 높을 경우 공간 overhead는 감소하지만 조회 비용이 올라간다. 만약 capacity값이 최대 항목 수를 하중 요인으로 나눈 값보다 클 경우 rebuilt 작업은 발생하지 않기때문에 초기에 선언할때 고려해야한다. 만약 많은 데이터가 `HashMap`에 저장되어야 하는 경우엔 초기에 capacity값을 충분히 고려하면 hash table을 확장하면서 데이터 구조를 rebuilt하는것보다 더 효율적으로 데이터를 저장할 수 있기 때문이다.

<br>

hashCode() 함수가 만약 동일한 key값을 return 한다면 성능상에 이슈가 있을 수 있는점에 유의해야한다.

<br>

이 `HashMap`은 동기화 처리가 되어있지 않기 때문에 multi-thread환경에서 사용은 적절하지 않다. 만약 사용해야하는 경우 접근하는 thread들이 외부에서 동기화처리되거나 Collections.synchronizedMap 메서드를 사용하여 동기화된 `HashMap`을 얻어올 수 있다.



> ```java
> Map m = Collections.synchronizedMap(new HashMap(...));
> ```



<br>

`HashMap` 역시 keySet(), entrySet(), values() 의 iterator 호출 후 `HashMap` 내부에 데이터 변경이 있을 경우 ConcurrentModificationException 을 발생시키니 주의해야한다.



# Example

`HashMap`의 Collection view 메서드들인 keySet(), entrySet(), values() 을 활용하여 반복시키는 예제와 capacity, load factor와 `HashMap`간의 성능 이슈에 대한 예제로 마무리하도록 하겠다.



- keySet(), entrySet(), values()  활용하기



\- MyHashMap.class



```java
package data.structure9;

import java.util.HashMap;
import java.util.Iterator;
import java.util.Map.Entry;

public class MyHashMap {

	public static void main(String[] args) throws InterruptedException {

		HashMap<String, String> hashMap = new HashMap<>();
		
		for (int i = 0; i < 10; i++) {
			hashMap.put("foo" + i, "bar" + i);
		}
		
		Iterator<String> keyIterator = hashMap.keySet().iterator();
		
		System.out.println("## HashMap의 keySet()");
		
		while (keyIterator.hasNext()) {
			System.out.println(keyIterator.next());
		}
		
		Iterator<Entry<String, String>> entryIterator = hashMap.entrySet().iterator();
		
		System.out.println("## HashMap의 entrySet()");
		
		while (entryIterator.hasNext()) {
			Entry<String,String> entry = entryIterator.next();
			System.out.println(entry.getKey() + " : " + entry.getValue());
		}
		
		Iterator<String> valueIterator =  hashMap.values().iterator();
		
		System.out.println("## HashMap의 values()");
		
		while (valueIterator.hasNext()) {
			System.out.println(valueIterator.next());
		}
		
	
	}

}
```

<br>

\- console

```
## HashMap의 keySet()
foo6
foo7
foo8
foo9
foo0
foo1
foo2
foo3
foo4
foo5
## HashMap의 entrySet()
foo6 : bar6
foo7 : bar7
foo8 : bar8
foo9 : bar9
foo0 : bar0
foo1 : bar1
foo2 : bar2
foo3 : bar3
foo4 : bar4
foo5 : bar5
## HashMap의 values()
bar6
bar7
bar8
bar9
bar0
bar1
bar2
bar3
bar4
bar5

```

<br>



- `HashMap`과 capacity, load factor의 관계

여기에서는 

1. String:String 요소 백만개를 넣을때 capacity를 지정하지 않은 경우
2. String:String 요소 백만개를 넣을때 capacity만 지정한 경우
3. String:String 요소 백만개를 넣을때 capacity와 load factor를 0.4f 로 지정한경우
4. String:String 요소 백만개를 넣을때 capacity와 load factor를 0.95f 로 지정한경우



이렇게 총 4가지로 실행시켜보도록 하겠다.

```java
package data.structure9;

import java.util.HashMap;
import java.util.Map;

public class MyHashMap2 {

	public static void main(String[] args) throws InterruptedException {

		long start;
		long end;
		
		//capacity 없이 생성
		HashMap<String, String> hashMap = new HashMap<>();
		
		start = System.currentTimeMillis();
		addMilionElement(hashMap);
		end = System.currentTimeMillis();
		
		System.out.println( "capacity를 지정하지 않았을때 String:String 요소 오백만개 put() 소요 시간 : " + ( end - start )/1000.0 +"초");		
		
		
		//capacity 250만으로 생성
		hashMap = new HashMap<>(2500000);
		
		start = System.currentTimeMillis();
		addMilionElement(hashMap);
		end = System.currentTimeMillis();
		
		System.out.println( "capacity를 250만으로 지정하고 String:String 요소 오백만개 put() 소요 시간 : " + ( end - start )/1000.0 +"초");		
		
		
		//capacity 250만, load factor 0.4f 로 생성
		hashMap = new HashMap<>(2500000 , 0.4f);
		
		start = System.currentTimeMillis();
		addMilionElement(hashMap);
		end = System.currentTimeMillis();
		
		System.out.println( "capacity를 250만, load factor를 0.4f으로 지정하고 String:String 요소 오백만개 put() 소요 시간 : " + ( end - start )/1000.0 +"초");		
		
		//capacity 250만, load factor 0.95f 로 생성
		hashMap = new HashMap<>(2500000 , 0.95f);
		
		start = System.currentTimeMillis();
		addMilionElement(hashMap);
		end = System.currentTimeMillis();
		
		System.out.println( "capacity를 250만, load factor를 0.95f으로 지정하고 String:String 요소 오백만개 put() 소요 시간 : " + ( end - start )/1000.0 +"초");		
		
	}
	
	static void addMilionElement(Map<String, String> map) {
		for (int i = 0; i < 5000000; i++) {
			map.put("foo" + i, "bar + i");
		}
	}

}
```



\- console

```
capacity를 지정하지 않았을때 String:String 요소 500만개 put() 소요 시간 : 1.992초
capacity를 250만으로 지정하고 String:String 요소 500만개 put() 소요 시간 : 1.114초
capacity를 250만, load factor를 0.4f으로 지정하고 String:String 요소 500만개 put() 소요 시간 : 0.873초
capacity를 250만, load factor를 0.95f으로 지정하고 String:String 요소 500만개 put() 소요 시간 : 0.629초
===================================================================================
capacity를 지정하지 않았을때 String:String 요소 500만개 put() 소요 시간 : 2.354초
capacity를 250만으로 지정하고 String:String 요소 500만개 put() 소요 시간 : 1.117초
capacity를 250만, load factor를 0.4f으로 지정하고 String:String 요소 500만개 put() 소요 시간 : 0.793초
capacity를 250만, load factor를 0.95f으로 지정하고 String:String 요소 500만개 put() 소요 시간 : 0.64초

```

사실 이게 정말 정확한 자료가 될지는 의문이지만 capacity와 load factor를 지정하고 안하고의 차이가 꽤 큰편이다 나같은경우는 지금 put()만 실험해봤지만 get()같은 경우는 아마 맨 마지막 4번이 가장 오래걸릴것으로 예측된다.





***



다음시간에는 HashMap을 상속한 LinkedHashMap을 까보도록 하겠습니다.



<br>

포스팅은 여기까지 하겠습니다. 

예제 : https://github.com/sup2is/java-example/tree/master/java-data-structure9

<br>

퍼가실때는 링크와 출처를 반드시 명시해주세요. 감사합니다.

<br>



참고 : https://docs.oracle.com/javase/8/docs/api/