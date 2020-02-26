---

layout: post
title: "Java의 EnumMap"
tags: [JAVA, DataStructure, Map, EnumMap]
comments: true
nav_order: 12
parent: DataStructure
grand_parent: Java
---



오늘은 `EnumMap`을 한번 까보도록 하겠다.

***

# EnumMap<K extends Enum<K>,V>

다음은 `EnumMap`의 구조이다 

> ```java
> public class EnumMap<K extends Enum<K>,V>
> extends AbstractMap<K,V>
> implements Serializable, Cloneable
> ```

역시 이름 그대로 key값엔 Enum 타입밖에 들어오지 못한다. `EnumMap`의 내부는 배열로 표현되는데 이 포현은 간결하고 효율적이다. 

`EnumMap`은 key값이 자연정렬된 값을 나타내는데  keySet(), entrySet() and values() 같은 collection-views 의 iterators를 사용할 때 나타난다.

조금 재미있는거는 지금까지 본 Collection 중에 Iterator 호출 후 해당 Collection에 변형이 있으면 거의   ConcurrentModificationException을 호출하는데 `EnumMap`의 collection-views에 반환된 Iterators는 일관성이 약하기때문에 ConcurrentModificationException을 호출하지 않는다 자세한건 아래 예제에서 확인해보도록 하자.

`EnumMap`은 key값으로 null을 허용하지 않는다.  null값을 넣는 순간 NPE를 던지는데 null값을 remove하거나 테스트할때는 동작한다. 이것 역시 아래 예제에서 확인해보자.

`EnumMap`은 동기화 되어있지 않기 때문에 multi-thread 환경에서의 사용은 자제해야한다. 만약 thread-safe한 `EnumMap`이 필요하다면 아래와 같이 사용하면 된다.

> ```java
>      Map<EnumKey, V> m
>          = Collections.synchronizedMap(new EnumMap<EnumKey, V>(...));
> ```



`EnumMap`의 기본적인 모든 동작은 일관된 시간안에 동작하고 `EnumMap`은 HashMap보다 빠르게 동작할 수 있다.



# Example



예제는 최대한 간단하게 짜보도록 하겠다.



1. `EnumMap`의 collection-views 호출 후 내부 데이터 변경
2. HashMap과 `EnumMap`의 성능 비교



1번 부터 보도록 하자

\- MyEnumMap.class

```java
package data.structure12;

import java.util.EnumMap;
import java.util.HashMap;
import java.util.Map.Entry;

public class MyEnumMap {

	public static void main(String[] args) throws Exception {
		
		
		EnumMap<Fruit, String> fruitMap = new EnumMap<>(Fruit.class);
		
		for (Fruit f : Fruit.values()) {
			fruitMap.put(f, f.toString() + " is delicious");
		}
		
		HashMap<String, String> hm = new HashMap<>();

		for (Entry<Fruit, String> e : fruitMap.entrySet()) {
			hm.put(e.getKey().toString(), e.getValue());
		}
		
		for (Entry<Fruit, String> e : fruitMap.entrySet()) {
			
			if(e.getKey().equals(Fruit.MANGO)) {
				//iterator 내부에서 hashMap의 데이터 셋 변경 -> 에러안남
				fruitMap.remove(Fruit.MANGO);
			}
		}
		
		System.out.println(fruitMap.toString());
		
		for (Entry<String, String> e : hm.entrySet()) {
			
			if(e.getKey().equals(Fruit.HUCKLEBERRY.toString())) {
				//iterator 내부에서 hashMap의 데이터 셋 변경 -> 에러발생
				hm.remove(Fruit.HUCKLEBERRY.toString());
			}
		}
		
		System.out.println(hm.toString());
		
	}
	
	enum Fruit {
		APPLE, BLUEBERRY, COCONUT, DURIAN, GRAPE, HUCKLEBERRY, JACKFRUIT, MANGO, ORANGE, PINEAPPLE
	}
	
}

```

위와 같이 iterator 반복시에 collection 내부 변경을 줘봤다. 결과는 아래와 같다.

\- console

```
{APPLE=APPLE is delicious, BLUEBERRY=BLUEBERRY is delicious, COCONUT=COCONUT is delicious, DURIAN=DURIAN is delicious, GRAPE=GRAPE is delicious, HUCKLEBERRY=HUCKLEBERRY is delicious, JACKFRUIT=JACKFRUIT is delicious, ORANGE=ORANGE is delicious, PINEAPPLE=PINEAPPLE is delicious}
Exception in thread "main" java.util.ConcurrentModificationException
	at java.util.HashMap$HashIterator.nextNode(HashMap.java:1445)
	at java.util.HashMap$EntryIterator.next(HashMap.java:1479)
	at java.util.HashMap$EntryIterator.next(HashMap.java:1477)
	at data.structure12.MyEnumMap.main(MyEnumMap.java:36)

```

`EnumMap`의 경우 iterator 내부에서 collection 데이터를 변경해도 ConcurrentModificationException을 던지지 않지만 HashMap은 ConcurrentModificationException을 호출하는것을 확인할 수 있다.



2번은 조금 억지인 감이 없지않아 있지만 ... 그래도 작성했으니 한번 확인해보자 수많은 enum 타입을 모두 선언할 수 없었기 때문에 내가 그나마 아는 springframework의 HttpStatus enum을 함께 사용했다.



\- MyEnumMap2.class

```java
package data.structure12;

import java.util.EnumMap;
import java.util.HashMap;
import java.util.Map.Entry;

import org.springframework.http.HttpStatus;

public class MyEnumMap2 {

	public static void main(String[] args) throws Exception {
		
		long start;
		long end;
		
		EnumMap<HttpStatus, String> httpStatusMap = new EnumMap<>(HttpStatus.class);
		
		start = System.nanoTime();
		
		for (HttpStatus h : HttpStatus.values()) {
			httpStatusMap.put(h, h.getReasonPhrase());
		}
		
		end = System.nanoTime();
		
		System.out.println( "EnumMap에 HttpStatus key:value값 insert 시간 : " + ( end - start )/10000000.0 +"초");
		
		HashMap<String, String> hm = new HashMap<>();

		start = System.nanoTime();
		
		for (HttpStatus e : HttpStatus.values()) {
			hm.put(e.toString(), e.getReasonPhrase());
		}
		
		end = System.nanoTime();
		
		System.out.println( "HashMap에 HttpStatus key:value값 insert 시간 : " + ( end - start )/10000000.0 +"초");
		
	}
}

```



\- console

```
EnumMap에 HttpStatus key:value값 insert 시간 : 0.00234초
HashMap에 HttpStatus key:value값 insert 시간 : 0.0156499초
===========================================================
EnumMap에 HttpStatus key:value값 insert 시간 : 0.00418초
HashMap에 HttpStatus key:value값 insert 시간 : 0.0226801초
===========================================================
EnumMap에 HttpStatus key:value값 insert 시간 : 0.0022399초
HashMap에 HttpStatus key:value값 insert 시간 : 0.02146초

```



당연히 인간의 눈으로 체감하기는 불가능하고 그냥 이건 이렇고 저건 저런거다라는 삽질을 그냥 해본거니 그냥 `EnumMap`이 HashMap보다 빠른편이다 라고 생각하는게 마음 편할듯하다...



***



음 .. 사실 Concurrent*Map,  Hashtable 등등 Map을 구현한 많은 class가 있지만 여기서 다루지 않은 Map을 구현한 class는 thread-safe한 특성이 있음만 알고 일단은 Set으로 넘어가도록 하겠다. 나중에 필요하면 다시 포스팅하는걸로 ...

<br>

포스팅은 여기까지 하겠습니다. 

예제 : https://github.com/sup2is/java-example/tree/master/java-data-structure12

<br>

퍼가실때는 링크와 출처를 반드시 명시해주세요. 감사합니다.

<br>



참고 : https://docs.oracle.com/javase/8/docs/api/