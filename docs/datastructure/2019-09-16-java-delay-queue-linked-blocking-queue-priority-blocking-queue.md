---

layout: post
title: "Java의 DelayQueue, LinkedBlockingQueue, PriorityBlockingQueue"
tags: [JAVA, DataStructure, Queue, BlockingQueue, DelayQueue, LinkedBlockingQueue, PriorityBlockingQueue]
comments: true
nav_order: 7
parent: DataStructure
grand_parent: Java
---





이편은 전편과 이어져있으니 관심이 있다면 [이곳](https://sup2is.github.io/java-data-structure-6/)을 확인해도 괜찮을 듯 하다.  이번시간에는 DelayQueue, LinkedBlockingQueue, PriorityBlockingQueue에 대해서 간략하게 알아보도록 하자.

***

# DelayQueue<E>

`DelayQueue`는 다음 구조를 갖도록 설계 되었다.

> ```java
> public class DelayQueue<E extends Delayed> extends AbstractQueue<E>
>     implements BlockingQueue<E> {
> ```

가장 눈에 띄는 부분은 아마 제네릭 타입이 될 것 같은데 Delayed 타입의 sub 객체만 `DelayQueue`의 인스턴스를 생성할 수 있다. 저번시간에 알아본 ArrayBlockingQueue와는 다르게 capacity를 지정하지 않아도 인스턴스를 생성할 수 있다. 내부 배열 크기를 알아서 늘려도록 unbounded하게 설계 되었다. `DelayQueue`는 null요소를 허용하지 않는다.

<br>

Delayed 라는 interface를 한번 확인해 보자

> ```java
> public interface Delayed
> extends Comparable<Delayed>
> ```

`DelayQueue`를 구현하기 위해서는 Delayed 라는 interface를 구현한 class를 생성해야한다. 이 Delayed라는 interface의 역할은 말 그대로 주어진 시간 이후에 동작해야하는 객체를 나타내기 위해 설계 되었다. 이 Delayed를 구현하기 위해서는 Comparable의 compareTo()와 Delayed의 getDelay() 메서드를 구현하면 된다.

<br>

`DelayQueue`는 Delayed 타입의 객체를 통해서 지연된 처리를 할 수 있도록 해주는 interface이다. 아래에도 언급하지만 take()라는 메서드는 Delayed의 getDelay() 메서드를 호출하는데 리턴된 값이 0이 아니면 리턴된 시간만큼 대기를 반복한다. `DelayQueue`는 시간 지연처리 관련에 적합한 자료구조이다.



# Example

사실 이 예제가 얼마나 좋은 예제인지는 모르겠고 이 `DelayQueue`의 사용이 적절한곳이 어딘지 몰라서 조금 헤맸다 그냥 이런거구나.. 참고만 하자

<br>

\- MyDelayQueue.class

```java
package data.structure7;

import java.util.concurrent.DelayQueue;
import java.util.concurrent.Delayed;
import java.util.concurrent.TimeUnit;

public class MyDelayQueue {

	public static void main(String[] args) throws InterruptedException {
		
		DelayQueue<Delayed> dq = new DelayQueue<>();
		
		MyDelayed md1 = new MyDelayed("MyDelayed-1", 5000000000L); // 5초동안 큐에 대기
		MyDelayed md2 = new MyDelayed("MyDelayed-2", 2000000000L); // 2초동안 큐에 대기
		MyDelayed md3 = new MyDelayed("MyDelayed-3", 10000000000L); // 10초동안 큐에 대기
		MyDelayed md4 = new MyDelayed("MyDelayed-4", 8000000000L); // 8초동안 큐에 대기
		MyDelayed md5 = new MyDelayed("MyDelayed-5", 6000000000L); // 6초동안 큐에 대기
		
		dq.add(md1);
		dq.add(md2);
		dq.add(md3);
		dq.add(md4);
		dq.add(md5);
		
		while (!dq.isEmpty()) {
			MyDelayed md = (MyDelayed) dq.take();
			System.out.println("### : " + md.name + "반환!!");
		}
	}
}

class MyDelayed implements Delayed {
	
	String name;
	final long DELAY_TIME = 1000000000L; //모든 Delayed는 1초씩 증가
	final long EXPIRE_TIME; 
	long accumulateTime = 0; //누적시간
	
	public MyDelayed(String name ,long expire_time) {
		this.name = name;
		this.EXPIRE_TIME = expire_time;
	}

	@Override
	public long getDelay(TimeUnit unit) {
		this.accumulateTime += DELAY_TIME;
		System.out.println(name + "가 반환되기까지 남은 시간" + (EXPIRE_TIME - accumulateTime)/1000000000L + "초");
		return unit.toNanos(EXPIRE_TIME - accumulateTime > 0 ? DELAY_TIME : 0);
	}
	
	@Override
	public int compareTo(Delayed o) {
		return Long.compare(this.EXPIRE_TIME , ((MyDelayed)o).EXPIRE_TIME);
	}

}
```



Delayed를 구현한 MyDelayed class는 name과 expire_time 이라는 파라미터를 받는다. expire_time은 나노초로 나타내는데 나노초는 십억분의1초이다. 아주 깊고 자세하게 보지는 않았지만 DelayQueue의 take() 메서드에서 나노초 단위로 시간을 계산하기때문에 나도 나노초 단위로 프로그램을 작성했다.

<br>

> ```java
>     /**
>      * Retrieves and removes the head of this queue, waiting if necessary
>      * until an element with an expired delay is available on this queue.
>      *
>      * @return the head of this queue
>      * @throws InterruptedException {@inheritDoc}
>      */
>     public E take() throws InterruptedException {
>         final ReentrantLock lock = this.lock;
>         lock.lockInterruptibly();
>         try {
>             for (;;) {
>                 E first = q.peek();
>                 if (first == null)
>                     available.await();
>                 else {
>                     long delay = first.getDelay(NANOSECONDS);
>                     if (delay <= 0)
>                         return q.poll();
>                     first = null; // don't retain ref while waiting
>                     if (leader != null)
>                         available.await();
>                     else {
>                         Thread thisThread = Thread.currentThread();
>                         leader = thisThread;
>                         try {
>                             available.awaitNanos(delay);
>                         } finally {
>                             if (leader == thisThread)
>                                 leader = null;
>                         }
>                     }
>                 }
>             }
>         } finally {
>             if (leader == null && q.peek() != null)
>                 available.signal();
>             lock.unlock();
>         }
>     }
> ```

<br>

주저리주저리 설명보다는 그냥 어떻게 실행되는지 확인하는게 더좋을듯 하다.

![녹화_2019_09_17_09_44_14_539](https://user-images.githubusercontent.com/30790184/65002831-8ab0a200-d930-11e9-95f2-1f7629a830c0.gif)





<br>



# LinkedBlockingQueue<E>

`LinkedBlockingQueue`는 다음과 같은 구조를 갖고 있다.

> ```java
> public class LinkedBlockingQueue<E>
> extends AbstractQueue<E>
> implements BlockingQueue<E>, Serializable
> ```

`LinkedBlockingQueue`는 capacity를 설정해서 인스턴스화를 할 수도 있고 그냥 생성할 수도 있다. FIFO기반의 queue로 구현되었고 FIFO의 특징그대로 head에 있는 요소가 대기열에 가장 오래된 요소, tail에 있는 요소가 대기열의 가장 최근요소이다.

<br>

`LinkedBlockingQueue` 역시 연결리스트로 작성이 되었기때문에 내부에 Node라는 inner class가 있다. `LinkedBlockingQueue`의 특징은 일반 array기반 List보다 동작방식은 빠르지만 multi-thread환경에서의 동작은 성능을 예측하기 어렵다...?

> ```
> Linked queues typically have higher throughput than array-based queues but less predictable performance in most concurrent applications.
> ```

<br>



# Example

그냥 간단하게 ArrayBlockingQueue와 `LinkedBlockingQueue`에서 int 요소 5백만개를 넣을때 성능 차이만 알아보도록 하겠다.



\- MyLinkedBlockingQueue.class

```java
package data.structure7;

import java.util.concurrent.ArrayBlockingQueue;
import java.util.concurrent.LinkedBlockingQueue;

public class MyLinkedBlockingQueue {

	
	public static void main(String[] args) {
		LinkedBlockingQueue<Integer> lbq = new LinkedBlockingQueue<>();
		long start;
		long end;
	
		start = System.currentTimeMillis();
		for (int i = 0; i < 5000000; i++) {
			lbq.add(i);
		}
		end = System.currentTimeMillis();
		
		System.out.println( "LinkedBlockingQueue의 요소 오백만개 넣는 시간 : " + ( end - start )/1000.0 +"초");		

		ArrayBlockingQueue<Integer> abq = new ArrayBlockingQueue<>(5000000);
		
		start = System.currentTimeMillis();
		for (int i = 0; i < 5000000; i++) {
			abq.add(i);
		}
		end = System.currentTimeMillis();
		
		System.out.println( "ArrayBlockingQueue의 요소 오백만개 넣는 시간 : " + ( end - start )/1000.0 +"초");
		
	}
	

}

```



<br>

\- console

```
LinkedBlockingQueue의 요소 오백만개 넣는 시간 : 0.896초
ArrayBlockingQueue의 요소 오백만개 넣는 시간 : 4.069초
===================================================
LinkedBlockingQueue의 요소 오백만개 넣는 시간 : 0.941초
ArrayBlockingQueue의 요소 오백만개 넣는 시간 : 4.051초
```

성능차이가 꽤 큰 편이다.



# PriorityBlockingQueue<E>

마지막으로 `PriorityBlockingQueue`를 보도록하자.

`PriorityBlockingQueue`의 구조는 다음과 같다.

> ```java
> public class PriorityBlockingQueue<E>
> extends AbstractQueue<E>
> implements BlockingQueue<E>, Serializable
> ```

이 `PriorityBlockingQueue`는 PriorityQueue와 아주 흡사하고 추가로 BlockingQueue의 기능까지 사용할 수 있도록 구현되었다. `PriorityBlockingQueue`는 unbounded하고 null 요소를 허용하지 않는다.  PriorityQueue와 마찬가지로 정렬불가능한 요소의 추가를 허용하지 않는다.

<br>

iterator사용도 주의해야하는데 이점도 PriorityQueue와 똑같다. iterator사용시 정렬된 요소기반으로 동작하지 않기때문에 매우주의해야한다.

<br>

정렬이 필요한 두 요소가 `PriorityBlockingQueue`의 입장에서 동일하다면 어떤것이 먼저 take()될지의 순서는 보장하지 않는다. 이 경우에 순서를 지정해야하는 경우는 직접 정의하여 사용할 수 있다. 아래는 javadoc에서 제공하는 FIFOEntry class 의 예제다 이 FIFOEntry는 더이상 비교할 수 없는 동일한 객체를 seq로 다시 정렬해주어 같은 요소일경우 먼저 들어온 객체가 먼저 나간다.

> ```java
>  class FIFOEntry<E extends Comparable<? super E>>
>      implements Comparable<FIFOEntry<E>> {
>    static final AtomicLong seq = new AtomicLong(0);
>    final long seqNum;
>    final E entry;
>    public FIFOEntry(E entry) {
>      seqNum = seq.getAndIncrement();
>      this.entry = entry;
>    }
>    public E getEntry() { return entry; }
>    public int compareTo(FIFOEntry<E> other) {
>      int res = entry.compareTo(other.entry);
>      if (res == 0 && other.entry != this.entry)
>        res = (seqNum < other.seqNum ? -1 : 1);
>      return res;
>    }
>  }
> ```



<br>

# Example

위 javadoc의 FIFOEntry class를 활용하여 나이가 같은 요소일 경우 먼저 들어온 요소가 먼저 나가도록 Priority & FIFO 기능을 구현하는 `PriorityBlockingQueue`를 생성해보도록 하겠다



\- MyPriorityBlockingQueue.class



```java
package data.structure7;

import java.util.concurrent.PriorityBlockingQueue;
import java.util.concurrent.atomic.AtomicLong;

public class MyPriorityBlockingQueue {

	public static void main(String[] args) throws InterruptedException {
		
		PriorityBlockingQueue<FIFOEntry> enforceOrderingPbq = new PriorityBlockingQueue<>();
		
		for (int i = 0; i < 10; i++) {
			User user = new User("User" + i , 20);
			FIFOEntry<User> fifoEntry = new FIFOEntry(user);
			enforceOrderingPbq.add(fifoEntry);
		}
		
		enforceOrderingPbq.add(new FIFOEntry(new User("User99", 15)));
		
		while (!enforceOrderingPbq.isEmpty()) {
			FIFOEntry<User> fifoEntry =  enforceOrderingPbq.take();
			System.out.println(fifoEntry.seqNum + "의 시퀀스 넘버를 가진 " + fifoEntry.getEntry().name );
		}
		
		System.out.println("=======================================================");
		
		PriorityBlockingQueue<User> defaultPbq = new PriorityBlockingQueue<>();
		for (int i = 0; i < 10; i++) {
			User user = new User("User" + i , 20);
			defaultPbq.add(user);
		}
		
		defaultPbq.add(new User("User99", 15));
		
		while (!defaultPbq.isEmpty()) {
			User user = defaultPbq.take();
			System.out.println(user.name);
		}
		
	}
}

class User implements Comparable<User>{
	
	String name;
	int age;
	
	public User(String name, int age) {
		super();
		this.name = name;
		this.age = age;
	}

	@Override
	public int compareTo(User o) {
		return Integer.compare(this.age, o.age);
	}
	
}

class FIFOEntry<E extends Comparable<? super E>> implements Comparable<FIFOEntry<E>> {
	static final AtomicLong seq = new AtomicLong(0);
	final long seqNum;
	final E entry;

	public FIFOEntry(E entry) {
		seqNum = seq.getAndIncrement();
		this.entry = entry;
	}

	public E getEntry() {
		return entry;
	}

	@Override
	public int compareTo(FIFOEntry<E> other) {
		int res = entry.compareTo(other.entry);
		if (res == 0 && other.entry != this.entry)
			res = (seqNum < other.seqNum ? -1 : 1);
		return res;
	}


}

```



<br>

\- console

```
// enforceOrderingPbq : 같은 요소일 경우 seq기반으로 재정렬하는 queue의 실행
10의 시퀀스 넘버를 가진 User99
0의 시퀀스 넘버를 가진 User0
1의 시퀀스 넘버를 가진 User1
2의 시퀀스 넘버를 가진 User2
3의 시퀀스 넘버를 가진 User3
4의 시퀀스 넘버를 가진 User4
5의 시퀀스 넘버를 가진 User5
6의 시퀀스 넘버를 가진 User6
7의 시퀀스 넘버를 가진 User7
8의 시퀀스 넘버를 가진 User8
9의 시퀀스 넘버를 가진 User9
=======================================================
// defaultPbq : 같은 요소일 경우 순서를 보장하지 않는 queue의 실행
User99
User4
User9
User8
User7
User6
User5
User1
User3
User2
User0

```

<br>

실행결과가 아주 확연히 다른것을 확인할 수 있다. enforceOrderingPbq는 요소가 같을경우 FIFO를 보장하기때문에 먼저 들어간 User0에서 9까지 순서대로 take()되는것에 비해 defaultPbq는 queue 내부에서 더이상 정렬이 불가능하기때문에 4,9,8 ... 로 불안정하게 요소를 take()해 온다. 



***



Queue interface는 이정도로 마치고 다음시간에는 Deque에 대해서 알아보도록 하겠습니다!!



<br>

포스팅은 여기까지 하겠습니다. 

예제 : https://github.com/sup2is/java-example/tree/master/java-data-structure7

<br>

퍼가실때는 링크와 출처를 반드시 명시해주세요. 감사합니다.

<br>



참고 : https://docs.oracle.com/javase/8/docs/api/