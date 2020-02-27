---

layout: post
title: "BlockingQueue, ArrayBlockingQueue"
tags: [JAVA, DataStructure, Queue, BlockingQueue, ArrayBlockingQueue]
comments: true
date: 2019-09-10
nav_order: 6
parent: DataStructure
grand_parent: Java
---



쉴새없이 달리는것같아서 기분이 좋다. 근데 Queue를 구현하는 class가 너무 많다;; 천천히 해야지 ... 이번시간에는 BlockingQueue interface에 대해 알아보자 

***



# BlockingQueue<E>

`BlockingQueue` interface의 내부 구조는 다음과 같다.

> ```java
> public interface BlockingQueue<E>
> extends Queue<E>
> ```

Queue interface가 super이기때문에 `BlockingQueue` interface를 구현하는 클래스는 모두 Queue를 구현해야한다. [ArrayBlockingQueue](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/ArrayBlockingQueue.html), [DelayQueue](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/DelayQueue.html), [LinkedBlockingDeque](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/LinkedBlockingDeque.html), [LinkedBlockingQueue](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/LinkedBlockingQueue.html), [LinkedTransferQueue](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/LinkedTransferQueue.html), [PriorityBlockingQueue](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/PriorityBlockingQueue.html), [SynchronousQueue](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/SynchronousQueue.html)  모두 `BlockingQueue`를 구현하고있다. 이중에는 `BlockingQueue`의 sub interface인 BlockingDeque나 TransferQueue를 구현하는 것도 있는데 나중에 자세히 살펴보도록 하겠다.

<br>

저번시간에 blocking과 non-blocking에 대해서 아주 간단하게 언급했는데 지금 이 `BlockingQueue` 에서 Blocking이란 단어 역시 같은 맥락이다. `BlockingQueue` 는 Queue 이외의 다음과 같은 동작을 지원한다

1. 요소를 검색할때 이 `BlockingQueue`를 구현한 자료구조가 비어있지 않을때까지 기다리고 
2. 요소를 저장할때 이 `BlockingQueue`를 구현한 자료구조가 비어있을때까지 기다리는것이다. 

만약 이 내용이 헷갈린다면 [다음](https://homoefficio.github.io/2017/02/19/Blocking-NonBlocking-Synchronous-Asynchronous/)의 링크를 다시한번 확인하자

<br>

이 `BlockingQueue`는 지금까지 봤던 Queue 자료구조와는 조금 다른 방식으로 구현이 되어있다.

|             | *Throws exception*                                           | *Special value*                                              | *Blocks*                                                     | *Times out*                                                  |
| ----------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **Insert**  | [`add(e)`](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/BlockingQueue.html#add-E-) | [`offer(e)`](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/BlockingQueue.html#offer-E-) | [`put(e)`](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/BlockingQueue.html#put-E-) | [`offer(e, time, unit)`](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/BlockingQueue.html#offer-E-long-java.util.concurrent.TimeUnit-) |
| **Remove**  | [`remove()`](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/BlockingQueue.html#remove-java.lang.Object-) | [`poll()`](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/BlockingQueue.html#poll-long-java.util.concurrent.TimeUnit-) | [`take()`](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/BlockingQueue.html#take--) | [`poll(time, unit)`](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/BlockingQueue.html#poll-long-java.util.concurrent.TimeUnit-) |
| **Examine** | [`element()`](https://docs.oracle.com/javase/8/docs/api/java/util/Queue.html#element--) | [`peek()`](https://docs.oracle.com/javase/8/docs/api/java/util/Queue.html#peek--) | *not applicable*                                             | *not applicable*                                             |

가장 맨위 열에 해당하는 내용이 바로 그 부분인데 add()와 offer()는 별 차이 없이 add()의 경우 요소를 넣지 못하면  예외를 발생시키고 offer()는 boolean 값을 리턴한다. 이부분은 AbstractQueue에 있기 때문에 `BlockingQueue` 만의 특성은 아닌것 같다.

조금 살펴봐야 하는 부분이 이제 put(), take() 와 time out 관련 메서드이다.

put(), take() 같은 경우는 현재 접근한 thread의 동작이 완료될때까지 무제한 대기시키는 것이고

offer(e, timeout, unit), poll(timeout, unit) 은 주어신 시간안에서만 동작을 대기시키시는 것이다.

> ```java
>     /**
>      * Inserts the specified element into this queue, waiting if necessary
>      * for space to become available.
>      *
>      * @param e the element to add
>      * @throws InterruptedException if interrupted while waiting
>      * @throws ClassCastException if the class of the specified element
>      *         prevents it from being added to this queue
>      * @throws NullPointerException if the specified element is null
>      * @throws IllegalArgumentException if some property of the specified
>      *         element prevents it from being added to this queue
>      */
>     void put(E e) throws InterruptedException;
> 
> ...
>     /**
>      * Retrieves and removes the head of this queue, waiting if necessary
>      * until an element becomes available.
>      *
>      * @return the head of this queue
>      * @throws InterruptedException if interrupted while waiting
>      */
>     E take() throws InterruptedException;
> ...
>     /**
>      * Retrieves and removes the head of this queue, waiting up to the
>      * specified wait time if necessary for an element to become available.
>      *
>      * @param timeout how long to wait before giving up, in units of
>      *        {@code unit}
>      * @param unit a {@code TimeUnit} determining how to interpret the
>      *        {@code timeout} parameter
>      * @return the head of this queue, or {@code null} if the
>      *         specified waiting time elapses before an element is available
>      * @throws InterruptedException if interrupted while waiting
>      */
>     E poll(long timeout, TimeUnit unit)
>         throws InterruptedException;
> ...
>     /**
>      * Inserts the specified element into this queue, waiting up to the
>      * specified wait time if necessary for space to become available.
>      *
>      * @param e the element to add
>      * @param timeout how long to wait before giving up, in units of
>      *        {@code unit}
>      * @param unit a {@code TimeUnit} determining how to interpret the
>      *        {@code timeout} parameter
>      * @return {@code true} if successful, or {@code false} if
>      *         the specified waiting time elapses before space is available
>      * @throws InterruptedException if interrupted while waiting
>      * @throws ClassCastException if the class of the specified element
>      *         prevents it from being added to this queue
>      * @throws NullPointerException if the specified element is null
>      * @throws IllegalArgumentException if some property of the specified
>      *         element prevents it from being added to this queue
>      */
>     boolean offer(E e, long timeout, TimeUnit unit)
>         throws InterruptedException;
> ...
> 
> ```

<br>

`BlockingQueue` 역시 null 요소를 허용하지 않는데 이 이유는 Queue에 요소가 더이상 없을때 poll() 메서드는 null을 반환하기 때문이다.

<br>

`BlockingQueue`는 capacity로 구현될 수 있는데 뒤에 알아볼 `ArrayBlockingQueue`가 그렇게 설계 되었다. 그나마 기본생성자 역할을 하는게 다음 생성자인데

> ```java
>     /**
>      * Creates an {@code ArrayBlockingQueue} with the given (fixed)
>      * capacity and default access policy.
>      *
>      * @param capacity the capacity of this queue
>      * @throws IllegalArgumentException if {@code capacity < 1}
>      */
>     public ArrayBlockingQueue(int capacity) {
>         this(capacity, false);
>     }
> ```

`ArrayBlockingQueue`는 초기에 size를 지정해야만 인스턴스를 생성할수있고 애초에 지정한 배열사이즈를 넘기는순간 AbstractQueue에서 throw new IllegalStateException("Queue full") 를 호출할 것이다. 따라서 `ArrayBlockingQueue`는  인스턴스 생성때 배열의 크기를 지정해줘야한다. 

<br>

`BlockingQueue`를 구현하는 class는 주로 생산자-소비자 역할을 하는 Queue로 설계되었고 remove() 같은 경우는 이미 Collection에 있어서 구현한 메서드이지만 성능이 좋지 않기때문에 사용을 지양해야한다.

<br>

`BlockingQueue`는 thread-safe하도록 설계되었고 어떠한 작업마다 내부에서 lock()과 unlock()을 동시에 실행하기때문에 동시성을 보장한다. 그러나 대량작업의 addAll(), removeAll()같은 경우는 사용할때 매우 주의해야하는데 따로 명시되어있지 않는 한 addAll()같은 경우의 실행중 예외를 던질 수 있다. 10개의 요소가 담긴 자료구조중에서 7번째 요소를 넣는 도중에 에러가 발생할 수 있다는 것이다.

<br>

# ArrayBlockingQueue<E>

`ArrayBlockingQueue`는 `BlockingQueue`을 구현한 class인데 구조는 다음과 같다.

> ```java
> public class ArrayBlockingQueue<E>
> extends AbstractQueue<E>
> implements BlockingQueue<E>, Serializable
> ```

<br>

기본적으로 `BlockingQueue`를 구현하기때문에 위에서 언급한 put(), take() 등을 구현해야한다. `ArrayBlockingQueue`도 역시 배열기반의 `BlockingQueue`인데 `ArrayBlockingQueue`는 FIFO로 구현되어있다. FIFO이기때문에 head는 가장 오래된 요소, tail은 가장 최근에 들어온 요소가 된다.

<br>

위에서 언급한대로 `ArrayBlockingQueue`는 초기에 capacity값에 따라 배열이 생성되고 이후에 수정이 불가능하다. 만약 `ArrayBlockingQueue`의 array가 꽉 찬 상태에서 추가적으로 요소를 더 넣거나 `ArrayBlockingQueue` 빈 상태에서 요소를 꺼내올때 `ArrayBlockingQueue`는 block 상태가 된다. 기본적으로 생산자-소비자 관계를 어느정도 보장하는 것이다. 하지만 이 관계의 block 현상을 완전하게 보장하지는 않는다.



# Example

javadoc에서 생산자-소비자에 대한 예제가 있는데 대충 아래와 같다

> ```java
>  class Producer implements Runnable {
>    private final BlockingQueue queue;
>    Producer(BlockingQueue q) { queue = q; }
>    public void run() {
>      try {
>        while (true) { queue.put(produce()); }
>      } catch (InterruptedException ex) { ... handle ...}
>    }
>    Object produce() { ... }
>  }
> 
>  class Consumer implements Runnable {
>    private final BlockingQueue queue;
>    Consumer(BlockingQueue q) { queue = q; }
>    public void run() {
>      try {
>        while (true) { consume(queue.take()); }
>      } catch (InterruptedException ex) { ... handle ...}
>    }
>    void consume(Object x) { ... }
>  }
> 
>  class Setup {
>    void main() {
>      BlockingQueue q = new SomeQueueImplementation();
>      Producer p = new Producer(q);
>      Consumer c1 = new Consumer(q);
>      Consumer c2 = new Consumer(q);
>      new Thread(p).start();
>      new Thread(c1).start();
>      new Thread(c2).start();
>    }
>  }
> ```

<br>

예제에서  `ArrayBlockingQueue`을 사용해서 직접 구현해보는 시간을 갖겠다.



<br>

\-Producer.class

```java
package data.structure6;

import java.util.concurrent.BlockingQueue;

public class Producer implements Runnable{

	private final BlockingQueue<String> queue;

	final String[] GOODS = new String[] {"아이폰", "갤럭시", "에어팟", "무풍에어컨", "스타일러"};
	
	
	public Producer (BlockingQueue<String> bq) {
		this.queue = bq;
	}

	@Override
	public void run() {
		while(true) {
			try {
				long time = ((long) (Math.random()* 10000)) / 2;
				System.out.println("생산자는 앞으로 약 " + time+ "(millisecond)초 동안 block됩니다. 고객들은 기다려주세요");
				Thread.sleep(time);
				queue.put(produce());
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}
	}
	
	private String produce() {
		return GOODS[((int)(Math.random()* 10) % 5)];
	}
	
}

```



Producer class의 역할은 생산자다. 이 생산자는 약 1~5초까지의 물품 생산시간을 갖고 아이폰, 갤럭시 등등의 물품을 찍어내는 역할을 한다.  생산전에 일정시간동안 block되는걸 consumer들에게 알려주고 생산되는 즉시 물품을 queue에 올린다.



<br>

\- Consumer.class

```java
package data.structure6;

import java.util.concurrent.BlockingQueue;

public class Consumer implements Runnable{

	private final BlockingQueue<String> queue;
	private final String name;
	
	public Consumer (BlockingQueue<String> bq, String name) {
		this.queue = bq;
		this.name = name;
	}

	@Override
	public void run() {

		while(true) {
			try {
				consume(queue.take());
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}
	}
	
	private void consume(String string) {
		System.out.println(name + "의 이름을 가진 thread가 " + string + "을 구매했습니다.");
	}
	
}

```

이 Consumer class의 역할은 소비자역할을 한다. 이 소비자는 물건을 닥치는대로 마구마구 사기 때문에 queue에 올라온 어느 물건이던 자기 차례가 되면 바로 사버린다. 구매 직후에는 자신이 구매한 물품의 이름을 알려준다.



\- Main.class

```java
package data.structure6;

import java.util.concurrent.ArrayBlockingQueue;
import java.util.concurrent.BlockingQueue;

public class Main{

	public static void main(String[] args) {
		
		BlockingQueue<String> abq = new ArrayBlockingQueue<>(100);
		Producer p = new Producer(abq);
		Consumer c1 = new Consumer(abq, "고객1");
		Consumer c2 = new Consumer(abq, "고객2");
		Consumer c3 = new Consumer(abq, "고객3");
		Consumer c4 = new Consumer(abq, "고객4");
		Consumer c5 = new Consumer(abq, "고객5");
		Consumer c6 = new Consumer(abq, "고객6");
		
		new Thread(p).start();
		new Thread(c1).start();
		new Thread(c2).start();
		new Thread(c3).start();
		new Thread(c4).start();
		new Thread(c5).start();
		new Thread(c6).start();
		
	}
	
}



```

준비가 끝이 났으니 이제 한번 실행시켜보자.

생산자와 소비자가 같은 ArrayBlockingQueue 의 인스턴스를 공유하도록 설정해주고 소비자는 각각 고객1~6까지의 이름을 지어주어서 생성해준다. 실행 결과먼저 확인해보자.

<br>

![녹화_2019_09_16_15_06_44_127](https://user-images.githubusercontent.com/30790184/64937088-d5c8a780-d893-11e9-99cc-fcf292cdd597.gif)



위에 gif에서 확인할 수 있는 것처럼 ArrayBlockingQueue 에 더이상 꺼낼 요소가 없으면 접근한 thread가 block되어 무제한 대기상태가된다. 그리고 위 그림에서는 고객이 1~6까지 차례대로 ArrayBlockingQueue 에 접근하는것같지만 thread의 점유순서를 백퍼센트 보장하지 않으니 주의해야한다. 예제처럼 1~6 으로 차례대로 start시켰지만 1,3,4,5,2,6 순서로 될 수도 있다.

<br>

ArrayBlockingQueue 의 내부를 잠깐 살펴보면

```java
    /**
     * Inserts the specified element at the tail of this queue, waiting
     * for space to become available if the queue is full.
     *
     * @throws InterruptedException {@inheritDoc}
     * @throws NullPointerException {@inheritDoc}
     */
    public void put(E e) throws InterruptedException {
        checkNotNull(e);
        final ReentrantLock lock = this.lock;
        lock.lockInterruptibly();
        try {
            while (count == items.length)
                notFull.await();
            enqueue(e);
        } finally {
            lock.unlock();
        }
    }

..
    
    public E take() throws InterruptedException {
        final ReentrantLock lock = this.lock;
        lock.lockInterruptibly();
        try {
            while (count == 0)
                notEmpty.await();
            return dequeue();
        } finally {
            lock.unlock();
        }
    }
```


put()의 enqueue()를 수행하기 전에 현재 queue의 상태를 보고 꽉 차있을 경우 notFull.await()를 무한정 대기시킨다. 마찬가지로 take() 역시 현재 queue의 상태를 확인하여 queue가 비어있을경우  notEmpty.await()로 무한정 대기 시킨다.

결론적으로 이 BlockingQueue의 가장 큰 특징은 take()와 put()이 될 것 같다. 내부를 좀 더 깊숙히 파보고 싶긴한데 나중에 기회가 되면 다시 보도록 하겠다.



***



이 BlockingQueue를 구현한 class는 ArrayBlockingQueue 이외에도 DelayQueue, LinkedBlockingQueue, PriorityBlockingQueue가 있는데 다음시간에 한번 까보도록 하자!



<br>

포스팅은 여기까지 하겠습니다. 

예제 : https://github.com/sup2is/java-example/tree/master/java-data-structure6

<br>

퍼가실때는 링크와 출처를 반드시 명시해주세요. 감사합니다.

<br>



참고 : https://docs.oracle.com/javase/8/docs/api/