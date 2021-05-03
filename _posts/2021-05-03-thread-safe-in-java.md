---
layout: post
title: "Java에서 Thread-Safe하게 프로그래밍하기"
tags: [Java, Thread-Safe]
date: 2021-05-03
comments: true
---

<br>



# Overview

이번시간에는 자바에서 Thread-safe하게 프로그래밍 하는 방법에 대해서 알아보도록 하겠다.



# Thread-safe란?

스레드는 프로세스의 하위 개념으로써 프로세스에 할당된 자원을 공유한다. 프로그램을 스레드로 분리하면 자연스럽게 병렬성을 이용할 수 있기 때문에 복잡한 애플리케이션의 성능을 향상시킬 수 있다.

제대로 잘 사용한다면 안정성있고 높은 생산성을 가진 프로그램을 만들 수 있다. 하지만 제대로 잘 사용하기 힘들 뿐더러 공유된 자원에 접근하는 스레드의 결과를 예측하기가 매우 어렵다는 단점이 있다.

따라서 멀티스레드를 정확한 방법으로 다루는 방법에 대해서 간략하게 소개하도록 하겠다.

# Lock 사용하기

스레드 안전성을 보장하는 방법으로 가장 첫번째. Lock을 사용하는 방법이다.

자바에서 가장 쉽고 간편하게 스레드 안전성을 보장하는 방법인데 바로 `synchronized` 키워드를 사용하는 방법이다.

```java
/**
 * Sequence
 *
 * @author Brian Goetz and Tim Peierls
 */

@ThreadSafe
public class Sequence {
    @GuardedBy("this") private int nextValue;

    public synchronized int getNext() {
        return nextValue++;
    }
}
```

위 코드는 `Sequence` 인스턴스의 `getNext()` 메서드의 동시성을 보장한다. 즉 `getNext()`에 접근할 수 있는 스레드를 단 한개만 보장한다는 뜻이다. 따라서 수많은 스레드들이 동시에 `getNext()`에 접근한다고 하더라도 정확한 `nextValue` 값을 보장받을 수 있다.

메서드에 `synchronized` 키워드를 붙여서 동기화하는 방법도 있지만 특정 코드 블록에 다음과 같이 `synchronized` 블록을 사용하는 방법도 있다.

```java
/**
 * PrivateLock
 * <p/>
 * Guarding state with a private lock
 *
 * @author Brian Goetz and Tim Peierls
 */
public class PrivateLock {
    private final Object myLock = new Object();
    @GuardedBy("myLock") Widget widget;

    void someMethod() {
        synchronized (myLock) {
            // Access or modify the state of widget
        }
    }
}
```

위 코드는 자바모니터 패턴을 활용해서 `synchronized` 를 사용했다. 자바 모니터 패턴의 추가 예제는 [https://blog.e-zest.com/java-monitor-pattern/](https://blog.e-zest.com/java-monitor-pattern/) 여기에서 확인해볼 수 있다.

# 자료구조 사용하기

불가피하게 `synchronized`를 사용해야한다면 어쩔수 없지만 선택이 가능하다면 자바에서 제공하는 자료구조를 사용하는게 더 좋은 방법이다. `java.concurrent` 패키지 하위에 존재하는 자료구조들인데 대표적으로 `Hashtable`, `ConcurrentHashMap`, `AtomicInteger`, `BlockingQueue` 등등이 있다.

동시성관련된 자료구조를 사용하는 방법은 좋은 방법이지만 어떤 자료구조를 사용하느냐에 따라서 성능이 조금 떨어질 수도 있다.

위에서 소개한 `synchronized`을 사용하는 방법의 가장 큰 단점은 좋은 성능을 보장하기가 어렵다는 것이다. 예를 들어 자바의 `Hashtable`의 주요 메서드들을 확인해보자.

```java
    public synchronized V get(Object key) {
        Entry<?,?> tab[] = table;
        int hash = key.hashCode();
        int index = (hash & 0x7FFFFFFF) % tab.length;
        for (Entry<?,?> e = tab[index] ; e != null ; e = e.next) {
            if ((e.hash == hash) && e.key.equals(key)) {
                return (V)e.value;
            }
        }
        return null;
    }
    
    ...
    
    
    public synchronized V put(K key, V value) {
        // Make sure the value is not null
        if (value == null) {
            throw new NullPointerException();
        }

        // Makes sure the key is not already in the hashtable.
        Entry<?,?> tab[] = table;
        int hash = key.hashCode();
        int index = (hash & 0x7FFFFFFF) % tab.length;
        @SuppressWarnings("unchecked")
        Entry<K,V> entry = (Entry<K,V>)tab[index];
        for(; entry != null ; entry = entry.next) {
            if ((entry.hash == hash) && entry.key.equals(key)) {
                V old = entry.value;
                entry.value = value;
                return old;
            }
        }

        addEntry(hash, key, value, index);
        return null;
    }
    
    ...
    
```

`get()`, `put()` 등등 주요 메서드들이 전부 `synchronized` 키워드에 감싸진 상태로 동시성을 보장하고 있다. 그말인즉슨 `Hashtable`의 동기화된 인스턴스 메서드를 호출할 수 있는 스레드는 반드시 1개라는 뜻이다. 동시에 호출이 불가능하다.

자바는 `Hashtable` 의 성능을 조금 더 증진시키기 위해 1.5 버전에서 `ConcurrentHashMap`을 도입했다. `ConcurrentHashMap`은 조금 더 세밀하게 동시성을 보장하기 때문에 더 좋은 성능을 보장한다.

![thread-safe-map-performance-benchmark1](https://user-images.githubusercontent.com/30790184/116840588-5aa33900-ac11-11eb-9daf-2922a66c57a7.png)

> http://asjava.com/core-java/thread-safe-hash-map-in-java-and-their-performance-benchmark/

다음은 `ConcurrentHashMap`의 주요 메서드이다. `Hashtable`과는 달리 `get()` 또는 `put()` 메서드에 `synchronized` 키워드가 없다.

```java
    public V get(Object key) {
        Node<K,V>[] tab; Node<K,V> e, p; int n, eh; K ek;
        int h = spread(key.hashCode());
        if ((tab = table) != null && (n = tab.length) > 0 &&
            (e = tabAt(tab, (n - 1) & h)) != null) {
            if ((eh = e.hash) == h) {
                if ((ek = e.key) == key || (ek != null && key.equals(ek)))
                    return e.val;
            }
            else if (eh < 0)
                return (p = e.find(h, key)) != null ? p.val : null;
            while ((e = e.next) != null) {
                if (e.hash == h &&
                    ((ek = e.key) == key || (ek != null && key.equals(ek))))
                    return e.val;
            }
        }
        return null;
    }
    
    ...
    
        
    public V put(K key, V value) {
        return putVal(key, value, false);
    }

    /** Implementation for put and putIfAbsent */
    final V putVal(K key, V value, boolean onlyIfAbsent) {
        if (key == null || value == null) throw new NullPointerException();
        int hash = spread(key.hashCode());
        int binCount = 0;
        for (Node<K,V>[] tab = table;;) {
            Node<K,V> f; int n, i, fh;
            if (tab == null || (n = tab.length) == 0)
                tab = initTable();
            else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
                if (casTabAt(tab, i, null,
                             new Node<K,V>(hash, key, value, null)))
                    break;                   // no lock when adding to empty bin
            }
            else if ((fh = f.hash) == MOVED)
                tab = helpTransfer(tab, f);
            else {
                V oldVal = null;
                synchronized (f) {
                    if (tabAt(tab, i) == f) {
                        if (fh >= 0) {
                            binCount = 1;
                            for (Node<K,V> e = f;; ++binCount) {
                                K ek;
                                if (e.hash == hash &&
                                    ((ek = e.key) == key ||
                                     (ek != null && key.equals(ek)))) {
                                    oldVal = e.val;
                                    if (!onlyIfAbsent)
                                        e.val = value;
                                    break;
                                }
                                Node<K,V> pred = e;
                                if ((e = e.next) == null) {
                                    pred.next = new Node<K,V>(hash, key,
                                                              value, null);
                                    break;
                                }
                            }
                        }
                        else if (f instanceof TreeBin) {
                            Node<K,V> p;
                            binCount = 2;
                            if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,
                                                           value)) != null) {
                                oldVal = p.val;
                                if (!onlyIfAbsent)
                                    p.val = value;
                            }
                        }
                    }
                }
                if (binCount != 0) {
                    if (binCount >= TREEIFY_THRESHOLD)
                        treeifyBin(tab, i);
                    if (oldVal != null)
                        return oldVal;
                    break;
                }
            }
        }
        addCount(1L, binCount);
        return null;
    }
        
```

`get()`메서드의 경우 `volatile` 변수를 사용해서 가시성을 보장하도록 되어있고 `put()` 메서드는 부분적으로 `synchronized` 키워드를 사용하는것을 볼 수 있다.

`ConcurrentHashMap`에 대한 정리글은 [https://pplenty.tistory.com/17](https://pplenty.tistory.com/17) 여기에서 확인할 수 있다.

# Stack 한정 프로그래밍

스레드 안전성을 보장하는 세번째 방법은 stack에 한정되도록 프로그래밍하는 방법이다.

스레드는 위에서 설명한것과 같이 프로세스에 할당된 자원을 공유하지만 각 스레드는 각기 별도의 스택과 지역변수를 갖도록 되어 있다. 따라서 모든 스레드는 각자 고유한 스택과 지역변수를 가진다는 특성을 잘 이해하면 동시성을 보장하도록 할 수 있다.

```java

/**
 * Animals
 * <p/>
 * Thread confinement of local primitive and reference variables
 *
 * @author Brian Goetz and Tim Peierls
 */
public class Animals {
    Ark ark;
    Species species;
    Gender gender;

    public int loadTheArk(Collection<Animal> candidates) {
        SortedSet<Animal> animals;
        int numPairs = 0;
        Animal candidate = null;

        // animals confined to method, don't let them escape!
        animals = new TreeSet<Animal>(new SpeciesGenderComparator());
        animals.addAll(candidates);
        for (Animal a : animals) {
            if (candidate == null || !candidate.isPotentialMate(a))
                candidate = a;
            else {
                ark.load(new AnimalPair(candidate, a));
                ++numPairs;
                candidate = null;
            }
        }
        return numPairs;
    }
}
```

위 예제에서 주의깊게 볼 점은 `animals` 라는 `SortedSet` 인스턴스는 지역변수로 선언되었다는 점이다. 넘어오는 `candidates` 파라미터를 복사(addAll) 한 뒤에 추가적인 작업을 실행하는데 지역변수, 즉 stack 내부에서만 사용했기 때문에 동시성을 보장할 수 있다.

위에서 설명한것처럼 각 스레드는 각기 별도의 스택과 지역변수를 갖도록 되어 있기 때문에 이런 방법으로도 동시성을 보장하도록 할 수 있다.

# ThreadLocal 사용하기

위에서 지역변수를 사용해서 동시성을 보장하는 방법은 간결하고 이해하기 쉽지만 저 메서드 스택을 벗어나는 순간 `animals`라는 변수의 참조가 없어지기 때문에 다른곳에서 `animals`를 사용할 수 없다는 단점이 있다.

위와 같은 단점을 해결하기 위해서 자바는 `ThreadLocal`이라는 클래스를 제공하고 있다.

`ThreadLocal`을 이용하면 쓰레드 영역에 변수를 설정할 수 있기 때문에, 특정 쓰레드가 실행하는 모든 코드에서 그 쓰레드에 설정된 변수 값을 사용할 수 있게 된다.   

대표적인 사용 예로 `SpringSecurity`에서 제공하는 `SecurityContextHolder`가 바로 `ThreadLocal` 적절한 예가 된다.

`ThreadLocal`에 대한 조금 더 자세한 사용법과 설명은 [https://javacan.tistory.com/entry/ThreadLocalUsage](https://javacan.tistory.com/entry/ThreadLocalUsage) 여기에서 확인할 수 있다.



# 불변객체 사용하기 feat. final keyword

스레드 안전성을 보장하는 마지막 방법으로 불변객체를 사용하는 방법이다.

불변객체란 객체가 선언된 이후로는 변경할 수 없는 객체임을 뜻하는데 자바에서는 대표적으로 `String`이 있다. 그렇다면 `String`은 기본적으로 스레드에 안전할까? 그렇다. `String`은 스레드에 안전하다.

아래는 불변객체를 생성하는 예제이다.

```java
import java.util.Date;
 
/**
* Always remember that your instance variables will be either mutable or immutable.
* Identify them and return new objects with copied content for all mutable objects.
* Immutable variables can be returned safely without extra effort.
* */
public final class ImmutableClass
{
 
    /**
    * Integer class is immutable as it does not provide any setter to change its content
    * */
    private final Integer immutableField1;
 
    /**
    * String class is immutable as it also does not provide setter to change its content
    * */
    private final String immutableField2;
 
    /**
    * Date class is mutable as it provide setters to change various date/time parts
    * */
    private final Date mutableField;
 
    //Default private constructor will ensure no unplanned construction of class
    private ImmutableClass(Integer fld1, String fld2, Date date)
    {
        this.immutableField1 = fld1;
        this.immutableField2 = fld2;
        this.mutableField = new Date(date.getTime());
    }
 
    //Factory method to store object creation logic in single place
    public static ImmutableClass createNewInstance(Integer fld1, String fld2, Date date)
    {
        return new ImmutableClass(fld1, fld2, date);
    }
 
    //Provide no setter methods
 
    /**
    * Integer class is immutable so we can return the instance variable as it is
    * */
    public Integer getImmutableField1() {
        return immutableField1;
    }
 
    /**
    * String class is also immutable so we can return the instance variable as it is
    * */
    public String getImmutableField2() {
        return immutableField2;
    }
 
    /**
    * Date class is mutable so we need a little care here.
    * We should not return the reference of original instance variable.
    * Instead a new Date object, with content copied to it, should be returned.
    * */
    public Date getMutableField() {
        return new Date(mutableField.getTime());
    }
 
    @Override
    public String toString() {
        return immutableField1 +" - "+ immutableField2 +" - "+ mutableField;
    }
}
```

> https://howtodoinjava.com/java/basics/how-to-make-a-java-class-immutable/

기본적으로 `setter` 메서드가 없고 생성자가 `private`으로 선언되어있으며 실제 인스턴스는 팩토리 메서드를 통해서 생성하도록 외부에 공개되었다. 그리고 모든 멤버변수들은 `final`로 선언되어있는데 `Integer`나 `String`은 기본적으로 불변이기 때문에 반환할때도 그대로 리턴해도 되지만 `Date`의 경우 가변객체이기때문에 반드시 방어적복사본을 생성하는 방법으로 프로그래밍 해야 동시성을 보장받을 수 있다.

적절한 `final` 키워드도 별다른 동기화작업이 없이도 동시성환경에서 자유롭게 사용할 수 있다. 불변객체와 비슷한관점으로 초기화된 이후에 변경될 수 없기 때문에 여러스레드가 동시에 접근해도 동일한 값을 보장받을 수 있기 때문이다.

# 마무리

이 글은 대부분 [자바병렬프로그래밍](http://www.yes24.com/Product/Goods/3015162)을 참조했다. 예전부터 읽어봐야지 읽어봐야지 하고 미뤄뒀던 책이었는데 만약 나와 같은 생각을 하고 있다면 읽어보는것을 강력 추천한다. 비록 java1.5, 1.6이 나오는 시점에 나왔던 책이지만 기본 원리에 대해서 잘 설명하고 예제도 잘 제공하기 때문에 이해하는데 크게 어렵지 않다.



<br>

***

포스팅은 여기까지 하겠습니다. 퍼가실때는 출처를 반드시 남겨주세요!

<br>

**References**

- 자바병렬프로그래밍 -에이콘-
- [https://www.javamex.com/tutorials/concurrenthashmap_scalability.shtml](https://www.javamex.com/tutorials/concurrenthashmap_scalability.shtml)
- [https://blog.e-zest.com/java-monitor-pattern/](https://blog.e-zest.com/java-monitor-pattern/)
- [https://howtodoinjava.com/java/basics/how-to-make-a-java-class-immutable/](https://howtodoinjava.com/java/basics/how-to-make-a-java-class-immutable/)
- [https://pplenty.tistory.com/17](https://pplenty.tistory.com/17)
- [https://javacan.tistory.com/entry/ThreadLocalUsage](https://javacan.tistory.com/entry/ThreadLocalUsage)
