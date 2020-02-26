---

layout: post
title: "JAVA 자료구조 뿌시기 -Map- #3 SortedMap, NavigableMap, TreeMap편"
tags: [JAVA, DataStructure, Map, SortedMap, NavigableMap, TreeMap]
comments: true
nav_order: 11
parent: Java
---



Tree 기반인 `TreeMap`을 살펴보기 이전에 `SortedMap`, `NavigableMap`을 먼저 보고 `TreeMap`을 까보자.

***



# SortedMap<K,V>

`SortedMap`의 구성은 다음과 같다.

> ```java
> public interface SortedMap<K,V>
> extends Map<K,V>
> ```



이 Map interface는 이름 그대로 정렬된 Map을 나타내는데 자연정렬 또는 인스턴스 생성시에 제공된 Comparator를 기반으로 정렬된다. 이 정렬은 collection-view인 entrySet(), keySet(), values()가 호출될 때 반영된다. 

모든 key들은 Comparable interface를 구현해야하는데 key들을 비교할때 ClassCastException을 발생해서는 안된다.

다음구문은 참고바란다

> Note that the ordering maintained by a sorted map (whether or not an explicit comparator is provided) must be *consistent with equals* if the sorted map is to correctly implement the `Map` interface. (See the `Comparable` interface or `Comparator` interface for a precise definition of *consistent with equals*.) This is so because the `Map` interface is defined in terms of the `equals` operation, but a sorted map performs all key comparisons using its `compareTo` (or `compare`) method, so two keys that are deemed equal by this method are, from the standpoint of the sorted map, equal. The behavior of a tree map *is* well-defined even if its ordering is inconsistent with equals; it just fails to obey the general contract of the `Map` interface.

~~equals() 에 관한 이야기인듯 ...?~~

<br>

모든 `SortedMap`을 구현하는 class들은 4개의 생성자들을 제공해야한다. 이 규약을 interface 내부에서 강제하지는 못하지만 준수해야한다. 



1.  생성자가 없는 자연정렬의 빈 sorted map
2. Comparator 인스턴스를 인자로 받는 빈 sorted map
3. Map 인스턴스를 인자로 받는 자연정렬된 copyed sorted map
4. `SortedMap` 인스턴스를 인자로 받고 인자로 받은 `SortedMap`과 같은 Comparator를 사용하는 sorted map

<br>

우리가 맨 마지막에 알아볼 `SortedMap`의 구현체인 `TreeMap`은 이 규약을 준수하고 있다.

> ```java
>     public TreeMap() {
>         comparator = null;
>     }
>     
>     //...
>     public TreeMap(Comparator<? super K> comparator) {
>         this.comparator = comparator;
>     }
>     //...
>     
>     public TreeMap(Map<? extends K, ? extends V> m) {
>         comparator = null;
>         putAll(m);
>     }
>     //...
>     
>     public TreeMap(SortedMap<K, ? extends V> m) {
>         comparator = m.comparator();
>         try {
>             buildFromSorted(m.size(), m.entrySet().iterator(), null, null);
>         } catch (java.io.IOException cannotHappen) {
>         } catch (ClassNotFoundException cannotHappen) {
>         }
>     }
>     
> ```



아래 내용도 참고바란다..

> **Note**: several methods return submaps with restricted key ranges. Such ranges are *half-open*, that is, they include their low endpoint but not their high endpoint (where applicable). If you need a *closed range* (which includes both endpoints), and the key type allows for calculation of the successor of a given key, merely request the subrange from `lowEndpoint` to `successor(highEndpoint)`. For example, suppose that `m` is a map whose keys are strings. The following idiom obtains a view containing all of the key-value mappings in `m` whose keys are between `low` and `high`, inclusive:
>
> ```
>    SortedMap<String, V> sub = m.subMap(low, high+"\0");
> ```
>
> A similar technique can be used to generate an *open range*`m``low``high`
>
> ```
>    SortedMap<String, V> sub = m.subMap(low+"\0", high);
> ```



`SortedMap`의 subMap() 메서드는 지정된 key의 범위를 `SortedMap` 인스턴스로 얻을 수 있는 메서드이다. 아래에 `TreeMap` 예제에서 알아볼꺼지만 fromKey와 toKey를 지정했을때 fromKey는 포함하지만 toKey는 포함하지 않는다. 예를 들어 1 2 3 4 5 라는 key가 있을때 fromkey를 2 tokey를 4로 잡을 경우 반환되는 `SortedMap`의 인스턴스는 3 4이다.



<br>



# NavigableMap<K,V>

우리가 살펴볼 `TreeMap`은 `SortedMap`을 상속한 `NavigableMap`을 구현하고 있기 때문에 `NavigableMap`도 한번 봐보자.. ~~사실 document만 봐서는 크게 감이 안오지만 ..~~

> ```java
> public interface NavigableMap<K,V>
> extends SortedMap<K,V>
> ```



`NavigableMap`은 주어진 target을 이용해 가장 가까운 값을 반환한다. 메서드는 lowerEntry(), floorEntry(), ceilingEntry(), higherEntry() 가 있고 각각 return 타입은은 Map.Entry 이다. 만약 주어진 target 값, 즉 비교대상을 찾지 못한다면 null을 반환한다.

<br>

`NavigableMap`은 오름차순 또는 내림차순으로 접근, 통과가 가능하다.  subMap(), headMap(), tailMap() 의 경우 `SortedMap`과 기본 개념은 비슷하지만 inclusive 값을 이용해서 기준이 되는 key값을 포함할지 미포함할지 지정할 수 있다.

<br>

`NavigableMap`은 firstEntry(), pollFirstEntry(), lastEntry(), and pollLastEntry() 메서드를 갖고 있다.  정렬된 기준으로 node의 첫번째 노드를 꺼내낼때는 firstEntry(), 꺼내면서 삭제할때는 pollFirstEntry()를 사용하면 된다.



# TreeMap<K,V>

다음은 대망의 `TreeMap`이다

> ```java
> public class TreeMap<K,V>
> extends AbstractMap<K,V>
> implements NavigableMap<K,V>, Cloneable, Serializable
> ```

<br>

우선 `TreeMap`은 Tree 자료구조의 하나인 Red-Black 을 기반으로 `NavigableMap`을 구현한 class이다. Red-Black tree에 대한 내용을 자세하게 알고 싶다면 [여기](https://zeddios.tistory.com/237)를 참고해보자. 위에서 언급한 `SortedMap`도 구현해야하기 때문에 이 `TreeMap`의 기본 정렬 속성은 자연정렬 또는 인스턴스 생성시 제공된 Comparator 정렬을 따른다.

<br>

이 `TreeMap`의 containsKey(), get(), put(), remove() 메서드의 시간복잡도는 log(n) 을 보장한다. 



`TreeMap` 역시 동기화 처리가 되어있지 않기 때문에 multi-thread환경에서 사용은 적절하지 않다. 만약 사용해야하는 경우 접근하는 thread들이 외부에서 동기화처리되거나 Collections.synchronizedMap 메서드를 사용하여 동기화된 `TreeMap` 을 얻어올 수 있다.



> ```java
>    SortedMap m = Collections.synchronizedSortedMap(new TreeMap(...));
> ```



또 `TreeMap` 역시 keySet(), entrySet(), values() 의 iterator 호출 후 `TreeMap` 내부에 데이터 변경이 있을 경우 ConcurrentModificationException 을 발생시키니 주의해야한다.





# Example

Sorted, Navigable 한 `TreeMap`을 간단한 예제와 함께 사용해보도록 하겠다. 



시나리오는 20개정도의 랜덤한 int 타입의 key를 생성하고 key들의 middle값을 기반으로 하위 entrySet, 상위 entrySet을 구해보고 그 외에 여러가지로 활용해보는 시간을 갖자.



\- MyTreeMap.class

```java
package data.structure11;

import java.util.Arrays;
import java.util.Map.Entry;
import java.util.TreeMap;

public class MyTreeMap {

	public static void main(String[] args) throws InterruptedException {

		TreeMap<Integer, String> tm = new TreeMap<>();
		int[] arr = new int[20];
		
		for (int i = 0; i < 20; i++) {
			int t = (int)(Math.random() * 100) + 1;
			arr[i] = t;
			tm.put(t, "bar" + t);
		}
		
		Arrays.sort(arr);
		
		int middle = arr[arr.length/2 - 1];
		
		for (Entry<Integer, String> s: tm.entrySet()) {
			System.out.println(s);
		}
		
		System.out.println("middle key값 : " + middle);
		
		System.out.println("middle 값의 lowerEntry : " + tm.lowerEntry(middle).getKey() + "=" + tm.lowerEntry(middle).getValue());
		
		System.out.println("middle 값의 higherEntry : " + tm.higherEntry(middle).getKey() + "=" + tm.higherEntry(middle).getValue());
		
		System.out.println("middle 값을 기준으로 상위 entrySet ..");
		
		for (Entry<Integer, String> s : tm.headMap(middle).entrySet()) {
			System.out.println(s);
		}
		
		System.out.println("middle 값을 포함하는 하위 entrySet ..");
		
		for (Entry<Integer, String> s : tm.tailMap(middle).entrySet()) {
			System.out.println(s);
		}
		
		System.out.println("TreeMap의 최상단 노드 : " + tm.firstEntry().getKey() + "=" + tm.firstEntry().getValue());
		
		System.out.println("TreeMap의 최하단 노드 : " + tm.lastEntry().getKey() + "=" + tm.lastEntry().getValue());
		
		
	}

}

```



class의 구분없이 막 때려박으니까 조금 지저분하긴한데 아래 결과를 보면 조금 편할수도 ...

\- console

```
6=bar6
10=bar10
14=bar14
15=bar15
20=bar20
49=bar49
55=bar55
56=bar56
58=bar58
60=bar60
69=bar69
71=bar71
91=bar91
95=bar95
97=bar97
99=bar99
100=bar100

middle key값 : 58

middle 값의 lowerEntry : 56=bar56

middle 값의 higherEntry : 60=bar60

middle 값을 기준으로 상위 entrySet ..
6=bar6
10=bar10
14=bar14
15=bar15
20=bar20
49=bar49
55=bar55
56=bar56

middle 값을 포함하는 하위 entrySet ..
58=bar58
60=bar60
69=bar69
71=bar71
91=bar91
95=bar95
97=bar97
99=bar99
100=bar100

TreeMap의 최상단 노드 : 6=bar6
TreeMap의 최하단 노드 : 100=bar100

```



subMap() 같은 경우 key값이 `TreeMap` 내부에 반드시 존재해야하는것은 아니다.



\- MyTreeMap2.class

```java
package data.structure11;

import java.util.Arrays;
import java.util.SortedMap;
import java.util.Map.Entry;
import java.util.TreeMap;

public class MyTreeMap2 {

	public static void main(String[] args) throws InterruptedException {

		TreeMap<Integer, String> tm = new TreeMap<>();
		int[] arr = new int[20];
		
		for (int i = 0; i < 20; i++) {
			int t = (int)(Math.random() * 100) + 1;
			arr[i] = t;
			tm.put(t, "김아무개의 나이는 :" + t);
		}
		
		Arrays.sort(arr);
		
		int middle = arr[arr.length/2 - 1];
		
		for (Entry<Integer, String> s: tm.entrySet()) {
			System.out.println(s);
		}
		
		System.out.println("middle key값 : " + middle);
		
		SortedMap<Integer, String> NavigableRangeMap = tm.subMap(25 , true, 50, true);
		
		System.out.println("# 25살 ~ 50살의 김아무개 범위 subMap entry ..");
		
		for (Entry<Integer, String> t : NavigableRangeMap.entrySet()) {
			System.out.println(t);
		}
		
	}
}

```

정말 말 그대로 범위이기때문에 아래와 같은 결과를 출력한다. 

\- console

```
11=김아무개의 나이는 :11
13=김아무개의 나이는 :13
14=김아무개의 나이는 :14
16=김아무개의 나이는 :16
23=김아무개의 나이는 :23
24=김아무개의 나이는 :24
31=김아무개의 나이는 :31
50=김아무개의 나이는 :50
63=김아무개의 나이는 :63
65=김아무개의 나이는 :65
72=김아무개의 나이는 :72
78=김아무개의 나이는 :78
80=김아무개의 나이는 :80
85=김아무개의 나이는 :85
89=김아무개의 나이는 :89
92=김아무개의 나이는 :92
middle key값 : 50
# 25살 ~ 50살의 김아무개 범위 subMap entry ..
31=김아무개의 나이는 :31
50=김아무개의 나이는 :50

```

물론 매우 간단한 예제이기때문에 나이가 key값이 될수는 없을 것이다. 예제는 예제일뿐이니 참고바란다.



***



다음시간에는 EnumMap을 까보도록 하겠습니다.

<br>

포스팅은 여기까지 하겠습니다. 

예제 : https://github.com/sup2is/java-example/tree/master/java-data-structure11

<br>

퍼가실때는 링크와 출처를 반드시 명시해주세요. 감사합니다.

<br>



참고 : https://docs.oracle.com/javase/8/docs/api/