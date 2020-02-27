---

layout: post
title: "LinkedHashMap"
tags: [JAVA, DataStructure, Map, LinkedHashMap]
comments: true
date: 2019-10-31
nav_order: 10
parent: DataStructure
grand_parent: Java
---



HashMap을 상속한 LinkedHashMap을 까보자

***



# LinkedHashMap<K, V>

다음은 `LinkedHashMap`의 구현이다

> ```java
> public class LinkedHashMap<K,V>
> extends HashMap<K,V>
> implements Map<K,V>
> ```
>

다음과같이 HashMap을 상속받고있는데 이 `LinkedHashMap`은 Linked 되도록 설계 되었기때문에 내부는 꽤 복잡한 편이다.

`LinkedHashMap`은 보통 다른 설정이 없다면 기본 설정으로 Map에 삽입되는 key값의 순서를 기반으로 정렬된다.  하지만 기존에 같은 키값이 `LinkedHashMap`에 있을 수 있기 때문에 put() 직전에 contain()을 호출한다. 만약 `LinkedHashMap` 내부에 같은key 존재한다면 `LinkedHashMap`에 새로들어가는게 아니라 기존에 있던 key에 value값이 업데이트된다.

이 `LinkedHashMap`은 페이지 교체 알고리즘 중 하나인 LRU (Least Recently Used) 캐시를 구현하기에 적합하다. `LinkedHashMap`의 생성자 중 하나인 다음 생성자를 사용하면 위에 언급한 key값의 초기 순서 기반 정렬이 아니라 이미 같은 key값이 있더라도 그 key:value 쌍은 `LinkedHashMap` 후미에 들어오게 된다. 또한 get(), getOrDefault() 같은 key를 기반으로 value값을 꺼내는 메서드를 사용해도 사용된 key값은 `LinkedHashMap`의 후미로 들어온다.

<br>

> ```java
>     /**
>      * Constructs an empty <tt>LinkedHashMap</tt> instance with the
>      * specified initial capacity, load factor and ordering mode.
>      *
>      * @param  initialCapacity the initial capacity
>      * @param  loadFactor      the load factor
>      * @param  accessOrder     the ordering mode - <tt>true</tt> for
>      *         access-order, <tt>false</tt> for insertion-order
>      * @throws IllegalArgumentException if the initial capacity is negative
>      *         or the load factor is nonpositive
>      */
>     public LinkedHashMap(int initialCapacity,
>                          float loadFactor,
>                          boolean accessOrder) {
>         super(initialCapacity, loadFactor);
>         this.accessOrder = accessOrder;
>     }
> ```

<br>

조금 자세히 보고싶어서 조금 더 파봤다..

`LinkedHashMap`에는 put() 메서드를 재정의하지 않고 HashMap의 put()을 그대로 사용한다. HashMap에 putVal() 메서드에는 아래와 같은 분기가 있다. 

> ```java
> // putVal() 메서드 중 ...
> 	if (e != null) { // existing mapping for key
>         V oldValue = e.value;
>         if (!onlyIfAbsent || oldValue == null)
>             e.value = value;
>         afterNodeAccess(e);
>         return oldValue;
>     }
> ```

HashMap 에서 afterNodeAccess()는 뼈대만 있고 실제 호출되는 메서드는 `LinkedHashMap`의 afterNodeAccess() 를 호출한다.

> ```java
>     void afterNodeAccess(Node<K,V> e) { // move node to last
>         LinkedHashMap.Entry<K,V> last;
>         if (accessOrder && (last = tail) != e) {
>             LinkedHashMap.Entry<K,V> p =
>                 (LinkedHashMap.Entry<K,V>)e, b = p.before, a = p.after;
>             p.after = null;
>             if (b == null)
>                 head = a;
>             else
>                 b.after = a;
>             if (a != null)
>                 a.before = b;
>             else
>                 last = b;
>             if (last == null)
>                 head = p;
>             else {
>                 p.before = last;
>                 last.after = p;
>             }
>             tail = p;
>             ++modCount;
>         }
>     }
> 
> ```

따라서 accessOrder값에 의해 `LinkedHashMap`에 접근하는 put(), putIfAbsent(), getOrDefault(), computIfAbsent(), computIfPresent(), computIfPresent() 또는 computIfPresent()는 해당 key값을 `LinkedHashMap`의 후미로 값을 밀어넣게된다.

<br>

`LinkedHashMap`에는 removeEldestEntry() 라는 메서드가 있다. Eldest는 '가장 나이가 많은' 이라는 의미가 있는데 말 그대로 `LinkedHashMap`에서 가장 오래된 entry값을 삭제해준다. 아래에서 LRU를 구성하면서 함께 다뤄보도록 하겠다.

<br>

`LinkedHashMap`은 모든 Map의 동작들을 제공하며 null 역시 허용한다. 해시 함수가 버킷사이에 요소를 적절히 분산시킨다고 가정했을때 기본 add(), contains(), remove() 메서드들은 일정한 시간 성능을 제공한다. `LinkedHashMap`은 HashMap보다 성능이 약간 떨어질 수는 있는데 그 이유는 내부에서 연결리스트를 구성하기 때문이다. 한가지 예외가 있는데 `LinkedHashMap`은 capacity와 상관 없이 내부 인스턴스의 size만큼의 시간이 필요하지만 HashMap같은 경우는capacity 기반이기때문에 조금 더 오래 걸릴 수는 있다.

`LinkedHashMap` 역시 HashMap의 생성자인 capacity와 load factor 파라미터를 갖고 있다.  이전시간에 알아본 [JAVA 자료구조 뿌시기 -Map- #1 AbstractMap, HashMap편](https://sup2is.github.io/java-data-structure-9/)에서 capacity와 load factor가 HashMap에 끼치는 영향에 대해서 예제와 함께 알아봤었는데 `LinkedHashMap`은 capacity의 영향이 HashMap보다는 더 적다. 그 이유는 바로 위에 언급한대로 `LinkedHashMap`은 내부 인스턴스의 size 기반으로 동작하기 때문이다.

<br>

이 `LinkedHashMap` 역시 HashMap과 마찬가지로 동기화 처리가 되어있지 않기 때문에 multi-thread환경에서 사용은 적절하지 않다. 만약 사용해야하는 경우 접근하는 thread들이 외부에서 동기화처리되거나 Collections.synchronizedMap 메서드를 사용하여 동기화된 `LinkedHashMap` 을 얻어올 수 있다.

> ```java
> Map m = Collections.synchronizedMap(new `LinkedHashMap`(...));
> ```



`LinkedHashMap` 역시 keySet(), entrySet(), values() 의 iterator 호출 후 `LinkedHashMap` 내부에 데이터 변경이 있을 경우 ConcurrentModificationException 을 발생시키니 주의해야한다.



<br>

# Example

이 `LinkedHashMap`의 기본생성자, capacity와 load factor, accessOrder 파라미터를 받는 생성자를 비교해보고 accessOrder 속성을 사용해서 LRU 를 구성해보는 시간을 갖겠다.

<br>

\- MyLinkedHashMap.class

```java
package data.structure10;

import java.util.LinkedHashMap;
import java.util.Map.Entry;

public class MyLinkedHashMap {

	public static void main(String[] args) throws InterruptedException {

		LinkedHashMap<String, String> lhm = new LinkedHashMap<>();

		for (int i = 0; i < 10; i++) {
			lhm.put("foo" + i, "bar" + i);
		}

		lhm.put("foo5", "re-insert bar5");
		lhm.put("foo11", "bar11");

		for (Entry<String, String> string : lhm.entrySet()) {
			System.out.println(string);
		}

	}

}

```

다음과 같이 foo 를 key값으로 둔 10개가량의 String 인스턴스를 `LinkedHashMap`에 넣어주고 이후에 기존 foo5 라는 key에 새로운 값을 넣어주는 예제다.

<br>

\- console

```
foo0=bar0
foo1=bar1
foo2=bar2
foo3=bar3
foo4=bar4
foo5=re-insert bar5
foo6=bar6
foo7=bar7
foo8=bar8
foo9=bar9
foo11=bar11

```

예상했던것과 마찬가지로 foo5 라는 key값의 위치는 변경이 되지 않고 value 값만 update된 걸 확인할 수 있다.



<br>



다음으로 capacity와 load factor, accessOrder를 받는 생성자를 이용해서 위 예제와 조금 다른 동작을 하도록 구성해봤다.

\- MyLinkedHashMap2.class

```java
package data.structure10;

import java.util.LinkedHashMap;
import java.util.Map.Entry;

public class MyLinkedHashMap2 {

	public static void main(String[] args) throws InterruptedException {

		LinkedHashMap<String, String> lhm = new LinkedHashMap<>(1000, 0.75f, true);

		for (int i = 0; i < 10; i++) {
			lhm.put("foo" + i, "bar" + i);
		}

		lhm.put("foo5", "re-insert bar5");
		lhm.put("foo11", "bar11");
		lhm.get("foo3");
		
		for (Entry<String, String> string : lhm.entrySet()) {
			System.out.println(string);
		}
	}
}

```

첫번째 예제와 다른점이 있다면 `LinkedHashMap`을 생성할때 order 속성을 true로 줬다. 한번 실행해보자.

<br>

\- console

```
foo0=bar0
foo1=bar1
foo2=bar2
foo4=bar4
foo6=bar6
foo7=bar7
foo8=bar8
foo9=bar9
foo5=re-insert bar5
foo11=bar11
foo3=bar3
```

보이는것처럼 put() 또는 get()을 호출한 key값인 foo5, foo11, foo3 key들이 `LinkedHashMap`의 후미로 들어온것을 확인할 수 있다.



<br>

마지막으로 removeEldestEntry를 사용해서 고정 사이즈가 10인 LRU 캐시의 흉내를 한번 내보도록 하겠다.



\- MyLinkedHashMap3.class

```java
package data.structure10;

import java.util.LinkedHashMap;
import java.util.Map.Entry;

public class MyLinkedHashMap3 {

	public static void main(String[] args) throws InterruptedException {

		LinkedHashMap<String, String> lhm = new LinkedHashMap<String, String>(1000,0.75f,true) {
			
			private final int MAX = 10;
			
			protected boolean removeEldestEntry(java.util.Map.Entry<String,String> eldest) {
				return size() >= MAX;
			};
			
		};
		
		for (int i = 0; i < 10; i++) {
			lhm.put("foo" + i, "bar" + i);
		}

		lhm.put("foo5", "re-insert bar5");
		lhm.put("foo4", "re-insert bar4");
		lhm.put("foo12", "bar12");
		lhm.put("foo13", "bar13");
		lhm.put("foo14", "bar14");
		lhm.put("foo15", "bar15");
		lhm.put("foo5", "re-insert bar5");

		for (Entry<String, String> string : lhm.entrySet()) {
			System.out.println(string);
		}
		
	}
	
}

```



2번째 예제에서 removeEldestEntry()을 지정하고 MAX값을 10으로 지정했을때 나오는 결과값이다.

\- console

```
foo7=bar7
foo8=bar8
foo9=bar9
foo4=re-insert bar4
foo12=bar12
foo13=bar13
foo14=bar14
foo15=bar15
foo5=re-insert bar5

```





***



다음시간에는 NavigableMap<K,V>, SortedMap<K,V>, TreeMap<K,V> 을 까보도록 하겠습니다.



<br>

포스팅은 여기까지 하겠습니다. 

예제 : https://github.com/sup2is/java-example/tree/master/java-data-structure10

<br>

퍼가실때는 링크와 출처를 반드시 명시해주세요. 감사합니다.

<br>



참고 : https://docs.oracle.com/javase/8/docs/api/